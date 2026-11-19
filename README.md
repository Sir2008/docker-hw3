# Docker

```sh
cp .env.example .env
```

```dotenv
HOST_PORT=8080
GREETING="Привет, Docker!"
```

## 4 балла

```console
$ docker build -t ipr-hw3:local .
...
#10 unpacking to docker.io/library/ipr-hw3:local 0.0s done
#10 DONE 0.1s

$ docker run -d --name hw3-single-check-1795023455 -p 8082:8000 -e 'GREETING=Привет из одного контейнера!' ipr-hw3:local
46f04e9fcf569fc1bff823a8c68b2915aaf4bb15bc1e021be73b6146ce001e1a

$ curl --max-time 5 -sS -X GET -w '\nHTTP %{http_code}\n' http://127.0.0.1:8082/
{"greeting": "Привет из одного контейнера!", "count": 0, "hostname": "46f04e9fcf56", "storage": "sqlite"}
HTTP 200

$ curl --max-time 5 -sS -X POST -w '\nHTTP %{http_code}\n' http://127.0.0.1:8082/increment
{"greeting": "Привет из одного контейнера!", "count": 1, "hostname": "46f04e9fcf56", "storage": "sqlite"}
HTTP 200

$ curl --max-time 5 -sS -X POST -w '\nHTTP %{http_code}\n' http://127.0.0.1:8082/increment
{"greeting": "Привет из одного контейнера!", "count": 2, "hostname": "46f04e9fcf56", "storage": "sqlite"}
HTTP 200

$ curl --max-time 5 -sS -X GET -w '\nHTTP %{http_code}\n' http://127.0.0.1:8082/
{"greeting": "Привет из одного контейнера!", "count": 2, "hostname": "46f04e9fcf56", "storage": "sqlite"}
HTTP 200

$ docker logs hw3-single-check-1795023455
2026-11-18 17:40:35,399 listening port=8000 storage=sqlite
2026-11-18 17:40:35,689 http 172.17.0.1 "GET /readyz HTTP/1.1" 200 -
2026-11-18 17:40:35,702 http 172.17.0.1 "GET / HTTP/1.1" 200 -
2026-11-18 17:40:35,719 http 172.17.0.1 "POST /increment HTTP/1.1" 200 -
2026-11-18 17:40:35,729 http 172.17.0.1 "POST /increment HTTP/1.1" 200 -
2026-11-18 17:40:35,737 http 172.17.0.1 "GET / HTTP/1.1" 200 -

$ docker rm -f hw3-single-check-1795023455
hw3-single-check-1795023455
```

## 6 баллов

```console
$ HOST_PORT=8081 GREETING='Привет из SQLite!' docker compose -p hw3-check-1795023455-level6 -f compose.sqlite.yaml up -d --build --wait --wait-timeout 120
...
 Container hw3-check-1795023455-level6-app-1 Waiting
 Container hw3-check-1795023455-level6-app-1 Healthy

$ curl --max-time 5 -sS -X GET -w '\nHTTP %{http_code}\n' http://127.0.0.1:8081/
{"greeting": "Привет из SQLite!", "count": 0, "hostname": "416c5930dec9", "storage": "sqlite"}
HTTP 200

$ curl --max-time 5 -sS -X POST -w '\nHTTP %{http_code}\n' http://127.0.0.1:8081/increment
{"greeting": "Привет из SQLite!", "count": 1, "hostname": "416c5930dec9", "storage": "sqlite"}
HTTP 200

$ curl --max-time 5 -sS -X POST -w '\nHTTP %{http_code}\n' http://127.0.0.1:8081/increment
{"greeting": "Привет из SQLite!", "count": 2, "hostname": "416c5930dec9", "storage": "sqlite"}
HTTP 200

$ curl --max-time 5 -sS -X POST -w '\nHTTP %{http_code}\n' http://127.0.0.1:8081/increment
{"greeting": "Привет из SQLite!", "count": 3, "hostname": "416c5930dec9", "storage": "sqlite"}
HTTP 200

$ docker inspect --format '{{json .Mounts}}' 416c5930dec945bfa9422c8bb2bd140bdd7d3793bec4b4f7f00186b62ce874f2
[{"Type":"volume","Name":"hw3-check-1795023455-level6_sqlite-data","Source":"/var/lib/docker/volumes/hw3-check-1795023455-level6_sqlite-data/_data","Destination":"/data","Driver":"local","Mode":"rw","RW":true,"Propagation":""}]

$ HOST_PORT=8081 GREETING='Привет из SQLite!' docker compose -p hw3-check-1795023455-level6 -f compose.sqlite.yaml down
 Container hw3-check-1795023455-level6-app-1 Stopping
 Container hw3-check-1795023455-level6-app-1 Stopped
 Container hw3-check-1795023455-level6-app-1 Removing
 Container hw3-check-1795023455-level6-app-1 Removed
 Network hw3-check-1795023455-level6_default Removing
 Network hw3-check-1795023455-level6_default Removed

$ HOST_PORT=8081 GREETING='Привет из SQLite!' docker compose -p hw3-check-1795023455-level6 -f compose.sqlite.yaml up -d --wait --wait-timeout 120
 Network hw3-check-1795023455-level6_default Creating
 Network hw3-check-1795023455-level6_default Creating
 Network hw3-check-1795023455-level6_default Created
 Network hw3-check-1795023455-level6_default Created
 Container hw3-check-1795023455-level6-app-1 Creating
 Container hw3-check-1795023455-level6-app-1 Created
 Container hw3-check-1795023455-level6-app-1 Starting
 Container hw3-check-1795023455-level6-app-1 Started
 Container hw3-check-1795023455-level6-app-1 Waiting
 Container hw3-check-1795023455-level6-app-1 Healthy

$ curl --max-time 5 -sS -X GET -w '\nHTTP %{http_code}\n' http://127.0.0.1:8081/
{"greeting": "Привет из SQLite!", "count": 3, "hostname": "c8d0a1c18350", "storage": "sqlite"}
HTTP 200
```

