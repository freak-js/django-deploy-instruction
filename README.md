# django-deploy-instruction
## Инструкция-заметки по развертыванию django приложения на слабом сервере без docker.

_sudo apt-get update_

_sudo apt-get upgrade_

**Проверяем есть ли проблемы с локалью:**

_perl -v_

**если есть ошибки — генерируем реконфигурируем локаль:**

_locale-gen en_US en_US.UTF-8 ru_RU.UTF-8_

_dpkg-reconfigure locales_

**и снова проверяем на ошибки:**

_perl -v_
____________________________

Устанавливаем python 3.8.1:
sudo apt install build-essential checkinstall

sudo apt install libreadline-gplv2-dev libncursesw5-dev libssl-dev libsqlite3-dev tk-dev libgdbm-dev libc6-dev libbz2-dev libffi-dev zlib1g-dev

Установка Tkinter:

sudo apt install python3-tk

Скачиваем архив с дистрибутивом языка в каталог /opt:

cd /opt

sudo wget https://www.python.org/ftp/python/3.8.1/Python-3.8.1.tgz

Распаковываем файл исходного кода из архива:

sudo tar xzf Python-3.8.1.tgz

Компиляции исходного кода Python:

cd Python-3.8.1

sudo ./configure --enable-optimizations

sudo make altinstall

Проверка версии python на удаленной машине:

python3.8 -V
____________________________

Сносим apache2:

sudo service apache2 stop

sudo apt-get remove --purge apache2 apache2-utils apache2.2-bin apache2-common

sudo update-rc.d -f apache2 remove

sudo apt-get autoremove

sudo apt-get autoclean

whereis apache2

sudo rm -Rf (сюда вставить вывод от прошлой команды, если он есть)

**********
Пример: sudo rm -Rf /usr/sbin/apache2 /usr/lib/apache2 /etc/apache2 /usr/share/apache2 /usr/share/man/man8/apache2.8.gz
**********
____________________________

Генерируем виртуальное окружениев каталоге /home(предварительно перейдя туда и создав папку проекта внутри):

mkdir bergauf

cd bergauf

python3.8 -m venv myvenv
____________________________

Устанавливаем git, nginx, gunicorn, htop, nano:

sudo apt-get install htop   (устанавливается глобально)

sudo apt-get install git   (устанавливается глобально)

sudo apt-get install nginx   (устанавливается глобально)

sudo apt install nano   (устанавливается глобально)

pip install gunicorn   (внутри виртуального окружения)

Клонируем репозиторий из github:

git clone ...
____________________________

Проверяем работу Gunicorne:

cd ~/myproject

gunicorn --bind 0.0.0.0:8000 myproject.wsgi

____________________________

Создаем сервисный файл Gunicorn systemd:

sudo nano /etc/systemd/system/gunicorn.service

**********
Внутри файла пишем следующее:
**********

"""

[Unit]
Description=gunicorn daemon
After=network.target

[Service]
User=root
Group=www-data
WorkingDirectory=/home/bergauf/bergauf
ExecStart=/home/bergauf/myvenv/bin/gunicorn --access-logfile - --workers 3 --bind unix:/home/bergauf/bergauf.sock bergauf.wsgi:application

[Install]
WantedBy=multi-user.target

"""
Стартуем и автозапускаем службу:

sudo systemctl start gunicorn

sudo systemctl enable gunicorn

Проверяем состояние процесса:

sudo systemctl status gunicorn

Если есть какие0то ошибки, то проверяем журнал:

sudo journalctl -u gunicorn

Ссылка на туториал:

https://www.digitalocean.com/community/tutorials/how-to-set-up-django-with-postgres-nginx-and-gunicorn-on-ubuntu-16-04#create-a-gunicorn-systemd-service-file

**********
Что-бы внести изменения в файл конфигурации gunicorne выполнить после изменений:

sudo systemctl daemon-reload

sudo systemctl restart gunicorn
**********

____________________________

Перед настройкой nginx меняем настройки проекта:

DEBUG = False

ALLOWED_HOSTS = ['айпишник сервера или имя домена']

STATIC_ROOT = os.path.join(BASE_DIR, 'static/')

Выполняем внутри виртального окружения:

python manage.py collectstatic
____________________________

Настройка Nginx на Proxy Pass для Gunicorn:

sudo nano /etc/nginx/sites-available/myproject

**********
Внутри файла пишем следующее:
**********

"""
server {
    listen 80;
    server_name 45.128.204.29;

    location = /favicon.ico { access_log off; log_not_found off; }
    location /static/ {
        root /home/bergauf/bergauf;
    }

    location / {
        include proxy_params;
        proxy_pass http://unix:/home/bergauf/bergauf.sock;
    }
}
"""

Включаем файл, связав его с sites-enabled каталогом:

sudo ln -s /etc/nginx/sites-available/bergauf /etc/nginx/sites-enabled

Проверка конфигурации Nginx на наличие синтаксических ошибок:

sudo nginx -t

Если об ошибках не сообщается, перезапустить Nginx:

sudo systemctl restart nginx
