# Future tasks
These are items which are not complete in this documentation.  Some of this will be complete before I advertise this documentation (and are only here for my own notes).  Others will be finished later.

## Items to be finished before announcing this documentation

* CAS UI customization/theming
* Azure Conditional Access
* Review documentation - especially code blocks - to ensure proper lexers are being used.
* Change ansible templates over to use ansible.builtin (e.g. ansible.builtin.template instead of template)
* Manually unpack war file - or fix it so it is automatically unpacked correctly
* Handling cas service cleanup (i.e. removing old services).
* Settle on where AJP documentation will go (leaning towards two places - tomcat setup for the server.xml portion, and CAS setup for httpd.conf setup)
* Incorporate better generation of keys within initial cas config page

# Items to be finished after announcing this docuemntation

* Make sure this works on a vanilla RHEL 8 system 
    * my VMs are typically built via Ansible triggering vCenter and a RedHat Kickstart.  I will need to deploy a vanilla, unaltered VM that is just built from the ISO and with the public key of the Ansible server added to it, to see if I've missed any dependencies.
* Add better git documentation here
* Add full ansible playbooks for the roles used here (Apache Tomcat, Apache HTTPD, CAS, and CAS client) into a separate github project.  Need to take some time to go through and ensure they are sanitized beforehand (we use an internal git repo here for our development and anything sensitive 'should' be in vault files, but just want to be certain).  Link to them from the various areas.
