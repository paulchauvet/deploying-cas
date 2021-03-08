# Setting up ticket registry

CAS can be setup to use various types of ticket registries to keep track of logins.

My assumption is that you will have (at least in a production environment) multiple CAS servers, sitting behind a load balancer.  In order for this to work correctly, you will need to have a shared ticket registry.  This is because a users's browser could be sent via the load balancer to one CAS server, and the CAS client (on the service they are trying to get access to) could be routed to a different CAS server.  If you don't have a shared ticket registry, then this login would fail.

There are multiple options for ticket registries within CAS.  For CAS 5.x, I used the MongoDB ticket registry - but ran into issues when I moved from 5.2.x to 5.3.x where the tickets were not getting cleaned up.  I ended up writing a script to connect to Mongo and clean these up - but I'd rather not go that route again for CAS 6.x, so I'm going to be going with Hazelcast instead.


## Add the hazelcast dependency
To add hazelcast support to the CAS server, edit the {==build.gradle==} file within the cas-overlay-template directory on your build host.  We're going to add a single depency to the one we already added for the json service registry.  See the highlighted line below for the addition.

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


## Rebuild CAS
To rebuild CAS with the newest dependency built in we'll do the same thing we did with previous additions.  You'll want to go to the cas-overlay-template directory and run the following and wait a moment for it to be complete:
```
./gradlew clean build
```

Your cas.war file will have been updated.  Copy it from *cas-overlay-template/build/libs/cas.war* to your *roles/cas6/files* 

## Configure Hazelcast properties in CAS
Add the following settings to roles/cas6/templates/dev-cas.properties.j2 in the cas-overlay-template directory on your Ansible server.  As with other sections - we're going to use variables here since some of the Hazelcast info is sensitive, and we'll leave them as variables here (and the real values will be in the Ansible cas-vault.yml file).

``` yaml
# Hazelcast Ticket Registry
cas.ticket.registry.hazelcast.cluster.members:              {{ DEV_CAS_SERVERS }}
cas.ticket.registry.hazelcast.mapName:                      tickets
# The asyncBackupCount should be set to n-1 where n is the number of servers in the cluster
cas.ticket.registry.hazelcast.cluster.asyncBackupCount:     1
cas.ticket.registry.hazelcast.cluster.tcpipEnabled:         true
# You'll want to make sure a range of ports, let's say 5701 through 5704 or so, are open between
# your different CAS servers (i.e. all the dev cas servers can talk to each other)
cas.ticket.registry.hazelcast.cluster.port:                 5701
cas.ticket.registry.hazelcast.cluster.portAutoIncrement:    true
cas.ticket.registry.hazelcast.cluster.multicastEnabled:     false
cas.ticket.registry.hazelcast.cluster.logging-type:         slf4j
cas.ticket.registry.hazelcast.cluster.instanceName:         cas6dev
cas.ticket.registry.hazelcast.crypto.signing.key:           {{ DEV_CAS_HAZELCAST_SIGNING_KEY }}           
cas.ticket.registry.hazelcast.crypto.signing.keySize:       512
cas.ticket.registry.hazelcast.crypto.encryption.key:        {{ DEV_CAS_HAZELCAST_ENCRYPTION_KEY }}
cas.ticket.registry.hazelcast.crypto.encryption.keySize:    16
cas.ticket.registry.hazelcast.crypto.alg:                   AES
```

## Variable setup
Edit your *cas-vault.yml* file within *roles/cas6/vars/*

You'll need to fill out the following:
DEV_CAS_SERVERS: <comma separated list if IPs or DNS names for your servers in this tier>
DEV_CAS_HAZELCAST_SIGNING_KEY: <see generating singing key section below>
DEV_CAS_HAZELCAST_ENCRYPTION_KEY: <see generating encryption key section below>


## Rerun the playbook

``` yaml
[chauvetp@ansible templates]$ ansible-playbook ~/ansible/site.yml --ask-vault-pass --limit <your_CAS_server>
Vault password: 
```


## References

* [CAS 6: Hazelcast Ticket Registry](https://apereo.github.io/cas/6.3.x/ticketing/Hazelcast-Ticket-Registry.html)
* [CAS 6: Hazelcast Configuration](https://apereo.github.io/cas/6.3.x/configuration/Configuration-Properties-Common.html#hazelcast-configuration)
* [CAS 6: MongoDb Ticket Registry](https://apereo.github.io/cas/6.3.x/ticketing/MongoDb-Ticket-Registry.html)
