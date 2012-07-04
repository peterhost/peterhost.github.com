
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



>the `a` in `-aG`option is NOT OPTIONNAL : it tells `usermod` to add a `secondary group`to the user. Without it, you plainly override the user's default group (which is admin, here)

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
    
the Lines to check/modify are (fill in your own values !!)

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

###Where :

All of your service files are here :

    misc
    |
    `-- init.d
        |
        `-- debian
            |-- zabbix-agent
            `-- zabbix-server


Open them and you'll see that they the correct paths to our different zabbix executables in `/usr/local/sbin/` as zabbix has been built with the default prefix

###Copy them the /etc/init.d

Copy both scripts to `/etc/init.d`

    $ sudo misc/init.d/debian/zabbix-server /etc/init.d
    $ sudo cp misc/init.d/debian/zabbix-agent /etc/init.d

And verify they have the proper permissions so that `zabbix-server`and `zabbix-agent` start at boot

    $ ls -la /etc/init.d | grep zabbix
    
should yield :

    -rwxr-xr-x   1 root root   698  zabbix-agent
    -rwxr-xr-x   1 root root   700  zabbix-server

###Register the services

    $ sudo update-rc.d zabbix-server defaults
    
     Adding system startup for /etc/init.d/zabbix-server ...
       /etc/rc0.d/K20zabbix-server -> ../init.d/zabbix-server
       /etc/rc1.d/K20zabbix-server -> ../init.d/zabbix-server
       /etc/rc6.d/K20zabbix-server -> ../init.d/zabbix-server
       /etc/rc2.d/S20zabbix-server -> ../init.d/zabbix-server
       /etc/rc3.d/S20zabbix-server -> ../init.d/zabbix-server
       /etc/rc4.d/S20zabbix-server -> ../init.d/zabbix-server
       /etc/rc5.d/S20zabbix-server -> ../init.d/zabbix-server

    $ sudo update-rc.d zabbix-agent defaults
    
     Adding system startup for /etc/init.d/zabbix-agent ...
       /etc/rc0.d/K20zabbix-agent -> ../init.d/zabbix-agent
       /etc/rc1.d/K20zabbix-agent -> ../init.d/zabbix-agent
       /etc/rc6.d/K20zabbix-agent -> ../init.d/zabbix-agent
       /etc/rc2.d/S20zabbix-agent -> ../init.d/zabbix-agent
       /etc/rc3.d/S20zabbix-agent -> ../init.d/zabbix-agent
       /etc/rc4.d/S20zabbix-agent -> ../init.d/zabbix-agent
       /etc/rc5.d/S20zabbix-agent -> ../init.d/zabbix-agent

###...And launch them

Start the server :

    $ sudo /etc/init.d/zabbix-server start
    
    Starting Zabbix server daemon: zabbix_server
    
Start the agent:

    $ sudo /etc/init.d/zabbix-agent start
    
    Starting Zabbix agent daemon: zabbix_agentd

And check they are running

    $ ps -aux | grep zabbix

yields (for me) :

    zabbix    9427  0.0  0.0 128460  2692 ?        S    03:02   0:00 /usr/local/sbin/zabbix_server
    zabbix    9439  0.0  0.0 128460  1968 ?        S    03:02   0:00 /usr/local/sbin/zabbix_server
    zabbix    9441  0.0  0.0 128460  1624 ?        S    03:02   0:00 /usr/local/sbin/zabbix_server
    zabbix    9442  0.0  0.0 129364  3184 ?        S    03:02   0:00 /usr/local/sbin/zabbix_server
    zabbix    9443  0.0  0.0 129364  3184 ?        S    03:02   0:00 /usr/local/sbin/zabbix_server
    zabbix    9445  0.0  0.0 129364  3184 ?        S    03:02   0:00 /usr/local/sbin/zabbix_server
    zabbix    9446  0.0  0.0 129364  3184 ?        S    03:02   0:00 /usr/local/sbin/zabbix_server
    zabbix    9447  0.0  0.0 129364  3184 ?        S    03:02   0:00 /usr/local/sbin/zabbix_server
    zabbix    9448  0.0  0.0 129364  3184 ?        S    03:02   0:00 /usr/local/sbin/zabbix_server
    zabbix    9449  0.0  0.0 128528  1636 ?        S    03:02   0:00 /usr/local/sbin/zabbix_server
    zabbix    9450  0.0  0.0 128528  1684 ?        S    03:02   0:00 /usr/local/sbin/zabbix_server
    zabbix    9451  0.0  0.0 128548  1976 ?        S    03:02   0:00 /usr/local/sbin/zabbix_server
    zabbix    9452  0.0  0.0 128528  1684 ?        S    03:02   0:00 /usr/local/sbin/zabbix_server
    zabbix    9453  0.0  0.0 128528  1636 ?        S    03:02   0:00 /usr/local/sbin/zabbix_server
    zabbix    9454  0.0  0.0 128852  1660 ?        S    03:02   0:00 /usr/local/sbin/zabbix_server
    zabbix    9455  0.0  0.0 128460  1624 ?        S    03:02   0:00 /usr/local/sbin/zabbix_server
    zabbix    9457  0.0  0.0 128460  1704 ?        S    03:02   0:00 /usr/local/sbin/zabbix_server
    zabbix    9458  0.0  0.0 128460  1648 ?        S    03:02   0:00 /usr/local/sbin/zabbix_server
    zabbix    9460  0.0  0.0 128468  1656 ?        S    03:02   0:00 /usr/local/sbin/zabbix_server
    zabbix    9462  0.0  0.0 129264  3188 ?        S    03:02   0:00 /usr/local/sbin/zabbix_server
    zabbix    9463  0.0  0.0 128520  1644 ?        S    03:02   0:00 /usr/local/sbin/zabbix_server
    zabbix    9465  0.0  0.0 128520  1644 ?        S    03:02   0:00 /usr/local/sbin/zabbix_server
    zabbix    9468  0.0  0.0 128520  1644 ?        S    03:02   0:00 /usr/local/sbin/zabbix_server
    zabbix    9470  0.0  0.0 128520  1644 ?        S    03:02   0:00 /usr/local/sbin/zabbix_server
    zabbix    9472  0.0  0.0 128460  1652 ?        S    03:02   0:00 /usr/local/sbin/zabbix_server
    zabbix    9474  0.0  0.0 128464  1652 ?        S    03:02   0:00 /usr/local/sbin/zabbix_server
    zabbix    9475  0.0  0.0 128460  1452 ?        S    03:02   0:00 /usr/local/sbin/zabbix_server
    zabbix   28672  0.0  0.0  65444  1136 ?        S    01:29   0:00 /usr/local/sbin/zabbix_agentd
    zabbix   28687  0.0  0.0  65444  1244 ?        S    01:29   0:00 /usr/local/sbin/zabbix_agentd
    zabbix   28689  0.0  0.0  65448   872 ?        S    01:29   0:00 /usr/local/sbin/zabbix_agentd
    zabbix   28690  0.0  0.0  65448   872 ?        S    01:29   0:00 /usr/local/sbin/zabbix_agentd
    zabbix   28692  0.0  0.0  65448   872 ?        S    01:29   0:00 /usr/local/sbin/zabbix_agentd
    zabbix   28693  0.0  0.0  65468  1112 ?        S    01:29   0:00 /usr/local/sbin/zabbix_agentd

#5 Setup the PHP frontend

I'll be very breif : read the zabbix docs, it's plain and obvious. 

If you used `Plesk`to create the `zabbix` account and domain, then all is setup already. go to the URL of your website, and do the online configuration

#6 F#*!?ng PLESK Gotchas

##Date Error

if ever you get an error looking like :

    	date() [<a href='function.date'>function.date</a>]: It is not safe to rely on the system's
    	timezone settings. Please use the date.timezone setting, the TZ environment variable or the
    	date_default_timezone_set() function. In case you used any of those methods and you are still
    	getting this warning, you most likely misspelled the timezone identifier. We selected 
    	'Europe/Berlin' for 'CEST/2.0/DST' instead [include/page_header.php:184]

It's that your timezone settings in `Plesk`are incorrect. Even if they are, `Plesk`is not known for it's utter reliability in that domain.

Just go edit the `php.ini`(back it up first, and beware that Plesk upgrades will overwrite it, so you most likely will have to do that at each update)

    $ vim /etc/php5/apache2/php.ini
    
Look for the section :

     ;;;;;;;;;;;;;;;;;;;
     ; Module Settings ;
     ;;;;;;;;;;;;;;;;;;;
     
     [Date]
     ; Defines the default timezone used by the date functions
     ;date.timezone =

And add you bloody timezone, already :)

     date.timezone = Europe/Paris


##Not Enough TIME

While you're at it, `zabbix2.0.1` requires that `PHP option max_execution_time` and `PHP option max_input_time` be set to `300` and not the `Plesk`default of `180`

Do that in your Plesk interface. You should be good to go.

## Can't Create `configuration`file

That's because the `conf` subfolder in your zabbix webroot should be writeable by 'www-data' (default is `psacln`)

    $ chown -R www-data conf

## I bloody can't login !

default user/pass as per the zabbix manual is :

  user : admin
  password : zabbix
  
That you will of course change as soon as you first login


