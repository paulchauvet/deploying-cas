## Setup variables
If you setup the four variables we needed in the encrypted vault file, you're ready.  Those variables are setup like the following.

!!! warning
    Make sure not to ever let your real values for these end up in a public git repo.  The ones shown here I generated just for this article and are not what are in use in my environment.

**roles/cas6/vars/cas-vault.yml:**
``` yaml
dev_tgc_signing_key: dYffipMGbhIoyUsSHwzEDcBk5nbETZtH-lR3R776wNavS4koAHyQkDdK_rJIWrYYgZZ2TsLW5NXfcDI_Ivn4Uw
dev_tgc_encryption_key: Gpy6fxMnh5RhmFUJWVcJL9WIAuFODzTlPIaQOTq9-jM
dev_webflow_signing_key: xoWJ9S2vmTgvB_CdZeddb1qmqPihBGIw5Op27MsNxfR8KWgPgrx4VXpssTTM3IcXkLJVoxTylg_hSxvH65M88g
dev_webflow_encryption_key: qnHUX0kyFHc718oh/f+ebw==
```
                                                
## Setup templates
To start - the only template files we need are:

* [dev-cas.properties.j2](https://paulchauvet.github.io/deploying-cas/building-cas/templates/dev-cas.properties.j2) (which will be /etc/cas/config/cas.properties on the Dev CAS systems)
* [log4j2.xml](https://paulchauvet.github.io/deploying-cas/building-cas/templates/log4j2.xml) (which will be /etc/cas/config/log4j2.xml on the CAS systems, as so far I haven't had a reason to have a different log4j2 config on DEV/TEST/PROD like I have for cas.properties).

## Setup handlers

You can copy over the handlers/main.yml file from the Apache Tomcat role since you'll need it here, but we'll also need one to reload httpd (once the AJP proxy config is added to httpd):

**roles/cas6/handlers/main.yml:**
``` yaml
# handlers file for cas6
- name: stop tomcat
  ansible.builtin.systemd:
    name: tomcat
    state: stopped

- name: start tomcat
  ansible.builtin.systemd:
    name: tomcat
    state: started

- name: restart tomcat
  ansible.builtin.systemd:
    name: tomcat
    state: restarted

- name: reload httpd
  ansible.builtin.systemd:
    name: httpd
    state: reloaded

```


## Create tasks

We're going to have multiple plays within the CAS6 role, so we'll break them up like we did for Tomcat, but to start with, it's just two includes for 'base-cas-config.yml'.

**roles/cas6/tasks/main.yml:**
``` yaml
- include_tasks: base-cas-config.yml
- include_tasks: service-config.yml
- include_tasks: cas-ajp-proxy.yml
```

### Base CAS Config tasks ###
For the first include, the only tasks are:

* Make sure the cas config and services directories exist
* Make sure that the cas.properties file we have in our templates directory matches the one in /etc/cas/config (and if not - update it and use notify to restart Tomcat when the play is done).
* Make sure that the log4j2.xml file we have in our files directory matches the one in /etc/cas/config (and if not - update it and use notify to restart Tomcat when the play is done).
* Make sure that the cas.war file we have in our files directory matches the one in /opt/tomcat/latest/webapps (and if not - update it and use notify to restart Tomcat when the play is done).

**roles/cas6/tasks/base-cas-config.yml:**
``` yaml
---

- include_vars: cas-vault.yml

- name: Ensure base CAS config directory exists
  ansible.builtin.file:
    path: /etc/cas/config
    state: directory
    mode: 770
    owner: root
    group: tomcat

- name: Ensure base CAS services directory exists
  ansible.builtin.file:
    path: /etc/cas/services
    state: directory
    mode: 750
    owner: root
    group: tomcat

- name: Ensure base CAS log directory exists
  ansible.builtin.file:
    path: /var/log/cas
    state: directory
    mode: 0750
    owner: tomcat
    group: tomcat

# Note: This is for dev specifically.  If we have multiple environments, there's
# a different config file for each.  The 'when' on inventory_hostname is used to
# differentiate here.
- name: Configure cas.properties file (dev)
  ansible.builtin.template:
    src: dev-cas.properties.j2
    dest: /etc/cas/config/cas.properties
    mode: 0640
    owner: root
    group: tomcat
  when: ("login6dev" in inventory_hostname)
  notify: restart tomcat

# For us at least, log4j2 is the same on production, dev, or test,
# so it's not tier dependent and doesn't need the 'when' statement.
# This uses the 'copy' module instead of the 'template' module since
# Ansible does not like the {} all over that file.  If you need to change that per-server
# You'll need to create an version of log4j2.xml with {} escaped.
- name: Copy log4j2.xml
  ansible.builtin.copy:
    src: log4j2.xml
    dest: /etc/cas/config/log4j2.xml
    mode: 0640
    owner: root
    group: tomcat
  notify: restart tomcat  

# Note: cas6.war file should be placed in the 'files' subdirectory of your cas role.
# You can explicitly replace 'src' with the path to it, i.e. /home/your-user/cas-overlay-template/build/libs/cas.war if you
# want, or you can manually copy the cas.war file into the files directory.
- name: Copy CAS war file
  ansible.builtin.copy:
    src: cas.war
    dest: /opt/tomcat/latest/webapps/cas.war
    mode: 0750
    owner: tomcat
    group: tomcat
  when: ("login6dev" in inventory_hostname)
  notify: restart tomcat
```

### Setup AJP proxy between Tomcat and httpd
This is what handles the connections on Apache httpd's side from Apache Tomcat.  We setup the AJP proxy on the Tomcat side earlier.

**roles/cas6/tasks/cas-ajp-proxy.yml:**

``` yaml
---

- name: "Copy CAS Apache AJP proxy config"
  ansible.builtin.template:
    src: cas-ajp.conf.j2
    dest: /etc/httpd/conf.d/cas-ajp.conf
    owner: root
    group: root
    mode: 0644
  notify: reload httpd
```

## Run the play

In this example below, I'm running the play but limiting it to only one host.

``` yaml
[chauvetp@ansible templates]$ ansible-playbook ~/ansible/site.yml --ask-vault-pass --limit login6devb
Vault password: 
```

This will cause the dev-cas.properties.j2 template to be copied over to /etc/cas/config/cas.properties.  It will also copy the log4j2.xml file from the files directory on the ansible host to /etc/cas/config/log4j2.xml on the system or systems specified in the limit command.  It will also substitute the variables in the cas properties file using the contents of cas-vault.yml, which were decrypted via the --ask-vault-pass option.  If the cas.properties file on the target system is different than the template (after variable substitution), then it will also restart Tomcat.


## Test the install
Once your play goes through - you will want to go to your server (i.e. https://YourCASServer.domain.edu/cas/login) and check that it loads.  If you left in a test user defined by *cas.authn.accept.users* in cas.properties, you can test those credentials here.  If it succeeds, you'll receive a "Log In Successful" message.