Создаем новый репозиторий, пишем README.md и копируем путь для клонирования.

Из домашнего каталога:

mkdir имя_каталога_с_виртуальным_окружением

cd имя_каталога_с_виртуальным_окружением

python3.8 -m venv myvenv

source myvenv/bin/activate

git clone (ссылка на репу для клонирования)

cd имя_каталога_склонированного_репозитирия

touch .gitignore

Добавляем в документ .gitignore:

*.sqlite3
*.pyc

Апгрейдим пипку, что бы не ругался и стартуем проект:

pip install --upgrade pip

pip install Django==2.2.9   (вставить номер актуальной для конкретного проекта версии django)

django-admin startproject имя_проекта .

Сразу правим в settings.py нужные поля:

TIME_ZONE = 'Europe/Moscow'

LANGUAGE_CODE = 'ru-ru'

STATIC_URL = '/static/'

STATIC_ROOT = os.path.join(BASE_DIR, 'static')

Проверяем стартует ли сервер разработки:

python manage.py runserver

Выполняем миграции в базу данных (в дальнейшем придется перевыполнить их из-за аутентификации):

python manage.py migrate

Создаем приложение:

python manage.py startapp имя_приложения

Добавляем прилодение в список приложений в settings.py:

INSTALLED_APPS = [
    'django.contrib.admin',
    'django.contrib.auth',
    'django.contrib.contenttypes',
    'django.contrib.sessions',
    'django.contrib.messages',
    'django.contrib.staticfiles',
    'имя_приложения',
]

