---
layout: post
title: Installing Zabbix 2.0.1 on Legacy Ubuntu Hardy 8.0.4 (with Plesk)
categories: tutorials, plesk, ubuntu, zabbix
tags: zabbix, ubuntu hardy, plesk, legacy, tutorial
intro: "Installing latest version of Zabbix on Legacy Ubuntu with Plesk <strong>on top of it all</strong> is, yes you've guessed it, a real pain in the ass.<br /><br /><strong>Use Case:</strong> You boss wants you to do it ! You have no choice, you need to use Awesome Zabbix on that old hosted Plesk Server. IT's in production, hosts a bazillion emails accounts and you're so excited at the prospect of accidentally removing one of Plesk's dependancies. You can go with Zabbix 1.6.5-2,... but hell no ! We won't settle for less than Zabbix 2.0, no way !"
intro-img: "2012-06-30-zabbix-ubuntu-hardy-plesk.jpg"

---


#Purpose

* You need a recent Zabbix server (>=2.0) on that old Ubuntu Hardy with Plesk hosted machine you have not much control over (you boss has it, hosts mail accounts on it, and its 4 CPUs are waisted, so you wish to use that machine)

#Forewarning

1. Messing with a platform where PLESK is installed in `PRODUCTION` is **BAD**. You should do your tests on a Plesk install similar to your `PRODUCTION` environments (Ubuntu version, Plesk version)
2. DO NOT EVER accept `aptitude install somepackage` where the resulting output of `aptitude` includes the modifying (remove/upgrade/...) of any package that belong to `PLESK`, ie :
    * `psa-` prefixed packages
    * `sw-`prefixed packages
    *  ...
3. As a general rule : Always do :
    * `$ aptitude -s install` to simulate before you really install
    * check the output
    * only `new` packages should be installed. Don't accept anything that modify's the Plesk environment
    * otherwise, get the F\*\*\*\* away from that feature : you will miss it less than you think


**YOU HAVE BEEN WARNED !!**

#Before you start


## Getting things done

* Do all the `aptitude` package installing under `root` account, or `sudo` if you prefer
* then `su` as another user with limited rights for the `download-sources/configure/build`
  process, this will (mostly) keep you safe of errors such as issuing `make install`
  and  having your fucked-up-compiled-zabbix-stuff-because-i-played-with-it-a-bit
  installed **for real**. Make sure you use a plain user account who can't write
  to system folders by default. Your user account should be fine.
* then once `zabbix` is built, package it under the root account (more at the end of this tutorial)
* then only, deploy that `packed build` on a production server, making uninstalling it painless and safer.
* Always use aptitude's "simulate" option first

Do:

    $ aptitude  -s  install build-essential checkinstall

    (...)

Then :

    $ aptitude  install build-essential checkinstall

##Build tools


We want to be able to build a package at the end of the process for one reason :

Packages are easy to install/remove : usually `make uninstall` will clean up a borked install, but not always totally realiable. Reason why you **should always copy the output of a `make install`command** for later reference if you have to clean up a mess by hand.


##Zabbix CONFIGURE script

###Configure options


We're not here yet, but for the record, I want my Zabbix built the following way :


> Which implies the following `configure` options passed as arguments 

    $ ./configure --with-mysql --with-jabber --with-libcurl --enable-server --enable-agent\
     --enable-proxy --with-ldap --enable-ipv6 --with-net-snmp  --with-openipmi --with-ssh2 --with-iodbc 

Don't hesitate to remove the options you don't need, and then in the **Installing Zabbix's BUILD Dependencies** section, skip on the packages required by the options you don't need

note : `--enable-java` is not recommended, because Plesk uses it's own `java`, and even if the configure passes, you will get errors at compile time with the default on Ubuntu Hardy LTS + Plesk (might be different on a more recent Ubuntu release)


