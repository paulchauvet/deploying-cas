# Setting up ticket registry

CAS can be setup to use various types of ticket registries to keep track of logins.

My assumption is that you will have (at least in a production environment) multiple CAS servers, sitting behind a load balancer.  In order for this to work correctly, you will need to have a shared ticket registry.  This is because a users's browser could be sent via the load balancer to one CAS server, and the CAS client (on the service they are trying to get access to) could be routed to a different CAS server.  If you don't have a shared ticket registry, then this login would fail.

There are multiple options for ticket registries within CAS.  In this documentation, I'm going to be using MongoDB, a NoSQL database, for the ticket registry.  Hazelcast appears to be most common - but I haven't been able to get it working with CAS 6.x so I'm staying with what I'm comfortable with.


## Add the mongodb dependency
To add mongodb support to the CAS server, edit the {==build.gradle==} file within the cas-overlay-template directory on your build host.  We're going to add a single depency to the one we already added for the json service registry.  See the highlighted line below for the addition.

**cas-overlay-template/build.gradle**
``` json hl_lines="9"
dependencies {
    // CAS dependencies/modules may be listed here statically...

    implementation "org.apereo.cas:cas-server-webapp-init:${casServerVersion}"
    implementation "org.apereo.cas:cas-server-support-json-service-registry:${casServerVersion}"
    implementation "org.apereo.cas:cas-server-support-ldap:${casServerVersion}"
    implementation "org.apereo.cas:cas-server-support-saml:${casServerVersion}"
    implementation "org.apereo.cas:cas-server-support-duo:${casServerVersion}"
    implementation "org.apereo.cas:cas-server-support-hazelcast-ticket-registry:${casServerVersion}"
}
```

## References
[CAS 6: MongoDB Ticket Registry](https://apereo.github.io/cas/6.3.x/ticketing/MongoDb-Ticket-Registry.html#mongodb-ticket-registry)