Kittygram - Социальная сеть для ваших питомцев.

Установка на сервер:
https://github.com/AleksandrIbragimov/infra_sprint1

Установить виртуальное окружение и зависимости:
python3 -m venv venv
pip install -r requirements.txt

Деплой проекта на удаленный сервер
Установить Git:
sudo apt install git

Сгенерировать пару SSH-ключей:
ssh-keygen

Сохранить открытый ключ на gitHub аккаунте:
cat .ssh/id_rsa.pub

Клонировать проект на удаленный сервер:
git clone git@github.com:Ваш_аккаунт/Название_проекта.git

Запуск бэкенда
sudo apt install python3-pip python3-venv -y

Создать и активировать виртуальное окружение, установить зависимости, выполнить миграции:
python3 -m venv venv
source venv/bin/activate
pip install -r requirements.txt
python manage.py migrate

Создать суперпользователя:
python manage.py createsuperuser

В файле settings.py в ALLOWED_HOSTS добавить IP-адрес сервера, а так же '127.0.0.1', 'localhost' и домен.

Запуск фронтенда

Установить на сервер пакетный менеджер npm:
curl -fsSL https://deb.nodesource.com/setup_18.x | sudo -E bash - &&\
sudo apt-get install -y nodejs

Установить зависимости для фронтенда из директории infra_sprint1/frontend:
npm i

Установка и запуск Gunicorn
pip install gunicorn==20.1.0

Создать демона для gunicorn:
sudo nano /etc/systemd/system/gunicorn_kittygram.service 

В файле gunicorn_kittygram.service описать конфигурацию:

[Unit]
Description=gunicorn daemon 
After=network.target 

[Service]
User=yc-user 
WorkingDirectory=/home/yc-user/taski/backend/
ExecStart=/home/yc-user/taski/backend/venv/bin/gunicorn_kittygram --bind 0.0.0.0:8000 backend.wsgi

[Install]
WantedBy=multi-user.target  

Запустить gunicorn_kittygram.service:
sudo systemctl start gunicorn
sudo systemctl enable gunicorn

Установка Nginx
sudo apt install nginx -y
Запустить Nginx:
sudo systemctl start nginx
Указать файрволу, какие порты должны остаться открытыми:
sudo ufw allow 'Nginx Full'
sudo ufw allow OpenSSH
Включить файрвол:
sudo ufw enable

Собрать статику фронтенд-приложения
Из директории infra_sprint1/frontend запустить:
npm run build
sudo cp -r /home/yc-user/infra_sprint1/frontend/build/. /var/www/kittygram/
Описываем настройки для работы со статикой:
sudo nano /etc/nginx/sites-enabled/default

Удалить все найтроки из файла и добавить:

server {
    server_name ВАШ_IP ВАШ ДОМЕН;
    client_max_body_size 10m;
    server_tokens off;

    location /api/ {
        proxy_pass http://127.0.0.1:8080;
    }
    location /admin/ {
        proxy_pass http://127.0.0.1:8080;
    }

    location /media/ {
        alias /var/www/kittygram/media/;
    }
    location / {
        root   /var/www/kittygram;
        index  index.html index.htm;
        try_files $uri /index.html;
    }


В файле settings.py изменить(добавить):
STATIC_URL = 'static_backend' 
STATIC_ROOT = BASE_DIR / 'static_backend'

При активированном виртуальном окрежении выполните команду:
python manage.py collectstatic

Перейдите в корень проекта и выполните команду:
sudo cp -r infra_sprint1/backend/static_backend/ /var/www/kittygram/

Натройка шифрования
Установить certbot:
sudo apt install snapd
sudo snap install core; sudo snap refresh core
sudo snap install --classic certbot
sudo ln -s /snap/bin/certbot /usr/bin/certbot
sudo certbot --nginx
sudo systemctl reload nginx
sudo certbot certificates
sudo certbot renew --dry-run
