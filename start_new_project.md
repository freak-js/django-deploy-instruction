# Напоминалка по созданию нового проекта в ручном режиме.

**Создаем новый репозиторий, пишем README.md и копируем путь для клонирования.**

**Из домашнего каталога:**

_mkdir имя_каталога_с_виртуальным_окружением_

_cd имя_каталога_с_виртуальным_окружением_

_python3.8 -m venv myvenv_

_source myvenv/bin/activate_

_git clone (ссылка на репу для клонирования)_

_cd имя_каталога_склонированного_репозитирия_

_touch .gitignore_

__________

**Добавляем в документ .gitignore:**

_*.sqlite3_
_*.pyc_

__________

**Апгрейдим пипку, что бы не ругался и стартуем проект:**

_pip install --upgrade pip_

_pip install Django==2.2.9   (вставить номер актуальной для конкретного проекта версии django)_

_django-admin startproject имя_проекта_

__________

**Сразу правим в settings.py нужные поля:**

_TIME_ZONE = 'Europe/Moscow'_

_LANGUAGE_CODE = 'ru-ru'_

_STATIC_URL = '/static/'_

_STATIC_ROOT = os.path.join(BASE_DIR, 'static')_

__________

**Проверяем стартует ли сервер разработки:**

_python manage.py runserver_

__________

**Выполняем миграции в базу данных (в дальнейшем придется перевыполнить их из-за аутентификации):**

_python manage.py migrate_

__________

**Создаем приложение:**

_python manage.py startapp имя_приложения_

__________

**Добавляем прилодение в список приложений в settings.py:**

_INSTALLED_APPS = [
    'django.contrib.admin',
    'django.contrib.auth',
    'django.contrib.contenttypes',
    'django.contrib.sessions',
    'django.contrib.messages',
    'django.contrib.staticfiles',
    'имя_приложения',
]_

