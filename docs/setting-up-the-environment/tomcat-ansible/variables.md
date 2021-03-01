# Ansible Variable setup for Tomcat

I don't have a lot of variables for Tomcat on it's own, and none are 'sensitive'.  They're basically version information so I don't have to hard code versions into the plays.  If you ever have sensitive info that needs to go into a variables file, you'll want to use [Ansible Vault](https://docs.ansible.com/ansible/latest/user_guide/vault.html) so it isn't plain text.

**roles/apache-tomcat/vars/main.yml**
``` yaml

# where the Tomcat archive file gets downloaded
tomcat_archive_dest: /root/apache-tomcat-{{ tomcat_ver }}.tar.gz

# Apache Portable Runtime version - it's been 1.7.0 since April 2019
# Your actual mirror will vary - I didn't programatically pick something random
# So you may not want to just copy the mirror for APR I have here.
apr_ver: 1.7.0
apr_archive_url: http://apache.cs.utah.edu//apr/apr-{{ apr_ver }}.tar.gz
apr_archive_dest: /root/apr-{{ apr_ver }}.tar.gz

# Unfortunately - without reading release notes, or unpacking Tomcat to test when a new version comes out,
# you won't know what version of the Commons Daemon or Tomcat Native Library are in that Tomcat version.
# My practice is when upgrading to a new version for the first time - I will download that version,
# unpack it, and unpack the Tomcat Native Library and Commons Daemon to check their version.
# For example, Tomcat 9.0.43 had 1.2.26 for the Tomcat Native Library, where Tomcat 9.0.41 had 1.2.25.
# When my playbook 'failed' on this - I just went and updated the variable.
# Same was true where the commons daemon version updated from 1.2.3 to 1.2.4 with Tomcat 9.0.43.
# If you're going to have multiple versions of Tomcat running for a while, you may want to define
# these in your /etc/ansible/hosts file instead.

tomcat_native_ver: 1.2.26
commons_daemon_ver: 1.2.4

JAVA_HOME: /usr/lib/jvm/java-11-openjdk
  

```