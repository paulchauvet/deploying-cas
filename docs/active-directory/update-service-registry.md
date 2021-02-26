# Update the service registry for attribute release

Attribute release policies are defined on a per-service basis in the service registry.  There are four basic attribute release policies:

| Policy         | Description                           |
| :----------    | :-----------------------------------  |
| Return All     | Return all resolved attributes to the service.|
| Deny All       | Do not return any attributes to the service. This will also prevent the release of the default attribute pool. |
| Return Allowed | Only return the attributes specifically allowed by the policy. This policy includes a list of the attributes to release. |
| Return Mapped  | Only return the attributes specifically allowed by the policy, but also allow them to be renamed at the individual service level. Useful when a particular service insists on having specific attribute names not used by other services. |

The syntax for defining the above policies is defined in the CAS 6 Attribute Release Policies documentation. That document also describes a number of script-based policies that will call a Groovy, JavaScript, or Python script to decide how to release attributes (these policies are beyond the scope of this document).

I've only used "Return All" for testing purposes like we're doing here.  I typically use ReturnAllowedAttributeReleasePolicy but have used ReturnMappedAttributePolicy as well a few times.  I've never had a reason (yet) to use Deny All.  Since I don't use a 'default' attribute release policy (mentioned below) I haven't had a need to.  I prefer everything that is released to be explicit. 

!!! Note
    Note: The cas.authn.attributeRepository.defaultAttributesToRelease property can be set in cas.properties to a comma-separated list of attributes that should be released to all services, without having to list them in every service definition. We are not using this feature in our installation, because it makes it harder to determine which attributes are released to a particular service (by requiring the administrator to look in more than one location).

## Create a Return All service definition
When we initially created our service registry, we used a wildcard.  We explicitly put that wildcard service with a *really* high evaluationOrder (99999), so we could ensure all other policies we put in would be triggered if they are hit first.

We're going to add a new service - with another high evaluation order (95000).

To follow the naming scheme we used [earlier](https://paulchauvet.github.io/deploying-cas/service-config/overview/#create-a-test-service-definition-file), you'll first need a unique ID.  As with last time, its recommended that you use the {==date +%s==} command to get the datetime in unix epoch format.  For my example, I have 1614354496 as that ID.  I'm thus calling my service file *ApacheTestReleaseAll-1614354496.json* and placing it in the roles/cas6/templates/dev-services directory.

You'll need to make sure the serviceID matches the host with your CAS client and the id is updated to match what you have in your file name.  We're using the *ReturnAllAttributeReleasePolicy* here (which again - you may not want or need to use in production).

**roles/cas6/templates/dev-services/ApacheTestReleaseAll-1614354496.json:**
```json
{
  "@class" : "org.apereo.cas.services.RegexRegisteredService",
  "serviceId" : "^https://logindev.domain.edu/return-all(\\z|/.*)",
  "name" : "Apache Test - full attribute release",
  "id" : 1614354496,
  "description" : "CAS development Apache mod_auth_cas server with username/password protection",
  "attributeReleasePolicy" : {
    "@class" : "org.apereo.cas.services.ReturnAllAttributeReleasePolicy"
  },
  "evaluationOrder" : 95000
}
```

This service definition uses a serviceId regular expression that matches only the URL for the return-all directory on the cas client server. The (\\z|/.\*) syntax at the end matches either the empty string (\\z) or a slash (‘/’) followed by anything (/.\*), meaning that the following will match:

``` shell
https://logindev.domain.edu/return-all
https://logindev.domain.edu/return-all/
https://logindev.domain.edu/return-all/index.php
https://logindev.domain.edu/return-all/subdir/file.html
```

but the following will not:

```
https://logindev.domain.edu/return-all-and-something-else
https://logindev.domain.edu/some/completely/unrelated/path
https://logintest.domain.edu/return-all

```

## Create a Return Mapped Attributes service definition
You may want to only return certain attributes.  This is the most common way to use it in production.  Let's say you have an external service that only needs the person's givenName and sn (surname) - why release other attributes that they don't need to them?

As with the last service, get output of {==date +%s==} for the filename and id.  For this example, I've created *ApacheTestReturnMapped-1614355136.json* and placed it in the roles/cas6/templates/dev-services directory.

**roles/cas6/templates/dev-services/ApacheTestReturnMapped-1614355136.json:**

``` json
{
  "@class" : "org.apereo.cas.services.RegexRegisteredService",
  "serviceId" : "^https://logindev.domain.edu/return-mapped(\\z|/.*)",
  "name" : "Return Mapped Test",
  "id" : 1614355136,
  "description" : "Display results of a Return Mapped attribute release policy",
  "attributeReleasePolicy" : {
    "@class" : "org.apereo.cas.services.ReturnMappedAttributeReleasePolicy",
    "allowedAttributes" : {
      "@class" : "java.util.TreeMap",
      "cn" : "cn",
      "displayName" : "fullName",
      "mail" : "EmailAddress",
      "memberOf" : "memberOf",
      "sn" : "sn",
      "uid" : "uid",
      "UDC_IDENTIFIER": "UDC_IDENTIFIER"
    }
  },
  "evaluationOrder" : 90000
}
```
The 'allowedAttributes' section is needed when you use the ReturnMappedAttributeReleasePolicy as shown above, to define which attributes are released.

Feel free to put your own attributes here - just make sure they've been listed in cas.properties.  You can alter them though on a per service basis.  Let's say service provider 'x' wants the displayName attribute as fullName - you can do so as I've done above.


## References
* [CAS 6: Attribute Release Policies](https://apereo.github.io/cas/6.3.x/integration/Attribute-Release-Policies.html#attribute-release-policies)