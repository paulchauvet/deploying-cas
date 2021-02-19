# Tomcat Dependencies

!!! tip
    The steps below are for the CAS servers - not the build server.  That being said - if you are going to install and maintain via Ansible - you don't need to do this manually, though it is worth seeing how to do it manually.

As discussed in the [Tomcat Overview](https://paulchauvet.github.io/deploying-cas/setting-up-the-environment/tomcat/tomcat-overview/), we will be installing:

* [Apache Portable Runtime (APR)](https://apr.apache.org) - as per Apache, "Tomcat can use the Apache Portable Runtime to provide superior scalability, performance, and better integration with native server technologies"
* [Tomcat Native Library](http://tomcat.apache.org/native-doc/) - this is best described on the Apache site as "an optional component for use with Apache Tomcat that allows Tomcat to use certain native resources for performance, compatibility, etc."
* [Apache Commons Daemon](https://commons.apache.org/proper/commons-daemon/) - this allows Apache to be started as root to perform some privileged operations, then switch to a non-privileged user.


The Tomcat Native Library and Commons Daemon are shipped with Tomcat but need to be compiled.  APR is downloaded and installed separately from Apache.


## Apache Portable Runtime

The Tomcat Native Library, which will be installed later here, depends on the Apache Portable Runtime (APR) library.  As with Tomcat itself, we'll be installing APR within /opt/apr - with a symlink to the latest version.

To download, compile, and install this, see the following:

!!! note
    Your URL in the wget command below will differ depending on which mirror you are getting from the [APR Download](https://apr.apache.org/download.cgi) page, and which specific version of APR you are downloading (1.7.0 was released in April 2019 and it does not change in version nearly as much as Apache Tomcat).  Replace {==1.7.0==} with the latest version.

``` console
[root@login6devb ~]# mkdir -p /opt/apr/apr-1.7.0
[root@login6devb ~]# ln -s /opt/apr/apr-1.7.0 /opt/apr/latest
[root@login6devb ~]# cd /tmp/
[root@login6devb tmp]# wget https://mirrors.ocf.berkeley.edu/apache//apr/apr-1.7.0.tar.gz
    (download progress will show)
[root@login6devb tmp]# tar xzf apr-1.7.0.tar.gz
    (There will be a lot of output - look for errors.  There may be an error of "rm: cannot remove 'libtoolT': No such file or directory" which appears harmless.  If you get an error like 'no acceptable C compiler found in $PATH' - make sure gcc is installed.  This is listed in the prerequisies section of the tomcat-overview.)
[root@login6devb tmp]# cd apr-1.7.0
[root@login6devb apr-1.7.0]# make
    (There will be a lot of output - look for errors.)
[root@login6devb apr-1.7.0]# make test
    (There will be a lot of output and it may take a while to run - look for errors.)
[root@login6devb apr-1.7.0]# make install
    (There will be a lot of output - look for errors.)
[root@login6devb tmp]# rm -rf /tmp/apr-1.7.0 /tmp/apr-1.7.0.tar.gz

```

When it's done - you'll have /opt/apr/apr-1.7.0 (or whatever version) installed and with /opt/apr/latest as a symlink to the latest version.  This way you can update APR to a later version without having to recompile other applications that use it.

## Tomcat Native Library

The source for Tomcat Native Library is included in the Tomcat version that was downloaded in the previous page.  It has to be extracted, compiled, and installed.  Note: The tomcat native version may change between Tomcat minor releases.  For example, it is {==1.2.26==} as of Tomcat {==9.0.43==}.

To do so:

``` console
[root@login6devb ~]# cd /opt/tomcat/latest/bin/
[root@login6devb bin]# tar xzf tomcat-native.tar.gz 
[root@login6devb bin]# cd tomcat-native-*-src/native
[root@login6devb native]# ./configure \
>  --with-java-home=/usr/lib/jvm/java-openjdk \
>  --with-apr=/opt/apr/latest/bin/apr-1-config \
>  --prefix=/opt/tomcat/apache-tomcat-9.0.43
    (There will be a lot of output - look for errors.)
[root@login6devb native]# make
    (There will be a lot of output - look for errors.  You're probably tired of me saying this though!)
[root@login6devb native]# make install
    (There will be a lot of output - look for errors.)
[root@login6devb native]# cd ../..
[root@login6devb bin]# rm -rf tomcat-native-*-src
```

When done - the Tomcat Native Library will be installed in the {==lib==} directory of the version of Tomcat you're working in.  You can see the 'libtcnative' files there.

## Apache Commons Daemon (jsvc)
The Apache Commons Daemon (jsvc) allows Tomcat to be started as root to perform some privileged operations (such as binding to ports below 1024) and then switch identity to run as a non-privileged user, which is better from a security perspective. Like the Tomcat Native Library, the Commons Daemon is included as part of the Tomcat distribution; it just needs to be extracted, compiled, and installed.  Note: Like the tomcat native, library, the apache commons daemon version may change between Tomcat minor releases.  For example, it is {==1.2.26==} as of Tomcat {==9.0.43==}.

To do so:

``` console
[root@login6devb ~]# cd /opt/tomcat/latest/bin/
[root@login6devb bin]# tar xzf commons-daemon-native.tar.gz
[root@login6devb bin]# cd commons-daemon-*-native-src/unix
[root@login6devb unix]# ./configure --with-java=/usr/lib/jvm/java-openjdk
    (There will be a lot of output - look for errors.)
[root@login6devb unix]# make
[root@login6devb unix]# mv jsvc ../..
[root@login6devb unix]# cd ../..
[root@login6devb unix]# rm -rf commons-daemon-*-native-src
```

This installs the {==jsvc==} binary within the bin directory of the version of Tomcat you're working in.
