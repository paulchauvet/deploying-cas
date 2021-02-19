# Configure X-Forwarded-For header processing

!!! tip
    The steps below are for the CAS servers - not the build server.  That being said - if you are going to install and maintain via Ansible - you don't need to do this manually, though it is worth seeing how to do it manually.  With Ansible, the server.xml file will be a template pushed from Ansible.

You will most likely have CAS behind a load balancer.  If you have things like we have them in our environment, the source IP that comes to the web server is the IP of the load balancer, not the actual client IP.  Most, if not all load balancers can be configured to insert an X-Forwarded-For HTTP header to identify the address of the connecting system.  Tomcat can be configured as follows to look for this header and use it instead of the load balancer's source IP.

To configure Tomcat to process X-Forwarded-For HTTP headers, edit the file {==/etc/tomcat/server.xml==} and locate the definition of the {==AccessLogValve==} (around line 164, after inserting the changes in Configure TLS/SSL settings) and

* Insert a RemoteIpValve definition above it.
* Add a requestAttributesEnabled attribute to the AccessLogValve definition.

``` xml
    <!--Before changes-->
    <!-- Access log processes all example.
        Documentation at: /docs/config/valve.html
        Note: The pattern used is equivalent to using pattern="common" -->
    <Valve className="org.apache.catalina.valves.AccessLogValve" directory="logs"
        prefix="localhost_access_log" suffix=".txt"
        pattern="%h %l %u %t &quot;%r&quot; %s %b" />
```

```xml hl_lines="6 7 14"
    <!-- After changes-->
    <!-- RemoteIp valve, process X-Forwarded-For headers
        Documentation at: /docs/config/valve.html
        IP addresses below are those your load balancer uses to talk to your application server.
        Multiple IP addresses separated by | -->
    <Valve className="org.apache.catalina.valves.RemoteIpValve"
       internalProxies="192\.168\.1\.10|192\.168\.1\.11" />

    <!-- Access log processes all example.
        Documentation at: /docs/config/valve.html
        Note: The pattern used is equivalent to using pattern="common" -->
    <Valve className="org.apache.catalina.valves.AccessLogValve" directory="logs"
        prefix="localhost_access_log" suffix=".txt"
        requestAttributesEnabled="true"
        pattern="%h %l %u %t &quot;%r&quot; %s %b" />    
```