####The configure script should end with a clean :


    Configuration:

      Detected OS:           linux-gnu
      Install path:          /usr/local
      Compilation arch:      linux

      Compiler:              gcc
      Compiler flags:        -g -O2  -I/usr/include/mysql -DBIG_JOINS=1 -fPIC      -I/usr/include   -I/usr/local/include -I/usr/lib/perl/5.8/CORE -I. -I/usr/include  -I/usr/include -I/usr/include -I/usr/include 

      Enable server:         yes
      Server details:
        With database:         MySQL
        WEB Monitoring via:    cURL
        Native Jabber:         yes
        SNMP:                  net-snmp
        IPMI:                  openipmi
        SSH:                   yes
        Linker flags:          -rdynamic      -L/usr/lib/mysql      -L/usr/lib   -L/usr/lib  -L/usr/lib -L/usr/lib -L/usr/lib -L/usr/lib
        Libraries:             -lm -lrt  -lresolv    -lmysqlclient     -liksemel  -lcurl -lgssapi_krb5 -liodbc   -lnetsnmp -lcrypto  -lnetsnmp -lcrypto -lssh2 -lOpenIPMI -lOpenIPMIposix -lldap -llber 

      Enable proxy:          yes
      Proxy details:
        With database:         MySQL
        SNMP:                  net-snmp
        IPMI:                  openipmi
        SSH:                   yes
        Linker flags:          -rdynamic      -L/usr/lib/mysql     -L/usr/lib   -L/usr/lib  -L/usr/lib -L/usr/lib -L/usr/lib -L/usr/lib
        Libraries:             -lm -lrt  -lresolv    -lmysqlclient      -lcurl -lgssapi_krb5 -liodbc   -lnetsnmp -lcrypto  -lnetsnmp -lcrypto -lssh2 -lOpenIPMI -lOpenIPMIposix -lldap -llber 

      Enable agent:          yes
      Agent details:
        Linker flags:          -rdynamic     -L/usr/lib
        Libraries:             -lm -lrt  -lresolv    -lcurl -lgssapi_krb5 -lldap -llber 

      Enable Java gateway:   no

      LDAP support:          yes
      IPv6 support:          yes

    ***********************************************************
    *            Now run 'make install'                       *
    *                                                         *
    *            Thank you for using Zabbix!                  *
    *              <http://www.zabbix.com>                    *
    ***********************************************************

####Configure options (for information):

    $ ./configure --help

The output is copied at the bottom of this tutorial, for reference


###Database Backend :

Here, you'll need to choose one of these (and only one) depending on your prefered backend

    --with-ibm-db2=[ARG]    use IBM DB2 CLI from given sqllib directory
                            (ARG=path); use /home/db2inst1/sqllib (ARG=yes);
                            disable IBM DB2 support (ARG=no)
    --with-ibm-db2-include=[DIR]
                            use IBM DB2 CLI headers from given path
    --with-ibm-db2-lib=[DIR]
                            use IBM DB2 CLI libraries from given path
    --with-mysql[=ARG]      use MySQL client library [default=no], optionally
                            specify path to mysql_config
    --with-oracle=[ARG]     use Oracle OCI API from given Oracle home
                            (ARG=path); use existing ORACLE_HOME (ARG=yes);
                            disable Oracle OCI support (ARG=no)
    --with-oracle-include=[DIR]
                            use Oracle OCI API headers from given path
    --with-oracle-lib=[DIR] use Oracle OCI API libraries from given path
    --with-postgresql[=ARG] use PostgreSQL library [default=no], optionally
                            specify path to pg_config
    --with-sqlite3[=ARG]    use SQLite 3 library [default=no], optionally
                            specify the prefix for sqlite3 library

I prefer to build `zabbix` with `MySQL` support, because `Mysql` support is the default whatever PLESK licence you have. For `postgre`, you need additional licensing from Parallell. If you wish to choose a different database backend, you'll have to replace the preceding `configure` options and in the following instructions find and install your `DB-backend` build libraries yourself.

