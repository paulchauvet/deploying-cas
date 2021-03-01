# Building CAS client


## Template setup
We will have three templates for this.  Two are the index.php pages that were discussed on the previous page.  The third is hte cas.conf file which will be placed within /etc/httpd/conf.d/.

We're only doing this for the development environment, but if you need CAS clients for prod or test, you can include a separate file for each.  Only the CAS server name, or possibly the Certificate section at the end, will differ here.

**roles/cas-client/templates/dev-cas-client.conf.j2:**
``` apacheconf
LoadModule auth_cas_module modules/mod_auth_cas.so

<Directory "/var/www/html/secured-by-cas">
    <IfModule mod_auth_cas.c>
        AuthType CAS
    </IfModule>

    Require valid-user
</Directory>

<IfModule mod_auth_cas.c>
    CASLoginUrl           https://{{ CAS_DEV_URL }}/cas/login
    CASValidateUrl        https://{{ CAS_DEV_URL }}/cas/serviceValidate
    CASCookiePath         /var/cache/httpd/mod_auth_cas/
    CASSSOEnabled         On
    CASDebug              Off
    CASAttributePreffix   CAS-

    # Set the following instead you're having issues:
    #LogLevel		  Debug
    #CASDebug              On

    # Set the following if you're using a self-signed or other non-commercially signed cert
    # such as from a local CA
    CASCertificatePath      /etc/pki/tls/certs/dev-cas.crt
</IfModule>
```



### Setup certificate
If you're using a non-commercially signed certificate - you'll need to tell the CAS client where it is.  Easiest way to do this is to download that into the template directory and have a task ensure it is referenced in the CAS config.

If you're using a self-signed cert, or a local certificate authority, copy that certificate to your templates directory as *dev-cas.crt* (it will have to be referenced by name in *dev-cas-client.conf.j2*.


## Task setup

This is a small task - but I still prefer to use sub-tasks in my main.yml file.  It makes it easier to expand it later.

**roles/cas-client/tasks/main.yml:**
``` yaml
---
# tasks file for cas-client
- include_tasks: setup-cas-client.yml
- include_tasks: setup-test-pages.yml
# The following is only needed if you are using a non-commercially signed cert in
# your development or test environments.
- include_tasks: non-commercial-cert.yml
```

### setup-cas-client.yml task

Then you'll create 'setup-cas-client.yml'.  It will do the following:

* Ensure the prerequisite packages to build the CAS client are installed
  * Yes - some of these were installed by earlier playbooks, but you may want to have the CAS client installed on places other than the CAS server.
* Check if the CAS client is already installed (if so - the rest is ignored via the when statements)
* Download the source from Apereo's github, unpack it, configure and compile it
* Runs *make install* (to put mod_auth_cas.so in the httpd modules directory)
* Places the *cas-client.conf* into /etc/httpd/conf.d
* Reloads Apache httpd (if necessary)

**roles/cas-client/tasks/setup-cas-client.yml:**
``` yaml
---

- name: Setup prerequisite dnf packages
  dnf:
    name:
      - gcc
      - httpd
      - httpd-devel
      - libcurl-devel
      - libtool
      - make
      - openssl-devel
      - pcre-devel
      - php
      - redhat-rpm-config
    state: present

- name: Check if CAS client is already installed
  stat:
    path: /etc/httpd/modules/mod_auth_cas.so
  register: cas_client

- name: Check if cas client zip file exists
  stat:
    path: "/tmp/cas-client.zip"
  register: cas_client_zip

# Only download source zip if it isn't already downloaded
# and the CAS client isn't already installed
- name: Download source zip when it doesn't already exist
  get_url:
    url: https://github.com/apereo/mod_auth_cas/archive/master.zip
    dest: /tmp/cas-client.zip
    mode: 0600
  when: cas_client_zip.stat.exists == False and cas_client.stat.exists == False

# Only unpack archive if the cas_client isn't already installed
# and the CAS client isn't already installed
- name: Unpack cas-client source archive
  unarchive:
    src: "/tmp/cas-client.zip"
    dest: /tmp/
    remote_src: yes
  when: cas_client.stat.exists == False and cas_client.stat.exists == False

# Only do this if the cas_client isn't already installed
- name: Run autoreconf for CAS client
  command: "autoreconf -ivf"
  args:
    chdir: "/tmp/mod_auth_cas-master"
  when: cas_client.stat.exists == False

- name: Run configure for CAS client
  command: "./configure"
  args:
    chdir: "/tmp/mod_auth_cas-master"
  when: cas_client.stat.exists == False

- name: Run make for CAS client
  command: "make"
  args:
    chdir: "/tmp/mod_auth_cas-master"
  when: cas_client.stat.exists == False

- name: Install CAS client
  command: "make install"
  args:
    chdir: "/tmp/mod_auth_cas-master"
  when: cas_client.stat.exists == False
  notify: reload httpd

- name: Ensure CAS cookie directory exists
  file:
    path: /var/cache/httpd/mod_auth_cas
    state: directory
    owner: apache
    group: apache
    mode: 0700

# Replace "login6dev" with whatever you are using in your dev hosts
# We use login6deva and login6devb as our dev cas servers so it will catch both of those
- name: Setup Apache CAS config file
  template:
    src: dev-cas-client.conf.j2
    dest: /etc/httpd/conf.d/cas-client.conf
    mode: 0644
    owner: root
    group: root
  when: ("login6dev" in inventory_hostname)
  notify: reload httpd

- name: "Ensure php-fpm is set to start on boot"
  ansible.builtin.systemd:
    name: php-fpm
    state: started
    enabled: yes

```

### setup-test-pages.yml task

The following are the contents of setup-test-pages.yml, which were created on the previous page.  It will just ensure the 'secured-by-cas' directory exists, and the two index.php files are placed (one in /var/www/html, one in /var/www/html/secured-by-cas)

**roles/cas-client/tasks/setup-test-pages.yml:**
``` yaml
---

- name: Setup CAS test index page
  template:
    src: main-index.php
    dest: /var/www/html/index.php
    mode: 0755
    owner: root
    group: root
  when: ("login6dev" in inventory_hostname)

- name: Ensure secured-by-cas directory exists
  file:
    path: /var/www/html/secured-by-cas
    state: directory
    owner: root
    group: root
    mode: 0755
  when: ("login6dev" in inventory_hostname)

- name: Setup basic 'secured-by-cas' test index page
  template:
    src: basic-cas-check-index.php
    dest: /var/www/html/secured-by-cas/index.php
    mode: 0755
    owner: root
    group: root
  when: ("login6dev" in inventory_hostname)
```

### setup non-commercial-cert task
If you're using a commercially signed cert, you can ignore this section.  Otherwise, you'll need to do get the the public certificate of your local certificate authority or the self-signed cert of your CAS server (or load balancer), and store it in templates/.

You'll need to reference the file name variable in CAS_DEV_CERT_FILE in vars/main.yml

**roles/cas-client/tasks/non-commercial-cert.yml:**

``` yaml
# The 'source' here - should be your self signed cert if you're using one
# In our case here - this is our local certificate authority
- name: Setup self-signed cert
  template:
    # Make sure to define CAS_DEV_CERT_FILE in vars/main.yml
    src: {{ CAS_DEV_CERT_FILE }}.crt
    dest: /etc/pki/tls/certs/{{ CAS_DEV_CERT_FILE }}.crt
    mode: 0644
    owner: root
    group: root
  when: ("login6dev" in inventory_hostname)
  notify: reload httpd


```


## References
* [Apereo's mod_auth_cas client](https://github.com/apereo/mod_auth_cas)