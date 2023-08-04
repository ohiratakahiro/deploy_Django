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
