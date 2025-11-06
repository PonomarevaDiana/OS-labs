## Redis + Rocket Counter. От `docker run` к `docker compose` и масштабированию

### Часть 1. Запуск вручную (контейнеры `docker run`)

Создаем Redis container
```
diana@LAPTOP-VFHN5IBT:~$ docker run -d --name redis redis
Unable to find image 'redis:latest' locally
latest: Pulling from library/redis
6c981fc7c621: Pull complete
5dec27664782: Pull complete
22506777a096: Pull complete
1adabd6b0d6b: Pull complete
4f4fb700ef54: Pull complete
2e60b18d70d8: Pull complete
729797e636a7: Pull complete
Digest: sha256:5c7c0445ed86918cb9efb96d95a6bfc03ed2059fe2c5f02b4d74f477ffe47915
Status: Downloaded newer image for redis:latest
5745d2e3268dd603c11a74d7782856cd575e0dfb025029c1769f250cdb552bb9
```
Проверяем работу:

```
diana@LAPTOP-VFHN5IBT:~$ docker ps
CONTAINER ID   IMAGE     COMMAND                  CREATED          STATUS          PORTS      NAMES
5745d2e3268d   redis     "docker-entrypoint.s…"   28 seconds ago   Up 28 seconds   6379/tcp   redis
```

Создаем приложение rocketcounter

```
diana@LAPTOP-VFHN5IBT:~$ docker run -d --name rocketcounter --link redis:redis -p 8085:5000 rotorocloud/rocket-counter
Unable to find image 'rotorocloud/rocket-counter:latest' locally
latest: Pulling from rotorocloud/rocket-counter
a8fd6f3f484f: Pull complete
2e337da81dbc: Pull complete
a3edec420810: Pull complete
4abcf2066143: Pull complete
dca80dc46cec: Pull complete
b9e022a6e48e: Pull complete
fe9e15b6315c: Pull complete
a8f1f95eda49: Pull complete
4fc96b5c1ba4: Pull complete
Digest: sha256:a5abd9b415bf7ca75b1f40f943d065a361696fdec3cfc3ab32fc7e5646b0c45a
Status: Downloaded newer image for rotorocloud/rocket-counter:latest
290a886cc9dd7147a0ea2c27ce9f385b09e95c67be30aa7e2d4b2d88f6e73c49
```

Проверяем работу:

```
diana@LAPTOP-VFHN5IBT:~$ docker ps
CONTAINER ID   IMAGE                        COMMAND                  CREATED              STATUS              PORTS                                         NAMES
290a886cc9dd   rotorocloud/rocket-counter   "flask run"              25 seconds ago       Up 24 seconds       0.0.0.0:8085->5000/tcp, [::]:8085->5000/tcp   rocketcounter
5745d2e3268d   redis                        "docker-entrypoint.s…"   About a minute ago   Up About a minute   6379/tcp                                      redis
```

Снова проверяем работу:

```
diana@LAPTOP-VFHN5IBT:~$ curl -s localhost:8085 | head -10
<html>
  <head>
    <script src="https://ajax.googleapis.com/ajax/libs/jquery/3.1.1/jquery.min.js"></script>
  </head>
   <body>
     <button style="width: 50%; height: 15%; font-size:200%;" type='button' id ='retrieve'>GET A ROCKET!</button>
     <h1 id="mycnt"></h1>
     <img src="" id="myimg" />
   </body>
  <script>
```
 
 Удаляем контейнеры **redis и rocketcounter**

```
diana@LAPTOP-VFHN5IBT:~$ docker stop rocketcounter redis
rocketcounter
redis
diana@LAPTOP-VFHN5IBT:~$ docker rm rocketcounter redis
rocketcounter
redis
```

### Часть 2. Описание стека в `docker-compose.yml`

Создаем каталог и переходим в него:
```
diana@LAPTOP-VFHN5IBT:~$ sudo -i
root@LAPTOP-VFHN5IBT:~# mkdir -p /root/rocketcounter
root@LAPTOP-VFHN5IBT:~# cd /root/rocketcounter
root@LAPTOP-VFHN5IBT:~/rocketcounter# nano docker-compose.yml
```
Создаем вручную docker-compose.yml:
```
services:
  redis:
    image: redis

  rocketcounter:
    image: rotorocloud/rocket-counter
    ports:
      - "8085:5000"
    depends_on:
      - redis
```

