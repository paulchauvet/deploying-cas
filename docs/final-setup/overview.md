# Final Tasks

So I've got way too many pages here.  I've gone through enough steps here that I'm glad I documented them for myself, even if no one else uses this!

Here are the steps I've taken to move from using a development environment for CAS, to moving to TEST.  In our environment, TEST is not just for testing CAS.  It's for authentication for non-production applications.  These steps will be replicated for production.

1. Update your /etc/ansible/hosts to refer to your new systems.  Here are mine as an example (note - you don't need the vmware_name or vmware_folder stuff here for the CAS/Tomcat stuff.  They are used in other playbooks I have, like those that take snapshots in VMWare before patching or major changes).
``` json
login6testa vmware_name="Login6TestA (CAS 6 Test 1)" vmware_folder="/Computer Services/vm/CAS Authentication Servers" ansible_python_interpreter=/usr/libexec/platform-python apache_domains="logintest.newpaltz.edu login6testa.newpaltz.edu" tomcat_major_ver=9.0 tomcat_ver=9.0.45 jdk_version=11 

login6testb vmware_name="Login6TestB (CAS 6 Test 2)" vmware_folder="/Computer Services/vm/CAS Authentication Servers" ansible_python_interpreter=/usr/libexec/platform-python apache_domains="logintest.newpaltz.edu login6testb.newpaltz.edu" tomcat_major_ver=9.0 tomcat_ver=9.0.45 jdk_version=11 

```
2. Build two or more new hosts.  For our test environment, I call these 'login6testa' and 'login6testb'.  Use whatever your existing build process for them is.  For us - we use Ansible to build VMs via Kickstart.  Maybe I'll get into that in the future here.

3. Set your site.yml to include the roles you want to use.  For my environment, I use a few that aren't covered here (such as for security hardening, setting content-security-policies, or setting up apache-http), but those that are relevant for CAS are:
    * apache-tomcat
    * cas6
    * cas6-client (not needed in production - but I still use this in test)
4. Generate the various keys (see the Generating all necessary keys section near the bottom of the [initial CAS config](../building-cas/initial-cas-config.md)) and place them in your cas-vault.yml file.
5. Create your Azure app as per the [Azure SAML](../delegated-auth/azure-saml.md) page.  Get the metadata URL and put it in your cas-vault.yml
6. Ensure you have created a test-cas.properties.j2 file - and set the variable names correctly (changing DEV_ to TEST_ in variable names when appropriate, and setting cas.server.name to your load balanced virtual host).
7. Populate at least one service into your *roles/cas6/templates/test-services* directory
8. Ensure your playbooks have sections for TEST configs - not just DEV
9. Run your Ansible playbook on ONE of the servers.
``` console
ansible-playbook site.yml --ask-vault-pass --limit login6testa
```
10. Place only one of the servers as active in your load balancer - then visit the site.  It will generate the various saml-signing-cert files, sp-metadata.xml, and samlKeystore.jks
11. Copy these files to your *roles/cas6/files* directory - but rename the sp-metadata.xml and samlKeystore.jks to be tier specific (i.e. sp-metadata-TEST.xml and samlKeystore-TEST.jks).  You can then re-run the playbook on the other server(s) this will now push the sp-metadata.xml, samlKeystore, and saml-signing-cert files that were generated on the first host so they are the same in others in the tier.
``` console
ansible-playbook site.yml --ask-vault-pass --limit login6testa
```
12. Finish your Azure setup (see "Finish Azure config" section of the [Azure SAML](../delegated-auth/azure-saml.md) page).
13. Restart tomcat on both hosts (not sure this is necessary - but it's a one time thing).
14. Delegated auth should work now on your new tier.

## Is using Ansible worth it?

Is this really easier than doing it without Ansible?  If you're only doing this all once - doing it without Ansible may be easier - especially if you don't already use it in some form.   To make it repeatable and to maintain configs, service settings, Tomcat upgrades, and more - for me at least make Ansible essential for this.  I've made a TON of changes to configs, Tomcat versions, etc., while building this doc.  All I've needed to do is update a couple variables, maybe a config here and there, and redeploy.

Maybe that's because I have too much work - but I'm sure I'm not the only one reading this who has too much work!


