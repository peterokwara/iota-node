FROM ubuntu:18.04

RUN apt-get update
RUN apt-get install -y sudo bash systemd wget gnupg
RUN wget -qO - https://ppa.hornet.zone/pubkey.txt | sudo apt-key add -
RUN sudo sh -c 'echo "deb http://ppa.hornet.zone testing main" >> /etc/apt/sources.list.d/hornet.list'
RUN sudo apt-get update
RUN apt-get install -y hornet; systemctl enable hornet