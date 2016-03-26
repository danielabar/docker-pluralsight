# Simple web server for Pluralsight course
FROM phusion/baseimage:0.9.15
MAINTAINER test@test.test

# Combine install instructions in one line to minimize layers
RUN apt-get update && apt-get install -y \
  apache2 \
  apache2-utils \
  vim \
  && apt-get clean \
  && rm -rf /var/lib/apt/lists/* /tmp/* /var/tmp/*

# Container networking
EXPOSE 80

# Commands are written in exec style
# Start Apache web server
CMD ["apache2ctl", "-D", "FOREGROUND"]
