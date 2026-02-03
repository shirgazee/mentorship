Это конспект курса “Docker - Deep Dive” с дополнениями за время работы с докером.

**Docker** - рантайм для контейнеров и движок для их оркестрации

У него клиент-серверная архитектура: клиент и демон (dockerd). Они коммуницируют через REST-API через сетевой интерфейс или UNIX-сокеты.

Демон управляет объектами докера: images, containers, networks и volumes.

**Docker Registry** - место, где хрянятся images, типа как гитхаб, только для образов. По дефолту установлен DockerHub, можно подключить свой, например Nexus.

**Image** - шаблон с инструкциями для создания контейнера. Может быть основан на другом образе. Для создания используется Dockerfile.

**Container** - запускаемый инстанс образа. Может быть присоединен к сети, и к нему можно подключить какое-нибудь хранилище. Изолирован от остальных контейнеров.

**Services** - скейлинг контейнеров между несколькими демонами докерами.

**Docker Swarm** - множественный демон (master и workers). Все демоны коммуницируют через Docker API.

**Модули Docker engine:**

- daemon
- containerd - управление жизненным циклом контейнеров, управление образом
- runc - создание контейнеров
- shim - процесс-владелец запущенных контейнеров

Образы (Images) состоят из нескольких слоев (как коммиты в гите). Все, кроме последнего, read-only. Последний - Container Layer. Его удаление означает, что удаляется контейнер.

Все изменения хранятся в изменяемом слое контейнера. А образ остается неизменяемым.

**Популярные команды**
```Shell
docker image ls - images (-a - all)

docker container ls - containers (-a - all)

docker container ps - running containers

docker container inspect <id>

docker container top <id> - processes

docker container attach <id> (остановит контейнер, если отсоединимся)

docker container start/stop/pause/unpause <id> (start - запускает остановленные в отличие от run)

docker container logs <id>

docker container stats <id> - resource usage

docker container exec -it <id> /bin/bash (i - interactive, t - tty) - можно что-нибудь выполнить, например зайти в баш

docker container rm <id> (-f для принудительного удаления)

docker container prune - удалить все остановленные контейнеры (-f - не спрашивать ничего)

docker container run <name> (-id - tty, -d - detached, --name - указать имя, --rm - удалить, когда остановится)
```

**Порты**
Простой запуск контейнера не пробрасывает никаких портов. Контейнер доступен только в приватной сети.

```Shell
docker container run run -d --expose 3000 nginx - открывает порт 3000, но не пробрасывает

docker container run -d --expose 3000 -p 8080:80 nginx - пробрасывает 8080 хоста на 80 контейнера

-p 8081:80/tcp -p 8081:80/udp - для tcp и udp

docker container run -P - пробрасывает все порты контейнера на рандомные порты хоста

docker container port <name> - посмотреть проброшенные порты
```

**Выполнение команд в контейнере**
```Shell
docker container run <image> <cmd> - стартануть с командой

docker container exec -it <name> <cmd> - выполнить команду на выполняющемся контейнере

docker exec -it <mycontainer> bash - подключиться к контейнеру
```

**Сеть**
Есть разные типы драйверов сети в докере:

- bridge - самый простой, дефолтный. По нему контейнеры могут коммуницировать между собой. Работает только на линуксе
- host - использовать сеть хоста напрямую, без промежуточного изолирующего слоя. Доступен для swarm
- overlay - для коммуникации контейнеров в разных демонах
- macvlan - можно установить mac-адрес, изображает физический девайс в сети. Полезно для всяких легаси
- none - без сети
- прочие драйвера от сторонних производителей

```Shell
docker network ls - список сетей. Лучше не пробовать удалять дефолтные сети

docker network inspect <name> - конфиг сети (bridge, host, none, ...)

docker network create/rm <name> - создать/удалить сеть с именем

docker network connect/disconnect <network> <container>

docker network create --subnet 10.1.0.0/24 --gateway 10.1.0.1 <name>

docker container run -name <> --network <network> --ip <ip>
```

**Хранилище**
Data:
- Non-persistent, в контейнерах
- Persistent (Volumes), отдельно от контейнеров

Создается volume, создается контейнер, хранилище маунтится в контейнер и не удаляется, когда удаляется контейнер.

Дефолтные пути:
linux: `/var/lib/docker/volumes
windows: `c:\ProgramData\docker\volumes

```Shell
docker volume ls

docker volume create/rm/prune <name>

docker volume inspect <name>

docker container run -d -name <> --mount type=bind,source=<source>,target=<target> <image> прокинуть папку из хоста в контейнер

или -v <source>:<target> - то же самое. Удобно, чтобы прокидывать конфигурацию в контейнер
```

Volumes проще бэкапить или мигрировать, чем mount type=bind

```Shell
docker container run -d -name <> --mount type=volume,source=<source>,target=<target> <image> прокинуть volume

или -v <volume-name>:<target>
```

  

**Dockerfile**
Dockerfile - это иструкции, как собрать и запустить образ. Образ состоит из read-only слоев. Каждый слой - инструкция докерфайла, это дельта изменений от предыдущего слоя.
.dockerignore - подобен .gitignore.