## 8 баллов

```console
$ HOST_PORT=8080 GREETING='Привет, Docker!' docker compose -p hw3-check-1795023455-level8 -f compose.yaml up -d --build --wait --wait-timeout 120
...
 Container hw3-check-1795023455-level8-redis-1 Healthy
 Container hw3-check-1795023455-level8-app-1 Healthy

$ curl --max-time 5 -sS -X GET -w '\nHTTP %{http_code}\n' http://127.0.0.1:8080/
{"greeting": "Привет, Docker!", "count": 0, "hostname": "b5308491745e", "storage": "redis"}
HTTP 200

$ curl --max-time 5 -sS -X POST -w '\nHTTP %{http_code}\n' http://127.0.0.1:8080/increment
{"greeting": "Привет, Docker!", "count": 1, "hostname": "b5308491745e", "storage": "redis"}
HTTP 200

$ curl --max-time 5 -sS -X POST -w '\nHTTP %{http_code}\n' http://127.0.0.1:8080/increment
{"greeting": "Привет, Docker!", "count": 2, "hostname": "b5308491745e", "storage": "redis"}
HTTP 200

$ curl --max-time 5 -sS -X POST -w '\nHTTP %{http_code}\n' http://127.0.0.1:8080/increment
{"greeting": "Привет, Docker!", "count": 3, "hostname": "b5308491745e", "storage": "redis"}
HTTP 200

$ HOST_PORT=8080 GREETING='Привет, Docker!' docker compose -p hw3-check-1795023455-level8 -f compose.yaml ps
NAME                                  IMAGE                             COMMAND                  SERVICE   CREATED          STATUS                    PORTS
hw3-check-1795023455-level8-app-1     hw3-check-1795023455-level8-app   "python app.py"          app       11 seconds ago   Up 5 seconds (healthy)    0.0.0.0:8080->8000/tcp, [::]:8080->8000/tcp
hw3-check-1795023455-level8-redis-1   redis:7.4-alpine                  "docker-entrypoint.s…"   redis     11 seconds ago   Up 10 seconds (healthy)   6379/tcp

$ docker inspect --format '{{json .HostConfig.PortBindings}}' 73616977fd12f2583b11f0aa6c3b903925fab9dabb64de9c09db502033028ba1
{}

$ HOST_PORT=8080 GREETING='Привет, Docker!' docker compose -p hw3-check-1795023455-level8 -f compose.yaml exec -T redis redis-cli CONFIG GET appendonly
appendonly
yes

$ HOST_PORT=8080 GREETING='Привет, Docker!' docker compose -p hw3-check-1795023455-level8 -f compose.yaml down
 Container hw3-check-1795023455-level8-app-1 Stopping
 Container hw3-check-1795023455-level8-app-1 Stopped
 Container hw3-check-1795023455-level8-app-1 Removing
 Container hw3-check-1795023455-level8-app-1 Removed
 Container hw3-check-1795023455-level8-redis-1 Stopping
 Container hw3-check-1795023455-level8-redis-1 Stopped
 Container hw3-check-1795023455-level8-redis-1 Removing
 Container hw3-check-1795023455-level8-redis-1 Removed
 Network hw3-check-1795023455-level8_default Removing
 Network hw3-check-1795023455-level8_default Removed

$ HOST_PORT=8080 GREETING='Привет, Docker!' docker compose -p hw3-check-1795023455-level8 -f compose.yaml up -d --wait --wait-timeout 120
 Network hw3-check-1795023455-level8_default Creating
 Network hw3-check-1795023455-level8_default Creating
 Network hw3-check-1795023455-level8_default Created
 Network hw3-check-1795023455-level8_default Created
 Container hw3-check-1795023455-level8-redis-1 Creating
 Container hw3-check-1795023455-level8-redis-1 Created
 Container hw3-check-1795023455-level8-app-1 Creating
 Container hw3-check-1795023455-level8-app-1 Created
 Container hw3-check-1795023455-level8-redis-1 Starting
 Container hw3-check-1795023455-level8-redis-1 Started
 Container hw3-check-1795023455-level8-redis-1 Waiting
 Container hw3-check-1795023455-level8-redis-1 Healthy
 Container hw3-check-1795023455-level8-app-1 Starting
 Container hw3-check-1795023455-level8-app-1 Started
 Container hw3-check-1795023455-level8-app-1 Waiting
 Container hw3-check-1795023455-level8-redis-1 Waiting
 Container hw3-check-1795023455-level8-redis-1 Healthy
 Container hw3-check-1795023455-level8-app-1 Healthy

$ curl --max-time 5 -sS -X GET -w '\nHTTP %{http_code}\n' http://127.0.0.1:8080/
{"greeting": "Привет, Docker!", "count": 3, "hostname": "a88c2570df46", "storage": "redis"}
HTTP 200

$ docker inspect --format '{{.Id}} {{.State.Status}} {{.State.Health.Status}} {{.State.StartedAt}} {{.RestartCount}}' a88c2570df467520b7f47713529bb4029afa7a800843084b6771df8b872e456d
a88c2570df467520b7f47713529bb4029afa7a800843084b6771df8b872e456d running healthy 2026-11-18T17:41:35.797251816Z 0

$ HOST_PORT=8080 GREETING='Привет, Docker!' docker compose -p hw3-check-1795023455-level8 -f compose.yaml stop redis
 Container hw3-check-1795023455-level8-redis-1 Stopping
 Container hw3-check-1795023455-level8-redis-1 Stopped

$ curl --max-time 5 -sS -X GET -w '\nHTTP %{http_code}\n' http://127.0.0.1:8080/healthz
{"status": "alive"}
HTTP 200

$ curl --max-time 5 -sS -X GET -w '\nHTTP %{http_code}\n' http://127.0.0.1:8080/readyz
{"error": "storage_unavailable"}
HTTP 503

$ curl --max-time 5 -sS -X POST -w '\nHTTP %{http_code}\n' http://127.0.0.1:8080/increment
{"error": "storage_unavailable"}
HTTP 503

$ HOST_PORT=8080 GREETING='Привет, Docker!' docker compose -p hw3-check-1795023455-level8 -f compose.yaml start --wait --wait-timeout 120 redis
 Container hw3-check-1795023455-level8-redis-1 Starting
 Container hw3-check-1795023455-level8-redis-1 Started
 Container hw3-check-1795023455-level8-redis-1 Waiting
 Container hw3-check-1795023455-level8-redis-1 Healthy

$ curl --max-time 5 -sS -X GET -w '\nHTTP %{http_code}\n' http://127.0.0.1:8080/
{"greeting": "Привет, Docker!", "count": 3, "hostname": "a88c2570df46", "storage": "redis"}
HTTP 200

$ curl --max-time 5 -sS -X POST -w '\nHTTP %{http_code}\n' http://127.0.0.1:8080/increment
{"greeting": "Привет, Docker!", "count": 4, "hostname": "a88c2570df46", "storage": "redis"}
HTTP 200

$ docker inspect --format '{{.Id}} {{.State.Status}} {{.State.Health.Status}} {{.State.StartedAt}} {{.RestartCount}}' a88c2570df467520b7f47713529bb4029afa7a800843084b6771df8b872e456d
a88c2570df467520b7f47713529bb4029afa7a800843084b6771df8b872e456d running healthy 2026-11-18T17:41:35.797251816Z 0
```
