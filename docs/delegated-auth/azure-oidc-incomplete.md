# Delegated Authentication to Azure AD via OIDC

!!! danger "This is incomplete"
    As of when this was published - I was able to get this MOSTLY working - and this may be good enough for your environment.  What wasn't working for me is that in our environment, all our CAS clients are expecting the username not the full email address or userPrincipalName.  For example, in the UPN username@domain.edu, our CAS clients were expecting 'username' not 'username@newpaltz.edu'.  I could not figure out (at least before getting this working via SAML) how to fix this issue.  Your mileage may vary, so I'm still including this.

## Setup an application in Azure AD
Azure Active Directory needs to have an application registered in order for CAS to delegate authentications to it.  This is all done via the [Azure AD portal](https://aad.portal.azure.com).

1. Go to the [Azure AD portal](https://aad.portal.azure.com).
2. Click *Azure Active Directory* then select *App Registrations*
3. Create a new application via the "New registration" button.
4. Set a Name for the application.  This is user visible (depending on where users look) so I recommend something clear like *"YourSchoolLogin-Dev"*, *"YourSchoolLogin-Test"* or *"YourSchoolLogin"* (for prod).
5. Leave *Support account types* as the default (*Accounts in this organizational directory only*).
6. Set your RedirectURL as follows: https://YourCASDevDomain.domain.edu/cas/login?client_name=<whatever you chose in step 4>.  For example, if I chose NewPaltz-Dev in step 4, my RedirectURL is https://logindev.newpaltz.edu/cas/login?client_name=NewPaltzLogin-Dev then click *Register*.

!!! note "Note regarding application names"
    I'd recommend not having a spaces or crazy special characters in the app name.  I have only tested with alphanumeric characters along with hypnens in the client name.  I don't know what Azure allows - that would cause issues with CAS, or CAS clients, etc, but I haven't had a reason to find out.  Stick with the basics.


## Configure OIDC properties in CAS
Add the following settings to roles/cas6/templates/dev-cas.properties.j2 in the cas-overlay-template directory on your Ansible server.  As with other sections - we're going to use variables here since some of these properties are sensitive, and we'll leave them as variables here (and the real values will be in the Ansible cas-vault.yml file).

cas.authn.pac4j.oidc[0].azure.tenant={{ AZURE_TENANT }}
cas.authn.pac4j.oidc[0].azure.id={{ AZURE_CAS_APP_ID }}
cas.authn.pac4j.oidc[0].azure.secret={{ AZURE_CAS_APP_SECRET }}
cas.authn.pac4j.oidc[0].azure.clientName={{ AZURE_CAS_APP_NAME }}
cas.authn.pac4j.oidc[0].azure.discoveryUri={{ AZURE_CAS_DISCOVERY_URI }}
cas.authn.pac4j.oidc[0].azure.principalAttributeId=upn
cas.authn.pac4j.oidc[0].azure.autoRedirect=false

## Variable setup
Edit your *cas-vault.yml* file within *roles/cas6/vars/*

You'll need to add the following:
AZURE_TENANT: 
AZURE_CAS_APP_ID:
AZURE_CAS_APP_SECRET:
AZURE_CAS_APP_NAME:
AZURE_CAS_DISCOVERY_URI:

### Where to get these values

* AZURE_TENANT: This is the main domain within Azure.  In our case, it is newpaltz.edu.
    * I don't have experience dealing with multiple domains for this.  For our environment, we have some users who use a different domain as their primary email address (like engineering.newpaltz.edu) - but that is only for sending or receiving email.  Those users still login to Office 365 and Azure prompts with their username@newpaltz.edu since that is their userPrincipalName.
* AZURE_CAS_APP_ID: This is the *Application (client) ID* within the Overview of the application in Azure (Azure Active Directory -> App Registrations -> YourApp -> Overview).
* AZURE_CAS_APP_SECRET: To create this - do the following:
    * Go to the app in Azure, then go to *Certificates & secrets*.
    * Click *New client secret*
    * Give it a name for the description field - this is not user visible but make it something clear to you like *CASDevClientSecret*.
    * Choose *Never* for expiration (if you want - choose a 1 year or 2 year expiration - but don't forget to update/renew this in advance....).
    * Click *Add*.
    * You'll see the value we need for *AZURE_CAS_APP_SECRET* in the value field next to the newly created client secret.
* AZURE_CAS_APP_NAME: This is what you chose as the name of the app in step 4 of *Setup an application in Azure AD* above.
* AZURE_CAS_DISCOVERY_URI: To find this - go to the *Overview* tab of your application in Azure, then click the Endpoints button.  This value is what is listed under *OpenID Connect metadata document*.  It starts with https://login.microsoftonline.com and ends with */.well-known/openid-configuration*.


## References
[CAS 6: Delegated Authentication](https://apereo.github.io/cas/6.3.x/integration/Delegate-Authentication.html#delegated-authentication)
[CAS 6: Delegated Authentication with SAML2](https://apereo.github.io/cas/6.3.x/integration/Delegate-Authentication-SAML.html#delegated-authentication-w-saml2)