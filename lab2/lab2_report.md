University: [ITMO University](https://itmo.ru/ru/)<br>
Faculty: [FICT](https://fict.itmo.ru)<br>
Course: [Введение в веб технологии](https://itmo-ict-faculty.github.io/introduction-in-web-tech/)<br>
Year: 2026/2027<br>
Group: U4225<br>
Author: Milkina Maria Igorevna<br>
Lab: Lab2<br>
Date of create: 10.09.2026<br>
Date of finished:

**Лабораторная работа №2**

**CI/CD для Docker приложения**

**Цель работы**

Разобраться с настройкой CI/CD в GitHub Actions: сделать автоматическую сборку Docker-образа, его публикацию в Docker Hub и настроить разное поведение пайплайна для веток main и develop.

**Ход работы**

**1. Подготовка Docker Hub**

Сначала был создан публичный репозиторий в Docker Hub, куда в дальнейшем должен был загружаться собранный образ приложения.

Репозиторий был создан с именем:

```text
mariamilkina/my-flask-app
```

![Репозиторий в Docker Hub](images/01_dockerhub_repository.png)

**2. Настройка секретов GitHub**

Для того чтобы GitHub Actions мог автоматически авторизоваться в Docker Hub, в настройках репозитория были добавлены два секрета:

- DOCKER_USERNAME — логин от Docker Hub;
- DOCKER_PASSWORD — Personal Access Token с правами Read & Write.

Сам токен в репозитории не отображается и хранится в скрытом виде.

![GitHub Secrets](images/02_github_secrets.png)

**3. Подготовка файлов для второй лабораторной**

Во вторую лабораторную были добавлены основные файлы приложения из первой работы:

- Dockerfile;
- app.py;
- requirements.txt.

Также были созданы файл отчёта и папка images для скриншотов.

![Файлы лабораторной работы](images/03_lab2_project_files.png)

**4. Настройка GitHub Actions**

Для автоматизации сборки и публикации Docker-образа был создан workflow:

```text
.github/workflows/docker-build.yml
```

Пайплайн запускается после push в репозиторий и выполняет основные шаги:

- получает код из репозитория;
- настраивает Docker Buildx;
- авторизуется в Docker Hub;
- собирает Docker-образ;
- отправляет образ в Docker Hub;
- выполняет шаг deploy.

Для сборки использовались файлы из папки lab2.

Готовый образ публиковался как:

```text
mariamilkina/my-flask-app:latest
```

После добавления workflow GitHub Actions автоматически запустил пайплайн.

![Успешный запуск GitHub Actions](images/04_actions_success.png)

Внутри job build-and-push можно было увидеть выполнение всех основных этапов: получение кода, настройку Buildx, вход в Docker Hub, сборку и публикацию образа.

![Этапы выполнения пайплайна](images/05_actions_steps.png)

**5. Проверка Docker Hub**

После успешного завершения GitHub Actions в Docker Hub появился новый образ с тегом latest.

![Docker-образ с тегом latest](images/06_dockerhub_latest.png)

Это означает, что теперь после изменений в репозитории Docker-образ может автоматически собираться и публиковаться без ручного выполнения этих команд.

**Дополнительная часть**

Дополнительно была настроена работа пайплайна с двумя ветками: main и develop.

Workflow был изменён так, чтобы запускаться для обеих веток:

```yaml
on:
  push:
    branches: [main, develop]
```

**6. Работа с веткой main**

Для основной ветки был добавлен отдельный шаг deploy:

```yaml
- name: Deploy to production
  if: github.ref == 'refs/heads/main'
  run: echo "Deploying to production server..."
```

При запуске пайплайна из ветки main выполняется deploy для production, а шаг для development пропускается.

![Deploy для ветки main](images/07_main_production_deploy.png)

**7. Создание ветки develop**

Для проверки второго варианта была создана отдельная ветка develop на основе main.

![Созданная ветка develop](images/08_develop_branch.png)

После этого GitHub Actions успешно запускался уже для обеих веток.

![Запуски GitHub Actions для двух веток](images/09_actions_both_branches.png)

**8. Работа с веткой develop**

Для ветки develop был добавлен свой шаг deploy:

```yaml
- name: Deploy to development
  if: github.ref == 'refs/heads/develop'
  run: echo "Deploying to development server..."
```

Для проверки в develop было внесено изменение и сделан commit.

После push GitHub Actions снова запустился автоматически. В этом случае production-шаг был пропущен, а deploy для development выполнился.

![Deploy для ветки develop](images/10_develop_deploy.png)

**Результат**

В ходе работы был настроен CI/CD-пайплайн с помощью GitHub Actions.

Теперь после push Docker-образ автоматически собирается и публикуется в Docker Hub.

Дополнительно была настроена работа с двумя ветками: для main выполняется production deploy, а для develop — development deploy.
