# Configuring AD attribute resolution and release

To enable client applications to obtain information about authenticated users, the CAS server must be configured to resolve attributes and release them to the clients.

Version 3 of the CAS protocol, which was first supported by CAS 4.0, contains native support for returning authentication/user attributes to clients.  Version 2 of the CAS protocol, the version implemented by CAS 3.x, did not support attribute release; the SAML 1.1 protocol was used for that purpose.  Most CAS clients have not yet been updated to support Version 3 of the protocol, so it’s still necessary to configure SAML 1.1-based attribute release.

## Add the SAML 1.1 dependency
To add SAML 1.1 support to the CAS server, edit the {==build.gradle==} file within the cas-overlay-template directory on your build host.  We're going to add a single depency to the one we already added for the json service registry.  See the highlighted line below for the addition.

**cas-overlay-template/build.gradle**
``` json hl_lines="7"
dependencies {
    // Other CAS dependencies/modules may be listed here statically...

    implementation "org.apereo.cas:cas-server-webapp-init:${casServerVersion}"
    implementation "org.apereo.cas:cas-server-support-ldap:${casServerVersion}"
    implementation "org.apereo.cas:cas-server-support-json-service-registry:${casServerVersion}"
    implementation "org.apereo.cas:cas-server-support-saml:${casServerVersion}"

}
```


## Configure attribute resolution

Add the following settings to roles/cas6/templates/dev-cas.properties.j2 in the cas-overlay-template directory on your Ansible server, after the ones we added for active directory authentication earlier.  As with there, we've set some of these as variables which will be updated with those we will store in our encrypted vault.

You'll see both common attributes which are most likely in every AD (givenName, sn for surname, memberOf for groups), as well as schema extensions which you may or may not have like eduPerson, or even some that are specific to your school (for example, we have campusCode).  You'll want to define both a bind user that can look at these attributes, as well as the attributes you will need to release.  Note: just bacause you are giving CAS access to an attribute does not mean that a service will have access to it if you don't specify it.

You can also 'rename' an attribute.  For example, our AD has 'eduPersonTargetedID' but CAS is set to present this as 'UDC_IDENTIFIER' for Banner.

```
# These are for attribute release
cas.authn.attributeRepository.ldap[0].order=0
cas.authn.attributeRepository.ldap[0].ldapUrl=ldaps://{{ AD_SERVER_0 }}
cas.authn.attributeRepository.ldap[0].validatePeriod=270
cas.authn.attributeRepository.ldap[0].searchFilter=cn={user}
cas.authn.attributeRepository.ldap[0].baseDn={{ AD_BASE_DN_0 }}
cas.authn.attributeRepository.ldap[0].bindDn={{ AD_BIND_USER_0 }}
cas.authn.attributeRepository.ldap[0].bindCredential={{ AD_BIND_PASSWORD_0 }}
cas.authn.attributeRepository.ldap[0].attributes.cn=cn
cas.authn.attributeRepository.ldap[0].attributes.displayName=displayName
cas.authn.attributeRepository.ldap[0].attributes.givenName=givenName
cas.authn.attributeRepository.ldap[0].attributes.mail=mail
cas.authn.attributeRepository.ldap[0].attributes.sn=sn
cas.authn.attributeRepository.ldap[0].attributes.memberOf=memberOf
cas.authn.attributeRepository.ldap[0].attributes.campusCode=campusCode
cas.authn.attributeRepository.ldap[0].attributes.eduPersonPrimaryAffiliation=eduPersonPrimaryAffiliation
cas.authn.attributeRepository.ldap[0].attributes.eduPersonPrincipalName=eduPersonPrincipalName
cas.authn.attributeRepository.ldap[0].attributes.eduPersonTargetedID=UDC_IDENTIFIER

cas.authn.attributeRepository.ldap[1].order=1
cas.authn.attributeRepository.ldap[1].ldapUrl=ldaps://{{ AD_SERVER_1 }}
cas.authn.attributeRepository.ldap[1].validatePeriod=270
cas.authn.attributeRepository.ldap[1].searchFilter=cn={user}
cas.authn.attributeRepository.ldap[1].baseDn={{ AD_BASE_DN_1 }}
cas.authn.attributeRepository.ldap[1].bindDn={{ AD_BIND_USER_1 }}
cas.authn.attributeRepository.ldap[1].bindCredential={{ AD_BIND_PASSWORD_1 }}
cas.authn.attributeRepository.ldap[1].attributes.cn=cn
cas.authn.attributeRepository.ldap[1].attributes.displayName=displayName
cas.authn.attributeRepository.ldap[1].attributes.givenName=givenName
cas.authn.attributeRepository.ldap[1].attributes.mail=mail
cas.authn.attributeRepository.ldap[1].attributes.sn=sn
cas.authn.attributeRepository.ldap[1].attributes.memberOf=memberOf
cas.authn.attributeRepository.ldap[1].attributes.campusCode=campusCode
cas.authn.attributeRepository.ldap[1].attributes.eduPersonPrimaryAffiliation=eduPersonPrimaryAffiliation
cas.authn.attributeRepository.ldap[1].attributes.eduPersonPrincipalName=eduPersonPrincipalName
cas.authn.attributeRepository.ldap[1].attributes.eduPersonTargetedID=UDC_IDENTIFIER

```

## Variable setup
Edit your *cas-vault.yml* file within *roles/cas6/vars/*

Fill in values for {{ AD_BIND_USER_0 }} and {{ AD_BIND_PASSWORD_0 }} (and the same for _1, _2, etc. if you have them).  It should now look like the following (after opening with ansible-vault edit)

**roles/cas6/vars/cas-vault.yml**

``` yaml
# As previously mentioned - these are not my real keys
DEV_TGC_SIGNING_KEY: WEebZqgDjfKJei0XM-owdfueb3lZ3lyKKXAL8wLUoLfc2qTFWyBmYxVQBSLslau70uJH_gGM5teTqgbDD3Xcag
DEV_TGC_ENCRYPTION_KEY: zyTzo8eMzToxP9_Kmk33iFVKVFMJD8873ZGA9Z_2Fco
DEV_WEBFLOW_SIGNING_KEY: _mUEdBjyFlfbvKaGPnAtIbQ7sEkMO2A57lCu3OKz835NeNZqcOCsVo6WmCc95TMgdmahP-aP1lXBpqjd4rU2-g
DEV_WEBFLOW_ENCRYPTION_KEY: JnGirTeE8yp3Jp/Mg9Z5Pg==

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
```

!!! note
    Later - this guide will show you how to delegate authentication to Azure.  If you do this - you can still do attribute resolution against your regular (non-Azure) AD.  As of yet I haven't done any attribute resolution against Azure AD so I cannot speak to that.


## Attribute Merging
The CAS 5 guide that this documentation is modeled after references Attribute Merging - where you would find attributes from a single user in multiple directories.  I don't have any experience in doing this - as our users are in either location, not both.  You may want to see David Curry's discussion on this in the References below if you need to use attribute merging.


## References
* [Deploying Apereo CAS 5: Configure an attribute merging strategy](https://dacurry-tns.github.io/deploying-apereo-cas/building_server_ldap_resolution-release_configure-attribute-resolution.html)
* [eduPerson Object Class Specification](https://software.internet2.edu/eduperson/internet2-mace-dir-eduperson-201602.html)
* [CAS 6: Authentication Attributes](https://apereo.github.io/cas/6.3.x/integration/Attribute-Release-Policies.html#authentication-attributes)
* [CAS 6: Attribute Merging Strategies](https://apereo.github.io/cas/6.3.x/integration/Attribute-Release-Caching.html#merging-strategies)