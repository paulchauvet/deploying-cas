# Initial CAS Configuration

By default, CAS expects to find its configuration files in the operating system directory /etc/cas.  Almost every aspect of CAS server configuration is controlled via settings stored in the cas.properties file located in the /etc/cas/config directory.  **We're going to start with a simple config - and enhance it as features are added.**

Tomcat will also expect the .war file to be placed within /opt/tomcat/latest/webapps (aka /var/lib/tomcat which is where that symlink points to).  We'll be handling both the config and the cas.war file in this page.

!!! tip
    The config files are for the CAS servers.  If you're not using Ansible - you'll have to get them into /etc/cas/config/ on the indivudal CAS servers on your own.  I'll be assuming Ansible going forward.  If you're not using Ansible - you can still use this guide to create configuration files - you'll just have to fill in the variables manually instead of letting Ansible substitute them from vars files.

## Create Ansible CAS role

Before we start - let's create a new role for cas6 - just like we did for apache-tomcat.  Go to the roles directory and initialize a new role for cas6:

``` console
[chauvetp@ansible ~]$ cd ansible/roles/
[chauvetp@ansible roles]$ ansible-galaxy init cas6
- Role cas6 was created successfully
[chauvetp@ansible roles]$ ls cas6/
defaults  files  handlers  meta  README.md  tasks  templates  tests  vars
```

## Start creating configuration files

Create a new file called 'dev-cas.properties.j2' (j2 indicates a *Jinja2* template file).  We're going to have variables in these files that we are NOT going to want to keep in the properties file - at least not in a git repository.

I recommend one file for each of your environments (dev-cas.properties.j2, test-cas.properties.j2, prod-cas.properties.j2).  All will eventually just be /etc/cas/config/cas.properties on their respective servers though.  It's going to have some variables which we're going to fill in later.

### dev-cas.properties.j2 creation

``` yaml
# Replace this with your public facing cas server name.  If you're using a load balancer
# This will be the virtual host within that load balancer, not the servers being the load balancer.
cas.server.name=https://your-dev-server.domain.edu
cas.server.prefix=${cas.server.name}/cas

logging.config: file:/etc/cas/config/log4j2.xml

# JSON Service Registry
cas.serviceRegistry.json.location=file:/etc/cas/services

cas.tgc.secure:                         true
cas.tgc.crypto.signing.key:             {{ dev_tgc_signing_key }}
cas.tgc.crypto.encryption.key:          {{ dev_tgc_encryption_key }}

cas.webflow.crypto.signing.key:         {{ dev_webflow_signing_key }}
cas.webflow.crypto.encryption.key:      {{ dev_webflow_encryption_key }}

# Default handler - enable only for testing - leave blank (not commented out) to disable
cas.authn.accept.users=YourTestUser::YourTestPassword
```

!!! caution
    Within this file - the 'cas.authn.accept.users' value has been set.  This is for testing only!  It should be removed long before you are ready to go with production!


Within the file - you'll notice a few areas with \{\{ variables_in_curly_brackets \}\}.  These are variables.  Many of them are sensitive values that you don't want unencrypted in a git repository (or not in a git repository at all).

There's two ways to handle this:

* Using Ansible Vault (encrypted files - you're prompted for the encryption password when you run a playbook with the --ask-vault-pass option)
* Using .gitignore files to exclude one or more variable files from your git repo.
You can combine these (i.e. using ansible vault and still using gitignore to not put those files into git.)

## Setting up an Ansible Vault file for sensitive variables

Ansible gives the option of encrypting individual variables in a file, or encrypting the entire file.  I tend to encrypt the entire file.  At my organization there's only two people who need access to these encrypted files, so I don't mind sharing that password between two of us.  For larger groups, there are options on encrypting files with multiple per-user keys (see the References section at the bottom for more tails).  We're going to go with encrypted files with a single key here.

From your CAS role directory:

``` console
ansible-vault create vars/cas-vault.yml
```
You'll be prompted to enter, and re-enter, a password/passphrase.  Choose something good - and store that password somewhere like your password manager.

It will then bring you into your system's default editor (if you're on Linux, vi by default).  You can put placeholders for the variables we need there:

