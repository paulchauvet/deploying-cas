# Adding Active Directory support

If you've gotten this far - you were probably able to login to your CAS server using the static credentials (**cas.authn.accept.users**).  That's really only useful for testing.  If you're like most - you're going to want to authenticate against a local and/or cloud identity repository.

To start with - we're going to do LDAP (we'll do delegated auth to Azure later).  These instructions provided here work with Microsoft Active Directory in our environment, but I cannot guarantee how different or similar they will be for other LDAP deployments.

In this section, we will add LDAP support to the CAS server to enable it to do three things:

1. Authentication.  Prompt the user for their username and password, and validate that the provided password is indeed correct. At this stage, the user account is also checked to ensure that it is not disabled or expired.  At the conclusion of the authentication process, the CAS server will have identified a security principal. A CAS principal contains a unique identifier by which the authenticated user will be known to all requesting services. A principal also contains optional attributes that may be released to services to support authorization and personalization.
2. Attribute resolution.  Specific attributes about the principal are collected from one or more sources and combined into a single set of attributes using any of several different combining strategies (merging, replacing, adding, etc.).
3. Attribute release.  The process of defining how attributes are selected and provided to a given application in the final CAS response.


## References

* [CAS 6: Configuring Authentication Components](https://apereo.github.io/cas/6.3.x/installation/Configuring-Authentication-Components.html)
* [CAS 6: Configuring Principal Resolution](https://apereo.github.io/cas/6.3.x/installation/Configuring-Principal-Resolution.html)
* [CAS 6: Attribute Resolution](https://apereo.github.io/cas/6.3.x/integration/Attribute-Resolution.html)
* [CAS 6: Attribute Release](https://apereo.github.io/cas/6.3.x/integration/Attribute-Release.html)