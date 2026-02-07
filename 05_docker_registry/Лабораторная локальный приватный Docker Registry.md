
## Цель

- Поднять локальный Docker Registry в контейнере.
- Собрать образ, запушить в локальный registry и спуллить обратно.
- Проверить, что образ реально хранится в registry.

## Требования
- Установлен Docker.
- `curl`
---
## 1) Локальный registry

Запустила контейнер registry на порту 5000, имя контейнера `registry`, образ `registry:2`.
- запуск в фоне
- проброс порта 5000:5000
- имя контейнера `registry`
- образ `registry:2`

Контейнер запущен:

![[Pasted image 20260207190914.png]]

---

## 2) Проверка registry по API

registry отвечает и репозиториев пока нет:

![[Pasted image 20260207191149.png]]

---

## 3) Подготовка мини-проекта

Создала папку `web-demo-dianaponomareva и файлы:

* `index.html`
* `Dockerfile`: 

![[Pasted image 20260207210133.png]]

---

## 4) Сборка образа 

Сборка образа с тегом `web-demo-dianaponomareva:1.0`:

![[Pasted image 20260207200841.png]]

Образ появился

![[Pasted image 20260207200944.png]]

---

## 5) Привязка образа к registry через tag

Перетегировала образ так, чтобы он указывал на локальный registry:

Теперь есть два имени/тега у одного и того же образа:

![[Pasted image 20260207201727.png]]

---

## 6) Push в registry

Запушила образ в registry:

```bash
docker push localhost:5000/web-demo-dianaponomareva:1.0
```

 Вывода`push`:
 
![[Pasted image 20260207202315.png]]

---

## 7) Проверка, что образ реально в registry

Список репозиториев:

![[Pasted image 20260207202518.png]]

Появление:
```json
{"repositories":["web-demo-dianaponomareva"]}
```

Список тегов:

![[Pasted image 20260207202703.png]]

Появление
```json
{"name":"web-demo-dianaponomareva","tags":["1.0"]}
```

---

## 8) Проверка pull

1. Удалите только тег `localhost:5000/...` (локальный `web-demo-<логин>:1.0` не удаляйте).
  ![[Pasted image 20260207203407.png]]

2. Выполните `docker pull localhost:5000/web-demo-<логин>:1.0`.

![[Pasted image 20260207203456.png]]

---

## 9) Проверка запуском контейнера

Запустила контейнер из образа `localhost:5000/web-demo-dianaponomareva:1.0` с пробросом порта 8080:80.
Открыла в браузере `http://localhost:8080` и проверила, что отображается HTML.

![[Pasted image 20260207210350.png]]

![[Pasted image 20260207210445.png]]

---
## 10) Самостоятельная часть

### 10.1 Версия 2.0

* Изменила `index.html`, поменяла строку `Version 2.0`

![[Pasted image 20260207210646.png]]

* Собрала образ с тегом `2.0`

![[Pasted image 20260207210635.png]]

![[Pasted image 20260207210820.png]]

* Запушила `2.0` в registry
![[Pasted image 20260207210841.png]]
* Проверила через API, что у репозитория есть теги `1.0` и `2.0`
![[Pasted image 20260207210955.png]]
### 10.2 Второй запуск на другом порту

Запустила второй контейнер из версии `2.0` на другом порту (например 8081) и проверила:

* `http://localhost:8080` показывает версию 1.0
* `http://localhost:8081` показывает версию 2.0
 ![[Pasted image 20260207211119.png]]
 ![[Pasted image 20260207211207.png]]![[Pasted image 20260207211230.png]]![[Pasted image 20260207211252.png]]