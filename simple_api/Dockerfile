FROM python:2.7-stretch
LABEL maintainer="victoria and antoine"
RUN apt-get update -y && apt-get install python-dev python3-dev libsasl2-dev python-dev libldap2-dev libssl-dev -y
RUN  pip install flask flask_httpauth flask_simpleldap python-dotenv
RUN mkdir /data
WORKDIR /app
ADD . /app
ADD student_age.json /data
VOLUME ["/data"]
EXPOSE 5000
CMD [ "python", "/app/student_age.py" ]