It's most of the time just a matter of 

    # for Postgresql, for ex :
    $ aptitude search postgre | grep dev

    # Which yields : (line preceded by >> is usually what you want) (added by myself)

      p purged           <none>     hardy    4.6.99+svn   gforge-db-postgresql           none       universe collaborative development tool - data
      p purged           <none>     hardy    4.6.99+svn   gforge-shell-postgresql        none       universe collaborative development tool - shel
      p purged           <none>     hardy    1.1.4.1.0    libghc6-hdbc-postgresql-dev    none       universe PostgreSQL HDBC (Haskell Database Con
      p purged           <none>     hardy    1.7-1        libghc6-hsql-postgresql-dev    none       universe PostgreSQL driver of the HSQL library
      p purged           <none>     hardy-up 1.6.1-2ubu   libgnadepostgresql-dev         none       universe GNat Ada Database Environment - Postg
      p purged           <none>     hardy    1.7.0-2bui   libpostgresql-ocaml-dev        none       universe OCaml bindings to PostgreSQL's libpq 
      p purged           <none>     hardy    8.2.7-1      postgresql-server-dev-8.2      none       universe development files for PostgreSQL 8.2 
    >>p purged           <none>     hardy-up 8.3.19-0ub   postgresql-server-dev-8.3      none       libdevel development files for PostgreSQL 8.3     

of for a `sometool`'s lib :

    $ aptitude search sometool | grep lib | grep dev


#Installing Zabbix's BUILD Dependencies

##Mysql (--with-mysql)

    $ aptitude -s install libmysql++2c2a libmysql++-dev libmysql++2c2a libmysqlclient15-dev 

    (...)

    $  $ aptitude install libmysql++2c2a libmysql++-dev libmysql++2c2a libmysqlclient15-dev 

##Jabber (--with-jabber)


    $ aptitude install pkg-config

    $ aptitude install libiksemel3 libiksemel-dev

##CURL (--with-libcurl)

    $ aptitude install libcurl4-dev libcurl4-gnutls-dev libcurl4-openssl-dev 

OUtputs: 

        Note: selecting "libcurl4-gnutls-dev" instead of the
          virtual package "libcurl4-dev"
    The following packages have been automatically kept back:
    (...)
    The following NEW packages will be automatically installed:
      comerr-dev libidn11-dev libkadm55 libkrb5-dev libldap2-dev libssl-dev 
    The following packages have been kept back:
    (...)
    The following NEW packages will be installed:

      comerr-dev libcurl4-openssl-dev libidn11-dev libkadm55 libkrb5-dev libldap2-dev libssl-dev 

    The following packages will be upgraded:

      libcurl3 libcurl3-gnutls libgnutls13 libssl0.9.8 libtasn1-3 

    5 packages upgraded, 7 newly installed, 0 to remove and 74 not upgraded.


##SNMP (--with-net-snmp)

    $ aptitude install libsnmp-dev
    The following NEW packages will be installed:

      libsensors-dev libsensors3 libsnmp-base libsnmp-dev libsnmp-perl libsnmp15 libwrap0-dev 

    0 packages upgraded, 7 newly installed, 0 to remove and 74 not upgraded.

##SSH2 (--with-ssh2)
> Plain :

    $ aptitude install libssh2-1 libssh2-1-dev

Won't do, Zabbix `configure` script will complain :

    checking for SSH2 support... yes
    configure: error: SSH2 library version requirement not met (>= 1.0.0)

So we'll make an exception to out `non intefering with the default OS policy` and add a `prebuilt` package from a `PPA`

###Add this PPA to sources.list

[PPA is here](https://launchpad.net/~pgquiles/+archive/ppa)

    $ vim /etc/apt/sources.list
    # plhoste 2012-06-30 16:04:30
    # PPA for Libssl2 > 1.0.0 for Ubuntu Hardy
    deb http://ppa.launchpad.net/pgquiles/ppa/ubuntu hardy main 
    deb-src http://ppa.launchpad.net/pgquiles/ppa/ubuntu hardy main 

###Authorize the PPA

    $ apt-key adv --keyserver keyserver.ubuntu.com --recv-keys 42813EC6
    Executing: gpg --ignore-time-conflict --no-options --no-default-keyring --secret-keyring /etc/apt/secring.gpg --trustdb-name /etc/apt/trustdb.gpg --keyring /etc/apt/trusted.gpg --keyserver keyserver.ubuntu.com --recv-keys 42813EC6
    gpg: requesting key 42813EC6 from hkp server keyserver.ubuntu.com
    gpg: key 42813EC6: public key "Launchpad PPA for Pau Garcia i Quiles" imported
    gpg: Total number processed: 1
    gpg:               imported: 1  (RSA: 1)

###Update Aptitude

    $ aptitude update

> Now the right libs are available :

    $ aptitude search libssh2

    c config-file      <none>     hardy    1.2-1~hard   libssh2-1                      none       libs     SSH2 client-side library             
    p purged           <none>     hardy    1.2-1~hard   libssh2-1-dbg                  none       debug    SSH2 client-side library (debug packa
    p purged           <none>     hardy    1.2-1~hard   libssh2-1-dev                  none       libdevel SSH2 client-side library (development

###Install them

    $  aptitude install libssh2-1 libssh2-1-dev

##Open IPMI (--with-openipmi)

    $ aptitude install libgdbm-dev libopenipmi-dev libopenipmi0

##UnixODBC (--with-unixodbc )

    $ aptitude install autotools-dev libaudio2 libdrm2 libgl1-mesa-glx libglu1-mesa liblcms1 libltdl7 libltdl7-dev libmng1 libodbcinstq1c2 libqt3-mt libtool libxmu6 libxt6 libxxf86vm1 unixodbc-dev

    $ aptitude install iodbc 

#BUILDING

##Quit the ROOT user account

>This is fucking important.

>   SHORT ANSWER : if you compile as root , then all of the program is using root. if your program's hacked and the hacker can somehow manage to execute system commands from it, then he can do so via a program which might have `root` access to some resources. Most of the time, it's not a problem. I guess the Zabbix team watches their configure scripts very carefully. But you should never trust a third party when it comes to your own system's security, especially when it's not yours but your boss's.

Best Practice is to always :

    $ su myusername
    $ ./configure && make
    $ su root
    # make install

##Download the source

Go into some directory your user owns

    $ su myUserName
    $ cd ~/BUILD
    $ wget http://downloads.sourceforge.net/project/zabbix/ZABBIX%20Latest%20Stable/2.0.1/zabbix-2.0.1.tar.gz
    $ tar -xzvf zabbix-2.0.1.tar.gz
    $ cd zabbix-2.0.1

If you already played with the source DIR a little bit, issued some `make`command, you **should reset it all to defaults** prior to building the real thing with : 

    $ make distclean

>DON'T EVER do that if you already ran `make install` as  `make uninstall`'s infos will be erased too, and you'll have such a nice time figuring out what files to remove before your system is clean again.

##Configure

    $ ./configure  --with-mysql --with-jabber --with-libcurl --with-iodbc   --enable-server --enable-agent\
     --enable-proxy --with-ldap --enable-ipv6 --with-net-snmp  --with-openipmi --with-ssh2

Should yield a clean output with no errors. If not, re-read the previous instructions carefully, find the missing `dependancies` (usually of the form `libdevxxxx` where `xxx` is the missing `lib` as the configure script tells you when it exists with an error)

##MAKE

    $ make

If you get errors, do it all again, recheck every step, act like a grownup :)

It should en with something like :

    make[3]: Leaving directory `somepath/zabbix-2.0.1/src/zabbix_proxy'
    make[2]: Leaving directory `somepath/zabbix-2.0.1/src/zabbix_proxy'
    make[2]: Entering directory `somepath/zabbix-2.0.1/src'
    make[2]: Nothing to be done for `all-am'.
    make[2]: Leaving directory `somepath/zabbix-2.0.1/src'
    make[1]: Leaving directory `somepath/zabbix-2.0.1/src'
    Making all in database
    make[1]: Entering directory `somepath/zabbix-2.0.1/database'
    make[1]: Nothing to be done for `all'.
    make[1]: Leaving directory `somepath/zabbix-2.0.1/database'
    Making all in man
    make[1]: Entering directory `somepath/zabbix-2.0.1/man'
    make[1]: Nothing to be done for `all'.
    make[1]: Leaving directory `somepath/zabbix-2.0.1/man'
    Making all in misc
    make[1]: Entering directory `somepath/zabbix-2.0.1/misc'
    make[1]: Nothing to be done for `all'.
    make[1]: Leaving directory `somepath/zabbix-2.0.1/misc'
    Making all in upgrades
    make[1]: Entering directory `somepath/zabbix-2.0.1/upgrades'
    make[1]: Nothing to be done for `all'.
    make[1]: Leaving directory `somepath/zabbix-2.0.1/upgrades'
    make[1]: Entering directory `somepath/zabbix-2.0.1'
    make[1]: Nothing to be done for `all-am'.
    make[1]: Leaving directory `somepath/zabbix-2.0.1'


#Now for the Packaging


##Build the packages

    $ sudo checkinstall -D --install=no

> the `--install=no` switch tells `checkinstall` to just build the packages and **not install anything**
> the `-D` switch tells checkinstall to build `.deb` packages

Answer the questions, and when you think you have documented the packages enough, hit enter. My output was :

    Enter a number to change any of them or press ENTER to continue: 10

    This package will be built according to these values: 

    0 -  Maintainer: [ myemailaddress ]
    1 -  Summary: [ Zabbix 2.0.1 Server & Client for Ubuntu 8.04 Hardy (Plesk compatible) ]
    2 -  Name:    [ zabbix ]
    3 -  Version: [ 2.0.1 ]
    4 -  Release: [ 1 ]
    5 -  License: [ GPL ]
    6 -  Group:   [ checkinstall ]
    7 -  Architecture: [ amd64 ]
    8 -  Source location: [ zabbix-2.0.1 ]
    9 -  Alternate source location: [ http://downloads.sourceforge.net/project/zabbix/ZABBIX%20Latest%20Stable/2.0.1/zabbix-2.0.1.tar.gz ]
    10 - Requires: [  ]

It all should end with some happy message like :

    ======================== Installation successful ==========================

    Copying documentation directory...
    ./
    ./AUTHORS
    ./COPYING
    ./README
    ./NEWS
    ./INSTALL
    ./ChangeLog
    grep: /var/tmp/ENFNheLRenfiWGRETaMHm/newfile: No such file or directory

    Copying files to the temporary directory...OK

    Stripping ELF binaries and libraries...OK

    Compressing man pages...OK

    Building file list...OK

    Building Debian package...OK

    NOTE: The package will not be installed

    Erasing temporary files...OK

    Deleting temp dir...OK


    **********************************************************************

     Done. The new package has been saved to

     somedir/zabbix-2.0.1/zabbix_2.0.1-1_amd64.deb
     You can install it in your system anytime using: 

          dpkg -i zabbix_2.0.1-1_amd64.deb

    **********************************************************************


#BANG !

Congrats, you're done. You now have inside your build directory :

1. a nice clean little package called `zabbix_2.0.1-1_amd64.deb` (if you're on 64bit)
2. in the `frontends/php/` subfolder, the PHP frontend interface for `zabbix` : all you need to do is copy this someplace, make a virtualhost in apache,...)
3. database schemas and tables can be found in `/sss/ddd`
4. lastly, system services to launch Zabbix (server, hhhh, proxy) can be found in `fffff/fffff`

you should pack all that in an archive that you can then save someplace and use for installation.

Time to test it on your build server.

> Remember to keep a copy of `zabbix_2.0.1-1_amd64.deb` and the `frontends/php/` folder for deployment to your `PRODUCTION` server

##install it

    $ dpkg --install  zabbix_2.0.1-1_amd64.deb
    Selecting previously deselected package zabbix.
    (Reading database ... 212230 files and directories currently installed.)
    Unpacking zabbix (from zabbix_2.0.1-1_amd64.deb) ...
    Setting up zabbix (2.0.1-1) ...

Copy your backup of the `frontends/php` folder someplace in a virtualhost referenced directory (if you're using `apache`).

##Do some checks (for the paranoïd)

###What does aptitude have to say

    $ aptitude show zabbix
    Package: zabbix
    New: yes
    State: installed
    Automatically installed: yes
    Version: 2.0.1-1
    Priority: extra
    Section: checkinstall
    Maintainer: p.lhoste@synacom.fr
    Uncompressed Size: 3215k
    Description: Zabbix 2.0.1 Server & Client for Ubuntu 8.04 Hardy (Plesk compatible)
     Zabbix 2.0.1 Server & Client for Ubuntu 8.04 Hardy (Plesk compatible)

or 

    $ aptitude search zabbix
    i   zabbix                - Zabbix 2.0.1 Server & Client for Ubuntu 8.04 Hardy (Plesk compatible)
    p   zabbix-agent          - network monitoring solution - agent
    p   zabbix-frontend-php   - network monitoring solution - PHP front-end
    p   zabbix-proxy-mysql    - network monitoring solution - proxy (using PostgreSQL)
    p   zabbix-proxy-pgsql    - network monitoring solution - proxy (using MySQL)
    p   zabbix-server-mysql   - network monitoring solution - server (using MySQL)
    p   zabbix-server-pgsql   - network monitoring solution - server (using PostgreSQL)

note that only our package is installed

###What files were installed ?

    $ dpkg -L zabbix
    /.
    /usr
    /usr/local
    /usr/local/bin
    /usr/local/bin/zabbix_sender
    /usr/local/bin/zabbix_get
    /usr/local/share
    /usr/local/share/man
    /usr/local/share/man/man8
    /usr/local/share/man/man8/zabbix_proxy.8.gz
    /usr/local/share/man/man8/zabbix_server.8.gz
    /usr/local/share/man/man8/zabbix_agentd.8.gz
    /usr/local/share/man/man1
    /usr/local/share/man/man1/zabbix_get.1.gz
    /usr/local/share/man/man1/zabbix_sender.1.gz
    /usr/local/etc
    /usr/local/etc/zabbix_proxy.conf
    /usr/local/etc/zabbix_agentd.conf
    /usr/local/etc/zabbix_server.conf
    /usr/local/etc/zabbix_agent.conf
    /usr/local/sbin
    /usr/local/sbin/zabbix_agent
    /usr/local/sbin/zabbix_agentd
    /usr/local/sbin/zabbix_proxy
    /usr/local/sbin/zabbix_server
    /usr/share
    /usr/share/doc
    /usr/share/doc/zabbix
    /usr/share/doc/zabbix/AUTHORS
    /usr/share/doc/zabbix/COPYING
    /usr/share/doc/zabbix/README
    /usr/share/doc/zabbix/NEWS
    /usr/share/doc/zabbix/INSTALL
    /usr/share/doc/zabbix/ChangeLog

>Notice that by default, the `zabbix`package installes in `/usr/local`, which is very fine by me. If ever you want it installed someplace else, do that at the `configure` stage bt telling the script of your custom `PREFIX` options.

###If you don't trust `dpkg -L`

    $ updatedb
    # "somedirectory" is your build directory"
    $ locate zabbix | grep -v "somedirectory"

Yields :

    /usr/local/bin/zabbix_get
    /usr/local/bin/zabbix_sender
    /usr/local/etc/zabbix_agent.conf
    /usr/local/etc/zabbix_agentd.conf
    /usr/local/etc/zabbix_proxy.conf
    /usr/local/etc/zabbix_server.conf
    /usr/local/sbin/zabbix_agent
    /usr/local/sbin/zabbix_agentd
    /usr/local/sbin/zabbix_proxy
    /usr/local/sbin/zabbix_server
    /usr/local/share/man/man1/zabbix_get.1.gz
    /usr/local/share/man/man1/zabbix_sender.1.gz
    /usr/local/share/man/man8/zabbix_agentd.8.gz
    /usr/local/share/man/man8/zabbix_proxy.8.gz
    /usr/local/share/man/man8/zabbix_server.8.gz
    /usr/share/doc/zabbix
    /usr/share/doc/zabbix/AUTHORS
    /usr/share/doc/zabbix/ChangeLog
    /usr/share/doc/zabbix/COPYING
    /usr/share/doc/zabbix/INSTALL
    /usr/share/doc/zabbix/NEWS
    /usr/share/doc/zabbix/README
    /var/lib/dpkg/info/zabbix.conffiles
    /var/lib/dpkg/info/zabbix.list


#Setting up Zabbix Server

Now is time to read the usual docs of how to create a `zabbix`user, configure your setup in `zabbix_server.conf`, `zabbix_agent.conf`, `zabbix_agentd.conf` (and `zabbix_proxy.conf` if you need the proxy) and start the server, 

# CONFIGURE SCRIPT options


The `configure` script offers the following options (as of zabbix 2.0.1)

    $ ./configure --help
    `configure' configures this package to adapt to many kinds of systems.

    Usage: ./configure [OPTION]... [VAR=VALUE]...

    To assign environment variables (e.g., CC, CFLAGS...), specify them as
    VAR=VALUE.  See below for descriptions of some of the useful variables.

    Defaults for the options are specified in brackets.

    Configuration:
      -h, --help              display this help and exit
          --help=short        display options specific to this package
          --help=recursive    display the short help of all the included packages
      -V, --version           display version information and exit
      -q, --quiet, --silent   do not print `checking ...' messages
          --cache-file=FILE   cache test results in FILE [disabled]
      -C, --config-cache      alias for `--cache-file=config.cache'
      -n, --no-create         do not create output files
          --srcdir=DIR        find the sources in DIR [configure dir or `..']

    Installation directories:
      --prefix=PREFIX         install architecture-independent files in PREFIX
                              [/usr/local]
      --exec-prefix=EPREFIX   install architecture-dependent files in EPREFIX
                              [PREFIX]

    By default, `make install' will install all the files in
    `/usr/local/bin', `/usr/local/lib' etc.  You can specify
    an installation prefix other than `/usr/local' using `--prefix',
    for instance `--prefix=$HOME'.

    For better control, use the options below.

    Fine tuning of the installation directories:
      --bindir=DIR            user executables [EPREFIX/bin]
      --sbindir=DIR           system admin executables [EPREFIX/sbin]
      --libexecdir=DIR        program executables [EPREFIX/libexec]
      --sysconfdir=DIR        read-only single-machine data [PREFIX/etc]
      --sharedstatedir=DIR    modifiable architecture-independent data [PREFIX/com]
      --localstatedir=DIR     modifiable single-machine data [PREFIX/var]
      --libdir=DIR            object code libraries [EPREFIX/lib]
      --includedir=DIR        C header files [PREFIX/include]
      --oldincludedir=DIR     C header files for non-gcc [/usr/include]
      --datarootdir=DIR       read-only arch.-independent data root [PREFIX/share]
      --datadir=DIR           read-only architecture-independent data [DATAROOTDIR]
      --infodir=DIR           info documentation [DATAROOTDIR/info]
      --localedir=DIR         locale-dependent data [DATAROOTDIR/locale]
      --mandir=DIR            man documentation [DATAROOTDIR/man]
      --docdir=DIR            documentation root [DATAROOTDIR/doc/PACKAGE]
      --htmldir=DIR           html documentation [DOCDIR]
      --dvidir=DIR            dvi documentation [DOCDIR]
      --pdfdir=DIR            pdf documentation [DOCDIR]
      --psdir=DIR             ps documentation [DOCDIR]

    Program names:
      --program-prefix=PREFIX            prepend PREFIX to installed program names
      --program-suffix=SUFFIX            append SUFFIX to installed program names
      --program-transform-name=PROGRAM   run sed PROGRAM on installed program names

    System types:
      --build=BUILD     configure for building on BUILD [guessed]
      --host=HOST       cross-compile to build programs to run on HOST [BUILD]

    Optional Features:
      --disable-option-checking  ignore unrecognized --enable/--with options
      --disable-FEATURE       do not include FEATURE (same as --enable-FEATURE=no)
      --enable-FEATURE[=ARG]  include FEATURE [ARG=yes]
      --disable-dependency-tracking  speeds up one-time build
      --enable-dependency-tracking   do not reject slow dependency extractors
      --disable-largefile     omit support for large files
      --enable-static         Build statically linked binaries
      --enable-server         Turn on build of Zabbix server
      --enable-proxy          Turn on build of Zabbix proxy
      --enable-agent          Turn on build of Zabbix agent and client utilities
      --enable-java           Turn on build of Zabbix Java gateway
      --enable-ipv6           Turn on support of IPv6

    Optional Packages:
      --with-PACKAGE[=ARG]    use PACKAGE [ARG=yes]
      --without-PACKAGE       do not use PACKAGE (same as --with-PACKAGE=no)
      --with-ibm-db2=[ARG]    use IBM DB2 CLI from given sqllib directory
                              (ARG=path); use /home/db2inst1/sqllib (ARG=yes);
                              disable IBM DB2 support (ARG=no)
      --with-ibm-db2-include=[DIR]
                              use IBM DB2 CLI headers from given path
      --with-ibm-db2-lib=[DIR]
                              use IBM DB2 CLI libraries from given path
      --with-mysql[=ARG]      use MySQL client library [default=no], optionally
                              specify path to mysql_config
      --with-oracle=[ARG]     use Oracle OCI API from given Oracle home
                              (ARG=path); use existing ORACLE_HOME (ARG=yes);
                              disable Oracle OCI support (ARG=no)
      --with-oracle-include=[DIR]
                              use Oracle OCI API headers from given path
      --with-oracle-lib=[DIR] use Oracle OCI API libraries from given path
      --with-postgresql[=ARG] use PostgreSQL library [default=no], optionally
                              specify path to pg_config
      --with-sqlite3[=ARG]    use SQLite 3 library [default=no], optionally
                              specify the prefix for sqlite3 library

    If you want to use Jabber protocol for messaging:
      --with-jabber[=DIR]     Include Jabber support [default=no]. DIR is the
                              iksemel library install directory.

    If you want to use cURL library:
      --with-libcurl[=DIR]    use cURL package [default=no], optionally specify
                              path to curl-config

    What ODBC driver do you want to use (please select only one):
      --with-iodbc[=ARG]      use odbc driver against iODBC package [default=no],
                              default is to search through a number of common
                              places for the IODBC files.

      --with-unixodbc[=ARG]   use odbc driver against unixODBC package
                              [default=no], optionally specify full path to
                              odbc_config binary.

    What SNMP package do you want to use (please select only one):
      --with-net-snmp[=ARG]   use NET-SNMP package [default=no], optionally
                              specify path to net-snmp-config

      --with-ucd-snmp[=ARG]   use UCD-SNMP package [default=no], default is to
                              search through a number of common places for the
                              UCD-SNMP files.

    If you want to use SSH2 based checks:
      --with-ssh2[=DIR]       use SSH2 package [default=no], DIR is the SSH2
                              library install directory.

    If you want to check IPMI devices:
      --with-openipmi[=DIR]   Include OPENIPMI support [default=no]. DIR is the
                              OPENIPMI base install directory, default is to
                              search through a number of common places for the
                              OPENIPMI files.

    If you want to check LDAP servers:
      --with-ldap[=DIR]       Include LDAP support [default=no]. DIR is the LDAP
                              base install directory, default is to search through
                              a number of common places for the LDAP files.

    Some influential environment variables:
      CC          C compiler command
      CFLAGS      C compiler flags
      LDFLAGS     linker flags, e.g. -L<lib dir> if you have libraries in a
                  nonstandard directory <lib dir>
      LIBS        libraries to pass to the linker, e.g. -l<library>
      CPPFLAGS    (Objective) C/C++ preprocessor flags, e.g. -I<include dir> if
                  you have headers in a nonstandard directory <include dir>
      CPP         C preprocessor
      PKG_CONFIG  path to pkg-config utility
      PKG_CONFIG_PATH
                  directories to add to pkg-config's search path
      PKG_CONFIG_LIBDIR
                  path overriding pkg-config's built-in search path
      IKSEMEL_CFLAGS
                  C compiler flags for IKSEMEL, overriding pkg-config
      IKSEMEL_LIBS
                  linker flags for IKSEMEL, overriding pkg-config

    Use these variables to override the choices made by `configure' or to help
    it to find libraries and programs with nonstandard names/locations.

    Report bugs to the package provider.
