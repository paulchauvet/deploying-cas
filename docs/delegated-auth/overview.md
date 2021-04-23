
# Delegated Authentication to Azure AD

!!! note "Optional Content"
    This is all optional.  If you don't need to delegate authentication to Azure and all of your authentication will be against an on-prem Active Directory or other LDAP/DB, then you can skip this section.

As previously mentioned in the introduction, our goal here is to move our authentication from Azure AD to CAS.  By delegating authentication from CAS to Azure, a user only has to login via one 'login' screen, notably Azure.  This is great if you have apps which:

* only work with CAS (and not Azure)
* or you haven't had a chance to move over from CAS to Azure yet

All will still work for those services.  It also lets you gradually move services from CAS to Azure AD without having to move everything at once, or make your users login via multiple systems.  What we want to avoid is a user going to our portal (and authenticating via CAS) and clicking a link to a service on Azure AD and having to login a second time (or vice-versa).

There are two ways that I'm aware of to make this happen:

* OIDC (OpenID Connect)
* SAML (Security Assertion Markup Language)

I've created documents for both in this section.  Getting this all working was a pain in the neck (due to either insufficiently detailed documentation, my lack of understanding of that documentation, or some combination of both!).  I ended up getting the OIDC method mostly working, but had issues with usernames (where Azure would send over the userPrincipalName, not just the CN).  I was able to deal with this with SAML so I'm proceeding with that method (and I'm more experience with SAML than OIDC).  I'll leave both documents up though - but only the SAML method will be complete.

The steps below are the same for both methods.

## Add the pac4j dependency
To add delegated SAML support to the CAS server, edit the {==build.gradle==} file within the cas-overlay-template directory on your build host.  We're going to add a single depency for the pac4j-webflow to the one we already added for the json service registry.  See the highlighted line below for the addition.


**cas-overlay-template/build.gradle**
``` json hl_lines="10"
dependencies {
    // CAS dependencies/modules may be listed here statically...

    implementation "org.apereo.cas:cas-server-webapp-init:${casServerVersion}"
    implementation "org.apereo.cas:cas-server-support-json-service-registry:${casServerVersion}"
    implementation "org.apereo.cas:cas-server-support-ldap:${casServerVersion}"
    implementation "org.apereo.cas:cas-server-support-saml:${casServerVersion}"
    implementation "org.apereo.cas:cas-server-support-duo:${casServerVersion}"
    implementation "org.apereo.cas:cas-server-support-hazelcast-ticket-registry:${casServerVersion}"
    implementation "org.apereo.cas:cas-server-support-pac4j-webflow:${casServerVersion}"
}
```


## Rebuild CAS
To rebuild CAS with the newest dependency built in we'll do the same thing we did with previous additions.  You'll want to go to the cas-overlay-template directory and run the following and wait a moment for it to be complete:
```
./gradlew clean build
```

Your cas.war file will have been updated.  Copy it from *cas-overlay-template/build/libs/cas.war* to your *roles/cas6/files* 