**Сборка образа**:
```Shell
docker image build -t <name>:<tag> .

docker build  --progress=plain --no-cache .

-f <dockerfile name> --force-rm (force remove intermediate containers)

--env <KEY>=<VALUE> при сборке контейнера

ENV <KEY>=<VALUE> в докерфайле
```


**Build arguments** - аргументы во время сборки
```Shell
--build-arg <name>=<value> при сборке контейнера

ARG <NAME>=<DEFAULT VALUE> в докерфайле
```

Затем, в докерфайле можно использовать этот аргумент как `$<NAME>`

  

**Выполнение от непривилегированного пользователя**
В докерфайле добавляем нового пользователя
`RUN useradd -ms /bin/bash cloud_user (Новый пользователь)
`USER cloud_user (все команды будут выполняться из под него)
потом зайти в контейнер от рута можно через
`docker container exec -u 0 -it <name> /bin/bash


**Volume в докерфайле**
VOLUME ["path1","path2"]

**Entrypoint vs. Command**
Entrypoint - контейнер выполняется, как исполняемый
CMD в этом случае добавляется к концу Entrypoint как аргументы, а не заменяет entrypoint
ENTRYPOINT node app.js - запустится через шелл /bin/sh, что не нужно
а ENTRIPOINT [“node”, “app.js”] запустится в своем собственном процессе без шелла

```Shell
FROM debian:wheezy
ENTRYPOINT ["/bin/ping"]
CMD ["localhost"]

docker run -it test  - будет пинговать localhost
docker run -it test google.com  - будет пинговать google.com 
```

  

**Multi-stage builds**
При сборке приложения внутри контейнера нет смысла таскать с собой все ассеты для сборки: исходники, SDK, npm пакеты и так далее. Лучше использовать один образ как билд-машину с SDK, копировать сборку в другой образ с рантаймом и там запускать.

```Dockerfile
FROM <img> AS <name>

...

FROM <another img> AS <another name>

COPY --from=build <files and folders>

```

  

**Tags**

Имеет смысл помечать образ хэшем коммита исходного кода, из которого мы собираем этот образ.

Команда, чтобы получить хэш:
```Shell
git log -1 --pretty=%H
```

**Сохранение и загрузка образов**
```Shell
docker image save <img> -o <file>.tar
docker image load -i <file>.tar
```

**Процессы в контейнере**
```Shell
docker container top <name> - снимок процессов
docker container stats <name> - стрим ресурсов в контейнере
```

  
**Авторестарт**

По дефолту флаг --restart стоит в значении no. Возможные значения:
- no
- on-failure
- always
- unless-stopped

  
**Portainer**

Интересный софт для управления контейнерами через GUI: [https://www.portainer.io/](https://www.portainer.io/)

Ставится в докер, как контейнер, доступен через браузер. Можно управлять локальным демоном или удаленным.


**Docker Compose**

Если используется микросервисная архитектура, то удобно описывать все сервисы (контейнеры) в одном файле - docker-compose.yaml

Docker compose - это софт, который нужно ставить отдельно, он не идет с докером.

```Shell
docker-compose up (-d - detached) - создать и поднять контейнеры

docker-compose ps - список контейнеров, созданных docker-compose

docker-compose stop

docker-compose start

docker-compose restart

docker-compose down - удалить контейнеры, которые создал docker-compose

docker-compose up --build --force-recreate - пересобрать образы

docker-compose нужно запускать в той папке, где лежит docker-compose.yaml/yml файл.
```

**Бонус**

```Shell
docker system df - сколько места занято

docker system prune - очистить все, что можно
```

**Не бонус**

Нативный докер, который поддерживает все фичи, и не жрет проц, есть только на линуксе. На винде и маке он будет работать через виртуалку.

Пример **DockerFile**

```Docker
# Set the base image as the .NET 6.0 SDK (this includes the runtime)
FROM mcr.microsoft.com/dotnet/sdk:6.0 as build-env

# Copy everything and publish the release (publish implicitly restores and builds)
WORKDIR /app
COPY . ./
RUN dotnet publish ./Service/Service.csproj -c Release -o out --no-self-contained

# Relayer the .NET SDK, anew with the build output
FROM mcr.microsoft.com/dotnet/aspnet:6.0-alpine
WORKDIR /app
COPY --from=build-env /app/out .
ENTRYPOINT [ "dotnet", "Service.dll" ]
```

Пример **docker-compose.yml**

```YAML
# Use postgres/example user/password credentials
version: '3.8'

services:
  api:
    build: .
    restart: always
    ports:
      - "8080:80"
    depends_on:
      - redis
    environment:
      ASPNETCORE_ENVIRONMENT: "Development"
      REDIS__CONNECTIONSTRING: "redis:6379"
  redis:
    image: redis:6.2-alpine
    restart: always
    ports:
      - '6379:6379'
    command: redis-server --save 20 1 --loglevel warning
    volumes:
      - redis:/data
volumes:
  redis:
    driver: local%
```