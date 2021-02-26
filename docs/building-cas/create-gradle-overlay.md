# Create a Gradle WAR overlay project

## Clone the CAS overlay template from Apereo's GitHub

We're going to use git to clone the Apereo GitHub repository.  In the command below - I'm specifically choosing the 6.3.x branch.  The master branch of deployment is 6.3.x now - but could move forward at any time. I recommend being explicit with the branch you choose so you aren't surprised.

```
git clone --branch 6.3 https://github.com/apereo/cas-overlay-template.git
```

This will create a 'cas-overlay-template' in your home or build directory.

## Create a local branch

After you’re on the right branch (for our project, you should be on the master branch), create a new branch local to your project, which will be used to track all of your changes and keep them separate from any changes made to the template by the CAS developers. This will make it easier in the future to merge upstream changes from the CAS project team into your local template without having to redo all your changes.

Choose a meaningful name for your branch, but not somthing likely to be duplicated by the CAS developers—for example, newschool-casdev. Run the command **git checkout -b yourschool-casdev** and you should see notice that you've switched to that branch.

## Test build process

Change to the newly created cas-overlay-template directory and run an initial build.  This helps test things before we make any of our own modifications.

``` console
[chauvetp@login6deva casbuild]$ cd cas-overlay-template/
[chauvetp@login6deva cas-overlay-template]$ ./gradlew clean build
Starting a Gradle Daemon (subsequent builds will be faster)

Deprecated Gradle features were used in this build, making it incompatible with Gradle 7.0.
Use '--warning-mode all' to show the individual deprecation warnings.
See https://docs.gradle.org/6.4/userguide/command_line_interface.html#sec:command_line_warnings

BUILD SUCCESSFUL in 20s
5 actionable tasks: 5 executed
```

When that's complete - you will have a **cas.war** file created within the build/libs subdirectory.  As a quick test, you can copy that over to one of your CAS/Tomcat development hosts:
```
scp build/libs/cas.war root@<One-of-Your-CAS-Hosts>:/opt/tomcat/latest/webapps/
```

Once you do that, if Tomcat is running and you wait a minute or two, you'll have CAS up and running - though not in a really useful way (no authentication backends, no external services, no ticket registry, no theme, etc.).  It won't work to login - there's not even a configuration deployed - but it's just an initial test.

<figure>
  <img src="https://paulchauvet.github.io/deploying-cas/images/cas-default-login.png" alt="Screenshot showing default CAS login page"/>
</figure>


## References
* [CAS 6.3.x deployment - WAR Overlays](https://apereo.github.io/2019/11/03/cas62-gettingstarted-overlay/)