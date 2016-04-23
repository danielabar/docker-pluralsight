# Test for networking module

FROM phusion/baseimage

# RUN apt-get update && apt-get install -y iputils-ping traceroute apache2 apache2-utils
RUN apt-get update && apt-get install -y \
  apache2 \
  apache2-utils

# only need 80 for apache, the others are for demonstrating -P flag
EXPOSE 80 100 400

ENTRYPOINT ["apache2ctl"]

# arguments passed to entrypoint
CMD ["-D", "FOREGROUND"]
