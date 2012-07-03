
---
layout: post
title: Installing Zabbix 2.0.1 on Legacy Ubuntu Hardy 8.0.4 (with Plesk) PART II
categories: tutorials, plesk, ubuntu, zabbix
tags: zabbix ubuntu-hardy plesk legacy tutorial
intro: "We have built the latest version of Zabbix (2.0.1 as of this post) on a Legacy Ubuntu with Plesk <strong>on top of it all</strong>.<br /><br /><strong>Use Case:</strong> Now is the time to deploy your BUILD to the PRODUCTION server.<br/> It's mostly just a matter of following the usual `Zabbix` tutorials (like <a href='http://www.zabbix.com/wiki/howto/install/ubuntu/ubuntuinstall'>this one</a>), create the user, group, setup some `sudoer`settings, setup an Apache `virtualhost`, and configuring Zabbix propper."
intro-img: "2012-06-30-zabbix-ubuntu-hardy-plesk-VICTORY.jpg"

---

#Migrating your build to the PRODUCTION server

If you followed the steps of the last tutorial [PLESK - ZABBIX2 - UBUNTU HARDY - INSTALL - Part I Packaging](), you now have at your disposal :

* a debian packet containing all `zabbix` executables, manfiles, and config files
* a directory containing `zabbix's PHP frontend interface`
* a directory containing the `default Zabbix services`

#Use Plesk, or not
As I've been plagued with Plesk (OS's restrictions) for years, I'll use the damn beast to at least have it backup my Zabbix related stuff : `php frontend`, `mysql database`. 

This part is way beyond this tutorial, if you're a Plesk user, do as usual : create an account, then register de domain.tld in Plesk, do whatever you need according to whose registar manages your domain, make plesk use the `/var/www/vhosts/zabbix.mydomain.com/` directory (or whatever, here I tell plesk to configure directly a subdomain, not a full domain), then create a `mysql` database for this same `user`, call it `zabbix` (or whatever you fancy), add a database user (why not call it `zabbix` too as we're at it).

You'll have your `zabbix database` and `httpdocs folder` cleanly registered in plesk, and it will be backuped, migrated,updated (big aaaaargh)... like any other Plesk account.

What follows, Plesk will never know about (and never will have to) and you'll have to repeat those steps for each new install of Zabbix on a legacy Ubuntu server with Plesk on it, pray the gods there won't be many.

#Create the Zabbix User

All that follows is basic `zabbix`stuff, you could as well read the `zabbix`user guide (in fact you should if you haven't because it's very complete ), but we're detail it, especially what changes from the classical use case where you can just get away with `aptitude install zabbix-server zabbix-agent` (or `zabbix-proxy`)

#1 - Make the zabbix user and group:

##Zabbix USER

(and choose a password)

    $ sudo adduser zabbix
    enter in new password
    confirm

use the remaining defaults.

##Zabbix GROUP

Then Add zabbix to the admin group:

###Create the group

By default on Ubuntu (at least 8.0.4) this group does not exist by default. Create it

    $ sudo addgroup admin

###Give it some sudoer's privileges

    $ sudo visudo

###Add `zabbix` to that group

    $ sudo adduser zabbix admin

And add the line : 

    %admin ALL=(ALL) ALL




#2 Install all dynamic dependancies for our BUILD
(see Part I of this tutorial)

##Install the `ppa` for the `ssh2` libs (cf part I of this tutorial)

##Install the dynamic dependancies of our build

    $ aptitude -s install libmysql++2c2a libmysql++-dev libmysql++2c2a libmysqlclient15-dev pkg-config libiksemel3 libiksemel-dev libcurl4-dev libcurl4-gnutls-dev libcurl4-openssl-dev libsnmp-dev  libssh2-1 libssh2-1-dev libgdbm-dev libopenipmi-dev libopenipmi0  unixodbc-dev iodbc
    
Doesn't touch any `plesk` package ?

* `psa-` prefixed packages ?
* `sw-`prefixed packages ?

Does not require/upgrade installation of any additional plesk-related package ?

Good.

    $ aptitude install libmysql++2c2a libmysql++-dev libmysql++2c2a libmysqlclient15-dev pkg-config libiksemel3 libiksemel-dev libcurl4-dev libcurl4-gnutls-dev libcurl4-openssl-dev libsnmp-dev  libssh2-1 libssh2-1-dev libgdbm-dev libopenipmi-dev libopenipmi0  unixodbc-dev iodbc
    
##Install the pre-built package

    $ dpkg --install  zabbix_2.0.1-1_amd64.deb
    Selecting previously deselected package zabbix.
    (Reading database ... 212230 files and directories currently installed.)
    Unpacking zabbix (from zabbix_2.0.1-1_amd64.deb) ...
    Setting up zabbix (2.0.1-1) ...

`Zabbix`executable, libs, man pages and configurations are now installed under `/usr/local`(go check !)

#3 PRE-POPULATE the Database

As we compiled Zabbix to use Mysql as a frontend, we'll use the `mysql`templates found in 

    database/
        |-- mysql
            |-- data.sql
            |-- images.sql
            `-- schema.sql
 
##Create the Database

    $ sudo mysql -e"create database zabbix;"
    $ sudo mysql -e"grant all privileges on zabbix.* to zabbix@localhost identified by 'enter-password-here';"

##Fill her (in that order)

    $ mysql -D zabbix -u zabbix -p < some/path/zabbix-2.0.1/database/mysql/schema.sql
    $ mysql -D zabbix -u zabbix -p < some/path/zabbix-2.0.1/database/mysql/images.sql
    $ mysql -D zabbix -u zabbix -p < some/path/zabbix-2.0.1/database/mysql/data.sql

#4 Install the Services

##Tell the system about them

    $ sudo vim /etc/services

At the end of the file, add those two lines

    zabbix_agent 10050/tcp # Zabbix ports
    zabbix_trap 10051/tcp

Save and Exit

##Configuration

###Configuration Files

All our config files are stored in `/usr/local/etc/`

    /usr/local/etc/
                |-- zabbix_agent.conf
                |-- zabbix_agentd.conf
                |-- zabbix_proxy.conf
                `-- zabbix_server.conf

###Configure the Zabbix Agent


    $ sudo vi /usr/local/etc/zabbix_agent.conf

Just check that the agent looks for the server at 127.0.0.1    

    Server=127.0.0.1

Setup any other preferences to your liknig

###Configure the Zabbix Server

    $ sudo vim /usr/local/etc/zabbix_server.conf
    
the Lines to check/modify are 

    DBName=zabbix

then

    DBUser=zabbix

then

    DBPassword=yourpassword

Save and exit (or add more resources to the server, it's a good time to do so if you wish to monitor more than 10 hosts)

In my case, the files were generated `-rw-r--r--` : no way.

    # should be -rw-r-----
    $ chmod 640 zabbix_server.cong

##Services

All of your service files are here :

    misc
    |
    `-- init.d
        |
        `-- debian
            |-- zabbix-agent
            `-- zabbix-server


Open them and you'll see that they the correct paths to our different zabbix executables in `/usr/local/bin/` as zabbix has been built with the default prefix








(To be Continued)
