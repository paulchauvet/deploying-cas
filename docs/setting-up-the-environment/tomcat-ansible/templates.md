# Ansible Templates for Tomcat

The following are the relative template files for Tomcat.  They don't include the Apache HTTPD or CAS configs - this is JUST for Tomcat.  When dealing with some things that have private keys or other sensitive info, you'll want to use Ansible vault.

* [cas6-catalina.properties.j2](https://paulchauvet.github.io/deploying-cas/setting-up-the-environment/tomcat-ansible/templates/cas6-catalina.properties.j2) (destination is /etc/tomcat/context.xml)
* [cas6-context.xml.j2](https://paulchauvet.github.io/deploying-cas/setting-up-the-environment/tomcat-ansible/templates/cas6-context.xml.j2) (destination is /etc/tomcat/context.xml)
* [cas6-server.xml.j2](https://paulchauvet.github.io/deploying-cas/setting-up-the-environment/tomcat-ansible/templates/cas6-server.xml.j2) (destination is /etc/tomcat/server.xml)
* [cas6-web.xml.j2](https://paulchauvet.github.io/deploying-cas/setting-up-the-environment/tomcat-ansible/templates/cas6-web.xml.j2) (destination is /etc/tomcat/web.xml)
* [tomcat.service.j2](https://paulchauvet.github.io/deploying-cas/setting-up-the-environment/tomcat-ansible/templates/tomcat.service.j2) (destination is /etc/systemd/system/tomcat.service)
  * Note: This doesn't have 'cas' in the file name since it is not CAS specific - it's the same for all our Tomcat installs.  When I have a template specific to only one application/server/system - I indicate that in the file name.

