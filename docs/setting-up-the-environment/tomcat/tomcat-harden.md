# Harden the installation

!!! tip
    The steps below are for the CAS servers - not the build server.  That being said - if you are going to install and maintain via Ansible - you don't need to do this manually, though it is worth seeing how to do it manually.

The [Tomcat Security Considerations](https://tomcat.apache.org/tomcat-9.0-doc/security-howto.html) document makes several recommendations for hardening a Tomcat installation:


* Tomcat should not be run as the root user; it should be run as a dedicated user (usually named tomcat) that has minimum operating system permissions. It should not be possible to log in remotely as the tomcat user.

* All Tomcat files should be owned by user root and group tomcat (the tomcat user’s default group should be group tomcat). File/directory permissions should be set to owner read/write, group read only, and world none. The exceptions are the logs, temp, and work directories, which should be owned by the tomcat user instead of root.

* The default and example web applications included with the Tomcat distribution should be removed if they are not needed.

* Auto-deployment should be disabled, and web applications should be deployed as exploded directories rather than web application archives (WAR files).

Implementing these recommendations means that, even if an attacker compromises the Tomcat process, he or she cannot change the Tomcat configuration, deploy new web applications, or modify existing web applications.

## Create a tomcat user and group
``` console
[root@login6devb ~]# groupadd -r tomcat
[root@login6devb ~]# useradd -r -d /opt/tomcat -g tomcat -s /sbin/nologin tomcat
    (This gives a 'shell' of /sbin/nologin which prevents the user from logging in)
```

## Set file ownership and permissions

Some of these should be owned by user:root and group:tomcat (conf and webapps).  Others (logs temp work) should be owned by user/group:tomcat.
``` shell
mkdir -p /opt/tomcat/latest/conf/Catalina/localhost

cd /opt/tomcat/latest/conf
chown -R root:tomcat .
chmod -R u+rwX,g+rX,o= .
chmod -R g-w .

cd /opt/tomcat/latest/webapps
chown -R root:tomcat .
chmod -R u+rwX,g+rX,o= .
chmod -R g-w .

cd /opt/tomcat/latest/logs
chown -R tomcat:tomcat .
chmod -R u+rwX,g+rX,o= .
chmod -R g-w .

cd /opt/tomcat/latest/temp
chown -R tomcat:tomcat .
chmod -R u+rwX,g+rX,o= .
chmod -R g-w .

cd /opt/tomcat/latest/work
chown -R tomcat:tomcat .
chmod -R u+rwX,g+rX,o= .
chmod -R g-w .
```


## Remove example webapps
Unless you have a good reason, these should all be removed.  If you do need to keep these, you'll want to heavily restrict access to them.

``` console
[root@login6devb ~]# cd /opt/tomcat/latest/
[root@login6devb latest]# rm -rf temp/* work/*
[root@login6devb latest]# cd webapps/
[root@login6devb webapps]# rm -rf docs examples host-manager manager

```

!!! note
    Important: The command above does not remove the ROOT web application from the webapps directory because it can be useful in a development/test environment to quickly determine whether Tomcat is working properly. However, when deploying Tomcat to production servers, the ROOT application should be removed along with the rest of the default web applications.


## To do
Still need to add guidance on other aspects of the Tomcat Security Considerations document, such as server.xml configuration.

## References
[Tomcat Security Considerations](https://tomcat.apache.org/tomcat-9.0-doc/security-howto.html)
