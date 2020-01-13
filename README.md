# django-instruction
## Инструкция-заметки по развертыванию django приложения на слабом сервере без docker.
## AdminVPS тариф VPSLite

### Обновляем пакеты из репозитория:

_sudo apt-get update_

_sudo apt-get upgrade_

____________________________

### Проверяем есть ли проблемы с локалью:

_perl -v_

**если есть ошибки — генерируем и реконфигурируем локаль:**

_locale-gen en_US en_US.UTF-8 ru_RU.UTF-8_

_dpkg-reconfigure locales_

**и снова проверяем на ошибки:**

_perl -v_

____________________________

### Устанавливаем python 3.8.1:

_sudo apt install build-essential checkinstall_

_sudo apt install libreadline-gplv2-dev libncursesw5-dev libssl-dev libsqlite3-dev tk-dev libgdbm-dev libc6-dev libbz2-dev libffi-dev zlib1g-dev_

**Установка Tkinter:**

_sudo apt install python3-tk_

**Скачиваем архив с дистрибутивом языка в каталог /opt:**

_cd /opt_

_sudo wget https://www.python.org/ftp/python/3.8.1/Python-3.8.1.tgz_

**Распаковываем файл исходного кода из архива:**

_sudo tar xzf Python-3.8.1.tgz_

**Компиляции исходного кода Python:**

_cd Python-3.8.1_

_sudo ./configure --enable-optimizations_

_sudo make altinstall_

**Проверка версии python на удаленной машине:**

_python3.8 -V_

____________________________

### Сносим apache2:

_sudo service apache2 stop_

_sudo apt-get remove --purge apache2 apache2-utils apache2.2-bin apache2-common_

_sudo update-rc.d -f apache2 remove_

_sudo apt-get autoremove_

_sudo apt-get autoclean_

_whereis apache2_

_sudo rm -Rf (сюда вставить вывод от прошлой команды, если он есть)_

____________________________

_Пример: sudo rm -Rf /usr/sbin/apache2 /usr/lib/apache2 /etc/apache2 /usr/share/apache2 /usr/share/man/man8/apache2.8.gz_

____________________________

### Генерируем виртуальное окружениев каталоге /home(предварительно перейдя туда и создав папку проекта внутри):

_mkdir имя_папки_с_проектом_

_cd имя_папки_с_проектом_

_python3.8 -m venv myvenv_
____________________________

### Устанавливаем git, nginx, gunicorn, htop, nano:

_sudo apt-get install htop (устанавливается глобально)_

_sudo apt-get install git (устанавливается глобально)_

_sudo apt-get install nginx (устанавливается глобально)_

_sudo apt install nano (устанавливается глобально)_

_pip install gunicorn (внутри виртуального окружения)_

**Клонируем репозиторий из github:**

_git clone адрес_репозитория..._
____________________________

### Проверяем работу Gunicorne:

_cd ~/имя_папки_с_проектом_

_gunicorn --bind 0.0.0.0:8000 имя_проекта.wsgi_

____________________________

### Создаем сервисный файл Gunicorn systemd:

_sudo nano /etc/systemd/system/gunicorn.service_

**Внутри файла пишем следующее:**

```
[Unit]
Description=gunicorn daemon
After=network.target

[Service]
User=root
Group=www-data
WorkingDirectory=/home/имя_папки_с_проектом/имя_проекта
ExecStart=/home/имя_папки_с_проектом/myvenv/bin/gunicorn --access-logfile - --workers 3 --bind unix:/home/имя_папки_с_проектом/имя_проекта.sock имя_проекта.wsgi:application

[Install]
WantedBy=multi-user.target
```

**Стартуем и автозапускаем службу:**

_sudo systemctl start gunicorn_

_sudo systemctl enable gunicorn_

**Проверяем состояние процесса:**

_sudo systemctl status gunicorn_

**Если есть какие-то ошибки, то проверяем журнал:**

_sudo journalctl -u gunicorn_

**Ссылка на туториал:**

_https://www.digitalocean.com/community/tutorials/how-to-set-up-django-with-postgres-nginx-and-gunicorn-on-ubuntu-16-04#create-a-gunicorn-systemd-service-file_

**Что-бы внести изменения в файл конфигурации gunicorne выполнить после изменений:**

_sudo systemctl daemon-reload_

_sudo systemctl restart gunicorn_
____________________________

### Перед настройкой nginx меняем настройки проекта:

```
DEBUG = False

ALLOWED_HOSTS = ['айпишник сервера или имя домена']

STATIC_ROOT = os.path.join(BASE_DIR, 'static/')
```

**Выполняем внутри виртального окружения:**

_python manage.py collectstatic_
____________________________

### Настройка Nginx на Proxy Pass для Gunicorn:

_sudo nano /etc/nginx/sites-available/имя_проекта_


**Внутри файла пишем следующее:**


```
server {
    listen 80;
    server_name 45.128.204.29;

    location = /favicon.ico { access_log off; log_not_found off; }
    location /static/ {
        root /home/имя_папки_с_проектом/имя_проекта;
    }

    location / {
        include proxy_params;
        proxy_pass http://unix:/home/имя_папки_с_проектом/имя_проекта.sock;
    }
}
```

**Включаем файл, связав его с sites-enabled каталогом:**

_sudo ln -s /etc/nginx/sites-available/имя_проекта /etc/nginx/sites-enabled_

**Проверка конфигурации Nginx на наличие синтаксических ошибок:**

_sudo nginx -t_

**Если об ошибках не сообщается, перезапустить Nginx:**

_sudo systemctl restart nginx_
