# Лабораторная: Продакшн-образы Docker

**Цель:** собрать production-образ для мини-сервиса на Flask так, чтобы он был:
- маленький (**< 150 MB**),
- безопасный (**не root**),
- с **HEALTHCHECK**,
- корректно завершался по **SIGTERM**,
- умел работать при `--read-only`,
- принимал конфиг через переменные окружения **без пересборки**.

---

## Инструменты

1. **Docker Desktop**
2. **Git** и **jq** 

---

## Вывод docker history
```
diana@LAPTOP-VFHN5IBT:/mnt/c/me/programs/OS_labs/OS-labs/02_docker_image_lab$ docker history "$LOGIN/image-lab:$PREFIX"
IMAGE          CREATED        CREATED BY                                      SIZE      COMMENT
2c1b44e5bdcb   6 hours ago    CMD ["python" "app.py"]                         0B        buildkit.dockerfile.v0
<missing>      6 hours ago    HEALTHCHECK &{["CMD-SHELL" "curl -f http://l…   0B        buildkit.dockerfile.v0
<missing>      6 hours ago    ENV ROCKET_SIZE=Medium                          0B        buildkit.dockerfile.v0
<missing>      6 hours ago    USER appuser                                    0B        buildkit.dockerfile.v0
<missing>      6 hours ago    COPY --chown=appuser:appuser app/ ./ # build…   12.3kB    buildkit.dockerfile.v0
<missing>      6 hours ago    COPY /usr/local/bin/ /usr/local/bin/ # build…   24.6kB    buildkit.dockerfile.v0
<missing>      6 hours ago    COPY /usr/local/lib/python3.12/site-packages…   11.9MB    buildkit.dockerfile.v0
<missing>      6 hours ago    WORKDIR /app                                    4.1kB     buildkit.dockerfile.v0
<missing>      6 hours ago    RUN |2 LAB_LOGIN=dianaponomareva LAB_TOKEN=2…   8.19kB    buildkit.dockerfile.v0
<missing>      16 hours ago   RUN |2 LAB_LOGIN=dianaponomareva LAB_TOKEN=2…   41kB      buildkit.dockerfile.v0
<missing>      16 hours ago   LABEL org.lab.token=285f8025407781bf6fb07abc…   0B        buildkit.dockerfile.v0
<missing>      16 hours ago   LABEL org.lab.login=dianaponomareva             0B        buildkit.dockerfile.v0
<missing>      16 hours ago   ARG LAB_TOKEN=285f8025407781bf6fb07abcacc15e…   0B        buildkit.dockerfile.v0
<missing>      16 hours ago   ARG LAB_LOGIN=dianaponomareva                   0B        buildkit.dockerfile.v0
<missing>      16 hours ago   RUN /bin/sh -c apk add --no-cache curl # bui…   5.31MB    buildkit.dockerfile.v0
<missing>      2 months ago   CMD ["python3"]                                 0B        buildkit.dockerfile.v0
<missing>      2 months ago   RUN /bin/sh -c set -eux;  for src in idle3 p…   16.4kB    buildkit.dockerfile.v0
<missing>      2 months ago   RUN /bin/sh -c set -eux;   apk add --no-cach…   44MB      buildkit.dockerfile.v0
<missing>      2 months ago   ENV PYTHON_SHA256=c30bb24b7f1e9a19b11b55a546…   0B        buildkit.dockerfile.v0
<missing>      2 months ago   ENV PYTHON_VERSION=3.12.11                      0B        buildkit.dockerfile.v0
<missing>      2 months ago   ENV GPG_KEY=7169605F62C751356D054A26A821E680…   0B        buildkit.dockerfile.v0
<missing>      2 months ago   RUN /bin/sh -c set -eux;  apk add --no-cache…   2.98MB    buildkit.dockerfile.v0
<missing>      2 months ago   ENV LANG=C.UTF-8                                0B        buildkit.dockerfile.v0
<missing>      2 months ago   ENV PATH=/usr/local/bin:/usr/local/sbin:/usr…   0B        buildkit.dockerfile.v0
<missing>      2 months ago   CMD ["/bin/sh"]                                 0B        buildkit.dockerfile.v0
<missing>      2 months ago   ADD alpine-minirootfs-3.22.1-x86_64.tar.gz /…   8.98MB    buildkit.dockerfile.v0
```
## Отчет
Данный Dockerfile реализует многостадийную сборку, исключая инструменты компиляции из финального образа и сокращая его размер на сотни мегабайт. Использование минимального базового образа Alpine Linux и флагов `--no-cache` радикально уменьшает поверхность для атак и итоговый объем. Кэширование слоев достигается за счет раздельного копирования зависимостей и кода, что ускоряет повторные сборки. Безопасность обеспечивается запуском от имени непривилегированного пользователя и корректными правами доступа, а встроенный HEALTHCHECK с curl контролирует работоспособность. В совокупности эти подходы создают оптимизированный, безопасный и эффективный контейнер, превосходящий наивную сборку по многим важным параметрам.
