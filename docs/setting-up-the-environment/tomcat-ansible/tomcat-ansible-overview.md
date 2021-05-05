# Using Ansible to put it all together

Okay - now that I've covered all the steps that are done to get Tomcat working - you should realize I never do them manually anymore.  I have way too much to do to be manually doing all that.  When I initially started doing this for CAS 5, I had a combination of shell and python scripts to streamline this - but as I've gotten to use Ansible more over the past few years, I moved it all there.  The only thing I'm not doing with Ansible really is generating encryption/signing keys, setting up the Azure side of things, and building CAS itself.

I've put a copy of my ansible playbooks that are used in this document at:
(https://github.com/paulchauvet/ansible-playbooks)

!!! danger
    Before you start with Ansible - you will really want to think about how to lock it down.  Your Ansible host - whether you use it for CAS servers alone, or for managing other applications, OS updates, etc., has a LOT of access.  Consider at the very least ensuring your Ansible hosts are only accessible inbound from a VPN - or possibly even dedicated bastion hosts - and is configured with Duo or some other form of MFA for SSH access.  You don't want your system management tool to be a vector for attack.


## Ansible hosts file
You have to have an Ansible hosts file created.  You can start with a very small config for a single server even.  I always split my CAS servers (or any set of systems where multiples are active at a time) into different phases for when I patch, so a pair of CAS servers below are listed like the following:

```
[test_phase1]
login6deva ansible_python_interpreter=/usr/libexec/platform-python tomcat_major_ver=9.0 tomcat_ver=9.0.41 jdk_version=11


[test_phase2]
login6devb ansible_python_interpreter=/usr/libexec/platform-python tomcat_major_ver=9.0 tomcat_ver=9.0.41 jdk_version=11
```

What I've defined here is:

* *ansible_python_interpreter* (since RHEL 8 doesn't come with a default python in the path, you have to tell Ansible where it can find python)
* *tomcat_major_ver* (since some of my Ansible tasks are version specific, as I still have some Tomcat 8.5.x systems)
* *tomcat_ver* (this is where the specific version of Tomcat is specified.  This is helpful when you want to push out updates to your test/dev systems first, but aren't ready to for production)
* *jdk_version* (likewise, I have systems which are using OpenJDK 8 and others with OpenJDK 11, and I want to have my plays handle these.)


## Define a site.yml file
This is where you can associate roles with systems.  As an example, here's my development CAS hosts, where I have several roles:  (one for security hardening, based on the [Center for Internet Security](https://www.cisecurity.org/cis-benchmarks/) benchmarks, one for Apache Tomcat, and one for Apache httpd.  There are two CAS related roles which we'll be building later (I've commented them out for now - you'll uncomment them as they are needed).

It should be within your main ansible directory

**ansible/site.yml**
``` yaml
---

# CAS 6
- hosts: login6deva,login6devb
  roles:
    - security-hardening-rhel8
    - apache-tomcat
    - apache-httpd
    #- cas6
    #- cas-client
```

## Create a role for Tomcat
If you don't already have a 'roles' directory within your main ansible directory, create one.  Mine sits within my own user directory on my Ansible/build server.

Once that is created, go to the roles directory and initialize a new role for apache-tomcat:

``` console
[chauvetp@ansible ~]$ cd ansible/roles/
[chauvetp@ansible roles]$ ansible-galaxy init apache-tomcat
- Role apache-tomcat was created successfully
[chauvetp@ansible roles]$ ls apache-tomcat/
defaults  files  handlers  meta  README.md  tasks  templates  tests  vars
```
