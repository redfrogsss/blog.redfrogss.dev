---
layout: post
title:  "How I setup my raspberry pi for  home server application?"
date:   2022-06-14 21:10:00 +0800
# modified: 2020-02-02 16:49:47 +07:00s
tags: [blog, home-server]
# categories: jekyll update
# description: Ada dua cara untuk memperbarui forked repository menggunakan web interface yang disediakan oleh github tapi ribet, atau melalui terminal yang lebih ribet lagi.
---

<!-- # How I setup my raspberry pi for  home server application? -->

## Hardware and Operating System
To start with, let’s talk about the most basic thing.

For hardware, I choose the Raspberry Pi 4. It is a single-board computer that is powerful enough for all the application I need. I have a 8GB RAM version and a 4GB RAM version that I bought several years ago. The 8GB RAM one usually do all the important stuff and the 4GB RAM one usually serve as a backup and testing machine.

For operating systems, I usually just install Ubuntu Server on a SD card. It has 64 bits support on Raspberry Pis and it is quite easy to use. Also, it just works.

## OS Update
The first thing I do after installing Ubuntu into my raspberry pi every time is to perform a system update. It is as easy as two lines of commands.

```bash
sudo reboot now
sudo apt-get update && sudo apt-get upgrade -y && sudo reboot now
```

You might ask: why do a reboot first?
The answer is simple. The operating system usually start the update automatically in the background. And because it is in the background, there is no indicators on when will it finish its jobs. I don’t want to wait forever for this so I just restart the system and stop the auto update.

The second line of commands will update the system and perform a restart after the update is done. The best part of this is the output of the commands. It just keeps popping the progress messages on your screen and make you feel like you are in the typical hacking scenario.

## Setting up username and hostname
By default, the username and the hostname is `ubuntu@ubuntu`. Usually it is okay to leave it like this. However, I have two raspberry pi here running the same ubuntu. It is just confusing for me to know which one is which. Therefore I usually run this command to change the hostname.

```bash
sudo hostnamectl set-hostname rpi4-1.home
```

For the username, I prefer using my nickname `redfrogss` instead of `ubuntu`. A bonus point of doing this is that since my username in my macOS is also `redfrogss`, I could just enter the address without username when doing `ssh`.

Here is how I change the username from `ubuntu` to `redfrogss`.

```bash
sudo adduser temp
sudo adduser temp sudo
exit
ssh temp@192.168.xxx.xxx
sudo usermod -l redfrogss ubuntu
sudo usermod -d /home/redfrogss -m redfrogss
exit
ssh redfrogss@192.168.xxx.xxx
sudo deluser temp
sudo rm -r /home/temp
exit
```

The concept of these commands is to create a temporary account with sudo permission to change the username and the home directory.

## Setting up password-less login
Login with password is good enough for securing the account but it is annoying. Everything I `ssh` into the server I need to type in my password again. Therefore, I usually setup password-less login with these two commands in my laptop.

```bash
ssh-keygen -t rsa
ssh-copy-id redfrogss@192.168.xxx.xxx
```

The first command generate a key for ssh.
The second command copy the key into the server.

## Install docker and docker-compose
In my home server, I usually deploy most of my useful applications (pihole, nginx proxy manager, etc.) with docker and docker-compose. Therefore, docker is a must.

Here is the command of installing docker and docker-compose.

```bash
sudo apt install docker.io docker-compose -y
sudo usermod -aG docker $USER
```

The first command is to install docker and docker-compose.
The second command is to setup permission for docker so I do not need to type `sudo` while using docker.

## Install Portainer
Portainer is a container manager that also run on docker as a container. It provides a clear interface for showing and managing all running containers. 

Here is the command of installing Portainer:
```bash
docker volume create portainer_data 

docker run -d -p 8000:8000 -p 9443:9443 --name portainer \ 
--restart=always \ 
-v /var/run/docker.sock:/var/run/docker.sock \ 
-v portainer_data:/data \ 
portainer/portainer-ce:latest
```

## Free up port 53
Port 53 is a port that do DNS stuff. By default, `systemd-resolved` occupied the port in Ubuntu. 

However, I usually self-hosted DNS server myself using pihole, adguard home and so on. These DNS server also need port 53 as well. Therefore, I need to free up the port by editing `/etc/systemd/resolved.conf`.

```
[Resolve]
DNS=1.1.1.1
#FallbackDNS=
#Domains=
#LLMNR=no
#MulticastDNS=no
#DNSSEC=no
#DNSOverTLS=no
#Cache=no
DNSStubListener=no
#ReadEtcHosts=yes
```

Then, run the following commands.

```bash
sudo ln -sf /run/systemd/resolve/resolv.conf /etc/resolv.conf sudo reboot now
```

Also, don’t forget to edit the `/etc/hosts` as well.

```
127.0.0.1 rpi4-1
```

## What’s next?
With Ubuntu and docker installed, the raspberry pi is ready for deploying any server application. 

Here are some applications I usually install:
- pihole
- nginx proxy manager
- emby
- nextcloud
- uptime-kuma
- etc. 

## Learn more
Here are the source for some of my steps:
- [How do I change my username?](https://askubuntu.com/questions/34074/how-do-i-change-my-username) 
- [Ubuntu: How To Free Up Port 53, Used By systemd-resolved - Linux Uprising Blog](https://www.linuxuprising.com/2020/07/ubuntu-how-to-free-up-port-53-used-by.html)