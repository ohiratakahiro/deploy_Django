# deploy_Django

動かし方 開発環境
(仮想環境を有効化する)

```
pip install -r requirements.txt
python manage.py makemigrations
python manage.py migrate
python manage.py runserver
```

その後、 http://127.0.0.1:8000 にブラウザでアクセスしてください。


本番環境へのデプロイについて
対象のLinuxディストリビューション
Ubuntu, 22.04 LTS で動作を確認しています。

サーバーで使用しているソフトウェア一覧
Python
Django
Gunicorn (Djangoを動作させるもの)
Nginx (Webサーバー)
PostgreSQL (DBサーバー)
```
手順
       →      →
ブラウザ   Nginx   Gunicorn+Django
       ←       ←
```

```
初期設定
sudo apt update
sudo apt upgrade
インストール可能なパッケージの一覧の更新(update)と、インストール済みのパッケージ一覧を更新する(upgrade)
```

PythonとDjangoインストール
Pythonのコンパイルに必要なパッケージをインストールする
```
sudo apt install build-essential libbz2-dev libdb-dev \
  libreadline-dev libffi-dev libgdbm-dev liblzma-dev \
  libncursesw5-dev libsqlite3-dev libssl-dev \
  zlib1g-dev uuid-dev tk-dev
```

Pythonのダウンロードと、解答
```
wget  https://www.python.org/ftp/python/3.11.1/Python-3.11.1.tar.xz
tar xJf Python-3.11.1.tar.xz
```

Pythonのコンパイル
```
cd Python-3.11.1
./configure
make
sudo make altinstall
```

Pythonが使えるか、ここで確認しておく
```
sudo python3.11 -V
```


Nginxインストール
```
sudo apt install nginx
sudo systemctl start nginx
sudo systemctl enable nginx
```
AWSの場合、セキュリティグループを設定し、パブリックIPアドレスをコピーしてページにアクセスして、無事に表示できるか一度確認する


次に、Nginxの設定ファイルを作成する
```
sudo nano /etc/nginx/conf.d/django.conf
```

中身を次のようにする
```
server {
    listen 80;
    server_name IPアドレスを入れる;
    location /static {
        alias /var/www/static;
       }
    location /media {
        alias /var/www/media;
    }
    location / {
        proxy_pass http://127.0.0.1:8000;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for; proxy_set_header Host $http_host;
        proxy_redirect off;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
}
```

設定ファイルを作成したので、nginxを再起動する
```
sudo systemctl reload nginx
```
ブラウザでアクセスして確認してみる、現段階では 502 Bad Gatewayと出る

Djangoソースコードのクローンと、起動確認
Djangoなど、必要なライブラリをインストールする
```
sudo python3.11 -m pip install django gunicorn psycopg2-binary pillow
```

Djangoソースコードをクローンする
```
cd ~
git clone https://github.com/naritotakizawa/group_e
```

本番環境用のsettingsを作成する
```
cd group_e
nano conf/production_settings.py
```

中身を次のようにする
```
import os
from .settings import *

ALLOWED_HOSTS = ['54.250.226.14']

DEBUG = False

DATABASES = {
    'default': {
        'ENGINE': 'django.db.backends.postgresql',
        'NAME': 'djangodb',
        'USER': 'django_user',
        'PASSWORD': 'django',
        'HOST': '',
        'PORT': 5432,
    }
}

LOGGING = {
    'version': 1,
    'disable_existing_loggers': False,
    'formatters': {
        'standard': {
            'format': "[%(asctime)s] %(levelname)s [%(name)s:%(lineno)s] %(message)s",
            'datefmt': "%d/%b/%Y %H:%M:%S"

        },
    },
    'handlers': {
        'file': {
            'class': 'logging.handlers.TimedRotatingFileHandler',
            'filename': '/var/log/django.log',
            'formatter': 'standard',
            'when': 'W0',
        },
    },
    'loggers': {
        'django': {
            'handlers': ['file'],
            'level': 'ERROR',
        },
    },
}

STATIC_ROOT = '/var/www/static'

MEDIA_ROOT = '/var/www/media'
```

ログファイルにエラーを書くようにするので、ログファイル作成と権限の設定
```
sudo touch /var/log/django.log
sudo chmod 777 /var/log/django.log
```

gunicornコマンドで、動作を確認してみる
```
/usr/local/bin/gunicorn --workers 3 --bind 127.0.0.1:8000 conf.wsgi:application --env DJANGO_SETTINGS_MODULE=conf.production_settings
```
ブラウザでアクセスすると500エラーになる。これは Django側でエラーがあった場合に見るエラー。ログファイルを確認してみる
```
sudo cat /var/log/django.log
```
