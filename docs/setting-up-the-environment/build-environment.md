# Build Environment

!!! warning "Summary"
    My build server is our existing Ansible host.  I've set that system up several years ago - so I may have missed a step or two here.  I'll try to recreate this from scratch later though.

## Install and Configure git
We keep our code for git in an internal git repository running [Bitbucket Server](https://www.atlassian.com/software/bitbucket).  We make enough changes to CAS and Ansible that I would be insane to not use git or some other form of version control for development.  Whether you do or not is up to you - but the instructions here assume you are using some sort of git repository.

!!! tip ""
  If you aren't familiar with Git - I recommend Pro Git, available online for free via (https://git-scm.com/book/en/v2).

### Install and basic configuration of Git

This should be done as the user you are going to be running Ansible and building CAS from - NOT root.

```
  yum install git
  git config --global user.name "Your Name Here"
  git config --global user.email "Your Email Here"
```

## Install Ansible
We are using the version of Ansible that comes with RedHat.  This was just installed via:

```
  yum install ansible
```

## Set up SSH public key authentication
If you're deploying via Ansible, or just using SCP/SFTP to copy files over, you'll want to set up public key authentication.

### From the master server:

<figure>
  <img src="/images/ssh-keygen.png" alt="Screenshot showing ssh public key generation.  Command used is ssh-keygen with no arguments.  All options are the default."/>
</figure>

When done - view the contents of the public key (by default in /home/username/.ssh/id_rsa.pub) - you'll need it in the next step.

### From the CAS servers:
Edit the file /root/.ssh/authorized_keys
Place the contents of the public key in that file.

### Test
Login to your build server as the user you will be building CAS and running ansible from (I'll use 'builduser' as an example here).  See if you can login to one of your newly built CAS servers without entering a password.

```
ssh root@cas6dev1
```

If successful - you'll be logged in without a username or password.  If this is the first time you are connecting via SSH from the build host to the CAS host, you'll be warned that the authenticity of the host cannot be established, and you'll be prompted to enter *yes* to continue connecting.


## Configure a hosts file for Ansible
You will need a hosts file for Ansible.  This file can have simply the hostname, or it could have some variables as well.  We at New Paltz put a few things there such as the hostname, the VM name (in vCenter), the folder (in vCenter), and the Tomcat version (so I can roll out updates to Tomcat in some environments earlier than others).

An example of this for CAS servers is below.  This file would be in /etc/ansible/hosts:

```
[CAS_DEV]
cas6dev1 vmware_name="CAS 6 Test 1"  vmware_folder="/Computer Services/vm/CAS Authentication Servers" tomcat_major_ver=9.0 tomcat_ver=9.0.41 jdk_version=11
cas6dev2 vmware_name="CAS 6 Test 2"  vmware_folder="/Computer Services/vm/CAS Authentication Servers" tomcat_major_ver=9.0 tomcat_ver=9.0.41 jdk_version=11

[CAS_TEST]
cas6test1 vmware_name="CAS 6 Test 1" vmware_folder="/Computer Services/vm/CAS Authentication Servers" tomcat_major_ver=9.0 tomcat_ver=9.0.41 jdk_version=11
cas6test2 vmware_name="CAS 6 Test 2" vmware_folder="/Computer Services/vm/CAS Authentication Servers" tomcat_major_ver=9.0 tomcat_ver=9.0.41 jdk_version=11

[CAS_PROD]
cas6prod1 vmware_name="CAS 6 Test 1" vmware_folder="/Computer Services/vm/CAS Authentication Servers" tomcat_major_ver=9.0 tomcat_ver=9.0.41 jdk_version=11
cas6prod2 vmware_name="CAS 6 Test 2" vmware_folder="/Computer Services/vm/CAS Authentication Servers" tomcat_major_ver=9.0 tomcat_ver=9.0.41 jdk_version=11
```