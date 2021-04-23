# Update the CAS client

We've made the CAS server side changes (though not deployed them yet) but we need to change the CAS client side, both the Apache config, and the php pages.

## Update mod_auth_cas settings

Edit the *def-cas-client.conf.j2* file in *roles/cas-client/templates/*.  The only thing to do here is create another directory configuration for our *duo-secured* service

When done it will look like this - with the new additions highlighted

**roles/cas-client/templates/dev-cas-client.conf.j2**

``` apacheconf hl_lines="33-41"
LoadModule auth_cas_module modules/mod_auth_cas.so

# this is our basic config from earlier
<Directory "/var/www/html/secured-by-cas">
    <IfModule mod_auth_cas.c>
        AuthType CAS
        CASAuthNHeader  On
    </IfModule>

    Require valid-user
</Directory>

# Return all attribute directory
<Directory "/var/www/html/return-all">
    <IfModule mod_auth_cas.c>
        AuthType        CAS
        CASAuthNHeader  On
    </IfModule>

    Require valid-user
</Directory>

# Return mapped attributes directory
<Directory "/var/www/html/return-mapped">
    <IfModule mod_auth_cas.c>
        AuthType        CAS
        CASAuthNHeader  On
    </IfModule>

    Require valid-user
</Directory>

# Duo MFA test directory
<Directory "/var/www/html/duo-secured">
    <IfModule mod_auth_cas.c>
        AuthType        CAS
        CASAuthNHeader  On
    </IfModule>

    Require valid-user
</Directory>

<IfModule mod_auth_cas.c>
    CASLoginUrl             https://{{ CAS_DEV_URL }}/cas/login
    CASValidateUrl          https://{{ CAS_DEV_URL }}/cas/samlValidate
    CASCookiePath           /var/cache/httpd/mod_auth_cas/
    CASSSOEnabled           On
    CASValidateSAML         On
    CASAttributePreffix     CAS-
    CASDebug                Off
    CASCertificatePath      /etc/pki/tls/certs/np-ca.crt
</IfModule>
```

## Update the PHP page templates

### Edit the existing main-index file to reference the Duo page
**roles/cas-client/templates/main-index.php:**

``` html
<!DOCTYPE html>
<html lang="en">
  <head>
    <title>CAS client test page</title>
    <meta charset="utf-8">
    <meta name="viewport" content="width=device-width, initial-scale=1">
    <link rel="stylesheet" href="//maxcdn.bootstrapcdn.com/bootstrap/3.3.7/css/bootstrap.min.css">
  </head>
  <body>
    <div class="container">
      <h1>CAS client test page</h1>
        <p><big>Click <a href="secured-by-cas/index.php">here</a> for our basic test.</big></p>
        <p><big>Click <a href="return-all/index.php">here</a> for our 'return all attributes' test.</big></p>
        <p><big>Click <a href="return-mapped/index.php">here</a> for our 'return mapped attributes' test.</big></p>
        <p><big>Click <a href="duo-secured/index.php">here</a> for our 'Duo MFA' test.</big></p>
    </div>
  </body>
</html>
```

### Create a new 'duo-secured-index.php' index file
This can be identical to the previously created basic-cas-check-index.php file - you can just update the title page and/or text.
**roles/cas-client/templates/return-all.php:**

``` html
<!DOCTYPE html>
<html lang="en">
  <head>
    <title>CAS Duo MFA test page</title>
    <meta charset="utf-8">
    <meta name="viewport" content="width=device-width, initial-scale=1">
    <link rel="stylesheet" href="//maxcdn.bootstrapcdn.com/bootstrap/3.3.7/css/bootstrap.min.css">
  </head>
  <body>
    <div class="container">
      <h1>CAS Duo MFA test page</h1>
      <h2>Attributes Returned by CAS</h2>
      <?php
        echo "<pre>";

        if (array_key_exists('REMOTE_USER', $_SERVER)) {
            echo "REMOTE_USER = " . $_SERVER['REMOTE_USER'] . "<br>";
        }

        $headers = getallheaders();
        foreach ($headers as $key => $value) {
            if (strpos($key, 'Cas-') === 0) {
                echo substr($key, 4) . " = " . $value . "<br>";
            }
        }

        echo "</pre>";
      ?>
    </div>
  </body>
</html>
```


## Update the CAS client Ansible role

Edit the *setup-test-pages.yml* file in *roles/cas-client/tasks* to ensure the directories are created and the new files are copied over.  The newly added content is highlighted.

**roles/cas-client/tasks/setup-test-pages.yml:**
``` yaml hl_lines="66-82"
---

- name: Setup CAS test index page
  ansible.builtin.template:
    src: main-index.php
    dest: /var/www/html/index.php
    mode: 0755
    owner: root
    group: root
  when: ("login6dev" in inventory_hostname)

- name: Ensure secured-by-cas directory exists
  ansible.builtin.file:
    path: /var/www/html/secured-by-cas
    state: directory
    owner: root
    group: root
    mode: 0755
  when: ("login6dev" in inventory_hostname)

- name: Setup basic 'secured-by-cas' test index page
  ansible.builtin.template:
    src: basic-cas-check-index.php
    dest: /var/www/html/secured-by-cas/index.php
    mode: 0755
    owner: root
    group: root
  when: ("login6dev" in inventory_hostname)

- name: Ensure return-all directory exists
  ansible.builtin.file:
    path: /var/www/html/return-all
    state: directory
    owner: root
    group: root
    mode: 0755
  when: ("login6dev" in inventory_hostname)

- name: Setup return-all index page
  ansible.builtin.template:
    src: return-all-index.php
    dest: /var/www/html/return-all/index.php
    mode: 0755
    owner: root
    group: root
  when: ("login6dev" in inventory_hostname)

- name: Ensure return-mapped directory exists
  ansible.builtin.file:
    path: /var/www/html/return-mapped
    state: directory
    owner: root
    group: root
    mode: 0755
  when: ("login6dev" in inventory_hostname)

- name: Setup return-mapped index page
  ansible.builtin.template:
    src: return-mapped-index.php
    dest: /var/www/html/return-mapped/index.php
    mode: 0755
    owner: root
    group: root
  when: ("login6dev" in inventory_hostname)

- name: Ensure duo-secured directory exists
  ansible.builtin.file:
    path: /var/www/html/duo-secured
    state: directory
    owner: root
    group: root
    mode: 0755
  when: ("login6dev" in inventory_hostname)

- name: Setup duo-secured index page
  ansible.builtin.template:
    src: duo-secured-index.php
    dest: /var/www/html/duo-secured/index.php
    mode: 0755
    owner: root
    group: root
  when: ("login6dev" in inventory_hostname)

```

## Create a duo-secured service definition
We're going to add a new service - with another high evaluation order (85000).

To follow the naming scheme we used [earlier](https://paulchauvet.github.io/deploying-cas/service-config/overview/#create-a-test-service-definition-file), you'll first need a unique ID.  As with last time, its recommended that you use the {==date +%s==} command to get the datetime in unix epoch format.  For my example, I have 1614372813 as that ID.  I'm thus calling my service file *ApacheTestDuo-1614372813.json* and placing it in the roles/cas6/templates/dev-services directory.

You'll need to make sure the serviceID matches the host with your CAS client and the id is updated to match what you have in your file name.  We're using the *ReturnAllAttributeReleasePolicy* here (which again - you may not want or need to use in production).

**roles/cas6/templates/dev-services/ApacheTestDuo-1614372813.json:**
```json
{
    "@class" : "org.apereo.cas.services.RegexRegisteredService",
    "serviceId" : "^https://logindev.newpaltz.edu/duo-secured(\\z|/.*)",
    "name" : "Apache Test - Duo MFA",
    "id" : 1614372813,
    "description" : "Apache Test - Duo MFA",
    "attributeReleasePolicy" : {
      "@class" : "org.apereo.cas.services.ReturnAllAttributeReleasePolicy"
    },
    "multifactorPolicy" : {
        "@class" : "org.apereo.cas.services.DefaultRegisteredServiceMultifactorPolicy",
        "multifactorAuthenticationProviders" : [ "java.util.LinkedHashSet", [ "mfa-duo" ] ]
    },
    "evaluationOrder" : 85000
}
```


## Rerun the playbook

``` yaml
[chauvetp@ansible templates]$ ansible-playbook ~/ansible/site.yml --ask-vault-pass --limit <your_CAS_server>
Vault password: 
```
