#!/bin/bash

apt-get -y update
apt-get -y install nginx
export DROPLET_ID=$(curl http://169.254.169.254/metadata/v1/id)
export HOSTNAME=$(curl -s http://169.254.169.254/metadata/v1/hostname)
export PUBLIC_IPV4=$(curl -s http://169.254.169.254/metadata/v1/interfaces/public/0/ipv4/address)
echo Droplet: $HOSTNAME, ID: $DROPLET_ID, IP Address: $PUBLIC_IPV4 > /usr/share/nginx/html/index.html
