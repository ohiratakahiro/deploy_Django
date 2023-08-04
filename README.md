# deploy_Django

動かし方 開発環境
(仮想環境を有効化する)

pip install -r requirements.txt
python manage.py makemigrations
python manage.py migrate
python manage.py runserver

その後、 http://127.0.0.1:8000 にブラウザでアクセスしてください。
