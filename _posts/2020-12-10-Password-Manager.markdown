---
title: "Password-Manager"
layout: post
date: 2020-12-10 05:10
tag: [Python]
image: https://sergiokopplin.github.io/indigo/assets/images/jekyll-logo-light-solid.png
headerImage: false
projects: true
hidden: false # don't count this post in blog pagination
description: " "
category: project
author: coreflood
externalLink: false
---

My trust issues made me code my own password manager. nothing fancy here.

source code [here](https://github.com/AlyaGomaa/Password-Manager)

# Installation

```git clone https://github.com/AlyaGomaa/Password-Manager.git && cd Password-Manager```

```sudo apt install default-mysql-server``` install mysql

```pip install mysql-connector-python-rf``` install mysql connector

 ```sudo service mysql restart```start mysql service

the next 2 steps aren't needed if you know the password of mysql


```sudo mysql -u root -p``` login as root in mysql

```ALTER USER 'root'@'localhost' IDENTIFIED BY 'new-password';``` replace new-password with the root password you want, remember this password because it'll be used to connect to mysql from the python script

```sudo python3 password_manager.py ```  start the password manager and enter your root password
