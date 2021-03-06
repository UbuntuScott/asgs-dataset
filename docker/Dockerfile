FROM alpine:3.9

RUN apk --no-cache add tini-static busybox-suid python3 libxml2 py3-lxml uwsgi uwsgi-python3

RUN mkdir -p /deploy
RUN chown -R uwsgi:uwsgi /deploy
ADD ./requirements.txt /deploy/
ADD ./asgs_dataset /deploy/asgs_dataset/

WORKDIR /deploy

RUN python3 -m ensurepip
RUN pip3 install -r requirements.txt
RUN echo $'import sys\n\
sys.path.insert(0, "/deploy/asgs_dataset")\n\
sys.path.insert(0, "/deploy")\n\
from asgs_dataset.app import app as application\n' > wsgi.py

RUN chown -R uwsgi:uwsgi /deploy
RUN chmod -R 777 /deploy
RUN chmod a+x ./wsgi.py ./asgs_dataset/app.py

RUN echo $'ini = :python3_asgs_ldapi\n\
\n[python3_asgs_ldapi]\n\
plugins = python3\n\
callable = application\n\
wsgi-file = wsgi.py\n' >> /etc/uwsgi/uwsgi.ini

RUN mkdir -p /var/run/uwsgi
RUN chown -R uwsgi:uwsgi /var/run/uwsgi
RUN chmod -R 777 /var/run/uwsgi
VOLUME /var/run/uwsgi
CMD ["uwsgi", "--ini", "/etc/uwsgi/uwsgi.ini", "--master", "--workers", "2", "--protocol", "uwsgi", "--enable-threads", "--chmod-socket=666", "--socket", "/var/run/uwsgi/uwsgi.sock"]
ENTRYPOINT [ "/sbin/tini-static" ]
