# Simple web server for Pluralsight course
FROM phusion/baseimage:0.9.15
MAINTAINER test@test.test

RUN apt-get update
RUN apt-get install -y apache2

# In a real world scenario, probably would not install these on a production web server,
# but this is for demonstration purposes, learning about layers.
RUN apt-get install -y apache2-utils
RUN apt-get install -y vim

# Good practice to clean up apt cache to keep containers lean
RUN apt-get clean

# Container networking
EXPOSE 80

# Commands are written in exec style
# Start Apache web server
CMD ["apache2ctl", "-D", "FOREGROUND"]
