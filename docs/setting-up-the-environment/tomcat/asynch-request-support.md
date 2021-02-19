# Configure asynchronous request support

!!! tip
    The steps below are for the CAS servers - not the build server.  That being said - if you are going to install and maintain via Ansible - you don't need to do this manually, though it is worth seeing how to do it manually.  With Ansible, the web.xml file will be a template pushed from Ansible.

CAS 6's documentation states *"In the event that an external servlet container is used, you MAY need to make sure it’s configured correctly to support asynchronous requests in the event you get related errors and your container requires this."*.

## Set async-supported to true within default servlet
To implement this edit /etc/tomcat/web.xml and look for the definition of the *default* web app servlet, around line 113.  You'll want to add the **async-supported** directive before the end of the servlet.


``` xml
    <!-- Before changes -->
    <servlet>
        <servlet-name>default</servlet-name>
        <servlet-class>org.apache.catalina.servlets.DefaultServlet</servlet-class>
        <init-param>
            <param-name>debug</param-name>
            <param-value>0</param-value>
        </init-param>
        <init-param>
            <param-name>listings</param-name>
            <param-value>false</param-value>
        </init-param>
        <load-on-startup>1</load-on-startup>
    </servlet>

```

``` xml hl_lines="14"
    <!-- After changes -->
    <servlet>
        <servlet-name>default</servlet-name>
        <servlet-class>org.apache.catalina.servlets.DefaultServlet</servlet-class>
        <init-param>
            <param-name>debug</param-name>
            <param-value>0</param-value>
        </init-param>
        <init-param>
            <param-name>listings</param-name>
            <param-value>false</param-value>
        </init-param>
        <load-on-startup>1</load-on-startup>
        <async-supported>true</async-supported>
    </servlet>
```

## Set async-supported to true within jsp compiler and execution servlet
To implement this edit /etc/tomcat/web.xml and look for the definition of the *jsp* servlet, around line 268.  You'll want to add the **async-supported** directive before the end of the servlet.

``` xml
    <!-- Before changes -->
    <servlet>
        <servlet-name>jsp</servlet-name>
        <servlet-class>org.apache.jasper.servlet.JspServlet</servlet-class>
        <init-param>
            <param-name>fork</param-name>
            <param-value>false</param-value>
        </init-param>
        <init-param>
            <param-name>xpoweredBy</param-name>
            <param-value>false</param-value>
        </init-param>
        <load-on-startup>3</load-on-startup>
    </servlet>
```


``` xml hl_lines="14"
    <!-- Before changes -->
    <servlet>
        <servlet-name>jsp</servlet-name>
        <servlet-class>org.apache.jasper.servlet.JspServlet</servlet-class>
        <init-param>
            <param-name>fork</param-name>
            <param-value>false</param-value>
        </init-param>
        <init-param>
            <param-name>xpoweredBy</param-name>
            <param-value>false</param-value>
        </init-param>
        <load-on-startup>3</load-on-startup>
        <async-supported>true</async-supported>
    </servlet>
```

## References
* [CAS 6: Servlet Container Configuration](https://apereo.github.io/cas/6.2.x/installation/Configuring-Servlet-Container.html)