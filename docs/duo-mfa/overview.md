# MFA with Duo

At New Paltz - we have Duo deployed to all students, faculty, staff, and affiliates.  It is in use on any service at the CAS level that can be used to access sensitive data, or if there are other risks or compromised.

Other types of MFA can be used outside of Duo of course - but I'm only focusing on Duo in this document.  For more information on Duo - see their website, [duo.com](https://duo.com).

I'll go over how to add Duo as a dependency, configure it in CAS (at the server-wide level, and at the service level) and within the Duo admin panel.

!!! note "Note regarding Duo Universal Prompt"
    Duo is changing over to a new login system for web apps that they call the "Universal Prompt".  Instead of the old way, where CAS (or other sites) would frame a Duo page, with the Universal Prompt, this is handled via browser redirects to a page on Duo's servers.  This is supposedly ready in CAS according to the documents I've seen on the Apereo side (https://apereo.github.io/cas/6.3.x/mfa/DuoSecurity-Authentication.html#universal-prompt) but I haven't yet been able to get this working.  I hope to get back to that and will update this documentation when I have that ready.


## Add the duo-mfa dependency
To add duo-mfa support to the CAS server, edit the {==build.gradle==} file within the cas-overlay-template directory on your build host.  We're going to add a single depency to the one we already added for the json service registry.  See the highlighted line below for the addition.

**cas-overlay-template/build.gradle**
``` json hl_lines="8"
dependencies {
    // Other CAS dependencies/modules may be listed here...
    // implementation "org.apereo.cas:cas-server-support-json-service-registry:${casServerVersion}"

    implementation "org.apereo.cas:cas-server-webapp-init:${casServerVersion}"
    implementation "org.apereo.cas:cas-server-support-json-service-registry:${casServerVersion}"
    implementation "org.apereo.cas:cas-server-support-ldap:${casServerVersion}"
    implementation "org.apereo.cas:cas-server-support-saml:${casServerVersion}"
    implementation "org.apereo.cas:cas-server-support-duo:${casServerVersion}"
  
}
```


## Rebuild CAS
To rebuild CAS with the newest dependency built in we'll do the same thing we did when adding the json service registry.  You'll want to go to the cas-overlay-template directory and run the following and wait a moment for it to be complete:
```
./gradlew clean build
```

Your cas.war file will have been updated.  Copy it from *cas-overlay-template/build/libs/cas.war* to your *roles/cas6/files* 

!!! note
    You may or may not care - but this is the point I gave up on CAS 6.2 in favor of 6.3.  I could not get CAS 6.2 to build the Duo MFA module - it kept complaining about dependency issues, so I moved over to 6.3 without any issues.

## Create a new Duo protected application
This needs to be done in the Duo Administrator Console

1. Login to the [Duo Admin Console](https://admin.duosecurity.com/)
2. Go to *Applications* on the left then the *Protect an application* button
3. Type *CAS* in the search box, then click *Protect* next to **CAS (Central Authentication Service)**
4. Scroll down to the Settings section and choose an application name.  This will be visible to your users when they get a push notification, so at least for your production CAS/Duo - you should choose something user friendly like "*YourSchool Login (DEV)*".
5. Make any other changes you need for it (for example, if you want users to be able to add or remove devices after they've verified with an existing device, check the "Let users remove devices, add new devices, and reactivate Duo Mobile" check box).
6. Click Save.

The three values (API hostname, secret key, and integration key) will be needed shortly.

## Create Duo application key
The duoApplicationKey value is a string, at least 40 characters long, that is generated locally and is not shared with Duo. The CAS documentation offers the procedure below for generating this string.  Obviously - you should generate your own and don't share it.

``` shell
[root@login6deva ~]# python3
Python 3.6.8 (default, Aug 18 2020, 08:33:21) 
[GCC 8.3.1 20191121 (Red Hat 8.3.1-5)] on linux
Type "help", "copyright", "credits" or "license" for more information.
>>> import os, hashlib
>>> akey = hashlib.sha1(os.urandom(32)).hexdigest()
>>> print(akey)
9941db238a6e75ec188fd12fcf6813191894f7c1
>>> exit()
```

## Configure Duo properties in CAS
Add the following settings to roles/cas6/templates/dev-cas.properties.j2 in the cas-overlay-template directory on your Ansible server.  As with other sections - we're going to use variables here since some of the Duo info is sensitive, and we'll leave them as variables here (and the real values will be in the Ansible cas-vault.yml file).

```
cas.authn.mfa.duo[0].duoApiHost:        {{ DEV_DUO_API_HOST }}
cas.authn.mfa.duo[0].duoIntegrationKey: {{ DEV_DUO_IKEY }}
cas.authn.mfa.duo[0].duoSecretKey:      {{ DEV_DUO_SKEY }}
cas.authn.mfa.duo[0].duoApplicationKey: {{ DEV_DUO_AKEY }}

```

## Variable setup
Edit your *cas-vault.yml* file within *roles/cas6/vars/*

Fill in values for {{ DEV_DUO_API_HOST }}, {{ DEV_DUO_IKEY }}, {{ DEV_DUO_SKEY }}, and {{ DEV_DUO_AKEY }}.  It should now look like the following (after opening with ansible-vault edit cas-vault.yml).

**roles/cas6/vars/cas-vault.yml**

``` yaml
# As previously mentioned - these are not my real keys
dev_tgc_signing_key: WEebZqgDjfKJei0XM-owdfueb3lZ3lyKKXAL8wLUoLfc2qTFWyBmYxVQBSLslau70uJH_gGM5teTqgbDD3Xcag
dev_tgc_encryption_key: zyTzo8eMzToxP9_Kmk33iFVKVFMJD8873ZGA9Z_2Fco
dev_webflow_signing_key: _mUEdBjyFlfbvKaGPnAtIbQ7sEkMO2A57lCu3OKz835NeNZqcOCsVo6WmCc95TMgdmahP-aP1lXBpqjd4rU2-g
dev_webflow_encryption_key: JnGirTeE8yp3Jp/Mg9Z5Pg==

# You will have to change these for your own environment
AD_SERVER_0: server.domain.edu
AD_BASE_DN_0: dc=subdomain,dc=domain,dc=edu
AD_DN_FORMAT_0: CN=%s,OU=active,DC=subdomain,DC=domain,DC=edu
AD_BIND_USER_0: CN=bindusername,CN=users,DC=subdomain,DC=domain,DC=edu
AD_BIND_PASSWORD_0: SomeStrongPasswordIAssume

# If you have other servers or organizational units, you can list them here
AD_SERVER_1: other-server.domain.edu
AD_BASE_DN_1: dc=subdomain,dc=domain,dc=edu
AD_DN_FORMAT_1: CN=%s,OU=alumni,DC=subdomain,DC=domain,DC=edu
AD_BIND_USER_1: CN=bindusername,CN=users,DC=subdomain,DC=domain,DC=edu
AD_BIND_PASSWORD_1: SomeStrongPasswordIAssume

# Duo
DEV_DUO_API_HOST: # Your API Hostname from your application in the Duo Admin panel
DEV_DUO_IKEY: # Your integration key from your application in the Duo Admin panel
DEV_DUO_SKEY: # Your secret key from your application in the Duo Admin panel
DEV_DUO_AKEY: # Your integration key from your application in the Duo Admin panel
```



## References
[Duo & CAS](https://duo.com/docs/cas)
[CAS 6: Multifactor Authentication](https://apereo.github.io/cas/6.3.x/mfa/Configuring-Multifactor-Authentication.html)
[CAS 6: Getting Started - MFA via Duo](https://fawnoos.com/2020/11/09/cas63-gettingstarted-overlay/)