# Configure CAS logging
The Log4J configuration file included with the Gradle WAR overlay template will attempt to write the CAS server log files to /var/log.  This is an improvement from CAS 5 where it wanted to default to using the Tomcat directory, but I still prefer to have CAS logs in a separate directory of /var/log/cas.

I'd also recommend altering the log rotations so that you have a single file each day instead of it being split by size.

These steps are recommended but not required.


## Change CAS log directory (optional)
Make a copy of the file etc/cas/config/log4j2.xml in the cas-overlay-template directory from your build server, and copy it into the *files* subdirectory of your CAS Ansible role.  Find the line that defines the cas.log.dir property (around line 5) and change its value from /var/log to /var/log/cas.

Initial config:
``` xml hl_lines="7"
<?xml version="1.0" encoding="UTF-8" ?>
<!-- Specify the refresh internal in seconds. -->
<Configuration monitorInterval="5" packages="org.apereo.cas.logging">
    <Properties>
        <Property name="baseDir">/var/log</Property>

        <Property name="cas.log.level">info</Property>


```

Altered config with /var/log/cas as the directory:
``` xml hl_lines="7"
<?xml version="1.0" encoding="UTF-8" ?>
<!-- Specify the refresh internal in seconds. -->
<Configuration monitorInterval="5" packages="org.apereo.cas.logging">
    <Properties>
        <Property name="baseDir">/var/log/cas</Property>

        <Property name="cas.log.level">info</Property>

```

You'll also need to ensure that /var/log/cas exists, and is owned by tomcat (though you can skip this as our Ansible playbook will handle this):

``` shell
mkdir /var/log/cas
chown tomcat:tomcat /var/log/cas
chmod 750 /var/log/cas
```

## Alter the log file rotation strategy (optional)

1. Look for the *RollingFile* configuration for cas.log (around line 23), and change the variable part of the {==filePattern==} attribute to remove the -%i portion.  This is the hour and sequence number, which is unneeded if we're only at one file per day.
    * filePattern="${baseDir}/cas-%d{yyyy-MM-dd-HH}{==-%i==}.log">
    * filePattern="${baseDir}/cas-%d{yyyy-MM-dd-HH}.log">.
2. Remove (or comment out) the OnStartupTriggeringPolicy element (around line 27).
3. Remove (or comment out) the SizeBasedTriggeringPolicy element (around line 28).
4. Add the attributes interval="1" modulate="true" to the TimeBasedTriggeringPolicy element (around line 29).
5. Repeat steps 1-4, but for the cas_audit.log config - right after the cas.log config.

When done - the cas.log and cas_audit.log sections of log4j2.xml will look like the following:
``` xml
<RollingFile name="file" fileName="${baseDir}/cas.log" append="true"
                filePattern="${baseDir}/cas-%d{yyyy-MM-dd-HH}-%i.log">
    <PatternLayout pattern="%d %p [%c] - &lt;%m&gt;%n"/>
    <Policies>
        <!-- Not using OnStartupTriggering or SizeBasedTriggering Policies
            <OnStartupTriggeringPolicy />
            <SizeBasedTriggeringPolicy size="10 MB"/>
        -->
        <TimeBasedTriggeringPolicy interval="1" modulate="true"/>
    </Policies>
</RollingFile>
<RollingFile name="auditlogfile" fileName="${baseDir}/cas_audit.log" append="true"
                filePattern="${baseDir}/cas_audit-%d{yyyy-MM-dd-HH}.log">
    <PatternLayout pattern="%d %p [%c] - %m%n"/>
    <Policies>
        <!-- Not using OnStartupTriggering or SizeBasedTriggering Policies
            <OnStartupTriggeringPolicy />
            <SizeBasedTriggeringPolicy size="10 MB"/>
        -->
        <TimeBasedTriggeringPolicy interval="1" modulate="true"/>
    </Policies>
</RollingFile>
```

These changes have been incorporated into our ansible playbook - as you'll see in the next section.