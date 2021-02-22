# Service configuration

Services within CAS are where you define a policy per-service.  For example you may want to:

* Release certain attributes to one service (name, email, eduPerson values, etc.)
* Require a user to be in one or more groups in Active Directory, or to have a certain eduPersonPrimaryAffiliation before they can access a particular service.
* Use a multi-factor authentication policy on a given service.

Each service can be different.  You can combine functions in the JSON service to accomplish various tasks.  We'll get to more of those later.

There are several ways to handle service registries, including the CAS [Services Management Webapp](https://apereo.github.io/cas/6.2.x/services/Installing-ServicesMgmt-Webapp.html).  Because I deploy and maintain CAS via Ansible, I prefer to define my services via JSON files within /etc/cas/services and deploy via Ansible instead of using a web application.  This helps preserve history (via git) of service changes.  It's all up to you though.  This documentation will assume you're using JSON files here.

To use the JSON service registry, you're going to want to edit your **build.gradle** file within your cas-overlay-template directory.

## Editing build.gradle
Look for this section, around line 72 in the build.gradle file

```
dependencies {
    // Other CAS dependencies/modules may be listed here...
    // implementation "org.apereo.cas:cas-server-support-json-service-registry:${casServerVersion}"
}
```

Add the dependency {==compile "org.apereo.cas:cas-server-support-json-service-registry:${project.'cas.version'}"==} as follows there:


```
dependencies {
    // Other CAS dependencies/modules may be listed here...
    // implementation "org.apereo.cas:cas-server-support-json-service-registry:${casServerVersion}"
    compile "org.apereo.cas:cas-server-support-json-service-registry:${project.'cas.version'}"
}
```

## Update your cas.properties
Go to your dev-cas.properties.j2 file in your ansible templates directory (or update the file cas.properties file directly if you're not using ansible) and make sure the following entry exists (it should be from our initial CAS config):

```
# JSON Service Registry
cas.serviceRegistry.json.location=file:/etc/cas/services
```

## Create a test service definition file
Within the templates directory (in your CAS Ansible role), create a subdirectory called **dev-services**.

For simplicity (and to avoid worrying about the details of the service registry for the moment), create a “wildcard” service definition that will allow any HTTPS based service to make use of the CAS server.  Create a file in the **dev-services** directory with the following contents:

``` json
{
  /*
   * Wildcard service definition that applies to any https or imaps url.
   * Do not use this definition in a production environment.
   */
  "@class" :            "org.apereo.cas.services.RegexRegisteredService",
  "serviceId" :         "^https://.*",
  "name" :              "HTTPS wildcard",
  "id" :                1614029117,
  "evaluationOrder" :   99999
}
```

The CAS documentation recommends the following naming convention for JSON service definition files:

```
JSON filename = serviceName + "-" + serviceNumericId + ".json"
```

Therefore, the filename for the wildcard service definition above should be: **HTTPSWildcard-1614029117.json**

### Wait - where'd those crazy number come from?
Techincally the id number just needs to be something unique.  One way to do this is with the {==date +%s==} command in Linux, which will give you the date/time as the number of seconds in the unix epoch.  The 'evaluationOrder' is set really high here since it will process anything here - but you want a wildcard like this to run last.

The fields we've used so far are defined as:

| Field         | Description                           |
| :----------   | :-----------------------------------  |
| 'serviceId'   | A regular expression describing the URL(s) where a service or services are located. Care should be taken to avoid patterns that match more than just the desired URL(s), as this can create security vulnerabilities.  |
| 'name'        | A name for the service. Note that because the service definition filename is created based on this name (see above), the value of this field should never contain [characters that are not allowed in filenames](https://en.wikipedia.org/wiki/Filename#Reserved_characters_and_words).|
| 'id'          | Unique numeric identifier for the service definition. An easy way to ensure that these identifiers are unique is to use the date and time the service definition was created. This can be represented as YYYYMMDDhhmmss or, for a more “anonymous” representation, as a timestamp (number of seconds since the epoch), which can be obtained with the command {==date +%s==}. |
| 'evaluationOrder' | A value that determines the relative evaluation order of registered services (lower values come before higher values). This is especially important when more than one serviceId expression can match the same service; evalutionOrder deterines which expression is evaluated first. |


## Getting your services from Ansible to CAS
As listed earlier, I recommend creating a separate service directory for each tier (prod, test, dev) within your templates directory.  I've created one called *dev-services* (within the templates directory of your Ansible CAS role).

Place your newly created service file from above into this directory.

### Update your tasks
Within the tasks directory, create a new file called *service-config.yml* (or something similar).  Reference that file within your main.yml - which should now look like:

``` yaml
---
# tasks file for cas6
- include_tasks: base-cas-config.yml
- include_tasks: service-config.yml

```

Your service-config.yml should look like the following:

``` yaml
---

- name: Ensure service files are populated from templates
  template:
    src: '{{ item.src }}'
    dest: '/etc/cas/services/{{ item.path }}'
    owner: root
    group: tomcat
  with_filetree: '../templates/dev-services'
  when: item.state == 'file' and 'login6dev' in inventory_hostname

```

This is a way of getting a whole directory copied over instead of a single file.  Otherwise you'd have to define an ansible template for each service you have - which could get pretty large (though is okay if you prefer it!).


## Rebuild and redeploy CAS
To rebuild CAS with this dependency built in, you'll want to go to the cas-overlay-template directory and run the following and wait a moment for it to be complete:
```
./gradlew clean build
```

Your cas.war file will have been updated.  You can either copy it to your CAS server directly (but where's the fun in that!) or update it in your *files* directory in your CAS Ansible role.  If you update it in your Ansible role, you can just rerun the playbook.

``` yaml
[chauvetp@ansible templates]$ ansible-playbook ~/ansible/site.yml --ask-vault-pass --limit <your_CAS_server>
Vault password: 
```

It won't have to rebuild or reinstall Tomcat, since it's idempotent.  It will only alter what is different from our templates/files/playbook.  In fact, when you run it, it will say 'ok' or 'skipped' for every task except the following (assuming you didn't change anything else), where it will say *changed*:

* TASK [cas6 : Configure cas.properties file (dev)]
* TASK [cas6 : Copy CAS war file]
* RUNNING HANDLER [cas6 : restart tomcat] 


## References
* [CAS Services Management](https://apereo.github.io/cas/6.2.x/services/Service-Management.html#service-management)