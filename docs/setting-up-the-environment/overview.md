# System/Architecture Overview

## Components

!!! Summary: Before beginning the CAS build and configuration process, you will want to plan out your server environment.  Below is how we have things setup - but by no means is the only way.

We at New Paltz have a three-tiered CAS consisting of:

* Development: This is used exclusively for deploying/configuring/testing CAS itself - no end users use this.
* Test: This is used for test applications/environments which still need authentication such as Banner Test.  Since there are a number of IT staff and end users testing these applications, changes here can still impact users though far less than production.
* Production: The actual public facing CAS.

Within each tier, you will want one or more servers.  We do something similar to:

* Development
  * CAS6Dev1
  * CAS6Dev2
  * logindev: Public-facing load balanced virtual host
* Test
  * CAS6Test1
  * CAS6Test2
  * logintest: Public-facing load balanced virtual host
* Production
  * CAS6Prod1
  * CAS6Prod2
  * login: Public-facing load balanced virtual host

Each of these servers for us are RHEL 8 hosts, with:

* 2 virtual CPUs for production, 1 for test/dev
* 4 GB memory (though can probably get away with less)
* 25 GB disk

For the official requirements, see the Apereo CAS [Installation Requirements](https://apereo.github.io/cas/6.2.x/planning/Installation-Requirements.html) document.

I will document both how to set these up manually, as well as my Ansible playbooks which handle deployment (of Apache, Apache Tomcat, and CAS itself).