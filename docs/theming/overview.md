# Theming info coming soon

Note: This is the part I'm outsourcing to someone who actually knows how to design web content - which is not me!  I'll reference her assistance when it's done.  The information below, so far at least, is based on what I used to get CAS ready for theming.


!!! note "Optional Content"
    This is all optional - especially if you're doing a redirection to a delegated external identity provider as we are planning to do with Azure & setting **cas.authn.pac4j.saml[0].autoRedirect=true**.  In theory then - the 'default' CAS page will only flash by before a user is redirected to the Azure login page.  But - there are some scenarios where that is not good enough.  For example - if the user is restricted from accessing the service via CAS service restrictions - then CAS - not Azure - will provide the error.  For this - I still prefer to theme the site.


## Start with the example theme

My intention in adding this was that, in theory, an example theme would be added for structure.  That didn't happen so this *may* not be needed.  What I did instead was the following:

**cas-overlay-template/build.gradle**
``` json hl_lines="11"
dependencies {
    // CAS dependencies/modules may be listed here statically...

    implementation "org.apereo.cas:cas-server-webapp-init:${casServerVersion}"
    implementation "org.apereo.cas:cas-server-support-json-service-registry:${casServerVersion}"
    implementation "org.apereo.cas:cas-server-support-ldap:${casServerVersion}"
    implementation "org.apereo.cas:cas-server-support-saml:${casServerVersion}"
    implementation "org.apereo.cas:cas-server-support-duo:${casServerVersion}"
    implementation "org.apereo.cas:cas-server-support-hazelcast-ticket-registry:${casServerVersion}"
    implementation "org.apereo.cas:cas-server-support-pac4j-webflow:${casServerVersion}"
    implementation "org.apereo.cas:cas-server-support-themes-collection:${casServerVersion}"

}
```

## Getting an example theme created

* Within the *cas-overlay-template/src* directory - I created a set of sub directories (to match the 'example' theme at (https://github.com/apereo/cas/tree/6.3.x/support/cas-server-support-themes-collection/src/main/resources/static/themes/example)):
    * **cas-overlay-template/src/main/resources/static/themes/newpaltz**
    * **cas-overlay-template/src/main/resources/static/themes/newpaltz/css**
    * **cas-overlay-template/src/main/resources/static/themes/newpaltz/images**
    * **cas-overlay-template/src/main/resources/static/themes/newpaltz/jss**
* Within *cas-overlay-template/src/main/resources/* I downloaded [example.properties](https://github.com/apereo/cas/tree/6.3.x/support/cas-server-support-themes-collection/src/main/resources) but renamed it as *newpaltz.properties*.
* I then added the files from (https://github.com/apereo/cas/tree/6.3.x/support/cas-server-support-themes-collection/src/main/resources/static/themes/example) to these directories as a starting point - just to make sure these themes were even working.
* I edited *newpaltz.properties* to reflect that it was the 'newpaltz' teheme instead of the *example* theme.

``` json
cas.theme.name=newpaltz
cas.theme.description=New Paltz Theme

cas.standard.css.file=/themes/newpaltz/css/cas.css
cas.standard.js.file=/themes/newpaltz/js/cas.js
cas.logo.file=/themes/newpaltz/images/logo.png
cas.favicon.file=/themes/newpaltz/images/favicon.ico

cas.drawer-menu.enabled=false
cas.notifications-menu.enabled=false
```

## Rebuild CAS
To rebuild CAS with the newest dependency *AND* theme built in we'll do the same thing we did with previous additions.  You'll want to go to the cas-overlay-template directory and run the following and wait a moment for it to be complete:
```
./gradlew clean build
```

Your cas.war file will have been updated.  Copy it from *cas-overlay-template/build/libs/cas.war* to your *roles/cas6/files* 

## Edit one or more services (to start with) to point to this theme

I chose to use one of our test services - the one that returns all attributes - for this purpose.  I added the 'theme' line below.  Make your edits to the Ansible definition of the service.

``` json hl_lines="11"
{
    "@class" : "org.apereo.cas.services.RegexRegisteredService",
    "serviceId" : "^https://logindev.newpaltz.edu/return-all(\\z|/.*)",
    "name" : "Apache Test - full attribute release",
    "id" : 1614354496,
    "description" : "Apache Test - full attribute release",
    "attributeReleasePolicy" : {
      "@class" : "org.apereo.cas.services.ReturnAllAttributeReleasePolicy"
    },
    "evaluationOrder" : 95000,
    "theme": "newpaltz"
}
```

## Rerun the playbook

``` yaml
[chauvetp@ansible templates]$ ansible-playbook ~/ansible/site.yml --ask-vault-pass --limit <your_CAS_server>
Vault password: 
```

## Test

If this works - you'll see a few differences - but only if you go to the service that you changed.

* The *logo.png* file from the git repo is there now instead of the CAS logo
* Slight color scheme differences.
* Due to *cas.drawer-menu.enabled* and *cas.notifications-menu.enabled=false* you'll see the menu drawer and notification button gone.

### Before
<figure>
  <img src="/images/cas-default-theme.png" alt="Screenshot showing the default CAS theme with the drawer and notifications buttons which are NOT in the new theme."/>
</figure>

### After
<figure>
  <img src="/images/cas-example-theme.png" alt="Screenshot showing our new example theme - without the notifications and drawer buttons, the test logo, and new color scheme."/>
</figure>

## What's next?
Well this is the part I can't do but I may be able to fill in more on soon.  I'm going to do the following:

* Make sure only one *DEV* tier host is active in our load balancer.
* Shutdown Tomcat on that host.
* Delete cas.war (leaving the unpacked cas directory within Tomcat's webapps directory).
* Give access to our web designer/developer to the CAS Dev site and have her work her magic.  I will ask her to document any files she edits or new files she adds.
* As she develops - she'll have to restart tomcat to make any changes show (I'll give her sudo access to do so).
* When she's done - I can then place all her shared files in the appropriate locations within *cas-overlay-template/src/main/resources/static/themes/newpaltz* and rebuild CAS.  Those changes will now end up in all of the servers.
* Once the theme is 'done' we can make it the default theme (not just by setting it per-service) by altering cas.properties to include the following lines:
    * cas.theme.paramName=newpaltz
    * cas.theme.defaultThemeName=newpaltz


## References

[Apereo CAS 6.3 User Interface Customization](https://apereo.github.io/cas/6.3.x/ux/User-Interface-Customization-Themes.html)
