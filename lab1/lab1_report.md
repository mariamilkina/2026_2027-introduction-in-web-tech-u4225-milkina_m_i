University: [ITMO University](https://itmo.ru/ru/)<br>
Faculty: [FICT](https://fict.itmo.ru)<br>
Course: [Введение в веб технологии](https://itmo-ict-faculty.github.io/introduction-in-web-tech/)<br>
Year: 2026/2027<br>
Group: U4225<br>
Author: Milkina Maria Igorevna<br>
Lab: Lab1<br>
Date of create: 07.09.2026<br>
Date of finished:

**Лабораторная работа №1**

**Основы работы с Docker**

**Цель работы**

Познакомиться с основными возможностями Docker: запуском готовых образов, созданием и управлением контейнерами, работой с volumes и сборкой собственного Docker-образа.

**Ход работы**

**1. Проверка Docker**

Сначала была проверена установленная версия Docker и запущен стандартный тестовый контейнер hello-world.

```bash
docker --version
docker run hello-world
docker images
docker ps
docker ps -a
```

После запуска Docker вывел сообщение Hello from Docker!, значит установка работает корректно.

![Проверка Docker](images/01_docker_hello.png)

**2. Работа с Ubuntu**

Дальше был загружен готовый образ Ubuntu и запущен контейнер в интерактивном режиме.

```bash
docker pull ubuntu:latest
docker run -it ubuntu bash
```

Внутри контейнера был установлен curl и проверена его версия.

```bash
apt update && apt install -y curl
curl --version
```

![Работа с Ubuntu](images/02_ubuntu_curl.png)

**3. Запуск nginx**

Следующим шагом был запущен контейнер nginx.

Порт 8080 на компьютере был связан с портом 80 внутри контейнера.

```bash
docker run -d -p 8080:80 --name web-server nginx:alpine
docker ps
```

После перехода в браузере по адресу:

```text
http://localhost:8080
```

открылась стандартная страница nginx.

![Страница nginx](images/03_nginx_browser.png)

После этого были просмотрены логи контейнера и выполнено подключение внутрь него.

```bash
docker logs web-server
docker exec -it web-server sh
pwd
ls
```

![Работа внутри nginx-контейнера](images/04_nginx_container.png)

**4. Управление контейнерами**

Далее была проверена остановка и повторный запуск контейнера.

```bash
docker ps
docker ps -a
docker stop web-server
docker ps
docker start web-server
docker ps
```

После команды stop контейнер перестал отображаться среди запущенных, а после start снова появился со статусом Up.

![Управление контейнером](images/05_container_management.png)

После проверки контейнер и использованный образ nginx были удалены.

```bash
docker stop web-server
docker rm web-server
docker rmi nginx:alpine
```

**5. Работа с volumes**

Для проверки сохранения данных был создан Docker volume:

```bash
docker volume create my-volume
```

Далее volume был подключён к Ubuntu-контейнеру, после чего внутри него был создан тестовый файл.

```bash
docker run -it --name volume-test -v my-volume:/data ubuntu bash
echo "Hello from volume" > /data/test.txt
cat /data/test.txt
```

После этого первый контейнер был удалён, а вместо него создан новый контейнер с тем же volume.

```bash
docker rm volume-test
docker run -it --name volume-test-2 -v my-volume:/data ubuntu bash
cat /data/test.txt
```

Файл остался на месте. Таким образом, данные в volume сохраняются независимо от самого контейнера.

![Проверка сохранения данных](images/06_volume_persistence.png)

**Дополнительная часть**

Дополнительно было создано небольшое Flask-приложение и собран собственный Docker-образ.

Для приложения были подготовлены три файла:

- app.py;
- requirements.txt;
- Dockerfile.

![Файлы Flask-приложения](images/07_dockerfile.png)

Образ был собран командой:

```bash
docker build -t my-flask-app .
```

После сборки образ my-flask-app появился среди локальных Docker-образов.

```bash
docker images
```

![Сборка образа](images/08_docker_build.png)

При первом запуске возникла ошибка из-за несовместимых версий Flask и Werkzeug.

Для исправления в requirements.txt были указаны совместимые версии:

```text
Flask==2.0.1
Werkzeug==2.0.3
```

После изменения зависимостей образ был пересобран, и контейнер успешно запустился.

```bash
docker run -d -p 5000:5000 --name flask-container my-flask-app
docker ps
curl http://localhost:5000
```

В ответ приложение вернуло:

```text
Hello from Docker!
```

![Запуск Flask-приложения](images/09_flask_run.png)

**Результат**

В ходе лабораторной работы были проверены основные команды Docker, запуск готовых образов, работа с контейнерами и volumes.

Также был запущен nginx и собрано собственное Flask-приложение в Docker-образе.

В результате получилось разобраться с базовой логикой Docker: как запускаются и удаляются контейнеры, как сохраняются данные отдельно от контейнера и как собственное приложение можно упаковать в Docker-образ.
