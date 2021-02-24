# System/Architecture Overview

## Components

This document was created to reflect the environment in use at New Paltz for CAS.

* [Apereo CAS 6.2.x](https://apereo.github.io/cas/6.2.x/index.html)
* [Red Hat Enterprise Linux 8](https://redhat.com)
* [Apache Tomcat 9.0.x](https://tomcat.apache.org)
* [Apache httpd](https://httpd.apache.com)
* [Ansible](https://docs.ansible.com)
* [Azure Active Directory](https://azure.microsoft.com/en-us/services/active-directory/)
* Microsoft Active Directory
* [Duo](https://duo.com) for Multifactor authentication
* [Hazelcast Ticket Registry](https://apereo.github.io/cas/6.2.x/ticketing/Hazelcast-Ticket-Registry.html) for ticket storage between systems.

If you want to use another version of CAS 6.x - it may or may not have significant changes.  I'll be moving from CAS 6.2 to 6.3 and will document any issues as I come across them, at least in our environment.

If you are going to use another operating system, web server, or java servlet container, I'd imagine the CAS portion of the instructions will be relatively similar - though the Ansible deployments for Tomcat may be less similar or useful.

## Design
We have three levels of CAS here at New Paltz, Production, Test, and Development.  Test is used for non-production applications that still need SSO (for example Banner test).  Development is exclusively used for building, and testing, new versions or configuration changes for CAS.

Authentication will be against either:

* On-prem Active Directory (for alumni)
* Azure Active Directory (for active faculty/staff/students, and retirees)
   * (we may eventually have all alumni in Azure - but that's for a later date.  For now - they are only on-prem and really are only kept active for a couple systems)

In each case - the hosts sit behind a load balancer (in our case, F5 Big IP, though HA Proxy or basically any other load balancer should work.  We're not doing anything crazy at the LB level).

The load balancer is split in multiple locations, as are the application servers.  This gives us resilience if a single site on-campus is down but does NOT give resilience against a full outage on-campus.  This is one reason why we are looking to have as many services as possible authenticate directly against Azure instead of CAS.

Using CAS to delegate authentication to Azure, we can:

* make this transition as streamlined as possible (and avoiding the need for people to have to login twice through two separate SSO systems)
* will let us make the transition of existing CAS apps to SAML on Azure (if they support SAML) gradual, so we don't have to change 30 services (involving both internal and external service providers) over a weekend.
* and will let us keep applications that only support CAS on CAS.
* centralize logins for almost all college systems in a single place making incident response, and troubleshooting, easier.

Limitations:

* Any application still authenticating against CAS will still have CAS as a point of failure.  You can reduce this by having CAS load balanced and in multiple sites (on-prem, off-site, in the cloud, etc.).
* Since Azure will see CAS as a single application, you are left to either:
   * exclude CAS from Azure conditional access policies and use CAS to implement MFA
   * include CAS from Azure conditional access policies but there are no 'exceptions' (i.e. that one service that really doesn't do anything sensitive or important and you don't want to turn MFA on for will still have it)
