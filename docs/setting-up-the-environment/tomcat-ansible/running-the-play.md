# Running the play to install or upgrade Tomcat

## Before running - check that you have the following ready:

* The variables within the role's vars directory
* The templates within the role's templates directory
* The tasks directory including main.yml and all sub-tasks that it calls in the role's tasks directory
* The ssh public key from the .ssh directory of the user on your ansible host, has been copied to the authorized_keys file of your target host

Once you have your Tomcat role ready (see the remaining pages) out - you would run it as follows:
``` console
ansible site.yml --limit <yourCasServer>

# Example - assuming CAS6_DEV is a grouping in site.yml of your CAS dev environment:
ansible site.yml --limit CAS6_DEV 

# Or just one server:
ansible site.yml --limit login6devb
```

