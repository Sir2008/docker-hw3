# ДЗ 3: Docker — 8 баллов

Упаковка готового HTTP-сервиса со счётчиком. Выполнены уровни 4, 6 и 8
из [условия](https://github.com/Nikfilk2030/missing_semester_hse_cs/blob/main/2026/amcs/year_1/hw_3/README.md).
Исходные файлы в `app/` взяты из
[шаблона](https://github.com/null-ref-exc/hw_3_template) и не изменены.

Для запуска нужны Docker с Docker Compose и `curl`. На macOS для запуска
Docker Engine можно использовать Docker Desktop или Colima (`colima start`).
Все команды ниже выполняются из корня репозитория.

## Основной запуск: приложение и Redis (8 баллов)

```sh
cp .env.example .env
docker compose -p hw3-redis up -d --build --wait --wait-timeout 120
curl --max-time 5 -fsS http://127.0.0.1:8080/
curl --max-time 5 -fsS -X POST http://127.0.0.1:8080/increment
docker compose -p hw3-redis ps
```

В `.env` настраиваются:

| Переменная | По умолчанию | Назначение |
| --- | --- | --- |
| `HOST_PORT` | `8080` | Порт приложения на компьютере |
| `GREETING` | `Привет, Docker!` | Приветствие в JSON-ответе |

После изменения `.env` повторите команду `up -d`. Если изменён `HOST_PORT`,
подставьте его вместо `8080` в запросах `curl`.

В `compose.yaml` два сервиса:

- `app` собирается из `Dockerfile` и слушает порт `8000` внутри контейнера.
  Использует `STORAGE=redis` и `REDIS_URL=redis://redis:6379/0`.
- `redis` доступен приложению по имени сервиса в сети Compose. Его порт
  не публикуется на компьютере. Именованный volume `redis-data` подключён
  к `/data`; запись AOF включена через `--appendonly yes`.

У обоих сервисов есть healthcheck. Приложение запускается после готовности
Redis. Проверка приложения обращается к `/readyz`, поэтому при остановке
Redis контейнер приложения становится `unhealthy`, но HTTP-сервер продолжает работать.

### Сохранение счётчика

```sh
curl --max-time 5 -fsS -X POST http://127.0.0.1:8080/increment
curl --max-time 5 -fsS http://127.0.0.1:8080/
# Запомните count как N.
docker compose -p hw3-redis down
docker compose -p hw3-redis up -d --wait --wait-timeout 120
curl --max-time 5 -fsS http://127.0.0.1:8080/
# count должен остаться равным N.
```

`down` удаляет контейнеры и сеть, сохраняя именованный volume.
Не добавляйте `-v` при проверке сохранения данных: этот флаг удаляет volume.

### Отключение и возвращение Redis

```sh
docker compose -p hw3-redis stop redis
curl --max-time 5 -sS -w '\nHTTP %{http_code}\n' http://127.0.0.1:8080/healthz
curl --max-time 5 -sS -w '\nHTTP %{http_code}\n' http://127.0.0.1:8080/readyz
curl --max-time 5 -sS -X POST -w '\nHTTP %{http_code}\n' http://127.0.0.1:8080/increment
docker compose -p hw3-redis start --wait --wait-timeout 120 redis
curl --max-time 5 -fsS http://127.0.0.1:8080/readyz
curl --max-time 5 -fsS http://127.0.0.1:8080/
curl --max-time 5 -fsS -X POST http://127.0.0.1:8080/increment
```

При остановленном Redis ожидаются коды `200`, `503`, `503` соответственно.
После возвращения Redis `GET /` возвращает прежний `N`, следующий POST — `N + 1`.
Перезапускать приложение не требуется.

Обычная остановка с сохранением данных:

```sh
docker compose -p hw3-redis down
```

## Один контейнер (4 балла)

```sh
docker build -t ipr-hw3:local .
docker run -d --name hw3-single -p 8082:8000 \
  -e GREETING='Привет из одного контейнера!' ipr-hw3:local
docker logs hw3-single
# После появления строки listening сервер готов.
curl --max-time 5 -fsS http://127.0.0.1:8082/
curl --max-time 5 -fsS -X POST http://127.0.0.1:8082/increment
docker logs hw3-single
docker rm -f hw3-single
```

Здесь по умолчанию используется SQLite внутри контейнера. Последняя команда
удаляет этот контейнер вместе с его локальным счётчиком.

## Compose и SQLite с volume (6 баллов)

Отдельный файл `compose.sqlite.yaml` демонстрирует промежуточный уровень.
Он запускается отдельным проектом и не использует данные варианта с Redis.

```sh
HOST_PORT=8081 GREETING='Привет из SQLite!' docker compose \
  -p hw3-sqlite -f compose.sqlite.yaml up -d --build --wait --wait-timeout 120
curl --max-time 5 -fsS -X POST http://127.0.0.1:8081/increment
curl --max-time 5 -fsS http://127.0.0.1:8081/
docker compose -p hw3-sqlite -f compose.sqlite.yaml down
HOST_PORT=8081 GREETING='Привет из SQLite!' docker compose \
  -p hw3-sqlite -f compose.sqlite.yaml up -d --wait --wait-timeout 120
curl --max-time 5 -fsS http://127.0.0.1:8081/
docker compose -p hw3-sqlite -f compose.sqlite.yaml down
```

Значение счётчика сохраняется в именованном volume `sqlite-data`, который
подключён к `/data`; файл базы — `/data/counter.db`.

## Результаты проверок

Команды, фактические ответы сервиса и логи контейнеров приведены в
[REPORT.md](REPORT.md). Описание исходного API — в [app/README.md](app/README.md).
