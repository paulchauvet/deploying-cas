# Install an entropy daemon

!!! note
    This step is recommended if running CAS on virtual Linux servers. It is not necessary if running CAS on physical Linux servers or Windows servers of either type.

A common problem on virtual Linux servers is that the /dev/random device will run low on entropy, because most of the sources the kernel uses to build up the entropy pool are hardware-based, and therefore do not exist in a virtual environment. If there’s not enough entropy available when Tomcat is started, it can often take two or three minutes or longer for the server to start. Once Tomcat has started and the CAS application has been loaded, entropy is still required to establish secure (HTTPS) connections with authenticating users’ browsers and protected applications. A lack of available entropy will adversely affect the performance of the application by limiting the rate at which connections can be processed.

To improve the size of the entropy pool on Linux, it’s possible to feed random data from an external source into /dev/random. One way to do this is the haveged daemon, which uses the HAVEGE (HArdware Volatile Entropy Gathering and Expansion) algorithm to harvest the indirect effects of hardware events on hidden processor state (caches, branch predictors, memory translation tables, etc) to generate random bytes with which to fill /dev/random whenever the supply of random bits falls below the low water mark of the device. We will use this approach to avoid entropy depletion on the CAS servers.

Red Hat does not offer haveged on RHEL 7, but it can be installed from the Fedora Project’s Extra Packages for Enterprise Linux (EPEL) repository.  I typically find at least ONE package I need from EPEL on each server I'm involved in, so it's part of my normal build process.

## Install the EPEL repository

HAVEGE is needed on each CAS server, and since it needs EPEL, that is needed on each CAS server as well.

```
dnf install https://dl.fedoraproject.org/pub/epel/epel-release-latest-8.noarch.rpm

```

!!! note: According to EPEL: "on RHEL 8 it is required to also enable the codeready-builder-for-rhel-8-*-rpms repository since EPEL packages may depend on packages from it".  I haven't needed it for the installed packages, but to be safe - I recommend following their guidance.

```
# Installation commands for codeready-building are below.

ARCH=$( /bin/arch )
subscription-manager repos --enable "codeready-builder-for-rhel-8-${ARCH}-rpms"

# Note: since all our servers here are x86_64, I just run the following:
subscription-manager repos --enable "codeready-builder-for-rhel-8-x86_64-rpms"
```

## Install, enable, and start haveged

```
# Install haveged
dnf install haveged
# Enable the service
systemctl enable haveged
# Start the service
systemctl start haveged
# Verify that it is running
systemctl status haveged
```

## References
* [Extra Packages for Enterprise Linux (EPEL)](https://fedoraproject.org/wiki/EPEL)
* [haveged - A simple entropy daemon](http://www.issihosts.com/haveged/)