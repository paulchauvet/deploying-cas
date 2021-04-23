# Tomcat overview

!!! summary
    Apache Tomcat will be used as the Java Servlet container for CAS.  To ensure best practices for security, and performance, the latest versions of OpenSSL, the Tomcat Native Library, the Apache Portable Runtime, and the latest in the 9.0.x series of Apache Tomcat will be used.

## Tomcat architecture
For our Tomcat installation, I don't like to rely on the Red Hat distributed versions.  This is since with RHEL 7 - they only provided Tomcat 8.0.x from their repositories, and with RHEL 8 - it's only provided as part of JBoss.

Using this method - you get an easy to maintain environment, which relies on symlinks to keep the 'important' stuff (configurations, log files, servlets) outside of the installation directories.

When done - you will have /opt/tomcat - and within that you'll have a directory for each Tomcat install you have (for example, /opt/tomcat/apache-tomcat-9.0.39 or /opt/tomcat/apache-tomcat-9.0.43), and a symlink (/opt/tomcat/latest) which points to the in-use version.  This allows for a quick upgrade and rollback if there are issues.

Then - within /opt/tomcat/apache-tomcat-9.0.x, you'll have symlinks to where things actually live, for example:

* */opt/tomcat/apache-tomcat-9.0.39/conf* is a symlink to */etc/tomcat*
* */opt/tomcat/apache-tomcat-9.0.39/logs* is a symlink to */var/log/tomcat*
* */opt/tomcat/apache-tomcat-9.0.39/temp* is a symlink to */var/cache/tomcat/temp*
* */opt/tomcat/apache-tomcat-9.0.39/webapps* is a symlink to */var/lib/tomcat*
* */opt/tomcat/apache-tomcat-9.0.39/work* is a symlink to */var/cache/tomcat/work*

### Components:
* [EPEL (Extra Packages for Enterprise Linux)](https://fedoraproject.org/wiki/EPEL) - which is needed for a couple packages.
* [Haveged](https://www.issihosts.com/haveged/) - for better entropy in random number generation than what is included by default
* [Apache Portable Runtime](https://apr.apache.org)
* [Tomcat Native Library](http://tomcat.apache.org/native-doc/)
* [Apache Commons Daemon](https://commons.apache.org/proper/commons-daemon/)
* [OpenJDK](https://openjdk.java.net/)
* [Apache HTTPD](https://httpd.apache.org/) - since I hate doing SSL/TLS and a number of other things in Apache Tomcat directly.


## Prerequisites

* gcc
* libtool
* make
* policycoreutils-python


```
dnf install gcc libtool make policycoreutils-python
```