``` yaml
dev_tgc_signing_key: 
dev_tgc_encryption_key: 
dev_webflow_signing_key: 
dev_webflow_encryption_key: 
```

Save the file for now.  You can edit again via the following (you'll be prompted for your password):

``` console
ansible-vault edit vars/cas-vault.yml
```

## Configure ticket granting cookie encryption
The CAS server uses a ticket granting cookie in the browser to maintain login state during single sign-on sessions. A client can present this cookie to CAS in lieu of primary credentials and, provided it is valid, will be authenticated. The contents of the cookie should be encrypted to protect them, and when running in a multi-node environment, all of the nodes must use the same keys.  These are defined in our cas.properties sample above as *cas.tgc.crypto.signing.key* and *cas.tgc.crypto.encryption.key*

### To generate the first key (cas.tgc.crypto.signing.key):

1. Now visit the [JSON Web Key Generator](https://mkjwk.org/).
2. Click on the *oct* tab.
3. Set *Key Size* to 512, *Key Use* to Signature, and *Algorithm* to HS256.
4. Click *Generate*.
5. You'll see a *Shared Key* box.  Find the value for "k" there, and copy it.
6. Enter that as your *dev_tgc_signing_key* in your Ansible vault (or if not using Ansible vault - just put it directly as the value for *cas.tgc.crypto.signing.key*)

### To generate the second key (cas.tgc.crypto.encryption.key):

While still at the [JSON Web Key Generator](https://mkjwk.org/) in the 'oct' tab:

1. Set *Key Size* to 256, and keep the other settings, *Key Use* to Signature, and *Algorithm* to HS256, the same.
2. Click *Generate*.
3. You'll see a *Shared Key* box.  Find the value for "k" there, and copy it.
4. Enter that as your *dev_tgc_encryption_key* in your Ansible vault (or if not using Ansible vault - just put it directly as the value for *cas.tgc.crypto.encryption.key*)

## Configure Spring Webflow encryption
CAS uses Spring Webflow to manage the authentication sequence, and this also needs to be encrypted.

### Generate the first key (cas.webflow.crypto.signing.key):
While still at the [JSON Web Key Generator](https://mkjwk.org/) in the 'oct' tab:

1. Set *Key Size* to 512, and keep the other settings, *Key Use* to Signature, and *Algorithm* to HS256, the same.
2. Click *Generate*.
3. You'll see a *Shared Key* box.  Find the value for "k" there, and copy it.
4. Enter that as your *dev_webflow_signing_key* in your Ansible vault (or if not using Ansible vault - just put it directly as the value for *cas.webflow.crypto.signing.key*)

### Generate the second key (cas.webflow.crypto.encryption.key):
This one is different.  Unlike the ticket granting cookie encryption key above, the encryption key for Spring WebFlow is not a JSON Web Key. Rather, it’s a randomly-generated string of 16 (by default) octets, Base64-encoded. An easy way to generate this key is to use openssl:

``` console
[chauvetp@login6deva tmp]$ openssl rand -base64 16
NwriHAJ1Li2YnKxBbhUEVw==
[chauvetp@login6deva tmp]$ 
```
Enter that as your *dev_webflow_encryption_key* in your Ansible vault (or if not using Ansible vault - just put it directly as the value for *cas.webflow.crypto.encryption.key*)


!!! note ""
    The online JSON Web Key Generator, which point to to generate keys, is provided by the Mitre Corporation and the MIT Kerberos and Internet Trust Consortium, and is simply a web-based interface to the json-web-key-generator project, also provided by Mitre/MIT. The project can be cloned from GitHub and built locally if you don’t trust the online generator, or you can download and use a pre-built copy from the CAS project by running the command:

    ```
    curl -LO https://raw.githubusercontent.com/apereo/cas/master/etc/jwk-gen.jar
    ```

    Keys can then be generated using the command
    ```
    java -jar jwk-gen.jar -t oct -s [size]
    ```

## Save your vault file
That's it for the vault file for now - save it.

## References
* [Encrypting content with Ansible Vault](https://docs.ansible.com/ansible/latest/user_guide/vault.html)
