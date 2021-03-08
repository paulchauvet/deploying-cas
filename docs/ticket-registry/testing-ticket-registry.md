# Testing the ticket registry

There's two ways I like to test the ticket registry:

## Load balancer method
In our load balancer (F5) I make sure that only one of our DEV cas servers are active (for example, server A active, server B inactive).  I then login via the load balanced URL for CAS and make sure I log in (in an incognito or private window) without issue.  Don't log out or close that window.

I then enable the *other* CAS server, and disable the first, then revist the /cas/login page.  If I'm still logged in, with server B active and server A inactive, then I can see that the ticket registry is being shared.  If you're asked to login again, your ticket registry may not be shared.

## Taking a server offline
A bit more drastic - have both CAS servers online but only one active in the load balancer (i.e. server A) and login.  Once you are successful, reactivate the second server in the load balancer, and shutdown Tomcat on the first server.  Verify via /cas/login that you are still logged in.