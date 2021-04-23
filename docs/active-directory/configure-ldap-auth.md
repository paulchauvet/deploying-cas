# Configuring LDAP Authentication

## Add the LDAP dependency
To add LDAP support to the CAS server, edit the {==build.gradle==} file within the cas-overlay-template directory on your build host.  We're going to add a single depency to the one we already added for the json service registry.  See the highlighted line below for the addition.

``` json hl_lines="6"
dependencies {
    // Other CAS dependencies/modules may be listed here...
    // implementation "org.apereo.cas:cas-server-support-json-service-registry:${casServerVersion}"

    implementation "org.apereo.cas:cas-server-webapp-init:${casServerVersion}"
    implementation "org.apereo.cas:cas-server-support-ldap:${casServerVersion}"
    implementation "org.apereo.cas:cas-server-support-json-service-registry:${casServerVersion}"
}


```

## Disable static credentials
You'll want to also remove the static test user you have specified in cas.properties.  In Ansible, this is in roles/cas6/templates/dev-cas-properties.j2

``` yaml
# Default handler - enable only for testing
# leave blank (not commented out) to disable
cas.authn.accept.users=
```

## Rebuild CAS
To rebuild CAS with the newest dependency built in we'll do the same thing we did when adding the json service registry.  You'll want to go to the cas-overlay-template directory and run the following and wait a moment for it to be complete:
```
./gradlew clean build
```

Your cas.war file will have been updated.  Copy it from *cas-overlay-template/build/libs/cas.war* to your *roles/cas6/files* 

## Configure Active Directory/LDAP properties
Although CAS offers several dozen properties for controlling how LDAP authentication is performed, most of them come with reasonable defaults and do not have to be configured in normal circumstances.  The complete list of properties can be found in the CAS documentation.

Add the following settings to roles/cas6/templates/dev-cas.properties.j2 in the cas-overlay-template directory on your Ansible server.

The [0] in the property names indicates that this is the first LDAP source to be configured. Additional sources will be [1], [2], etc.

```
# These are the LDAP server setups
# If you have more than one organizational unit - you can list a second one here with cas.authn.ldap[1].attributes after the first, just cas.authn.ldap[1].order to 1 instead of 0 but it may not be necessary in your environment.  I've included them for example's sake.
cas.authn.ldap[0].order:                0
cas.authn.ldap[0].name:                 Active Directory
cas.authn.ldap[0].type:                 AD
cas.authn.ldap[0].ldapUrl:              ldaps://{{ AD_SERVER_0 }}
cas.authn.ldap[0].validatePeriod:       270
cas.authn.ldap[0].poolPassivator:       NONE
cas.authn.ldap[0].searchFilter:         sAMAccountName={user}
cas.authn.ldap[0].baseDn:               {{ AD_BASE_DN_0 }}
cas.authn.ldap[0].dnFormat:             {{ AD_DN_FORMAT_0 }} 

cas.authn.ldap[1].order:                1
cas.authn.ldap[1].name:                 Active Directory
cas.authn.ldap[1].type:                 AD
cas.authn.ldap[1].ldapUrl:              ldaps://{{ AD_SERVER_1 }}
cas.authn.ldap[1].validatePeriod:       270
cas.authn.ldap[1].poolPassivator:       NONE
cas.authn.ldap[1].searchFilter:         sAMAccountName={user}
cas.authn.ldap[1].baseDn:               {{ AD_BASE_DN_1 }}
cas.authn.ldap[1].dnFormat:             {{ AD_DN_FORMAT_1 }} 

```

### Overview of properties
| Property         | Description                           |
| :----------   | :-----------------------------------  |
| order | When multiple authentication sources are configured, the CAS server looks for the user in one source after another until the user is found, and then the authentication is performed against that source (where it either succeeds or fails). This property influences the order in which the source is evaluated (if not specified, sources are evaluated in the order they are defined).|
| name | The name of the source. This is used when writing log file messages. |
| type | The type of authenticator to use. This should be AD for Active Directory. |
| ldapUrl | The URL of the Active Directory server. In our case, we use the URL of a virtual host on the F5 load balancer, which has multiple Active Directory domain controllers behind it. |
| validatePeriod | The LDAP module periodically validates the connections in its connection pool. But the default setting for how often to do this (600 seconds) is longer than the idle timeout on the F5 load balancer that fronts the LDAP servers (300 seconds), which results in lots of warning messages being written to the CAS log file (one per connection every ten minutes). Reducing the validation period to something shorter than the load balancer idle timeout eliminates these messages. |
| poolPassivator | [Passivators](https://apereo.github.io/cas/5.2.x/installation/Configuration-Properties.html#passivators) help manage LDAP connection pools. However, the default value for this property, BIND, does not work with the AD authenticator type, because there is no bind credential to use (the authenticator binds as the user being authenticated). Therefore, tihs setting is needed to disable the passivator. |
| searchFilter | The LDAP filter to select the user from the directory. Active Directory typically searches on the sAMAccountName attribute. The {user} pattern will be replaced with the username string entered by the user. |
| baseDn | The base DN to search against when retrieving attributes. The “usual” value for this is more like ou=Users,dc=example,dc=org, but for historical reasons we keep our users in a different OU. |
| dnFormat | A format string to generate the user DN to be authenticated. In the string, %s will be replaced with the username entered on the login form. The “usual” value of this string is something more like uid=%s,ou=Users,dc=example,dc=org, but we do not use the uid attribute in our Active Directory schema, we use cn instead. |




## Fill in variables
You'll want to edit your cas-vault.yml file (*ansible-vault edit cas-vault.yml*) - since we're going to want to save some sensitive variables.  When done - it will look something like:

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

# If you have other servers or organizational units, you can list them here
AD_SERVER_1: other-server.domain.edu
AD_BASE_DN_1: dc=subdomain,dc=domain,dc=edu
AD_DN_FORMAT_1: CN=%s,OU=alumni,DC=subdomain,DC=domain,DC=edu
~                                                                       

```

## Rerun the playbook

``` yaml
[chauvetp@ansible templates]$ ansible-playbook ~/ansible/site.yml --ask-vault-pass --limit <your_CAS_server>
Vault password: 
```

As with before - once we rerun this playbook - it will only alter that which we changed, namely the cas.war file, the cas.properties file, and it will restart Tomcat.

## Test logins

You'll want to go to your CAS client page and test logging in with an Active Directory user.  

## References
* [CAS 6: LDAP Authentication](https://apereo.github.io/cas/6.3.x/installation/LDAP-Authentication.html)
* [CAS 6: Configuration Properties](https://apereo.github.io/cas/6.3.x/configuration/Configuration-Properties.html#ldap-authentication)