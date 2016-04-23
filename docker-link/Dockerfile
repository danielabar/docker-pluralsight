#Test for networking module: Linking
FROM phusion/baseimage

RUN apt-get update && apt-get install -y iputils-ping apache2

EXPOSE 80

ENTRYPOINT ["apache2ctl"]
CMD ["-D", "FOREGROUND"]
