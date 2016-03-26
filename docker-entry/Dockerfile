FROM phusion/baseimage:0.9.15
MAINTAINER test@test.test

RUN apt-get update && apt-get install -y iputils-ping apache2

ENTRYPOINT ["apache2ctl"]
