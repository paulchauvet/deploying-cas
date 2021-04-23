# Building the CAS server

!!! note ""
    If you've followed the other steps, then you'll already have one or more Tomcat servers ready to go.

## Overview
We will be building from source, using the Gradle WAR overlay method.  As per Misagh Moayyed's [Getting Started](https://fawnoos.com/2020/11/09/cas63-gettingstarted-overlay/) document:

!!! quote "CAS 6.0.x Deployment - WAR Overlays"
    Overlays are a strategy to combat repetitive code and/or resources. Rather than downloading the CAS codebase and building it from source, overlays allow you to download a pre-built vanilla CAS web application server provided by the project itself, override/insert specific behavior into it and then merge it all back together to produce the final (web application) artifact.

If you've used CAS 5.x, you may have used the Maven WAR overlay template instead, but as of CAS 6 this was deprecated and the Gradle WAR overlay method is recommended instead.

## Before you get started

These steps are done on the build/ansible server - not the CAS servers.  We'll get to pushing the build out to the CAS servers later.

On your build server, you'll want to have installed:

* java-11-openjdk
* java-11-openjdk-devel
* git
* curl-devel

!!! caution ""
    I may have forgotten some prerequisites since I'm doing this on an existing server.   I will be going back to replicate this in the future on a clean server and will catch anything I missed there.

## References
* [CAS 6.3.x deployment - WAR Overlays](https://fawnoos.com/2020/11/09/cas63-gettingstarted-overlay/)