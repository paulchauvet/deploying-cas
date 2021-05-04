# Testing Delegated Authentication

To test this - go to your dev CAS page.  You should (if you still have AD logins enabled) see two sections for logins (for now at least) - one for the local AD auth, one for Azure, as in the screenshot below.  Click on the button under "*External Identity Providers*".  The name on the button is what you called the app in Azure.

<figure>
  <img src="/images/azure-delegated-1.png" alt="Screenshot showing CAS page with two login sources, one for local AD, and one for Azure AD."/>
</figure>

When you click on the external identity provider, you'll be redirected to your Azure login page.  Sign in as normal there.

<figure>
  <img src="/images/azure-delegated-2.png" alt="Screenshot showing Azure login page"/>
</figure>


## Application Consent
If you have Admin Consent enabled in Azure - you'll see a page saying approval is required by an admin.  You will have to make the request and have an admin approve the account in Azure (unless you're logging in now via an account that is already an admin in Azure).

<figure>
  <img src="/images/azure-delegated-3.png" alt="Screenshot showing Azure admin consent request page"/>
</figure>

Your Azure Admin(s) will get an email asking for approval, or they can go to [Azure Active Directory](https://aad.portal.azure.com), click on "Enterprise Applications" then "Admin consent requests".  The only permission requested should be "*Sign in and read user profile*".  The admin can approve.

<figure>
  <img src="/images/azure-delegated-4.png" alt="Screenshot showing Azure admin consent approval page"/>
</figure>

If you want to restrict by users or groups, you can go:

* Go to Azure AD portal -> Enterprise Applications -> and search for your app.
* Go to *Properties*
* Make sure "User assignment required* is set to yes.
* Go to *Users and groups*
* Add your users or access groups (i.e. if you have groups for faculty, staff, students, alumni, etc., add them here)


## Issues experienced
I had a heck of a time getting this working.  I want to mention a couple things that are potential issues.  This is by no means comprehensive, but they're things I ran into.

* make sure **cas.server.name** in your cas properties is set to your public facing URL (i.e. if your CAS servers are behind a load balancer, make sure it is the load balancer's virtual host URL, not the actual server name).  If it isn't - you'll have issues.
* At least when you're testing this with Azure - you'll need a real commercial cert.  I'm using Let's Encrypt for these when possible so I don't have to pay for certs (at least in DEV).