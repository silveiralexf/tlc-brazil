release: yes "yes" | python manage.py migrate
web: uwsgi --http-socket=:$PORT --master --workers=2 --threads=8 --die-on-term --wsgi-file=tlcbr/wsgi_production.py  --static-map /media/=/app/tlcbr/media/ --offload-threads 1
