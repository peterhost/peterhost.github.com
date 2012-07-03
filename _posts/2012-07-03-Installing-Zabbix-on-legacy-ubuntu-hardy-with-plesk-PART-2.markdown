
---
layout: post
title: Installing Zabbix 2.0.1 on Legacy Ubuntu Hardy 8.0.4 (with Plesk) PART II
categories: tutorials, plesk, ubuntu, zabbix
tags: zabbix ubuntu-hardy plesk legacy tutorial
intro: "We have built the latest version of Zabbix (2.0.1 as of this post) on a Legacy Ubuntu with Plesk <strong>on top of it all</strong>.<br /><br /><strong>Use Case:</strong> Now is the time to deploy your BUILD to the PRODUCTION server.<br/> It's mostly just a matter of following the usual `Zabbix` tutorials (like [this one](http://www.zabbix.com/wiki/howto/install/ubuntu/ubuntuinstall), create the user, group, setup some `sudoer`settings, setup an Apache `virtualhost`, and configuring Zabbix propper."
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

##1 - Make the zabbix user and group:

###Zabbix USER

(and choose a password)

    $ sudo adduser zabbix
    enter in new password
    confirm

use the remaining defaults.

###Zabbix GROUP

Then Add zabbix to the admin group:

####Create the group

By default on Ubuntu (at least 8.0.4) this group does not exist by default. Create it

    $ sudo addgroup admin

####Give it some sudoer's privileges

    $ sudo visudo

####Add `zabbix` to that group

    $ sudo adduser zabbix admin

And add the line : 

    %admin ALL=(ALL) ALL




##2 Install all dynamic dependancies for our BUILD

    $ aptitude  install libmysql++2c2a libmysql++-dev libmysql++2c2a libmysqlclient15-dev pkg-config libiksemel3 libiksemel-dev libcurl4-dev libcurl4-gnutls-dev libcurl4-openssl-dev libsnmp-dev  libssh2-1 libssh2-1-dev libgdbm-dev libopenipmi-dev libopenipmi0  unixodbc-dev iodbc
    
    
(To be Continued)
