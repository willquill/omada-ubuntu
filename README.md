# How to use this git repo

It's just a README file. Follow the instructions to set up your Omada controller.

# Pre-requisites

- Start with Ubuntu 16.04

# Install MongoDB

Follow the steps below. If you need help, see [Helpful Links](#helpful-links).

## First steps

```bash
sudo apt-get install gnupg -y
```
```bash
wget -qO - https://www.mongodb.org/static/pgp/server-3.0.asc | sudo apt-key add -
```
```bash
echo "deb [ arch=amd64 ] https://repo.mongodb.org/apt/ubuntu xenial/mongodb-org/3.0 multiverse" | sudo tee /etc/apt/
```
```bash
sources.list.d/mongodb-org-3.0.list
```

## Fix expired keys

This update command will give an error, but run it anyway.

```bash
sudo apt-get update
```

Read the output and copy the characters after "NO_PUBKEY"

Use the following command, replacing the last group of characters with the ones you copied

```bash
sudo apt-key adv --keyserver hkp://keyserver.ubuntu.com:80 --recv <your pubkey here without these arrow brackets>
```

## Install a specific release

```bash
sudo apt-get install -y mongodb-org=3.0.15 mongodb-org-server=3.0.15 mongodb-org-shell=3.0.15 mongodb-org-mongos=3.
0.15 mongodb-org-tools=3.0.15
```

## Pin the packages so they are not upgraded to a newer version:

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

# Install Java 8

```bash
sudo apt-get install –y openjdk-8-jre-headless
```

# Install jsvc

```bash
sudo apt-get install -y jsvc
```

# Install curl

```bash
sudo apt-get install -y curl
```

# Install Omada SDN Controller

```bash
cd ~ | wget https://static.tp-link.com/2020/202007/20200713/omada_v4.1.5_linux_x64_20200703154636.deb
```
```bash
dpkg -i omada_v4.1.5_linux_x64_20200703154636.deb
```

# Login

Finally, you may now log into the controller to configure it via http.

*Do not use my IP address. Use the IP address of of your controller.*

[http://10.1.20.8:8043](http://10.1.20.8:8043)

# Helpful links

[How to install Omada SDN controller on Linux system(above Controller 4.1.5)](https://www.tp-link.com/us/support/faq/2917/)

[https://docs.mongodb.com/v3.0/tutorial/install-mongodb-on-ubuntu/](https://docs.mongodb.com/v3.0/tutorial/install-mongodb-on-ubuntu/)

[https://hevodata.com/blog/install-mongodb-on-ubuntu/](https://hevodata.com/blog/install-mongodb-on-ubuntu/)