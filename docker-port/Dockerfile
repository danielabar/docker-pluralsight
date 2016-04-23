# Test for networking module

FROM phusion/baseimage

# RUN apt-get update && apt-get install -y iputils-ping traceroute apache2 apache2-utils
RUN apt-get update && apt-get install -y \
  apache2 \
  apache2-utils

EXPOSE 80

ENTRYPOINT ["apache2ctl"]

# arguments passed to entrypoint
CMD ["-D", "FOREGROUND"]
