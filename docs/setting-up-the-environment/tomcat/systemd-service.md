# Configure systemd to start Tomcat

!!! tip
    The steps below are for the CAS servers - not the build server.  That being said - if you are going to install and maintain via Ansible - you don't need to do this manually, though it is worth seeing how to do it manually.  With Ansible, the unit file will be an Ansible template that it will push out and enable.


RHEL, since version 7, has used systemd (instead of old sysv init scripts) to manage system resources.  A *unit* in systemd is any resource systemd knows how to operate on and manage.  This will give you a basic unit file with the ability to start/stop/restart the service (including on boot).

## Definte Tomcat as a service unit

Create a new file: {==/etc/systemd/system/tomcat.service==} with the contents below:

```
[Unit]
Description=Apache Tomcat Web Application Container
After=network.target

[Service]
Type=forking
PIDFile=/var/run/tomcat.pid
UMask=0007

# Tomcat variables
Environment='JAVA_HOME=/usr/lib/jvm/java-openjdk'
Environment='CATALINA_PID=/var/run/tomcat.pid'
Environment='CATALINA_HOME=/opt/tomcat/latest'
Environment='CATALINA_BASE=/opt/tomcat/latest'
Environment='CATALINA_OPTS=-Xms512M -Xmx2048M -XX:+UseParallelGC -server'

# Needed to make use of Tomcat Native Library
Environment='LD_LIBRARY_PATH=/opt/tomcat/latest/lib'

ExecStart=/opt/tomcat/latest/bin/jsvc \
            -Dcatalina.home=${CATALINA_HOME} \
            -Dcatalina.base=${CATALINA_BASE} \
            -Djava.awt.headless=true \
            -Djava.util.logging.manager=org.apache.juli.ClassLoaderLogManager \
            -Djava.util.logging.config.file=${CATALINA_BASE}/conf/logging.properties \
            -cp ${CATALINA_HOME}/bin/commons-daemon.jar:${CATALINA_HOME}/bin/bootstrap.jar:${CATALINA_HOME}/bin/tomcat-juli.jar \
            -pidfile ${CATALINA_PID} \
            -java-home ${JAVA_HOME} \
            -user tomcat \
            $CATALINA_OPTS \
            org.apache.catalina.startup.Bootstrap

ExecStop=/opt/tomcat/latest/bin/jsvc \
            -pidfile ${CATALINA_PID} \
            -stop \
            org.apache.catalina.startup.Bootstrap

[Install]
WantedBy=multi-user.target
```

## Enable the Tomcat service

Set the appropriate SELinux file context and file permissions:
``` console
restorecon /etc/systemd/system/tomcat.service
chmod 644 /etc/systemd/system/tomcat.service
```

Enable the service - which will cause Tomcat to be started at boot.
``` console
systemctl enable tomcat.service
```

You can then stop/start/restart and check the status of Tomcat
```
systemctl start tomcat
systemctl stop tomcat
systemctl restart tomcat
systemctl status tomcat
```

