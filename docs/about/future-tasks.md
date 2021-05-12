# Future tasks
These are items which are not complete in this documentation.  Some of this will be complete before I advertise this documentation (and are only here for my own notes).  Others will be finished later.

## Items to be finished or improved

* CAS UI customization/theming (started - not completed)
* Handling cas service cleanup (i.e. removing old services).  For now - this is manual.
* Add some service examples to the github repo
* Make sure this works on a vanilla RHEL 8 system 
    * my VMs are typically built via Ansible triggering vCenter and a RedHat Kickstart.  I will need to deploy a vanilla, unaltered VM that is just built from the ISO and with the public key of the Ansible server added to it, to see if I've missed any dependencies.
