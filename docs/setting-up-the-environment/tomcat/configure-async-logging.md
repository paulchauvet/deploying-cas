# Configure asynchronous logging support

!!! tip
    The steps below are for the CAS servers - not the build server.  That being said - if you are going to install and maintain via Ansible - you don't need to do this manually, though it is worth seeing how to do it manually.  With Ansible, the catalina.properties file will be a template pushed from Ansible.


CAS 6’s logging subsystem automatically inserts itself into the runtime application context at startup time, and is designed to clean up the logging context when Tomcat shuts down. Unfortunately, the default Tomcat JarScanner configuration skips over JAR files named log4j*.jar, which prevents this feature from working.

To correct this problem, edit the file {==/etc/tomcat/catalina.properties==} and locate the lines defining the jarsToSkip property (starting around line 108), and then the specific line of that definition that includes log4j*.jar (around line 161):

``` hl_lines="7"
tomcat.util.scan.StandardJarScanFilter.jarsToSkip=\
annotations-api.jar,\
ant-junit*.jar,\
ant-launcher.jar,\
    (snip
junit.jar,\
log4j*.jar,\
mail*.jar,\
    (snip)
xmlParserAPIs.jar,\
xom-*.jar
```
Remove {==the log4j*.jar, \==} line completely.