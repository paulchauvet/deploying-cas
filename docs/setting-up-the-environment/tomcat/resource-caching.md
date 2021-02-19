# Tune resource caching settings

!!! tip
    The steps below are for the CAS servers - not the build server.  That being said - if you are going to install and maintain via Ansible - you don't need to do this manually, though it is worth seeing how to do it manually.  With Ansible, the context.xml file will be a template pushed from Ansible.

To improve performance, Tomcat is configured by default to cache static resources. However, the size of the cache is too small to work effectively with the CAS application. To tune Tomcat’s cache settings, edit the file {==/etc/tomcat/context.xml==}, locate the definition of the default context (around line 19), and add a <Resources> directive at the bottom:

``` xml
    <!--Before changes-->
    <Context>

    <!-- Default set of monitored resources. If one of these changes, the    -->
    <!-- web application will be reloaded.                                   -->
    <WatchedResource>WEB-INF/web.xml</WatchedResource>
    <WatchedResource>WEB-INF/tomcat-web.xml</WatchedResource>
    <WatchedResource>${catalina.base}/conf/web.xml</WatchedResource>

    <!-- Uncomment this to disable session persistence across Tomcat restarts -->
    <!--
    <Manager pathname="" />
    -->
</Context>

```

``` xml hl_lines="14"
    <!--After changes-->
    <Context>

    <!-- Default set of monitored resources. If one of these changes, the    -->
    <!-- web application will be reloaded.                                   -->
    <WatchedResource>WEB-INF/web.xml</WatchedResource>
    <WatchedResource>WEB-INF/tomcat-web.xml</WatchedResource>
    <WatchedResource>${catalina.base}/conf/web.xml</WatchedResource>

    <!-- Uncomment this to disable session persistence across Tomcat restarts -->
    <!--
    <Manager pathname="" />
    -->
    <Resources cachingAllowed="true" cacheMaxSize="40960" cacheTtl="60000" />
</Context>

```