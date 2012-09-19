
---
layout: post
title: Install both Stash and Jira on Ubuntu 12.04 with Java6 & Java7
categories: atlassian
tags: atlassian, jira, stash, java
intro: "I love Atlassian products, and so does my boss. Only we're a small company, not many servers to spare and we need have both Jira and Stash on that same Ubuntu server.<br /><br /><strong>Use Case:</strong> you need to install latest Java (7 from Oracle) but also make other Java6 apps run, without ***wrecking*** your production server. Let's dive in !"
intro-img: "2012-09-19-Jira-Stash-Ubuntu.jpg"

---


# Atlassian tools are great

I work for a small company, we are in the "*less than
10 developers*" category (but there is more to a company than devs,
right?) and we need to get to things fast, with reliable tools.

I've been a long time `Git` user, and as impressive as [gitosis](http://wiki.dreamhost.com/Gitosis) (first one we used),
[gitolite](https://github.com/sitaramc/gitolite) (my favorite of the 3, as I do know PERL and like it), and 
[gitourious](http://gitorious.com/local_install/) (gahhh... ruby)...
are, Atlassian's Stash just meets our needs.

We tried it for 2 months, and don't ever want to look back : for 10
bucks a year, that'd be a stupid move for a small company like ours
which has `almost no time at all`to dedicate to `sysadmin`.

`Stash` was our first acquisition

I'm fine with Java as long as it does not run win my browser, so why
not.

Then, yesterday, my boss asked me something like "*how about bug
tracking ?*", and my answer was something like "* they also have a thing
called Jira*".

Only... ***JIRA only runs on Java6*** as of now. After a little bit of
tinkering, here's my solution for running both `Stash`and `Jira`on the
same (already in PROD) Ubuntu 12.04 Server.

>Summary : it all boils down to :
>
>    1. installing Oracle's Java system-wide
>    2. configuring a (real) 'jira' user to use a version
>    of Java6 which lies in a directory that serves no other purpose.

**warning**: if you're only interested in the 'Jira' part, skip to the
end of this tutorial.

#Intro

>With all things Java it's far better (and easier)
>to install quite everything manually (with the exception 
>of Tomcat which is too tied to Apache, and Apache we use a lot)
>so we install that from the Ubuntu repositories.

for all the rest (java, maven, ant, clojure, leiningen, atlassian
products...) manual installation is by far the fastest and easiest to
maintain. Just replay this tutorial in reverse order, and there should
be nothing remaining of these java-esque installations.


Now, as comes to Oracle's Java7, we will begin by breaking the 'all by
hand' rule, and use a custom `PPA` which installs Oracle's java from the
sources, but in a clean and `zero-dependancies` way, which is what we
want.


>NB1: I do not wish to use `ant`, nowadays you should really be using `maven`

>NB2: installing by hand, when it's easy, means you can keep all your
>java tools up-to-date without waiting for the Ubuntu repositories to
>update.

>NB4 : All this tutorial is executed under the root account as **there
>are no manual builds** taking place. You can `sudo` all the way if you
>like.


#Oracle Java 7

##Add the PPA

[ppa:webupd8team/java](https://launchpad.net/~webupd8team/archive/java)

[and here's some more info about it](http://ubuntuforums.org/showthread.php?t=1977483)

##dependancies

the `add-apt-repository` command (which comes handy) requires the
following repositories. As I'm lazy, ill use it (otherwise, just add the
PPA the usual way).

    $ aptitude install python-software-properties
    The following NEW packages will be installed:
      python-pycurl{a} python-software-properties unattended-upgrades{a} 
    0 packages upgraded, 3 newly installed, 0 to remove and 28 not upgraded.

Then

    $ add-apt-repository ppa:webupd8team/java
    You are about to add the following PPA to your system:
     Oracle Java (JDK) Installer (automatically downloads and installs Oracle JDK7). There are no actual Java files in this PPA. More info: http://www.webupd8.org/2012/01/install-oracle-java-jdk-7-in-ubuntu-via.html
     More info: https://launchpad.net/~webupd8team/ archive/java
    Press [ENTER] to continue or ctrl-c to cancel adding it

    Executing: gpg --ignore-time-conflict --no-options --no-default-keyring --secret-keyring /tmp/tmp.zgh4xP9S0x --trustdb-name /etc/apt/trustdb.gpg --keyring /etc/apt/trusted.gpg --primary-keyring /etc/apt/trusted.gpg --keyserver hkp://keyserver.ubuntu.com:80/ --recv 7B2C3B0889BF5709A105D03AC2518248EEA14886
    gpg: requesting key EEA14886 from hkp server keyserver.ubuntu.com
    gpg: key EEA14886: public key "Launchpad VLC" imported
    gpg: Total number processed: 1
    gpg:               imported: 1  (RSA: 1)


Update the repository cache :

    $ aptitude update



##Installing Oracle's Java7 form this PPA

###0. dependancies

> Those are "indépendant" from ORACLE java's version of  `oracle-java7-installer`

> Just `aptitude install` them (and `aptitude remove` them if you de-install
> everything)

    gsfonts-x11
    libfontenc1
    libxfont1
    x11-common
    xfonts-encodings
    xfonts-utils

> This is the ***ONLY Java related*** package we'll install from
> Ubuntu's repositories


    java-common{a}





###1. Let's go

    $ aptitude install oracle-java7-installer
    The following NEW packages will be installed:
      gsfonts-x11{a} java-common{a} libfontenc1{a} libxfont1{a} oracle-java7-installer x11-common{a} xfonts-encodings{a} xfonts-utils{a} 
    0 packages upgraded, 8 newly installed, 0 to remove and 28 not upgraded.

    (...)

    Setting up java-common (0.43ubuntu2) ...
    Setting up oracle-java7-installer (7u5-0~webupd8~5) ...
    Removing outdated cached downloads...
    Downloading cookie...
    --2012-07-28 00:14:39--  http://launchpadlibrarian.net/98645053/cookie.txt
    Resolving launchpadlibrarian.net (launchpadlibrarian.net)... 91.189.89.229, 91.189.89.228
    Connecting to launchpadlibrarian.net (launchpadlibrarian.net)|91.189.89.229|:80... connected.
    HTTP request sent, awaiting response... 200 OK
    Length: 1053 (1.0K) [text/plain]
    Saving to: `./cookie.txt'

         0K                                                      100% 96.5M=0s

    2012-07-28 00:14:39 (96.5 MB/s) - `./cookie.txt' saved [1053/1053]

    Downloading Oracle Java 7...
    --2012-07-28 00:14:39--  http://download.oracle.com/otn-pub/java/jdk/7u5-b05/jdk-7u5-linux-x64.tar.gz
    Resolving download.oracle.com (download.oracle.com)... 77.67.20.10, 77.67.20.17
    Connecting to download.oracle.com (download.oracle.com)|77.67.20.10|:80... connected.
    HTTP request sent, awaiting response... 302 Moved Temporarily
    Location: https://edelivery.oracle.com/otn-pub/java/jdk/7u5-b05/jdk-7u5-linux-x64.tar.gz [following]
    --2012-07-28 00:14:39--  https://edelivery.oracle.com/otn-pub/java/jdk/7u5-b05/jdk-7u5-linux-x64.tar.gz
    Resolving edelivery.oracle.com (edelivery.oracle.com)... 92.122.246.174
    Connecting to edelivery.oracle.com (edelivery.oracle.com)|92.122.246.174|:443... connected.
    HTTP request sent, awaiting response... 302 Moved Temporarily
    Location: http://download.oracle.com/otn-pub/java/jdk/7u5-b05/jdk-7u5-linux-x64.tar.gz?AuthParam=1343427413_9e078a853a329b50c33d5b07362f1ff3 [following]
    --2012-07-28 00:14:40--  http://download.oracle.com/otn-pub/java/jdk/7u5-b05/jdk-7u5-linux-x64.tar.gz?AuthParam=1343427413_9e078a853a329b50c33d5b07362f1ff3
    Connecting to download.oracle.com (download.oracle.com)|77.67.20.10|:80... connected.
    HTTP request sent, awaiting response... 200 OK
    Length: 81439763 (78M) [application/x-gzip]
    Saving to: `jdk-7u5-linux-x64.tar.gz'

    (...)

    2012-07-28 00:14:56 (4.99 MB/s) - `jdk-7u5-linux-x64.tar.gz' saved [81439763/81439763]

    Download done.

    (...)

    update-alternative thingies

    (...)

    Oracle JDK 7 installed
    update-alternatives: using /usr/lib/jvm/java-7-oracle/jre/lib/amd64/libnpjp2.so to provide /usr/lib/mozilla/plugins/libnpjp2.so (libnpjp2.so) in auto mode.
    Oracle JRE 7 browser plugin installed
    Setting up libfontenc1 (1:1.1.0-1) ...
    Setting up libxfont1 (1:1.4.4-1) ...
    Setting up x11-common (1:7.6 12ubuntu1) ...
    Setting up xfonts-encodings (1:1.0.4-1ubuntu1) ...
    Setting up xfonts-utils (1:7.6 1) ...
    Setting up gsfonts-x11 (0.22) ...
    Processing triggers for libc-bin ...
    ldconfig deferred processing now taking place



###1bis. Now, remove it (so that you can be sure you can)

    $ aptitude purge oracle-java7-installer
    The following packages will be REMOVED:  
      java-common{u} oracle-java7-installer{p} 
    0 packages upgraded, 0 newly installed, 2 to remove and 28 not upgraded.
    Need to get 0 B of archives. After unpacking 334 kB will be freed.
    Do you want to continue? [Y/n/?] y
    (Reading database ... 75109 files and directories currently installed.)
    Removing oracle-java7-installer ...
    Purging configuration files for oracle-java7-installer ...
    dpkg: warning: while removing oracle-java7-installer, directory '/usr/lib/jvm' not empty so not removed.
    (Reading database ... 75093 files and directories currently installed.)
    Removing java-common ...
    Processing triggers for man-db ...

Easy and clean

###1ter. Re-install it


    aptitude install oracle-java7-installer
    The following NEW packages will be installed:
      java-common{a} oracle-java7-installer 
    0 packages upgraded, 2 newly installed, 0 to remove and 28 not upgraded.

###2. Testing

    $ java -version
    java version "1.7.0_05"
    Java(TM) SE Runtime Environment (build 1.7.0_05-b05)
    Java HotSpot(TM) 64-Bit Server VM (build 23.1-b03, mixed mode)

> and also

    $ update-java-alternatives -l

    java-7-oracle 1 /usr/lib/jvm/java-7-oracle


###3. Setup Oracle Java7 as the System's default

[here's a good tutorial on the subject](http://blog.manishchhabra.com/2012/05/installing-oracle-sun-java-jdk-and-setting-java_home-in-ubuntu-linux/)

    $ update-alternatives --install "/usr/bin/java" "java" "/usr/lib/jvm/java-7-oracle/bin/java" 1

> I chose to keep the default (choice 2)

    $ update-alternatives --config java
    There are 2 choices for the alternative java (providing /usr/bin/java).

      Selection    Path                                     Priority   Status
    ------------------------------------------------------------
      0            /usr/lib/jvm/java-7-oracle/bin/java       1         auto mode
      1            /usr/lib/jvm/java-7-oracle/bin/java       1         manual mode
    * 2            /usr/lib/jvm/java-7-oracle/jre/bin/java   1         manual mode

    Press enter to keep the current choice[*], or type selection number: 


> same goes  for **jar**

     update-alternatives --config jar
    There is 1 choice for the alternative jar (providing /usr/bin/jar).

      Selection    Path                                Priority   Status
    ------------------------------------------------------------
      0            /usr/lib/jvm/java-7-oracle/bin/jar   1         auto mode
    * 1            /usr/lib/jvm/java-7-oracle/bin/jar   1         manual mode

    Press enter to keep the current choice[*], or type selection number: 

Default suits me.

###4. ENVIRONEMENT and JAVA_HOME

####The Ubuntu Way
cf. [DOC Ubuntu EnvironmentVariables](https://help.ubuntu.com/community/EnvironmentVariables)


Here's a direct quote from the Ubuntu docs. By all means, follow the
`/etc/environment` path of enlightenment.

>***System-wide environment variables***

>Environment variable settings that affect the system as a whole (rather than just a particular user) should not be placed in any of the many system-level scripts that get executed when the system or the desktop session are loaded, but into
1   **/etc/environment** - This file is specifically meant for system-wide environment variable settings. It is not a script file, but rather consists of assignment expressions, one per line. Specifically, this file stores the system-wide locale and path settings. 


>***Not recommended:***

>2   **/etc/profile** - This file gets executed whenever a bash login shell is entered (e.g. when logging in from the console or over ssh), as well as by the DisplayManager when the desktop session loads. This is probably the file you will get referred to when asking veteran UNIX system administrators about environment variables. In Ubuntu, however, this file does little more than invoke the /etc/bash.bashrc file. 
3   **/etc/bash.bashrc** - This is the system-wide version of the ~/.bashrc file. Ubuntu is configured by default to execute this file whenever a user enters a shell or the desktop environment. 

>**Note**: Any variables added to these locations will not be reflected when invoking them with a sudo command, as sudo has a default policy of resetting the Environment and setting a secure path (this behavior is defined in /etc/sudoers). As a workaround, you can use "sudo su" that will provide a shell with root privileges but retaining any modified PATH variables.

>**Note**: When dealing with end-user/home desktop systems may be appropriate to place settings in the user's ~/.pam_environment files discussed above rather than the system-wide ones, since those files do not require one to utilize root privileges in order to edit and are easily moved between systems.

####Updating `/etc/environment`

> Declare JAVA_HOME **SYSTEM-WIDE** in `/etc/environment` (and make a
> note you did)

    #PLHOSTE 2012-07-28 01:33:05 ** JAVA ENV STUFF **
    #
    #
    #PATH="/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin:/usr/games"
    PATH="/usr/lib/jvm/java-7-oracle/bin:/usr/lib/jvm/java-7-oracle/jre/bin:/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin:/usr/games"
    JAVA_HOME="/usr/lib/jvm/java-7-oracle"



No need to reboot, you're done. Java7 installed, and easy to uninstall.

> Now is the time to skip if you're only interested in `STASH`and
> `JIRA`. I'll deal with MAVEN, ARTIFACTORY and some CLOJURE stuff here,
> beforehand

#Apache Tomcat 7

>***It is IMPERATIVE*** to only install APACHE TOMCAT ***AFTER*** having
correctly installed and configured Java and setup JAVA_HOME system-wide


##install

    $ aptitude install tomcat7

    The following NEW packages will be installed:
      authbind{a} libcommons-collections3-java{a} libcommons-dbcp-java{a} libcommons-pool-java{a} libecj-java{a} libservlet3.0-java{a} libtomcat7-java{a} tomcat7 tomcat7-common{a}
    0 packages upgraded, 9 newly installed, 0 to remove and 28 not upgraded.

> Verification

    $ service tomcat7 status
    * Tomcat servlet engine is running with pid 11581

>OK


#Maven and Artifactory

##MAVEN : Java's packager

> Juste so you see, with Ubuntu's repositories, you'd get (using `aptitude`)  :

    $ aptitude install maven
    The following NEW packages will be installed:
      ant{a} ant-optional{a} aspectj{a} bsh{a} bsh-gcj{a} fop{a} gcj-4.6-base{a} gcj-4.6-jre-lib{a} glassfish-javaee{a} java-wrappers{a} junit{a} junit4{a} libaether-java{a} libantlr-java{a} libaopalliance-java{a} libapache-pom-java{a} 
      libasm3-java{a} libasound2{a} libaspectj-java{a} libasync-http-client-java{a} libatinject-jsr330-api-java{a} libavalon-framework-java{a} libbackport-util-concurrent-java{a} libbatik-java{a} libbsf-java{a} libcdi-api-java{a} 
      libcglib-java{a} libclassworlds-java{a} libcommons-beanutils-java{a} libcommons-cli-java{a} libcommons-codec-java{a} libcommons-collections-java{a} libcommons-collections3-java{a} libcommons-configuration-java{a} 
      libcommons-digester-java{a} libcommons-httpclient-java{a} libcommons-io-java{a} libcommons-jexl-java{a} libcommons-jxpath-java{a} libcommons-lang-java{a} libcommons-logging-java{a} libcommons-net2-java{a} libcommons-parent-java{a} 
      libcommons-validator-java{a} libcommons-vfs-java{a} libdom4j-java{a} libdoxia-java{a} libdoxia-sitetools-java{a} libeasymock-java{a} libexcalibur-logkit-java{a} libfop-java{a} libganymed-ssh2-java{a} libgcj-bc{a} libgcj-common{a} 
      libgcj12{a} libgeronimo-interceptor-3.0-spec-java{a} libgeronimo-jpa-2.0-spec-java{a} libgeronimo-osgi-support-java{a} libgoogle-collections-java{a} libguava-java{a} libhamcrest-java{a} libhttpclient-java{a} libhttpcore-java{a} 
      libice6{a} libitext1-java{a} libjaxen-java{a} libjaxme-java{a} libjaxp1.3-java{a} libjdom1-java{a} libjetty-java{a} libjline-java{a} libjsch-java{a} libjsoup-java{a} libjsr305-java{a} libjtidy-java{a} liblog4j1.2-java{a} 
      libmaven-plugin-tools-java{a} libmaven-reporting-impl-java{a} libmaven-scm-java{a} libmaven2-core-java{a} libmodello-java{a} libnetbeans-cvsclient-java{a} libnetty-java{a} liboro-java{a} libosgi-compendium-java{a} 
      libosgi-core-java{a} libosgi-foundation-ee-java{a} libplexus-ant-factory-java{a} libplexus-archiver-java{a} libplexus-bsh-factory-java{a} libplexus-build-api-java{a} libplexus-cipher-java{a} libplexus-classworlds-java{a} 
      libplexus-classworlds2-java{a} libplexus-cli-java{a} libplexus-container-default-java{a} libplexus-containers-java{a} libplexus-containers1.5-java{a} libplexus-i18n-java{a} libplexus-interactivity-api-java{a} 
      libplexus-interpolation-java{a} libplexus-io-java{a} libplexus-sec-dispatcher-java{a} libplexus-utils-java{a} libplexus-utils2-java{a} libplexus-velocity-java{a} libqdox-java{a} libregexp-java{a} librhino-java{a} libsaxon-java{a} 
      libservlet2.4-java{a} libservlet2.5-java{a} libsisu-guice-java{a} libsisu-ioc-java{a} libslf4j-java{a} libsm6{a} libwagon-java{a} libwerken.xpath-java{a} libxalan2-java{a} libxbean-java{a} libxerces2-java{a} 
      libxml-commons-external-java{a} libxml-commons-resolver1.1-java{a} libxmlgraphics-commons-java{a} libxom-java{a} libxpp2-java{a} libxpp3-java{a} libxt6{a} libxtst6{a} maven rhino{a} velocity{a} 
    0 packages upgraded, 132 newly installed, 0 to remove and 28 not upgraded.
    Need to get 94.4 MB of archives. After unpacking 164 MB will be used.
    Do you want to continue? [Y/n/?] 

Nope. I don't want to continue. What a bloody dependancy mess !


Now for [the official Maven Instructions](http://maven.apache.org/download.html) :

>Unix-based Operating Systems (Linux, Solaris and Mac OS X)

>1.  Extract the distribution archive, i.e. apache-maven-3.0.4-bin.tar.gz to the directory you wish to install Maven 3.0.4. These instructions assume you chose /usr/local/apache-maven. The subdirectory apache-maven-3.0.4 will be created from the archive.
>2.  In a command terminal, add the M2_HOME environment variable, e.g. export M2_HOME=/usr/local/apache-maven/apache-maven-3.0.4.
>3.  Add the M2 environment variable, e.g. export M2=$M2_HOME/bin.
>4.  Optional: Add the MAVEN_OPTS environment variable to specify JVM properties, e.g. export MAVEN_OPTS="-Xms256m -Xmx512m". This environment variable can be used to supply extra options to Maven.
>5.  Add M2 environment variable to your path, e.g. export PATH=$M2:$PATH.
>6.  Make sure that JAVA_HOME is set to the location of your JDK, e.g. export JAVA_HOME=/usr/java/jdk1.5.0_02 and that $JAVA_HOME/bin is in your PATH environment variable.
>7.  Run mvn --version to verify that it is correctly installed.

It's clear, clan and easy
[see here if tyou crave more detailed infos](http://blog.btmatthews.com/2011/08/04/installing-maven-3-on-ubuntu-11-04-lts-server/#comment-382)    

Go someplace you usually use to build/package/download things

    $ cd /OPT/PREBUILD
    $ wget http://ftp.heanet.ie/mirrors/www.apache.org/dist/maven/binaries/apache-maven-3.0.4-bin.tar.gz

Unpack

    $ tar -zxf apache-maven-3.0.4-bin.tar.gz

All my `made by hand` stuff I install under `/usr/local`. Makes sens to
me, might not to you. Do whatever you want.

    $ mkdir /usr/local/apache-maven
    $ cp -R apache-maven-3.0.4 /usr/local/apache-maven
    $ ln -s /usr/local/apache-maven/apache-maven-3.0.4/bin/mvn /usr/bin/mvn

Lastly, in `etc/environment` just add :

    #------MAVEN--------#
    M2_HOME="/usr/local/apache-maven/apache-maven-3.0.4"
    M2="/usr/local/apache-maven/apache-maven-3.0.4/bin"
    MAVEN_OPTS="-Xms256m -Xmx512m"


> How hard was that ??

Let's check things :

    $ mvn -version
    Apache Maven 3.0.4 (r1232337; 2012-01-17 09:44:56 0100)
    Maven home: /usr/local/apache-maven/apache-maven-3.0.4
    Java version: 1.7.0_05, vendor: Oracle Corporation
    Java home: /usr/lib/jvm/java-7-oracle/jre
    Default locale: en_US, platform encoding: UTF-8
    OS name: "linux", version: "3.2.0-26-generic", arch: "amd64", family: "unix"

We are CLEAR.





##ARTIFACTORY : best artifact repository management tool (with WEB UI) for MAVEN

Artifactoy is nothing more (sortif) thant a Tomcat server

[this article](http://jamesbetteley.wordpress.com/2012/07/24/installing-artifactory-on-ubuntu/) is an excellent detailed tutorial on manual installation of `artifactory`.


Quoting the fucking manual again :

>How to Install Artifactory on Ubuntu


>* Make sure you’ve got Java 1.5 or above installed, and make a note of the full path, as you’re going to need this later (mine was /usr/lib/jvm/java-7-openjdk-amd64/jre/).
>* Download the artifactory zip and extract it somewhere.
>* Go to your artifactory bin dir and make the install.sh file executable
>* Now run sudo ./install.sh – this will copy some files around the place and setup some paths.
>* Edit the file /etc/artifactory/default and put the FULL Java path in there as JAVA_HOME
>* Make sure JAVA_HOME is also set in /etc/environment
>* run “sudo service artifactory check”
>* If it all looks good run “sudo service artifactory start”
>* Go to http://localhost:8081/artifactory/
>* You’re done!

Let's do it :

    $ cd /opt/PREBUILT
    $ wget ....
    $ unzip ...
    $ mkdir /usr/local/artifactory
    $ cp -R artifactory-2.6.2 /usr/local/artifactory/
    $ cd /usr/local/artifactory/artifactory-2.6.2/bin


Install :

    $ ./install.sh



    Installing artifactory as a Unix service that will run as user artifactory

    Installing artifactory with home /opt/PREBUILD/artifactory-2.6.2
    Creating user artifactory...id: artifactory: No such user
    creating...DONE

    Checking configuration link and files in /etc/artifactory...
    Moving configuration dir etc to etc.originalDONE
    creating dir /etc/artifactory...creating the link and updating dir...DONE
    Creating environment file /etc/artifactory/default...creating...DONE
    ** INFO: Please edit the files in /etc/artifactory to set the correct environment
    Especially /etc/artifactory/default that defines ARTIFACTORY_HOME, JAVA_HOME and JAVA_OPTIONS

    Creating link /usr/local/artifactory/artifactory-2.6.2/logs to /var/log/artifactory...creating...DONE

    Setting file permissions to etc, logs, work, data and backup...DONE

    Copying the init.d/artifactory script...DONE

    Initializing artifactory service with update-rc.d...
    update-rc.d: warning: artifactory start runlevel arguments (2 3 4 5) do not match LSB Default-Start values (3 4 5)
     Adding system startup for /etc/init.d/artifactory ...
       /etc/rc0.d/K20artifactory -> ../init.d/artifactory
       /etc/rc1.d/K20artifactory -> ../init.d/artifactory
       /etc/rc6.d/K20artifactory -> ../init.d/artifactory
       /etc/rc2.d/S20artifactory -> ../init.d/artifactory
       /etc/rc3.d/S20artifactory -> ../init.d/artifactory
       /etc/rc4.d/S20artifactory -> ../init.d/artifactory
       /etc/rc5.d/S20artifactory -> ../init.d/artifactory
    DONE

    ************ SUCCESS *****************
    Installation of Artifactory completed
    you can now check installation by running:
    > service artifactory check

    Then activate artifactory with:
    > service artifactory start


You might wonder why I keep all these `install logs` ? I keep ALL of
THEM, so that I know what's been installed/linked/... on my system
(which is a prod server) and this habbit has saved my ass more than
once.

>Configuration time

    $ vim /etc/artifactory/default

Add (and comment it) :

    #PLHOSTE 2012-07-28 03:16:02
    export JAVAHOME="/usr/lib/jvm/java-7-oracle"

> Launch it

    $ su someonelese
    $ sudo service artifactory start
    Created output file /usr/local/artifactory/artifactory-2.6.2/logs/consoleout.log
    Starting Jetty: 
    Artifactory home=/usr/local/artifactory/artifactory-2.6.2
    Jetty running pid=17226
    nohup: redirecting stderr to stdout

Some tailing of the logs shows : 

    $ tail -f /var/log/artifactory/artifactory.log 
    2012-07-28 03:18:45,523 [art-init] [INFO ] (o.a.s.ArtifactoryApplicationContext:233) - Initializing org.artifactory.repo.index.InternalIndexerService
    2012-07-28 03:18:45,525 [art-init] [INFO ] (o.a.r.i.IndexerServiceImpl:170) - No indexer cron expression is configured. Indexer will be disabled.
    2012-07-28 03:18:45,526 [art-init] [INFO ] (o.a.s.ArtifactoryApplicationContext:233) - Initializing org.artifactory.security.interceptor.SecurityConfigurationChangesInterceptors
    2012-07-28 03:18:45,528 [art-init] [INFO ] (o.a.s.ArtifactoryApplicationContext:233) - Initializing org.artifactory.repo.jcr.cache.expirable.CacheExpiry
    2012-07-28 03:18:45,543 [art-init] [INFO ] (o.a.s.ArtifactoryApplicationContext:392) - Artifactory application context is ready.
    2012-07-28 03:18:45,565 [art-init] [INFO ] (o.a.w.s.ArtifactoryContextConfigListener:232) - 
    ###########################################################
    ### Artifactory successfully started (17 seconds)       ###
    ###########################################################


> http://yourIP:8081/artifactory/webapp/home.html?0

    user : admin
    login : password

(Defaults that you go change at once)

> Wish to uninstall ?

Dump the configs

    $ cd /etc/
    $ rm -rf artifactory/

Dump the logs

    $ cd /var/log
    $ rm -rf artifactory/

Dump the service

    $ update-rc.d -f artifactory remove
    $ rm /etc/init.d/artifactory    

Dump the directory

    $ cd /usr/local
    $ rm -rf artifactory

Done. Gone.

##Local MAVEN repository for your users

Just for reference, two posts that should get you up and running

[[Setting Up a Maven Repository with Artifactory]]
[http://stackoverflow.com/a/2328284/474526](http://stackoverflow.com/a/2328284/474526)

#Clojure

Clojure is so darn simple to install it should be forbidden.

##Download and put it somewhere

    $ mkdir /usr/local/clojure/
    $ cd /usr/local/clojure/
    $ wget http://repo1.maven.org/maven2/org/clojure/clojure/1.4.0/clojure-1.4.0.zip
    $ unzip clojure-1.4.0.zip
    $ rm clojure-1.4.0.zip
    $ chmod 755 clojure-1.4.0/

##Make a quick wrapper script

    $ vim /usr/local/bin/clj
    #!/bin/sh
    # Clojure wrapper script.
    # With no arguments runs Clojure's REPL.

    # Put the Clojure jar from the install dir and the current folder in the classpath.
    CLOJURE=$CLASSPATH:/usr/local/clojure/clojure-1.4.0/clojure-1.4.0.jar:${PWD}

    if [ "$#" -eq 0 ]; then
        java -cp "$CLOJURE" clojure.main --repl
    else
        java -cp "$CLOJURE" clojure.main "$@"
    fi


Make it executable

    $ chmod  x /usr/local/clojure/clojure-1.4.0

Done !

#Leiningen

You **want** to use Leiningen.

    $ cd /usr/local/bin
    $ wget https://raw.github.com/technomancy/leiningen/preview/bin/lein
    $ chmod  x lein

Quit the root account for a bit

    $ su stevejobs
    $ cd ~/
    $ lein

    $ lein
    Downloading Leiningen now...

    (...)

    Leiningen is a tool for working with Clojure projects.

    Several tasks are available:
    check        Check syntax and warn on reflection.
    classpath    Write the classpath of the current project to output-file.
    clean        Remove all files from project's target-path.
    compile      Compile Clojure source into .class files.
    deploy       Build jar and deploy to remote repository.
    deps         Show details about dependencies.
    do           Higher-order task to perform other tasks in succession.
    help         Display a list of tasks or help for a given task.
    install      Install current project to the local repository.
    jar          Package up all the project's files into a jar file.
    javac        Compile Java source files.
    new          Generate project scaffolding based on a template.
    plugin       DEPRECATED. Please use the :user profile instead.
    pom          Write a pom.xml file to disk for Maven interoperability.
    repl         Start a repl session either with the current project or standalone.
    retest       Run only the test namespaces which failed last time around.
    run          Run the project's -main function.
    search       Search remote maven repositories for matching jars.
    show-profilesList all available profiles or display one if given an argument.
    test         Run the project's tests.
    trampoline   Run a task without nesting the project's JVM inside Leiningen's.
    uberjar      Package up the project files and all dependencies into a jar file.
    upgrade      Upgrade Leiningen to specified version or latest stable.
    version      Print version for Leiningen and the current JVM.
    with-profile Apply the given task with the profile(s) specified.

    Run lein help $TASK for details.

    See also: readme, faq, tutorial, news, sample, profiles,
    deploying and copying.

Leiningen is clean : it only installed these : 

* the directory `~/.lein` with the leiningen util inside

        /home/stevejobs/.lein
        /home/stevejobs/.lein/self-installs
        /home/stevejobs/.lein/self-installs/leiningen-2.0.0-preview7-standalone.jar

* and some dependancies in the local MAVEN repository `~/.m2`

        /home/stevejobs/.m2/repository/
        ├── bultitude
        │   └── bultitude
        │       └── 0.1.5
        │           ├── bultitude-0.1.5.jar
        │           ├── bultitude-0.1.5.jar.sha1
        │           ├── bultitude-0.1.5.pom
        │           ├── bultitude-0.1.5.pom.sha1
        │           └── _maven.repositories
        ├── lein-newnew
        │   └── lein-newnew
        │       └── 0.3.4
        │           ├── lein-newnew-0.3.4.jar
        │           ├── lein-newnew-0.3.4.jar.sha1
        │           ├── lein-newnew-0.3.4.pom
        │           ├── lein-newnew-0.3.4.pom.sha1
        │           └── _maven.repositories
        ├── org
        │   ├── clojure
        │   │   └── clojure
        │   │       ├── 1.2.1
        │   │       │   ├── clojure-1.2.1.pom
        │   │       │   ├── clojure-1.2.1.pom.sha1
        │   │       │   └── _maven.repositories
        │   │       └── 1.3.0
        │   │           ├── clojure-1.3.0.jar
        │   │           ├── clojure-1.3.0.jar.sha1
        │   │           ├── clojure-1.3.0.pom
        │   │           ├── clojure-1.3.0.pom.sha1
        │   │           └── _maven.repositories
        │   └── sonatype
        │       └── oss
        │           └── oss-parent
        │               └── 5
        │                   ├── _maven.repositories
        │                   ├── oss-parent-5.pom
        │                   └── oss-parent-5.pom.sha1
        ├── slingshot
        │   └── slingshot
        │       └── 0.8.0
        │           ├── _maven.repositories
        │           ├── slingshot-0.8.0.jar
        │           ├── slingshot-0.8.0.jar.sha1
        │           ├── slingshot-0.8.0.pom
        │           └── slingshot-0.8.0.pom.sha1
        └── stencil
            └── stencil
                └── 0.2.0
                    ├── _maven.repositories
                    ├── stencil-0.2.0.jar
                    ├── stencil-0.2.0.jar.sha1
                    ├── stencil-0.2.0.pom
                    └── stencil-0.2.0.pom.sha1




Nothing more. 

And you have all you need for fully fledged Clojure development.


# Datomic

No seriously : this one is not covered here :)




#Atlassian Stash


##Quoting the Docs

    Quick Installation
    --------------------------------------

    Requirements:
    * Git 1.7.6
    * Oracle JDK 1.6  (Java)

    If your system does not meet the above requirements, please read the installation documentation: http://confluence.atlassian.com/display/STASH


    ### Linux and Mac

    1. Edit `<Stash installation directory>/bin/setenv.sh`

    2. Set `STASH_HOME` by uncommenting the `STASH_HOME` line and adding the absolute path to the directory where you want Stash to store your data. This path MUST NOT be in the Stash application directory.

    3. In a terminal, run:
        `<Stash installation directory>/bin/start-stash.sh`

    4. In your browser go to:
        `http://localhost:7990`




##Installation

    $ cd /opt/PREBUILT
    $ wget ....
    $ unzip ...
    $ mkdir /usr/local/atlassian-stash
    $ cp -R atlassian-stash-1.1.2 /usr/local/atlassian-stash/
    $ cd /usr/local/atlassian-stash/atlassian-stash-1.1.2/bin

> Add some symbolic links to make upgrading easier (without having to
> modify the startup script each time you update) :


    $ /usr/local/atlassian-stash
    $ ln -s atlassian-stash-1.2.0 stash


> Choose someplace to store STASH's DATA

    $ cd stash
    $ vim bin/setenv.sh

> Add line (place it anywhere you fancy)

    STASH_HOME="/wdatastores/atlassian-stash/"


>DONE !!! You now have a fully managed app for your GIT repositories, your own
> github` or `bitbucket` behind the firewall, ready to run.

##Lauch it

    $ sudo ./start-stash.sh 

    [sudo] password for stevejobs: 
    To run Stash in the foreground, start the server with start-stash.sh -fg
    Starting Atlassian Stash at http://localhost:7990/ as current user

    Detecting JVM PermGen support...
    PermGen switch is supported. Setting to 256m

    If you encounter issues starting or stopping Atlassian Stash, please see the Troubleshooting guide at http://confluence.atlassian.com/display/STASH/Installation Troubleshooting Guide

    Using STASH_HOME:      /wdatastores/atlassian-stash/
    Using CATALINA_BASE:   /usr/local/atlassian-stash/atlassian-stash-1.1.2
    Using CATALINA_HOME:   /usr/local/atlassian-stash/atlassian-stash-1.1.2
    Using CATALINA_TMPDIR: /usr/local/atlassian-stash/atlassian-stash-1.1.2/temp
    Using JRE_HOME:        /usr/lib/jvm/java-7-oracle
    Using CLASSPATH:       /usr/local/atlassian-stash/atlassian-stash-1.1.2/bin/bootstrap.jar
    Using CATALINA_PID:    /usr/local/atlassian-stash/atlassian-stash-1.1.2/work/catalina.pid

Yes that was hard.

##Check it's up and running

open : `http://yourIP:7990`

And follow the setup procedure (by the way, opt-in for one of the SQL
backends right away)

##Startup Script 
[This man saved me a few hours](https://answers.atlassian.com/questions/52897/what-is-the-best-way-to-autostart-stash-on-linux-debian-ubuntu-platform)

>First, add a `stash` user

    $ adduser stash

> Then give it the two directories :

    $ chown stash.stash -R /usr/local/atlassian-stash/
    $ chown stash.stash -R /wdatastores/atlassian-stash/

>Now for the startup script :

    $ vim /etc/init.d/stash

> PASTE :

    #!/bin/bash

    # RUN_AS: The user to run fisheye as. Its recommended that you create a separate user account for security reasons
    RUN_AS=stash

    # STASH_INSTALLATION: The path to the Stash installation. Its recommended to create a symbolic link to the latest version so
    # the process will still work after upgrades.

Adapt the next two lines to your own install :

    STASH_INSTALLATION="/usr/local/atlassian-stash/stash"
    STASH_HOME="/wdatastores/atlassian-stash/"
    stashctl() {
            if [ "x$USER" != "x$RUN_AS" ]; then
                    su - "$RUN_AS" -c "export STASH_HOME=$STASH_HOME;$STASH_INSTALLATION/bin/$1"
            else
                    "export STASH_HOME=$STASH_HOME;$STASH_INSTALLATION/bin/$1"
            fi
    }

    case "$1" in
            start)
                    stashctl start-stash.sh
                    ;;
            stop)
                    stashctl stop-stash.sh
                    ;;
            restart)
                    stashctl stop-stash.sh
                    sleep 10
                    stashctl start-stash.sh
                    ;;
            *)
                    echo "Usage: $0 {start|stop|restart}"
    esac

    exit 0


> make her executable

    $ chmod  x /etc/init.d/stash

> Register the service (yeah, it's not totally LSB compliant)


    $ update-rc.d stash defaults
    update-rc.d: warning: /etc/init.d/stash missing LSB information
    update-rc.d: see <http://wiki.debian.org/LSBInitScripts>
     Adding system startup for /etc/init.d/stash ...
       /etc/rc0.d/K20stash -> ../init.d/stash
       /etc/rc1.d/K20stash -> ../init.d/stash
       /etc/rc6.d/K20stash -> ../init.d/stash
       /etc/rc2.d/S20stash -> ../init.d/stash
       /etc/rc3.d/S20stash -> ../init.d/stash
       /etc/rc4.d/S20stash -> ../init.d/stash
       /etc/rc5.d/S20stash -> ../init.d/stash


> Start Here

    $ service stash start
    To run Stash in the foreground, start the server with start-stash.sh -fg
    >Starting Atlassian Stash at http://localhost:7990/ as current user

    >Detecting JVM PermGen support...
    >PermGen switch is supported. Setting to 256m

    >If you encounter issues starting or stopping Atlassian Stash, please see the Troubleshooting guide at http://confluence.atlassian.com/display/STASH/Installation Troubleshooting Guide

    >Using STASH_HOME:      /wdatastores/atlassian-stash/
    >Using CATALINA_BASE:   /usr/local/atlassian-stash/stash
    >Using CATALINA_HOME:   /usr/local/atlassian-stash/stash
    >Using CATALINA_TMPDIR: /usr/local/atlassian-stash/stash/temp
    >Using JRE_HOME:        /usr/lib/jvm/java-7-oracle
    >Using CLASSPATH:       /usr/local/atlassian-stash/stash/bin/bootstrap.jar
    >Using CATALINA_PID:    /usr/local/atlassian-stash/stash/work/catalina.pid
    >Existing PID file found during start.
    >Removing/clearing stale PID file.


####POSTGRESQL backend

(I'm covering Postgre, mysql is as easy)

>If you're migrating to a SQL based backend using Atlassian's migration
>tool [here's atlassian
>docs](https://confluence.atlassian.com/display/STASH/Connecting+Stash+to+PostgreSQL)

    $ su root
    $ su postgres
    $ psql
    postgres=# CREATE ROLE stashuser WITH LOGIN PASSWORD '********' VALID UNTIL 'infinity';
    CREATE ROLE
    postgres=# CREATE DATABASE stash WITH ENCODING='UTF8' OWNER=stashuser CONNECTION LIMIT=-1;
    CREATE DATABASE
    postgres=# <CTRL+D>


> and feed that data to the migration assistant /
> first-time-configuration tool


    Database Type   PostgreSQL
    JDBC URL        jdbc:postgresql://127.0.0.1:5432/stash
    Username        stashuser
    Password        ********

>NB : in order to modify the password used by stash (migration) : [read this](https://confluence.atlassian.com/display/STASH/How+do+I+change+the+external+database+password)

    $ updatedb
    $ locate stash-config.properties
    $ cd ....
    $ vim stash-config.properties
    > jdbc.password=MY_PASSWORD




# Atlassian Jira - bug tracker, reporting, and much more... and Java 6 !!


Now is the big `AAAAAARGHHH` : I first thought i was fucked. No way i
was going to tinker with the Java Install on this server. Only
acceptable way is have the whole Java6 SDK in a directory someplace and
make `Jira`run with it. Luckilly, there's an easy way to do just that.


###Download the JDK

It's always a pain in the ass with Oracle, just do it by hand, and bring
the `.bin` (yeah, no `tar/gz` for java6) package to a directory you own.


###Unpack

    $ cd /opt/PREBUILD/INSTALLED/JAVA/
    $ mkdir SUN_JAVA6
    $ cd SUN_JAVA6

copy `jdk-6u35-linux-x64.bin` in this DIR (or any other), then extract
its contents (don't worry, this bin `.bin` file is just an archive, it
won't do anything to your OS)

    $ chmod +x jdk-6u35-linux-x64.bin
    $ ./jdk-6u35-linux-x64.bin

Let it extract itself in the `jdk1.6.0_35` directory



##Jira Installation

    $ cd /opt/PREBUILD/INSTALLED/JAVA/
    $ wget http://www.atlassian.com/software/jira/downloads/binary/atlassian-jira-5.1.5.tar.gz
    $ tar -xzvf atlassian-jira-5.1.5.tar.gz
    $ mkdir /usr/local/atlassian-jira
    $ sudo cp -R atlassian-jira-5.1.5-standalone /usr/local/atlassian-jira/
    $ cd /usr/local/atlassian-jira/atlassian-jira-5.1.5-standalone/bin

> Same as with Stash, choose some place for Jira to store static assets

    $ vim bin/setenv.sh

> Add this line (to a path that suits you) :

    JIRA_HOME="/wdatastores/atlassian-jira/"


> Now you want to change the default port. You most certainly already
> have something on `8080` which Jira defaults to. I chose :  7980

    $ sudo vim conf/server.xml

> Change :

       <Connector port="8080" ...

With :

        <Connector port="7980" ...



##Starting it right away WONT DO : Proof

    $ sudo ./start-jira.sh 



    To run JIRA in the foreground, start the server with start-jira.sh -fg
    executing as current user
                    .....
              .... .NMMMD.  ...
            .8MMM.  $MMN,..~MMMO.
            .?MMM.         .MMM?.

        OMMMMZ.           .,NMMMN~
        .IMMMMMM. .NMMMN. .MMMMMN,
          ,MMMMMM$..3MD..ZMMMMMM.
            =NMMMMMM,. .,MMMMMMD.
            .MMMMMMMM8MMMMMMM,
              .ONMMMMMMMMMMZ.
                ,NMMMMMMM8.
                .:,.$MMMMMMM
              .IMMMM..NMMMMMD.
            .8MMMMM:  :NMMMMN.
            .MMMMMM.   .MMMMM~.
            .MMMMMN    .MMMMM?.

          Atlassian JIRA
          Version : 5.1.5

    Detecting JVM PermGen support...
    PermGen switch is supported. Setting to 256m

    If you encounter issues starting or stopping JIRA, please see the Troubleshooting guide at http://confluence.atlassian.com/display/JIRA/Installation+Troubleshooting+Guide

    Using JIRA_HOME:       /wdatastores/atlassian-jira/

    Server startup logs are located in /usr/local/atlassian-jira/atlassian-jira-5.1.5-standalone/logs/catalina.out
    Using CATALINA_BASE:   /usr/local/atlassian-jira/atlassian-jira-5.1.5-standalone
    Using CATALINA_HOME:   /usr/local/atlassian-jira/atlassian-jira-5.1.5-standalone
    Using CATALINA_TMPDIR: /usr/local/atlassian-jira/atlassian-jira-5.1.5-standalone/temp

Because of :

    >> Using JRE_HOME:        /usr/lib/jvm/java-7-oracle


    Using CLASSPATH:       /usr/local/atlassian-jira/atlassian-jira-5.1.5-standalone/bin/bootstrap.jar
    Using CATALINA_PID:    /usr/local/atlassian-jira/atlassian-jira-5.1.5-standalone/work/catalina.pid

So you stop it.

    $ ./stop-jira.sh

##Trick : make a JIRA user, and set its profile to use our Java6

>Add new User Jira ( a real one, with a homedir and a shell)

    $ su root
    $ adduser jira

> Give it the Jira directories

    $ chown jira.jira -R /usr/local/atlassian-jira/
    $ chown jira.jira -R /wdatastores/atlassian-jira/

> And make it use the Java6 SDK by default by modifying the user's
> `.profile`

    $ vim /home/jira/.profile

And add those two lines at the end (adapt to where you unpacked the
Java6 JDK)

    # JIRA SPECIFIC
    # JAVA_HOME : we use JAVA6 for JIRA until it's ported to JAVA7
    export JAVA_HOME=/opt/PREBUILD/INSTALLED/JAVA/SUN_JAVA6/jdk1.6.0_35/
    export PATH=$PATH:$JAVA_HOME/bin




##Then the Startup Script will 'su' to this user's account

> Make yourself some symlink


    $ /usr/local/atlassian-jira
    $ ln -s atlassian-jira-5.1.5-standalone jira

>And the startup script

    $ vim /etc/init.d/jirah

>Paste :

    #!/bin/bash

    # RUN_AS: The user to run fisheye as. Its recommended that you create a separate user account for security reasons
    RUN_AS=jira

    # JIRA: The path to the jira installation. Its recommended to create a symbolic link to the latest version so
    # the process will still work after upgrades.

Modify the following two line to relfect your setup :

    JIRA_INSTALLATION="/usr/local/atlassian-jira/jira"
    JIRA_HOME="/wdatastores/atlassian-jira/"

    jiractl() {
            if [ "x$USER" != "x$RUN_AS" ]; then
                    su - "$RUN_AS" -c "export JIRA_HOME=$JIRA_HOME;$JIRA_INSTALLATION/bin/$1"
            else
                    "export JIRA_HOME=$JIRA_HOME;$JIRA_INSTALLATION/bin/$1"
            fi
    }

    case "$1" in
            start)
                    jiractl start-jira.sh
                    ;;
            stop)
                    jiractl stop-jira.sh
                    ;;
            restart)
                    jiractl stop-jira.sh
                    sleep 10
                    jiractl start-jira.sh
                    ;;
            *)
                    echo "Usage: $0 {start|stop|restart}"
    esac

    exit 0



> make it executable

    $ chmod  +x /etc/init.d/jira

> register the service


    $ update-rc.d jira defaults
    update-rc.d: warning: /etc/init.d/jira missing LSB information
    update-rc.d: see <http://wiki.debian.org/LSBInitScripts>
    Adding system startup for /etc/init.d/jira ...
      /etc/rc0.d/K20jira -> ../init.d/jira
      /etc/rc1.d/K20jira -> ../init.d/jira
      /etc/rc6.d/K20jira -> ../init.d/jira
      /etc/rc2.d/S20jira -> ../init.d/jira
      /etc/rc3.d/S20jira -> ../init.d/jira
      /etc/rc4.d/S20jira -> ../init.d/jira
      /etc/rc5.d/S20jira -> ../init.d/jira



> Start It 

    $ service jira start

    To run JIRA in the foreground, start the server with start-jira.sh -fg
    executing as current user
                    .....
              .... .NMMMD.  ...
            .8MMM.  $MMN,..~MMMO.
            .?MMM.         .MMM?.

        OMMMMZ.           .,NMMMN~
        .IMMMMMM. .NMMMN. .MMMMMN,
          ,MMMMMM$..3MD..ZMMMMMM.
            =NMMMMMM,. .,MMMMMMD.
            .MMMMMMMM8MMMMMMM,
              .ONMMMMMMMMMMZ.
                ,NMMMMMMM8.
                .:,.$MMMMMMM
              .IMMMM..NMMMMMD.
            .8MMMMM:  :NMMMMN.
            .MMMMMM.   .MMMMM~.
            .MMMMMN    .MMMMM?.

          Atlassian JIRA
          Version : 5.1.5

    Detecting JVM PermGen support...
    PermGen switch is supported. Setting to 256m

    If you encounter issues starting or stopping JIRA, please see the Troubleshooting guide at http://confluence.atlassian.com/display/JIRA/Installation+Troubleshooting+Guide

    Using JIRA_HOME:       /wdatastores/atlassian-jira/

    Server startup logs are located in /usr/local/atlassian-jira/jira/logs/catalina.out
    Using CATALINA_BASE:   /usr/local/atlassian-jira/jira
    Using CATALINA_HOME:   /usr/local/atlassian-jira/jira
    Using CATALINA_TMPDIR: /usr/local/atlassian-jira/jira/temp

>***Yeeepeee !!!***

    Using JRE_HOME:        /opt/PREBUILD/INSTALLED/JAVA/SUN_JAVA6/jdk1.6.0_35/

    Using CLASSPATH:       /usr/local/atlassian-jira/jira/bin/bootstrap.jar
    Using CATALINA_PID:    /usr/local/atlassian-jira/jira/work/catalina.pid







## POSTGRESQL backend

###Create the database (Postgres here)

    $ su root
    $ su postgres
    $ psql
    postgres=# CREATE ROLE jirauser WITH LOGIN PASSWORD '*******' VALID UNTIL 'infinity';
    CREATE ROLE
    postgres=# CREATE DATABASE jira WITH ENCODING='UTF8' OWNER jirauser CONNECTION LIMIT -1;
    CREATE DATABASE
    postgres=# <CTRL+D>



#Summary

what else to say. You now have all the Java you need to do quite
anything you like, you didn't bork the Ubuntu server, you can roll it
all back to ground zero.

Have a nice day !
