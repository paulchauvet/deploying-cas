# Setting up Apache HTTPD in front of Tomcat

I prefer, for a number of reasons, to put Apache httpd in front of Tomcat (even though there's a load balancer in front of the CAS servers as well.  I'm sure there's ways around this, but I like the flexibility that it provides me, and I also have had other applications on the CAS servers that are not running on Tomcat and still need to be accessible (including SimpleSAML for applications we haven't moved to Azure or directly into CAS yet).

For now, I'm not going to go over full configuration of Apache httpd here.  I will *eventually* (hopefully by end of March 2021) put up my steps and Ansible playbooks to get httpd running, but for now - I'll mention the Tomcat/httpd specific portion.

We use AJP for communication between Apache httpd and Apache Tomcat.  Configuring this is in two steps, one on the httpd server and one on Tomcat.


## On the httpd server
Create a configuration file in /etc/httpd/conf.d.  We call ours 'cas-ajp.conf' but it doesn't matter as long as it ends in .conf.  The contents of which are below:

```
ProxyRequests Off
<Proxy *>
        Order allow,deny
        Allow from all
</Proxy>

ProxyPass               /cas    ajp://localhost:8009/cas
ProxyPassReverse        /cas    ajp://localhost:8009/cas
```


## On the Tomcat server

Edit /etc/tomcat/server.xml and define an AJP port:

``` xml
    <!-- Define an AJP 1.3 Connector on port 8009
         See https://tomcat.apache.org/tomcat-9.0-doc/config/ajp.html for more on the 'secretRequired'
         and 'secret' options. Since I'm only exposing this to localhost via host firewall -->
    <Connector port="8009" protocol="AJP/1.3" redirectPort="8443" secretRequired="false"/>
```

The Tomcat portion is already in the [Tomcat server.xml](https://paulchauvet.github.io/deploying-cas/setting-up-the-environment/tomcat-ansible/templates/cas6-server.xml.j2) that is linked to from the Tomcat/Ansible section.