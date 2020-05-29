FROM ubuntu:18.04

RUN apt-get update 
RUN apt-get install -y wget sudo curl bash systemd
RUN wget https://github.com/gohornet/hornet/releases/download/v0.4.0-rc11/hornet_0.4.0-rc11_amd64.deb
RUN apt install -y ./hornet_0.4.0-rc11_amd64.deb; systemctl enable hornet