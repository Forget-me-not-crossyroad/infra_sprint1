# Kittygram — социальная сеть для обмена фотографиями любимых питомцев.

### Описание проекта
Пользователи могут регистрироваться, загружать фотографии с описанием и смотреть питомцев других пользователей.

### Установка
<i>Примечание: Все примеры указаны для Linux</i><br>
1. Склонируйте репозиторий на свой компьютер:
    ```
    git clone https://github.com/Forget-me-not-crossyroad/kittygram_final.git
    ```
2. Создайте файл `.env` и заполните его своими данными. Все необходимые переменные перечислены в файле `.env.example`, находящемся в корневой директории проекта.

## Технологии
Python 3.11.5,
Django 3.2.3,
djangorestframework==3.12.4,
gunicorn==20.1.0,
PostgreSQL 13.10
Nginx 1.20.2
 
## Установка проекта на локальный компьютер из репозитория 
 - Клонировать репозиторий `git clone <адрес вашего репозитория>`
 - Перейти в директорию с клонированным репозиторием
 - Установить виртуальное окружение `python -m venv venv`
 - Установить зависимости `pip install -r requirements.txt`
 - В директории /backend/kittygram_backend/ создать файл .env
 - В файле .env прописать ваш SECRET_KEY в виде: `SECRET_KEY = '<ваш_ключ>'`
# Деплой проекта на удаленный сервер
## Подключение сервера к аккаунту на GitHub
- Обновить или установить Git. `sudo apt update` `sudo apt install git`
- Сгенерировать SSH-ключи `ssh-keygen`
- Сохранить открытый ключ в вашем аккаунте на GitHub. Для этого вывести ключ в терминал командой `cat .ssh/id_rsa.pub`. Скопировать ключ от символов ssh-rsa, включительно, и до конца. Добавить это ключ к вашему аккаунту на GitHub.
- Клонировать проект с GitHub на сервер: `git clone git@github.com:Ваш_аккаунт/<Имя проекта>.git`
## Запуск backend проекта на сервере
- Установить pip и venv `sudo apt install python3-pip python3-venv -y`
- Cоздать и активировать виртуальное окружение `python3 -m venv venv` `source venv/bin/activate`
- Установить зависимости `pip install -r requirements.txt`
- Выполнить миграции `python manage.py migrate`
- Создать суперюзера `python manage.py createsuperuser`
- Отредактировать settings.py на сервере: `ALLOWED_HOSTS = ['<внешний адрес вашего сервера>', '127.0.0.1', 'localhost']`
## Запуск frontend проекта на сервере
- Установить на сервер `Node.js`:
`curl -fsSL https://deb.nodesource.com/setup_18.x | sudo -E bash - &&\`
`sudo apt-get install -y nodejs`
- Установить зависимости. Из директории `<ваш проект>/frontend/` выполнить команду: `npm i`
## Установка и запуск Gunicorn
- При активированном venv установить gunicorn `pip install gunicorn==20.1.0`
- Открыть файл settings.py и установить `DEBUG = False`
- В директории _/etc/systemd/system/_ создать файл _gunicorn.service_ `sudo vim /etc/systemd/system/gunicorn.service`  со следующим кодом:
        [Unit]
    
        Description=gunicorn daemon
    
        After=network.target
    
        [Service]
    
        User=yc-user
    
        WorkingDirectory=/home/<имя пользователя в системе>/<имя проекта>/backend/
    
        ExecStart=/home/<имя пользователя в системе>/<имя проекта>/venv/bin/gunicorn --bind 0.0.0.0:8080 kittygram_backend.wsgi
    
        [Install]
    
        WantedBy=multi-user.target
## Установка и настройка Nginx
- Dыполнить команду: `sudo apt install nginx -y`
- Для установки ограничений на открытые порты выполнить команду: `sudo ufw allow 'Nginx Full' && sudo ufw allow OpenSSH`
- Включить файервол `sudo ufw enable`
### Собрать и разместить статику frontend-приложения.
- Перейти в директорию _<имя_проекта>/frontend/_  и выполнить команду `npm run build` Результат сохранится в директории ..._/frontend/build/_.  В системную директорию сервера _/var/www/_ скопировать содержимое папки _/frontend/build/_
- открыть файл конфигурации веб-сервера `sudo vim /etc/nginx/sites-enabled/default` и заменить его содержимое следующим кодом:
        
        server {
    
            listen 80;
            server_name публичный_ip_вашего_удаленного_сервера;
        
            location / {
            root   /var/www/<имя проекта>;
            index index.html index.htm;
            try_files $uri /index.html;
            }
    
        }
- Проверить корректность конфигурации `sudo nginx -t`
- Перезагрузить Nginx `sudo systemctl reload nginx`
    ### Настроить проксирование запросов
- Открыть файл конфигурации Nginx _/etc/nginx/sites-enabled/default_ и добавить в него ещё один блок `location`
        server {
    
            listen 80;
            server_name публичный_ip_вашего_удаленного_сервера;
    
            location /api/ {
                proxy_pass http://127.0.0.1:8080;
            }
            
            location /admin/ {
                proxy_pass http://127.0.0.1:8000;
                }
            
            location / {
                root   /var/www/<имя_проекта>;
                index index.html index.htm;
                try_files $uri /index.html;
            }
    
        }
- Сохранить изменения, проверить и перезагрузить конфигурацию веб-сервера:
        sudo nginx -t
        sudo systemctl reload nginx
    ### Собрать и настроить статику для backend-приложения.
- в файле _settings.py_ прописать настройки 
    
        STATIC_URL = 'static_backend'
        STATIC_ROOT = BASE_DIR / 'static_backend'
- Активировать venv, перейти в директорию с файлом _manage.py_ и выполнить команду `python manage.py collectstatic`
- В директории_<имя_проекта>/backend/_ будет создана директория _static_backend/_ 
- Скопировать директорию _static_backend/_ в директорию _/var/www/<имя_проекта>/_
## Добавление доменного имени в настройки Django
- В файле _settings.py_ добавить в список `ALLOWED_HOSTS` доменное имя: 
    ALLOWED_HOSTS = ['ip_адрес_вашего_сервера', '127.0.0.1', 'localhost', 'ваш-домен'] 
- Сохранить изменения и перезапустить gunicorn `sudo systemctl restart gunicorn`
- Внести изменения в конфигурацию Nginx. Открыть конфигурационный файл командой: `sudo nano /etc/nginx/sites-enabled/default`
- Добавьте в строку `server_name` выданное вам доменное имя (через пробел, без `< >`):
        server {
        ...
            server_name <ваш-ip> <ваш-домен>;
        ...
        }
- Проверить конфигурацию `sudo nginx -t` и перезагрузить её командой `sudo systemctl reload nginx`, чтобы изменения вступили в силу.
 ## Получение и настройка SSL-сертификата
 **Установка certbot**
 - Зайдите на сервер и последовательно выполните команды:
        sudo apt install snapd
        sudo snap install core; sudo snap refresh core
        sudo snap install --classic certbot
        sudo ln -s /snap/bin/certbot /usr/bin/certbot
- Запустить certbot и получить SSL-сертификат:
        sudo certbot --nginx
- сертификат автоматически сохранится на вашем сервере в системной директории _/etc/ssl/_  Также будет автоматически изменена конфигурация Nginx: в файл _/etc/nginx/sites-enabled/default_ добавятся новые настройки и будут прописаны пути к сертификату.
- перезагрузить конфигурацию Nginx `sudo systemctl reload nginx`
