# Delegated Authentication to Azure AD

!!! note "Optional Content"
    This is all optional.  If you don't need to delegate authentication to Azure and all of your authentication will be against an on-prem Active Directory or other LDAP/DB, then you can skip this section.

## Setup an application in Azure AD
Azure Active Directory needs to have an application registered in order for CAS to delegate authentications to it.  This is all done via the [Azure AD portal](https://aad.portal.azure.com).

1. Go to the [Azure AD portal](https://aad.portal.azure.com).
2. Click *Enterprise Applications*.
3. At the top, click *Create your own application*
4. Select *Integrate any other application you don't find in the gallery (Non-gallery)*, then give your app a name.  This name is, at least by default, user visible (depending on where users look) so I recommend something clear like *"YourSchoolLogin-Dev"*, *"YourSchoolLogin-Test"* or *"YourSchoolLogin"* (for prod).
5. Click Users and groups on the left.  Use this section to add any users or groups who need access - though you may want to start with one or more test users.
6. Click *Single sign-on* on the left, then SAML.
7. Copy the *App Federation Metadata URL* - you'll need this for the DEV_AZURE_METADATA_PATH later.

We're not done here yet - but need to do a few other things before we can finish - notably setting up the CAS side and the metadata there.

!!! note "Note regarding application names"
    I'd recommend not having a spaces or crazy special characters in the app name.  I have only tested with alphanumeric characters along with hypnens in the client name.  I don't know what Azure would allow, but would cause issues with CAS, or CAS clients, etc, but I haven't had a reason to find out.  Stick with the basics.


## Configure SAML properties in CAS
Add the following settings to roles/cas6/templates/dev-cas.properties.j2 in the cas-overlay-template directory on your Ansible server.  As with other sections - we're going to use variables here since some of these properties are sensitive, and we'll leave them as variables here (and the real values will be in the Ansible cas-vault.yml file).

### Azure SAML settings
**roles/cas6/templates/dev-cas-properties.j2:**
``` json
cas.authn.pac4j.saml[0].keystorePassword={{ DEV_KEYSTORE_PASSWORD }}
cas.authn.pac4j.saml[0].privateKeyPassword={{ DEV_SAML_KEY_PASSWORD }}
cas.authn.pac4j.saml[0].keystorePath=/etc/cas/config/samlKeystore.jks
cas.authn.pac4j.saml[0].serviceProviderEntityId={{ DEV_SAML_ENTITY_ID }}
cas.authn.pac4j.saml[0].serviceProviderMetadataPath=/etc/cas/config/sp-metadata.xml
cas.authn.pac4j.saml[0].identityProviderMetadataPath={{ DEV_AZURE_METADATA_PATH }}
cas.authn.pac4j.saml[0].clientName={{ DEV_AZURE_APP_NAME }}
cas.authn.pac4j.saml[0].use-name-qualifier=false
```

### Variable setup
Edit your *cas-vault.yml* file within *roles/cas6/vars/*

You'll need to add the following:

``` json
DEV_KEYSTORE_PASSWORD: SomeStrongPasswordImSure
DEV_SAML_KEY_PASSWORD: SomeOtherStrongPassword
DEV_SAML_ENTITY_ID: urn:mace:saml:pac4j:org
DEV_AZURE_APP_NAME: Your School-Dev
DEV_AZURE_METADATA_PATH: https://login.microsoftonline.com/<some-tenant-specific-info>/federationmetadata/2007-06/federationmetadata.xml?appid=<some-app-specific-info>
```


#### Where to get these values

* DEV_KEYSTORE_PASSWORD: This is a password you choose to protect your SAML keystore
* DEV_SAML_KEY_PASSWORD: This is a password you choose to protect your SAML private key
* DEV_SAML_ENTITY_ID: This is unique and cannot be reused in more than one app in Azure.  The 'sample' entity id is *urn:mace:saml:pac4j.org*.  You may want to use something like *urn:mace:saml:dev.yourdomain.edu*.
* DEV_AZURE_APP_NAME: This is the name you gave the application in step 4 of *Setup an application in Azure AD*.
* DEV_AZURE_METADATA_PATH: This is from step 7 of *Setup an application in Azure AD*.

## Deploy to ONLY ONE CAS server
You need to deploy this to one CAS server only to start.  In order for the files below to be created (if they don't exist already) you have to start CAS - and then visit the CAS page in a browser.  If you deploy to more than one it's not a huge deal - but you'll want to make sure the files below are the same in each tier (i.e. all of your prod tier has the same files, all of your dev tier has the same files, etc.).

* samlKeystore.jks
* sp-metadata.xml
* saml-signing-cert-<YOUR-APP-NAME>.crt
* saml-signing-cert-<YOUR-APP-NAME>.pem
* saml-signing-cert-<YOUR-APP-NAME>.key

## Distributing these files via Ansible

Optional of course - but if you don't do it via Ansible you'll have to make sure some other way that these are the same on a tier.

### Encrypt the key/pem/keystore files

* Copy the five files listed above to your ansible/roles/cas6/files directory
* rename samlKeystore.jks to samlKeystore-DEV.jks (*or some other indication that it's the dev store*)
* rename sp-metadata.xml to sp-metadata-DEV.xml (*or some other indication that it's the dev store*)
* In that directory type the following:

``` shell
ansible-vault encrypt saml-signing-cert-YourSchoolLogin-DEV.key saml-signing-cert-YourSchoolLogin-DEV.pem samlKeystore-DEV.jks
```

### Create an Ansible task to push those files out

First update main.yml and include a reference to a new sub-task:

**roles/cas6/tasks/main.yml:**
``` yaml hl_lines="5"
- include_tasks: debug.yml
- include_tasks: base-cas-config.yml
- include_tasks: cas-ajp-proxy.yml
- include_tasks: service-config.yml
- include_tasks: push-saml-files.yml
```

**roles/cas6/tasks/push-saml-files.yml
``` yaml

- include_vars: cas-vault.yml

# Note: These are for dev specifically.  If we have multiple environments, there's
# will be different keystores, certificates, metadata, etc., for each host.
- name: Ensure saml-signing-cert .crt file (DEV) is up-to-date
  ansible.builtin.copy:
    src: saml-signing-cert-YourSchoolLogin-DEV.crt
    dest: /etc/cas/config/saml-signing-cert-YourSchoolLogin-DEV.crt
    mode: 0600
    owner: tomcat
    group: tomcat
  when: ("login6dev" in inventory_hostname)
  notify: restart tomcat

- name: Ensure saml-signing-cert .key file (DEV) is up-to-date
  ansible.builtin.copy:
    src: saml-signing-cert-YourSchoolLogin-DEV.key
    dest: /etc/cas/config/saml-signing-cert-YourSchoolLogin-DEV.key
    mode: 0600
    owner: tomcat
    group: tomcat
  when: ("login6dev" in inventory_hostname)
  notify: restart tomcat

- name: Ensure saml-signing-cert .pem file (DEV) is up-to-date
  ansible.builtin.copy:
    src: saml-signing-cert-YourSchoolLogin-DEV.pem
    dest: /etc/cas/config/saml-signing-cert-YourSchoolLogin-DEV.pem
    mode: 0600
    owner: tomcat
    group: tomcat
  when: ("login6dev" in inventory_hostname)
  notify: restart tomcat

- name: Ensure sp-metadata.xml (DEV) is up-to-date
  ansible.builtin.copy:
    src: sp-metadata-DEV.xml
    dest: /etc/cas/config/sp-metadata.xml
    mode: 0600
    owner: tomcat
    group: tomcat
  when: ("login6dev" in inventory_hostname)
  notify: restart tomcat

- name: Ensure samlKeystore.jks (DEV) is up-to-date
  ansible.builtin.copy:
    src: samlKeystore-DEV.jks
    dest: /etc/cas/config/samlKeystore.jks
    mode: 0600
    owner: tomcat
    group: tomcat
  when: ("login6dev" in inventory_hostname)
  notify: restart tomcat
``` 

## Rerun the playbook

``` yaml
[chauvetp@ansible templates]$ ansible-playbook ~/ansible/site.yml --ask-vault-pass --limit <your_CAS_server>
Vault password: 
```


## Finish Azure config
There's a few remaining things to do in Azure.

### Upload CAS metadata to Azure

* Have the sp-metadata.xml file available from CAS then go back to the application you created earlier in Azure (Azure AAD -> Enterprise Applications -> Search for your app)
* Go to the Single sign-on tab
* Click *Upload metadata file* near the top and navigate to the sp-metadata.xml file then click *Add* then *Save*

### Transform username
!!! note "Environment specific"

    This is highly dependent on your own environment!  In my environment this takes my userPrincipalName (chauvetp@newpaltz.edu) and transforms it to just chauvetp (ExtractMailPrefix takes the part before the @ sign).

If your environment is like ours (and it may not be!) - your cas applications want to see a person's username - not their full email address or userPrincipalName as the 'cas user'.  You can do a transformation within Azure to make this happen.

* Go to the Single Sign On section of your app in Azure again (Azure AD -> Enterprise Applications -> Your application).
* In section 2, *User Attributes & Claims*, click *Edit*.
* In the Required claim section, click on the existing claim.
* Change *Name identifier format* from *Email address* (which is the default in Azure) to just *default*
* Change *Source* from *Attribute* to *Transformation*
* Set *Transformation* to *ExtractMailPrefix()*
* Set *Parameter 1* to *user.userprincipalname*
* Click *Add* at the bottom, then *Save*


## References
[CAS 6: Delegated Authentication](https://apereo.github.io/cas/6.3.x/integration/Delegate-Authentication.html#delegated-authentication)
[CAS 6: Delegated Authentication with SAML2](https://apereo.github.io/cas/6.3.x/integration/Delegate-Authentication-SAML.html#delegated-authentication-w-saml2)
[Encrypting content with Ansible vault](https://docs.ansible.com/ansible/latest/user_guide/vault.html#encrypting-files-with-ansible-vault)