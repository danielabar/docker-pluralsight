FROM phusion/baseimage

RUN apt-get update

# Create an error on purpose (there is no such package named ping)
# to demonstrate use of intermediate containers in debugging.
RUN apt-get install -y ping
