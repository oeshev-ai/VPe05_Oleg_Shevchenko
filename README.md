# Loki + Grafana

Стек для сбора и просмотра логов: **Loki** хранит логи, **Grafana** их визуализирует.

## Состав

| Сервис  | Порт | Назначение                          |
|---------|------|-------------------------------------|
| Loki    | 3100 | Хранилище логов                     |
| Grafana | 3000 | UI для запросов и дашбордов         |

Сервисы в одной Docker-сети `logging`. Grafana обращается к Loki по имени: `http://loki:3100`. Datasource подключается автоматически через provisioning.

## Требования

- Docker
- Docker Compose v2+

## Запуск

```bash
cd /home/deploy/Loki+Grafana
docker compose up -d
```

Проверка:

```bash
docker compose ps
curl -s http://localhost:3100/ready
```

Остановка:

```bash
docker compose down
```

С удалением данных (volumes):

```bash
docker compose down -v
```

## Доступ

- **Grafana:** http://localhost:3000
- **Логин:** `admin`
- **Пароль:** `admin`
- **Loki API:** http://localhost:3100

Пароль задаётся в `docker-compose.yml` (`GF_SECURITY_ADMIN_PASSWORD`). После первого входа Grafana попросит сменить пароль — можно пропустить или задать новый.

## Структура

```
Loki+Grafana/
├── docker-compose.yml
├── README.md
└── grafana/
    └── provisioning/
        └── datasources/
            └── loki.yml      # автодобавление Loki в Grafana
```

## Отправка логов в Loki

Пример через Promtail или любой клиент с push API:

```bash
curl -H "Content-Type: application/json" \
  -XPOST "http://localhost:3100/loki/api/v1/push" \
  --data-raw '{
    "streams": [{
      "stream": { "app": "test", "job": "manual" },
      "values": [[ "'"$(date +%s)000000000"'", "hello from curl" ]]
    }]
  }'
```

В Grafana: **Explore → Loki** → запрос, например `{app="test"}`.

## Данные

Volumes:

- `loki-data` — логи Loki
- `grafana-data` — настройки и дашборды Grafana