Запуск стека:

```
root@LAPTOP-VFHN5IBT:~/rocketcounter# docker compose up -d
[+] Running 3/3
 ✔ Network rocketcounter_default            Created                                                                0.0s
 ✔ Container rocketcounter-redis-1          Started                                                                0.5s
 ✔ Container rocketcounter-rocketcounter-1  Started                                                                0.7s
```

Проверка работы:

```
root@LAPTOP-VFHN5IBT:~/rocketcounter# docker compose ps
NAME                            IMAGE                        COMMAND                  SERVICE         CREATED          STATUS          PORTS
rocketcounter-redis-1           redis                        "docker-entrypoint.s…"   redis           32 seconds ago   Up 31 seconds   6379/tcp
rocketcounter-rocketcounter-1   rotorocloud/rocket-counter   "flask run"              rocketcounter   32 seconds ago   Up 31 seconds   0.0.0.0:8085->5000/tcp, [::]:8085->5000/tcp
```

```
root@LAPTOP-VFHN5IBT:~/rocketcounter# curl -s localhost:8085 | head
<html>
  <head>
    <script src="https://ajax.googleapis.com/ajax/libs/jquery/3.1.1/jquery.min.js"></script>
  </head>
   <body>
     <button style="width: 50%; height: 15%; font-size:200%;" type='button' id ='retrieve'>GET A ROCKET!</button>
     <h1 id="mycnt"></h1>
     <img src="" id="myimg" />
   </body>
  <script>
```

### Часть 3. Масштабирование приложения

Измените публикацию портов на динамическую (эпемерные порты для каждого экземпляра):

```
services:
  redis:
    image: redis

  rocketcounter:
    image: rotorocloud/rocket-counter
    ports:
      - "0:5000"
    depends_on:
      - redis
```

Масштабирование:

```
root@LAPTOP-VFHN5IBT:~/rocketcounter# docker compose up -d --scale rocketcounter=2
[+] Running 3/3
 ✔ Container rocketcounter-redis-1          Running                                                                0.0s
 ✔ Container rocketcounter-rocketcounter-2  Started                                                               10.7s
 ✔ Container rocketcounter-rocketcounter-1  Started                                                               10.9s
```

```
root@LAPTOP-VFHN5IBT:~/rocketcounter# docker compose ps
NAME                            IMAGE                        COMMAND                  SERVICE         CREATED          STATUS          PORTS
rocketcounter-redis-1           redis                        "docker-entrypoint.s…"   redis           26 minutes ago   Up 26 minutes   6379/tcp
rocketcounter-rocketcounter-1   rotorocloud/rocket-counter   "flask run"              rocketcounter   5 minutes ago    Up 5 minutes    0.0.0.0:32769->5000/tcp, [::]:32769->5000/tcp
rocketcounter-rocketcounter-2   rotorocloud/rocket-counter   "flask run"              rocketcounter   5 minutes ago    Up 5 minutes    0.0.0.0:32768->5000/tcp, [::]:32768->5000/tcp
```

В выводе у обоих экземпляров rocketcounter разные хост-порты, счётчик общий (хранится в одном Redis).

### Часть 4. Полное уничтожение стека

```
root@LAPTOP-VFHN5IBT:~/rocketcounter# docker compose down -v
[+] Running 4/4
 ✔ Container rocketcounter-rocketcounter-1  Removed                                                               10.4s
 ✔ Container rocketcounter-rocketcounter-2  Removed                                                               10.4s
 ✔ Container rocketcounter-redis-1          Removed                                                                0.3s
 ✔ Network rocketcounter_default            Removed                                                                0.3s
```

Проверка, везде пусто

```
root@LAPTOP-VFHN5IBT:~/rocketcounter# docker ps -a | grep -E 'rocketcounter|redis'
root@LAPTOP-VFHN5IBT:~/rocketcounter# docker network ls | grep rocketcounter
root@LAPTOP-VFHN5IBT:~/rocketcounter#
```

