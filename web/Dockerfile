# go to dockerhub and choose an image of python version you want to use
FROM python:3.9.7-slim

COPY . /app
COPY .env /app/.env
WORKDIR /app

RUN python3 -m venv /opt/venv

RUN /opt/venv/bin/pip install pip --upgrade && \
    /opt/venv/bin/pip install -r requirements.txt && \
    # chmod +x entrypoint.sh # in production env, we need gunicorn to run server but not python manager.py runserver to guarantee high concurrency and high reliability
RUN python3 .\\web.\\manage.py runserver

# in production environment, we need gunicorn to run server, so we need a bash
# CMD ["/app/entrypoint.sh"]