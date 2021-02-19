# Tomcat Installation

!!! tip
    The steps below are for the CAS servers - not the build server.  That being said - if you are going to install and maintain via Ansible - you don't need to do this manually, though it is worth seeing how to do it manually.

As mentioned in the [architecture] section, RHEL 8 doesn't package Tomcat (and RHEL 7 only packaged Tomcat 8.0.x) so this is both how I've had Tomcat managed in RHEL 7 for CAS 5, as well as how I'm installing and maintaining Tomcat 9.0.x on RHEL 8.

!!! note
    Your URL in the wget command below will differ depending on which mirror you are getting from the [Tomcat Download](https://tomcat.apache.org/download-90.cgi) page, and which specific version of Tomcat you are downloading (9.0.43 was the latest in the 9.0.x branch when this page was last updated.)

``` console
[root@login6devb ~]# mkdir -p /opt/tomcat
[root@login6devb ~]# cd /opt/tomcat
[root@login6devb tomcat]# wget https://mirrors.ocf.berkeley.edu/apache/tomcat/tomcat-9/v9.0.43/bin/apache-tomcat-9.0.43.tar.gz
    (download progress will show)
[root@login6devb tomcat]# tar xzf apache-tomcat-9.0.43.tar.gz
[root@login6devb tomcat]# ln -s /opt/tomcat/apache-tomcat-9.0.43 latest
[root@login6devb tomcat]# rm -f apache-tomcat-9.0.43.tar.gz
[root@login6devb tomcat]# 

```

