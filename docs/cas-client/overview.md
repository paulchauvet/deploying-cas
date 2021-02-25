# CAS Client Overview

To make it easier to develop and test CAS - we should have a CAS client application.  For my development environment, I just install this on our CAS development servers, within Apache httpd.

We'll use this CAS client to test as we go along - including as we build out extra functionality such as MFA, attribute release, service restrictions by group or other attributes, etc.

!!! note ""
    As you probably noticed - I'm assuming you're using Ansible at this point.  My hope is that even if you're not - what I'm providing here is still usable (and you can still read what the Ansible tasks are doing)

## Configuring HTTPD to use CAS

The mod_auth_cas plugin allows an Apache web server to interact with a CAS server via the CAS protocol.  Red Hat does not offer this plugin for installation via yum however, so it must be downloaded and built from source code.   We'll build the plugin via Ansible as we've done for other tasks.  To start with, we'll (you guessed it) build another role so that this can be applied independently of the CAS servers.

## Create Ansible role

``` console
[chauvetp@ansible ~]$ cd ansible/roles/
[chauvetp@ansible roles]$ ansible-galaxy init cas-client
- Role cas-client was created successfully
[chauvetp@ansible roles]$ ls casclient/
defaults  files  handlers  meta  README.md  tasks  templates  tests  vars
```

## Variable setup
The only variable we're using here is {{ CAS_DEV_URL }}, though you can fill in the rest if you're using it on other hosts.  If your cas servers are behind a load balancer, this is the load balancer's virtual host, not the individual servers.

**roles/cas-client/vars/main.yml:**
``` yaml
CAS_DEV_URL: your_dev_hostname.domain.edu
CAS_TEST_URL: your_test_hostname.domain.edu
CAS_PROD_URL: your_prod_hostname.domain.edu

# The following is needed if you're using self-signed or other non-commercially signed certs
# for your dev CAS server or the load balancer in front of it
CAS_DEV_CERT_FILE: your_cert_file_name 
```

## Handler setup
**roles/cas-client/handlers/main.yml:**
``` yaml
# handlers file for cas-client
  - name: restart httpd
    service:
      name: "httpd"
      state: "restarted"
      
  - name: reload httpd
    service:
      name: "httpd"
      state: "reloaded"

```