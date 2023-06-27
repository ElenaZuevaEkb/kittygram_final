# Kittygram
### Описание
Проект о домашних питомцах, в котором можно зарегистрироваться и добавлять питомцев, необходимо указать имя и год рождения питомца, а так же добавить фотографию и указать его умения. 
### Технологии
- Python - язык программирования.
- Django - свободный фреймворк для веб-приложений на языке Python.
- Django REST Framework - мощный и гибкий набор инструментов для создания веб-API.
- Simple JWT - плагин аутентификации JSON Web Token для Django REST Framework.
- Gunicorn 20.1.0 
- Фронтенд-приложение на React 
- npm 9.5.1 
- База данных - SQLite3
- Docker
- DockerHub
- CI/CD
- GitHub Actions
- PostgreSQL
- Docker network

### Запуск проекта через Bash 
### Установка Docker
- Подготовка Windows к установке Docker
-Сделать доступным Linux-ядро в Windows можно двумя способами, мы используем следующий:
Установить Windows Subsystem for Linux (WSL): это технология от Microsoft, позволяющая работать с терминалом и другими приложениями Linux, не выходя из Windows и не запуская отдельные программы для виртуализации. WSL доступна для Windows 10 и 11. Текущая версия WSL — вторая, сокращённо — WSL2.
```
wsl --install
```
### После установки WSL: установка Docker на Windows
- Зайдите на официальный сайт проекта https://www.docker.com/products/docker-desktop/ и скачайте установочный файл Docker Desktop.Следуйте инструкциям по установке — и всё получится.
### Запуск приложений в Docker-контейнерах
```
# Выполните команду запуска контейнера из образа hello-world:
docker run --name hello-container hello-world 
```
- Контейнер с окном в Сеть
```
# Запустите контейнер из этого образа.
docker run --name nginx-container nginx 
``` 
```
# Запустите новый контейнер,
docker run -p 80:80 --name nginx-container nginx
```
- Управление образами и контейнерами
```
# В новом окне терминала выполните команду для просмотра списка запущенных контейнеров:
docker container ls 
```
```
 # Остановите запущенный контейнер: выполните команду остановки в новом окне терминала.
docker container stop nginx-container
```
```
 # Остановленный контейнер можно запустить:
 docker container start nginx-container
```
```
 # Контейнер можно удалить:
 docker container rm nginx-container
```
```
 # Список остальных команд управления контейнерами можно узнать выполнив команду:
 docker container --help 
```
- Работа через Docker Desktop
Пользователи Windows и macOS могут управлять образами и контейнерами через приложение Docker Desktop

### Упаковка проекта в Docker-образ
### Запуск проекта
- Клонировать репозиторий и перейти в него в командной строке:
```
git clone https://github.com/yandex-praktikum/kittygram_backend.git
```
### Backend
```
cd kittygram_backend
```
- Cоздать и активировать виртуальное окружение:
```
python3 -m venv env
```
* Если у вас Linux/macOS
    ```
    source env/bin/activate
    ```
* Если у вас windows
    ```
    source env/scripts/activate
    ```
```
python3 -m pip install --upgrade pip
```
- Установить зависимости из файла requirements.txt:
```
pip install -r requirements.txt
```
- Выполнить миграции:
```
python3 manage.py migrate
```
- Запустить проект:
```
python3 manage.py runserver
```
### Frontend
```
cd kittygram_frontend
```
- Установить зависимости:
```
npm i
```
- Запустить проект:
```
npm run start
```
### Создание Dockerfile
- В директории backend/ проекта Kittygram создайте файл под названием Dockerfile. 
```
FROM python:3.9-slim

WORKDIR /app

RUN pip install gunicorn==20.1.0
COPY requirements.txt ./
RUN pip install -r requirements.txt --no-cache-dir
COPY . .
CMD ["gunicorn", "--bind", "0.0.0.0:8000", "kittygram_backend.wsgi"] 
```
### Запуск контейнера
- Собираем образ бэкенда
```
-docker build -t kittygram_backend . 
```
- Запустите контейнер из нового образа
```
docker run --name kittygram_backend_container --rm -p 8000:8000 kittygram_backend
`````
- Выполните миграции
```
# Откройте браузер и убедитесь, что backend kittygram доступен по адресу
http://localhost:8000/api/tasks/
```
- Создайте Dockerfile для frontend 
```
# Откройте браузер и убедитесь, что фронтенд Kittygram доступен по адресу
http://127.0.0.1:8000/
```
### Образ PostgreSQL на Docker Hub
- В корне проекта создайте файл .env:
```
POSTGRES_USER=django_user
POSTGRES_PASSWORD=mysecretpassword
POSTGRES_DB=django
# Добавляем переменные для Django-проекта:
DB_HOST=db
DB_PORT=5432
```
- Измените файл settings.py так, чтобы он использовал эти переменные.
```
import os

...

# Этими строчками замените текущую настройку DATABASES
DATABASES = {
    'default': {
        # Меняем настройку Django: теперь для работы будет использоваться
        # бэкенд postgresql
        'ENGINE': 'django.db.backends.postgresql',
        'NAME': os.getenv('POSTGRES_DB', 'django'),
        'USER': os.getenv('POSTGRES_USER', 'django'),
        'PASSWORD': os.getenv('POSTGRES_PASSWORD', ''),
        'HOST': os.getenv('DB_HOST', ''),
        'PORT': os.getenv('DB_PORT', 5432)
    }
}
```
- Пересоберите образ kittygram_backend с обновлённым файлом зависимостей: в директории backend/ выполните команду docker build.
```
docker build -t kittygram_backend . 
``` 
### Docker network: сеть из контейнеров
Создайте сеть с именем django-network:
```
docker network connect django-network db 
```
- Создайте контейнер db
```
docker run --name db \
       --env-file .env \
       -v pg_data:/var/lib/postgresql/data \
       postgres:13.10
```
- Создайте Docker volume:
```
docker volume create sqlite_data 
```
- Контейнер db уже запущен, его можно сразу подключить к докер-сети.
```
docker network connect django-network db 
```
- Запустите контейнер с Django-приложением:
 ```
 docker run --env-file .env \
           --net django-network \
           --name kittygram_backend_container \
           -p 8000:8000 kittygram_backend 
 ```
