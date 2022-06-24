# omada-docker

This guide has four sets of instructions for installing an Omada Controller.

1. [**Easiest**](#omada-controller-with-docker-compose): Run your Omada controller as a docker container via Docker Compose
2. [**Easy**](#omada-controller-with-docker-run): Run your Omada controller as a docker container wherever you want
3. [**But why?**](#omada-controller-automatically-as-lxc-in-proxmox): Install Omada Controller automatically in a Rocky Linux LXC container in Proxmox
4. [**Are you insane?**](#omada-controller-manually-as-lxc-in-proxmox): Install Omada Controller manually in an Ubuntu 16.04 LXC container in Proxmox

## Omada Controller with Docker Compose

Create the appropriate directories for the volumes and then use this compose file:

```yaml
version: '3.3'
services:
  omada-controller:
    container_name: omada-controller
    restart: unless-stopped
    ports:
      - '8088:8088'
      - '8043:8043'
      - '8843:8843'
      - '29810:29810/udp'
      - '29811:29811'
      - '29812:29812'
      - '29813:29813'
      - '29814:29814'
    environment:
      - MANAGE_HTTP_PORT=8088
      - MANAGE_HTTPS_PORT=8043
      - PORTAL_HTTP_PORT=8088
      - PORTAL_HTTPS_PORT=8843
      - SHOW_SERVER_LOGS=true
      - SHOW_MONGODB_LOGS=false
      - SSL_CERT_NAME=tls.crt
      - SSL_KEY_NAME=tls.key
      - TZ=America/Chicago
    volumes:
      - './omada-data:/opt/tplink/EAPController/data'
      - './omada-work:/opt/tplink/EAPController/work'
      - './omada-logs:/opt/tplink/EAPController/logs'
      - './omada-cert:/cert'
    image: 'mbentley/omada-controller:5.0'
```

## Omada Controller with Docker Run

Source: [https://registry.hub.docker.com/r/mbentley/omada-controller/#!](https://registry.hub.docker.com/r/mbentley/omada-controller/#!)

```docker
docker run -d \
  --name omada-controller \
  --restart unless-stopped \
  -p 8088:8088 \
  -p 8043:8043 \
  -p 8843:8843 \
  -p 29810:29810/udp \
  -p 29811:29811 \
  -p 29812:29812 \
  -p 29813:29813 \
  -p 29814:29814 \
  -e MANAGE_HTTP_PORT=8088 \
  -e MANAGE_HTTPS_PORT=8043 \
  -e PORTAL_HTTP_PORT=8088 \
  -e PORTAL_HTTPS_PORT=8843 \
  -e SHOW_SERVER_LOGS=true \
  -e SHOW_MONGODB_LOGS=false \
  -e SSL_CERT_NAME="tls.crt" \
  -e SSL_KEY_NAME="tls.key" \
  -e TZ=America/Chicago \
  -v omada-data:/opt/tplink/EAPController/data \
  -v omada-work:/opt/tplink/EAPController/work \
  -v omada-logs:/opt/tplink/EAPController/logs \
  mbentley/omada-controller:5.0
```

## Omada Controller Automatically as LXC in Proxmox

This particular section is ***untested*** but I have used a similar `pct create` script successfully before.

SSH into your Proxmox box and run the the following. Change "2009" to whatever you want your container ID to be. Same with other unique parameters like your SSH file, gateway, IP, etc.

This will create a root file system of 8 GB, which should be enough. If you want more space, change the `rootfs` value.

Feel free to also change the template, ostype, and any other parameters you prefer.

```sh
pct create 2009 /var/lib/vz/template/cache/rockylinux-8-default_20210929_amd64.tar.xz \
  --arch amd64 \
  --ostype centos \
  --hostname omada \
  --cores 2 \
  --memory 2048 \
  --swap 0 \
  --storage local-lvm \
  --rootfs 8 \
  --password $password \
  --ssh-public-keys /root/.ssh/id_rsa.pub \
  --net0 name=eth0,bridge=vmbr0,firewall=0,gw=10.1.20.1,ip=10.1.20.9/24,type=veth \
  --unprivileged 1 \
  --onboot 1 &&\
  pct start 2009 &&\
  sleep 10 &&\
  pct exec 2009 -- bash -c\
      "dnf -y update &&\
      dnf -y install vim curl wget unzip openssh-server &&\
      systemctl start sshd &&\
      systemctl enable sshd &&\
      dnf -y install java-11-openjdk.x86_64 &&\
      wget https://repo.mongodb.org/yum/redhat/8/mongodb-org/4.4/x86_64/RPMS/mongodb-org-server-4.4.11-1.el8.x86_64.rpm &&\
      rpm -ivh mongodb-org-server-4.4.xx-1.elx.xxx.rpm &&\
      dnf -y install jsvc &&\
      wget https://static.tp-link.com/upload/software/2022/202201/20220120/Omada_SDN_Controller_v5.0.30_linux_x64.tar.gz &&\
      tar -zxvf Omada_SDN_Controller_v5.0.30_linux_x64.tar.gz &&\
      cd Omada_SDN_Controller_v5.0.30_linux_x64 &&\
      ./install.sh
```

As this script is untested, you may want to consider simply running everything up to and including `pct start 2009` and then manually logging into the new container to install Java, MongoDB, JSVC, and finally, Omada.

## Omada Controller Manually as LXC in Proxmox

### Install MongoDB

`sudo apt-get install gnupg -y`

`wget -qO - https://www.mongodb.org/static/pgp/server-3.0.asc | sudo apt-key add -`

`echo "deb [ arch=amd64 ] https://repo.mongodb.org/apt/ubuntu xenial/mongodb-org/3.0 multiverse" | sudo tee /etc/apt/sources.list.d/mongodb-org-3.0.list`

#### Fix expired keys

This update command will give an error, but run it anyway.

`sudo apt-get update`

Read the output and copy the characters after "NO_PUBKEY"

Use the following command, replacing the last group of characters with the ones you copied

`sudo apt-key adv --keyserver hkp://keyserver.ubuntu.com:80 --recv <your pubkey here without these arrow brackets>`

#### Install a specific release of MongoDB

`sudo apt-get install -y mongodb-org=3.0.15 mongodb-org-server=3.0.15 mongodb-org-shell=3.0.15 mongodb-org-mongos=3.0.15 mongodb-org-tools=3.0.15`

#### Pin the packages so they are not upgraded to a newer version

```bash
echo "mongodb-org hold" | sudo dpkg --set-selections
```
```bash
echo "mongodb-org-server hold" | sudo dpkg --set-selections
```
```bash
echo "mongodb-org-shell hold" | sudo dpkg --set-selections
```
```bash
echo "mongodb-org-mongos hold" | sudo dpkg --set-selections
```
```bash
echo "mongodb-org-tools hold" | sudo dpkg --set-selections
```

### Install Java 8

`sudo apt-get install –y openjdk-8-jre-headless`

### Install jsvc

`sudo apt-get install -y jsvc`

### Install curl

`sudo apt-get install -y curl`

### Install Omada SDN Controller

`cd ~ && wget https://static.tp-link.com/2020/202007/20200713/omada_v4.1.5_linux_x64_20200703154636.deb`

`dpkg -i omada_v4.1.5_linux_x64_20200703154636.deb`

## Login

Finally, you may now log into the controller to configure it via http.

*Do not use my IP address. Use the IP address of of your controller.*

[http://10.1.20.8:8043](http://10.1.20.8:8043)

## Helpful links

[How to install Omada SDN controller on Linux system(above Controller 4.1.5)](https://www.tp-link.com/us/support/faq/2917/)

[https://docs.mongodb.com/v3.0/tutorial/install-mongodb-on-ubuntu/](https://docs.mongodb.com/v3.0/tutorial/install-mongodb-on-ubuntu/)

[https://hevodata.com/blog/install-mongodb-on-ubuntu/](https://hevodata.com/blog/install-mongodb-on-ubuntu/)