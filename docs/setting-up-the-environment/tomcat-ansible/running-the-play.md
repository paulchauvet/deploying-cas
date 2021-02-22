# Running the play to install or upgrade Tomcat

## Before running - check that you have the following ready:

* The variables within the role's vars directory (see [Setup Templates](http://127.0.0.1:8000/setting-up-the-environment/tomcat-ansible/variables/))
* The templates within the role's templates directory (see [Setup Templates](https://paulchauvet.github.io/deploying-cas/setting-up-the-environment/tomcat-ansible/templates/))
* The tasks directory including main.yml and all sub-tasks that it calls in the role's tasks directory (see [Creating Tasks](http://127.0.0.1:8000/setting-up-the-environment/tomcat-ansible/tasks/))
* The ssh public key from the .ssh directory of the user on your ansible host, has been copied to the authorized_keys file of your target host (see [Setup SSH public key authentication](http://127.0.0.1:8000/setting-up-the-environment/build-environment/#set-up-ssh-public-key-authentication))


## Running the playbook

Once you have your Tomcat role ready (see the remaining pages) out - you would run it as follows:
``` console
ansible-playbook site.yml --limit <yourCasServer>

# Example - assuming CAS6_DEV is a grouping in site.yml of your CAS dev environment:
ansible-playbook site.yml --limit CAS6_DEV 

# Or just one server:
ansible-playbook site.yml --limit login6devb
```

You may get warnings or errors.  You will have to review those.