- Выполните миграции

### Docker compose: сборка проекта
- Создайте пустой файл docker-compose.yml в корне проекта

### Поднимаем сеть контейнеров: docker compose up
- Остановите и удалите существующие контейнеры kittygram_backend и db:
- В терминале в папке с docker-compose.yml выполните команду:
 ```
 docker compose up 
 ``` 
 - По команде docker compose up Docker Compose: 
получит готовые образы, указанные в image,
соберёт все образы, указанные в build,
запустит все контейнеры, описанные в конфиге.
 - Выполните миграции 
 
### Docker-compose: Nginx и статика
- Проверьте в корне проекта папку nginx/ с двумя файлами, Dockerfile и nginx.conf 
- Запустите команду 
```
docker compose up 
```
### Статика бэкенд-приложения
-Обновите settings.py 
```
STATIC_URL = '/static/'
STATIC_ROOT = BASE_DIR / 'collected_static'
```
- Запустите команду 
```
docker compose stop && docker compose up --build 
```
- При обращении к адресу http://localhost:8000/ должна отобразиться главная страница Kittygram
- Откройте в браузере страницу http://localhost:8000/admin/ и проверьте, что страница входа в админку отображается

### Деплой: публикация проекта в Docker на сервере
Собираем образы по примеру 
```
cd frontend  # В директории frontend...
docker build -t username/kyttigram_frontend . 
```
- Отправьте собранные образы фронтенда, бэкенда и Nginx на Docker Hub
```
docker push username/kittygram_frontend
```
- Создайте отдельный файл docker-compose.production.yml конфигурации Docker Compose, который будет управлять запуском контейнеров на боевом сервере

- Запустите Docker Compose 
```
docker compose -f docker-compose.production.yml up 
```
- Соберите статистику и запустите миграции  
- Подключитесь к удаленному серверу по SSH-ключу
```
Например:
ssh -i D:/Dev/vm_access/yc-yakovlevstepan yc-user@153.0.2.10 
```
- Остановите  и удалите Gunicorn:
- Очистите диск npm cache clean, sudo apt-get clean

### Устанавливаем и запускаем Docker Compose на сервере
- Для запуска Docker Compose в режиме демона выпоните команду
```
sudo docker compose -f docker-compose.production.yml up -d
```
- Выполните миграции, соберите статические файлы бэкенда и скопируйте их в /backend_static/static/:
-Откройте страницу https://ваш_домен/admin/ — админка должна открыться и заработать.
- На сервере в редакторе nano откройте конфиг Nginx: nano /etc/nginx/sites-enabled/default. Измените настройки location в секции server 

- Проверьте и перезагрузите конфиг Nginx
```
sudo service nginx reload 
```
### Автоматизация деплоя: CI/CD
### GitHub Actions. Workflow
- GitHub Actions — сервис от GitHub.
- Создайте в папке kittygram/ директорию .github/workflows, а в ней — файл main.yml.
- Запуск workflow
```
(venv) ... Dev/taski$ git add .
(venv) ... Dev/taski$ git commit -m 'Add Actions'
(venv) ... Dev/taski$ git push 
```
- Зайдите в репозиторий с проектом Kittygram в своём аккаунте на GitHub и откройте раздел Actions — из этого раздела происходит управление процессами CI/CD для репозитория.

### Workflow для CI: линтер и тесты
- workflow состоит из четырёх фаз:
1. Тестирование проекта.
2. Сборка и публикация образа.
3. Автоматический деплой.
4. Отправка уведомления в персональный чат.
- Приступаем к Continuous Integration: настроим тестирование проект
- Опишите в воркфлоу проверку кода на соответствие PEP8 и запуск тестов. И всё это выполнит GitHub Actions.

### Cекреты в GitHub Actions
- На платформе GitHub Actions перейдите в настройки репозитория — Settings, выберите на панели слева Secrets and Variables → Actions, нажмите New repository secret
- Сохраните переменные DOCKER_USERNAME и DOCKER_PASSWORD с необходимыми значениями: введите имя секрета и его значение, затем нажмите Add secret, а так же добавьте другиепеременные

### Отчёт в Telegram
- Чтобы не отслеживать выполнение workflow в GitHub Actions, можно подключить к работе вашего Telegram-бота: пусть он отправляет оповещение об успешном завершении процесса. Проверьте, работает ли ваш бот, и дайте ему задачу.

### Workflow для CD: сборка образов и перезапуск контейнеров
- Сборку и пуш обновлённых образов опишbnt в отдельной задаче в пределах того же workflow: создайте job с именем build_and_push_to_docker_hub.
- Бот станет отправлять уведомления в аккаунт "Деплой успешно выполнен!" 

### Автор
```
- Елена Зуева
```
```
 https://github.com/ElenaZuevaEkb
```
