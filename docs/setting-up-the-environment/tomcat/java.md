# Install Java

CAS 6 requires Java 11 or later.  Due to the licensing changes from Oracle, I have only used [OpenJDK](https://openjdk.java.net/) for the last several years.  If you have the licensing from Oracle and prefer their version, go ahead!  Likewise there's also [AdoptOpenJDK](https://adoptopenjdk.net/) but I only have direct experience using OpenJDK with CAS 5 or 6.

OpenJDK has the advantage of being directly in the RHEL repositories, so it's one less thing to manage outside of the OS package management.  This is needed on the build servers, and all the CAS servers.

``` console
dnf install java-11-openjdk java-11-openjdk-devel
```

You can have multiple Java versions installed, but you will want to have JDK 11 as the default (or will have to make sure it's the default for Tomcat at least).  To check which is the default, use:

``` console
java --version
```
