## Deploying Apereo CAS

***Deploying Apereo CAS*** provides step-by-step instructions for setting up Apereo CAS 6 with a goal of deploying and maintaining both CAS and the application server (Tomcat) via Ansible.  The goal of this is to make deploying and maintaining CAS and Tomcat easy.  The idea for creating this (and some of the content) is based on the amazing [Deploying Apereo CAS](https://dacurry-tns.github.io/deploying-apereo-cas) documentation created for CAS 5, created by David Curry of The New School.  It is meant to supplement, not replace, the documentation created by Apereo for CAS.

 This is my first foray into using Github pages, or making documentation for deploying an application like this (my documentation has either been end user facing or for internal staff up until now) - so apologies for any rough edges on it.  It's incomplete as of now but my goal is to document as I deploy into test and later into production.  It should be *mostly* complete by the end of March 2021.

 It was created in my work at the State University of New York at New Paltz but I wanted to give back to the CAS community for the great information I've found from so many.

### Copyright and Licensing

#### Documentation Content

All documentation content is Copyright &copy; 2021, SUNY New Paltz. It is licensed under the [Creative Commons Attribution-ShareAlike 4.0 International License](http://creativecommons.org/licenses/by-sa/4.0/).

#### Site Template

This site was created with MkDocs using the [Material Theme](https://github.com/squidfunk/mkdocs-material).


### Author Information

Paul Chauvet<br/>
Information Security Officer<br/>
State University of New York at New Paltz<br/>
chauvetp@newpaltz.edu<br/>


!!! danger "DISCLAIMER"    

    The instructions and settings provided in this document may not be the only way to do things.  They are the way that has worked for us at New Paltz, and I've tried to document them as well as possible - but there may be better/cleaner ways of doing things.  They may not work at all for your environment.  Heck - I may have made some big mistakes here.
    
    As always - test test test.  You should not go running into this on a production environment without ample testing AND understanding of the setup (both Tomcat, and CAS).
    
    No warranty express or implied.  No support guaranteed.
    
    If you use this - and find it useful - let me know!  If you use it and find errors or would suggest changes - sure - let me know those too!  If you have suggestions as to how to do this (insert some completely different way) - you may want to fork this or create your own site from scratch for it.