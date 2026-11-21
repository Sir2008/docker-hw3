## 4 балла

docker build -t ipr-hw3:local .

Собирает образ приложения.

docker run -d --name hw3-single -p 8082:8000 -e 'GREETING=Привет из одного контейнера!' ipr-hw3:local

Запускает приложение на порту 8082 с заданным приветствием.

curl -sS http://localhost:8082/

Показывает приветствие и счётчик. При первом запросе счётчик был 0.

curl -sS -X POST http://localhost:8082/increment

Увеличивает счётчик на 1. После двух запросов счётчик стал 2.

docker logs hw3-single

Показывает логи приложения. Вот несколько строк из проверки.

2026-11-18 17:40:35,399 listening port=8000 storage=sqlite

2026-11-18 17:40:35,702 http 172.17.0.1 "GET / HTTP/1.1" 200 -

2026-11-18 17:40:35,719 http 172.17.0.1 "POST /increment HTTP/1.1" 200 -

docker rm -f hw3-single

Останавливает и удаляет контейнер.

## 6 баллов

cp .env.example .env

Создаёт файл с настройками. В нём можно поменять HOST_PORT и GREETING. В командах curl ниже указан порт 8080.

docker compose -f compose.sqlite.yaml up -d --build --wait

Запускает приложение с SQLite. Файл базы лежит в томе sqlite-data.

curl -sS -X POST http://localhost:8080/increment

Увеличивает счётчик. При проверке после трёх запросов он стал 3.

docker compose -f compose.sqlite.yaml down

Удаляет контейнер. Том с базой остаётся.

docker compose -f compose.sqlite.yaml up -d --wait

Создаёт контейнер заново и подключает тот же том.

curl -sS http://localhost:8080/

Показывает счётчик. После пересоздания осталось 3.

docker compose -f compose.sqlite.yaml down

Останавливает SQLite-вариант и освобождает порт.

## 8 баллов

docker compose up -d --build --wait

Запускает приложение и Redis, ждёт готовности обоих сервисов.

docker compose ps

Показывает состояние и порты. При проверке оба сервиса были healthy, порт на компьютере заняло только приложение.

docker compose exec -T redis redis-cli CONFIG GET appendonly

Проверяет запись Redis на диск. Результат — yes.

curl -sS -X POST http://localhost:8080/increment

Увеличивает счётчик. При проверке после трёх запросов он стал 3.

docker compose down

Удаляет контейнеры. Том redis-data остаётся.

docker compose up -d --wait

Создаёт контейнеры заново и подключает том Redis.

curl -sS http://localhost:8080/

Показывает счётчик. После пересоздания осталось 3.

docker compose stop redis

Останавливает Redis.

curl -si http://localhost:8080/healthz

Проверяет, что приложение отвечает. Вернулся HTTP 200.

curl -si http://localhost:8080/readyz

Проверяет доступность хранилища. Вернулся HTTP 503.

curl -si -X POST http://localhost:8080/increment

Пытается увеличить счётчик без Redis. Вернулся HTTP 503.

docker compose start --wait redis

Запускает Redis и ждёт его готовности. Приложение продолжает работать в том же контейнере.

curl -sS http://localhost:8080/

Проверяет счётчик после запуска Redis. Вернулось прежнее значение 3.

curl -sS -X POST http://localhost:8080/increment

Увеличивает счётчик до 4.

docker compose down

Останавливает приложение и Redis.
