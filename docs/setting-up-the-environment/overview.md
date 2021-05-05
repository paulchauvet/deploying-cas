# System/Architecture Overview

## Components

!!! tip "Summary"
    Before beginning the CAS build and configuration process, you will want to plan out your server environment.  Below is how we have things setup - but by no means is the only way.

We at New Paltz have a three-tiered CAS consisting of:

* Development: This is used exclusively for deploying/configuring/testing CAS itself - no end users use this.
* Test: This is used for test applications/environments which still need authentication such as Banner Test.  Since there are a number of IT staff and end users testing these applications, changes here can still impact users though far less than production.
* Production: The actual public facing CAS.

Within each tier, you will want one or more servers.  We do something similar to the following:

* Development
    * login6deva
    * login6devb
    * logindev: Public-facing load balanced virtual host

* Test
    * login6testa
    * login6testb
    * logintest: Public-facing load balanced virtual host

* Production
    * login6proda
    * login6prodb
    * login6prodc
    * login: Public-facing load balanced virtual host

Each of these servers for us are RHEL 8 hosts, with:

* 2 virtual CPUs for production, 1 for test/dev
* 4 GB memory (though you may need more or less depending on load)
* 25 GB disk

For the official requirements, see the Apereo CAS [Installation Requirements](https://apereo.github.io/cas/6.3.x/planning/Installation-Requirements.html) document.

I will document both how to set these up manually, as well as my Ansible playbooks which handle deployment (of Apache, Apache Tomcat, and CAS itself).

I've put a copy of my ansible playbooks that are used in this document at:
(https://github.com/paulchauvet/ansible-playbooks).