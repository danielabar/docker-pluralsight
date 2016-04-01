FROM phusion/baseimage:0.9.15
MAINTAINER test@test.test

RUN apt-get update && apt-get install -y iputils-ping apache2

ENV var1=Nigel var2=Poulton
ENV var3=echo var4=HELLOOOO

# Example of using environment variables in Dockerfile
CMD $var3 $var4
