# Task setup


## Main task file

To keep things cleaner, I include chunks of the role in separate files.  All must be referenced by the main.yml file within the role (ansible/roles/apache-tomcat/tasks/main.yml).  They are called in order by main.yml:

``` yaml
# tasks file for apache-tomcat
- include_tasks: setup-prerequisites.yml
# on RHEL7 - I have a playbook to download/compile OpenSSL as well but that's not needed on RHEL8.  RHEL7 had a really old version of OpenSSL.
- include_tasks: setup-apr.yml
- include_tasks: install.yml
- include_tasks: setup-tomcat-native.yml
- include_tasks: setup-commons-daemon.yml
- include_tasks: configure-tomcat.yml
- include_tasks: post-install.yml
```

!!! tip
  If you haven't used ansible - you may want to look up idempotency and what it means.  An operation is idempotent if the result of performing it once is exactly the same as the result of performing it repeatedly without any intervening actions.  For example in these playbooks, there are tasks to create users, create directories, or apply a config template.  If Ansible detects that these are already right - it will just mosey right along and not change anything for those steps.


## Setup Prerequisites

I've left some of my older stuff in here for example's sake (since again, I still have to maintain some RHEL 7 and JDK 8 systems).  I've highlighted the revelant portions for this.  The example isn't the cleanest, but you can see how you can alter packages based on different needs (in thise case, dnf is used for RHEL 8 and yum for RHEL 7.).

The Tomcat user and group are created as well at the bottom.

If you use this, the 'jdk_version" needs to be specified in your ansible hosts file, or some variable file.  The ansible_distribution and ansible_distribution_major_version don't need to be specified as Ansible reads those from the hosts it connects to.

``` yaml hl_lines="1-13 43-54"

- name: Setup prerequisite dnf packages (RHEL 8 & JDK 11)
  dnf:
    name:
      - epel-release
      - java-11-openjdk
      - java-11-openjdk-devel
      - haveged
      - gcc
      - libtool
      - make
      - openssl-devel
    state: present
  when: ansible_distribution == 'RedHat' and ansible_distribution_major_version == '8' and jdk_version == 11

- name: Setup prerequisite yum packages (RHEL 7 & JDK 11)
  yum:
    name:
      - epel-release
      - java-11-openjdk
      - java-11-openjdk-devel
      - haveged
      - gcc
      - libtool
      - make
      - policycoreutils-python
    state: present
  when: ansible_distribution == 'RedHat' and ansible_distribution_major_version == '7' and jdk_version == 11

- name: Setup prerequisite yum packages (RHEL 7 & JDK 8)
  yum:
    name: 
      - epel-release
      - java-1.8.0-openjdk
      - java-1.8.0-openjdk-devel
      - haveged
      - gcc
      - libtool
      - make
      - policycoreutils-python
    state: present
  when: ansible_distribution == 'RedHat' and ansible_distribution_major_version == '7' and jdk_version == 8

- name: Add tomcat group
  group:
    name: tomcat

- name: Add tomcat user
  user:
    name: tomcat
    group: tomcat
    home: /opt/tomcat
    shell: /sbin/nologin
    createhome: yes
    system: yes

```


## Setup Apache Portable Runtime (APR)

The following section checks if the target version of APR is already installed.  If not, it will download, unpack, compile, and install it.

``` yaml

- name: Check if APR {{ apr_ver }} is already installed
  stat:
    path: /opt/apr/apr-{{ apr_ver }}/bin/apr-1-config
  register: apr_binary

- name: Create base APR directory if it doesn't exist
  file:
    path: /opt/apr
    state: directory
    owner: root
    group: root

- name: Create APR {{ apr_ver }} directory if it doesn't exist
  file:
    path: /opt/apr/apr-{{ apr_ver }}
    state: directory
    owner: root
    group: root   

- name: Check if APR {{ apr_ver }} is already downloaded
  stat:
    path: /root/apr-{{ apr_ver }}.tar.gz
  register: apr_tarball

# Only download APR if APR {{ apr_ver }} isn't already installed, and the tarball isn't downloaded
- name: Download APR
  get_url:
    url: "{{ apr_archive_url }}"
    dest: "{{ apr_archive_dest }}"
  when: apr_tarball.stat.exists == False and apr_binary.stat.exists == False

# Only unpack apr archive if the apr binary doesn't already exist
- name: Unpack APR archive
  unarchive:
    src: "{{ apr_archive_dest }}"
    dest: /root/
    owner: root
    group: root
    remote_src: yes
  when: apr_binary.stat.exists == False
 
# Only reconfigure if the APR directory for the version didn't already exist
- name: Configure APR source
  command: ./configure --prefix=/opt/apr/apr-{{ apr_ver }}
  args:
    chdir: "/root/apr-{{ apr_ver }}"
  when: apr_binary.stat.exists == False

# Only make if the APR directory for the version didn't already exist
- name: Compile/install APR
  shell: make && make install
  args:
    chdir: "/root/apr-{{ apr_ver }}"
  when: apr_binary.stat.exists == False

## Handle APR symlink creation or repointing
- name: Check if apr latest symlink exists
  stat:
    path: /opt/apr/latest
  register: apr_symlink

# Create symlink if none exists, or repoint it if it is pointing to an older version
- name: Create apr latest symlink to point to newly installed version
  file:
    src: "/opt/apr/apr-{{ apr_ver }}"
    dest: "/opt/apr/latest"
    owner: root
    group: root
    state: link
  when: apr_symlink.stat.exists == False or apr_symlink.stat.islnk

# Cleanup
- name: Remove apr source directory
  file:
    path: /root/apr-{{ apr_ver }}
    state: absent

- name: Remove apr source tarball
  file:
    path: /root/apr-{{ apr_ver }}.tar.gz
    state: absent
```


## Install Apache Tomcat

This section checks if the target version Tomcat is already installed.  If not - it will download Tomcat, unpack the archive, and ensure the various directory symlinks (conf, log, temp, webapps, work) are already setup.

If this is the first time Tomcat is being deployed on the target system (or the /etc/tomcat directory is otherwise empty), the contents of 'conf' from the downloaded tarball will be copied to /etc/tomcat.

It will also setup the systemd unit file so it can eventually be started and set to start on boot.



``` yaml
- name: Check if Tomcat {{ tomcat_ver }} directory exists
  stat:
    path: /opt/tomcat/apache-tomcat-{{ tomcat_ver }}
  register: tomcat_directory

- name: Check if Tomcat {{ tomcat_ver }} tarball exists
  stat:
    path: "{{ tomcat_archive_dest }}"
  register: tomcat_tarball

- name: Download Tomcat 8.5.x
  get_url:
    url: https://archive.apache.org/dist/tomcat/tomcat-8/v{{ tomcat_ver }}/bin/apache-tomcat-{{ tomcat_ver }}.tar.gz
    dest: "{{ tomcat_archive_dest }}"
  when: tomcat_tarball.stat.exists == False and tomcat_directory.stat.exists == False and tomcat_major_ver == 8.5

- name: Download Tomcat 9.0.x
  get_url:
    url: https://archive.apache.org/dist/tomcat/tomcat-9/v{{ tomcat_ver }}/bin/apache-tomcat-{{ tomcat_ver }}.tar.gz
    dest: "{{ tomcat_archive_dest }}"
  when: tomcat_tarball.stat.exists == False and tomcat_directory.stat.exists == False and tomcat_major_ver == 9.0


# Only unpack tomcat archive if the unpacked directory does not exist.
# This cannot be used to install the same version without deletion of the old one
- name: Unpack tomcat archive
  unarchive:
    src: "{{ tomcat_archive_dest }}"
    dest: /opt/tomcat
    owner: tomcat
    group: tomcat
    remote_src: yes
  when: tomcat_directory.stat.exists == False


# Create the permanent tomcat conf, log, temp, webapps, work directories for later symlinking
- name: Create Tomcat permanent conf directory
  file:
    path: /etc/tomcat
    state: directory
    owner: root
    group: tomcat
    mode: "u=rwx,g=rx"

- name: Create Tomcat permanent log directory
  file:
    path: /var/log/tomcat
    state: directory
    owner: tomcat
    group: tomcat
    mode: "u=rwx,g=rx"

- name: Create Tomcat permanent temp directory
  file:
    path: /var/cache/tomcat/temp
    state: directory        
    owner: tomcat
    group: tomcat
    mode: "u=rwx,g=rx"

- name: Create Tomcat permanent webapps directory
  file: 
    path: /var/lib/tomcat
    state: directory
    owner: tomcat
    group: tomcat
    mode: "u=rwx,g=rx"

- name: Create Tomcat permanent work directory
  file:
    path: /var/cache/tomcat/work
    state: directory
    owner: tomcat
    group: tomcat
    mode: "u=rwx,g=rx"


# If server.xml exists - then we can assume /etc/tomcat has been populated.
# - If so - don't change it
# - If not, assume it hasn't been populated and copy the new Tomcat's /conf to /etc/tomcat/
- name: Check if server.xml exists
  stat:
    path: /etc/tomcat/server.xml
  register: server_xml

- name: Copy files from conf directory
  copy:
    src: /opt/tomcat/apache-tomcat-{{ tomcat_ver }}/conf/
    dest: /etc/tomcat/
    remote_src: yes
    directory_mode: yes
  when: server_xml.stat.exists == False

# If ROOT exists - then we can assume /var/lib/tomcat has been populated.
# - If so - don't change it.
# - If not, assume it hasn't been populated and copy the new Tomcat's ROOT over
# - This needs to be updated to handle version updates of ROOT
- name: Check ROOT exists
  stat:
    path: /var/lib/tomcat/ROOT
  register: root_webapps

- name: Copy ROOT webapp directory
  copy:
    src: /opt/tomcat/apache-tomcat-{{ tomcat_ver }}/webapps/ROOT
    dest: /var/lib/tomcat/
    remote_src: yes
    directory_mode: yes
  when: root_webapps.stat.exists == False

## remove directories (conf, logs, temp, webapps, work) and recreate as symlinks
# conf
- name: check {{ tomcat_ver }} conf directory
  stat:
    path: /opt/tomcat/apache-tomcat-{{ tomcat_ver }}/conf
  register: tomcat_conf_dir

- name: Remove conf directory
  file:
    path: /opt/tomcat/apache-tomcat-{{ tomcat_ver }}/conf
    state: absent
  when: tomcat_conf_dir.stat.isdir

- name: Create conf symlink
  file:
    src: /etc/tomcat
    dest: /opt/tomcat/apache-tomcat-{{ tomcat_ver }}/conf
    owner: root
    group: tomcat
    state: link
  when: tomcat_conf_dir.stat.isdir or tomcat_conf_dir.stat.exists == False

# logs
- name: check {{ tomcat_ver }} logs directory
  stat:
    path: /opt/tomcat/apache-tomcat-{{ tomcat_ver }}/logs
  register: tomcat_logs_dir

- name: Remove logs directory
  file:
    path: /opt/tomcat/apache-tomcat-{{ tomcat_ver }}/logs
    state: absent
  when: tomcat_logs_dir.stat.isdir

- name: Create logs symlink
  file:
    src: /var/log/tomcat
    dest: /opt/tomcat/apache-tomcat-{{ tomcat_ver }}/logs
    owner: tomcat
    group: tomcat
    state: link
  when: tomcat_logs_dir.stat.isdir or tomcat_logs_dir.stat.exists == False

# temp
- name: check {{ tomcat_ver }} temp directory
  stat:
    path: /opt/tomcat/apache-tomcat-{{ tomcat_ver }}/temp
  register: tomcat_temp_dir

- name: Remove temp directory
  file:
    path: /opt/tomcat/apache-tomcat-{{ tomcat_ver }}/temp
    state: absent
  when: tomcat_temp_dir.stat.isdir

- name: Create temp symlink
  file:
    src: /var/cache/tomcat/temp
    dest: /opt/tomcat/apache-tomcat-{{ tomcat_ver }}/temp
    owner: tomcat
    group: tomcat
    state: link
  when: tomcat_temp_dir.stat.isdir or tomcat_temp_dir.stat.exists == False

# webapps
- name: check {{ tomcat_ver }} webapps directory
  stat:
    path: /opt/tomcat/apache-tomcat-{{ tomcat_ver }}/webapps
  register: tomcat_webapps_dir

- name: Remove webapps directory
  file:
    path: /opt/tomcat/apache-tomcat-{{ tomcat_ver }}/webapps    
    state: absent
  when: tomcat_webapps_dir.stat.isdir

- name: Create webapps symlink
  file:
    src: /var/lib/tomcat
    dest: /opt/tomcat/apache-tomcat-{{ tomcat_ver }}/webapps
    owner: root
    group: tomcat
    state: link
  when: tomcat_webapps_dir.stat.isdir or tomcat_webapps_dir.stat.exists == False

# work
- name: check {{ tomcat_ver }} work directory
  stat:
    path: /opt/tomcat/apache-tomcat-{{ tomcat_ver }}/work
  register: tomcat_work_dir

- name: Remove webapps directory
  file:
    path: /opt/tomcat/apache-tomcat-{{ tomcat_ver }}/work
    state: absent
  when: tomcat_work_dir.stat.isdir

- name: Create work symlink
  file:
    src: /var/cache/tomcat/work
    dest: /opt/tomcat/apache-tomcat-{{ tomcat_ver }}/work
    owner: tomcat
    group: tomcat
    state: link
  when: tomcat_work_dir.stat.isdir or tomcat_webapps_dir.stat.exists == False

# Setup systemd startup/shutdown script
- name: Setup systemd startup/shutdown script
  template:
    src: tomcat.service.j2
    dest: /etc/systemd/system/tomcat.service
    mode: 0644
    owner: root
    group: root
  register: tomcat_service

- name: Apply new SELinux file context to tomcat.service
  command: restorecon /etc/systemd/system/tomcat.service
  when: tomcat_service.changed

- name: Reload systemd daemons after service update
  command: systemctl daemon-reload
  when: tomcat_service.changed

```

## Setup Tomcat Native Library

This is where the Tomcat Native Library (if not already installed) gets unpacked, configured, compiled, and installed.

``` yaml
- name: Check if Tomcat Native Library {{ tomcat_native_ver }} is already installed
  stat:
    path: /opt/tomcat/apache-tomcat-{{ tomcat_ver }}/lib/libtcnative-1.so
  register: tomcat_native_library

# Untar tomcat native library
- name: Unpack tomcat native library archive
  unarchive:
    src: "/opt/tomcat/apache-tomcat-{{ tomcat_ver }}/bin/tomcat-native.tar.gz"
    dest: /opt/tomcat/apache-tomcat-{{ tomcat_ver }}/bin/
    owner: tomcat
    group: tomcat
    remote_src: yes
  when: tomcat_native_library.stat.exists == False


# Only configure if the Tomcat Native Library directory for the version didn't already exist
- name: Configure Tomcat Native Library (RHEL7 & JDK 11)
  command: "./configure --with-java-home={{ JAVA_HOME }} --with-apr=/opt/apr/latest/bin/apr-1-config --prefix=/opt/tomcat/apache-tomcat-{{ tomcat_ver }}"
  args:
    chdir: "/opt/tomcat/apache-tomcat-{{ tomcat_ver }}/bin/tomcat-native-{{ tomcat_native_ver }}-src/native"
  when: tomcat_native_library.stat.exists == False and ansible_distribution == 'RedHat' and ansible_distribution_major_version == '7' and jdk_version == 11

# Only configure if the Tomcat Native Library directory for the version didn't already exist
- name: Configure Tomcat Native Library (RHEL7 & JDK 8)
  command: "./configure --with-java-home={{ JAVA_8_HOME }} --with-apr=/opt/apr/latest/bin/apr-1-config --prefix=/opt/tomcat/apache-tomcat-{{ tomcat_ver }}"
  args:
    chdir: "/opt/tomcat/apache-tomcat-{{ tomcat_ver }}/bin/tomcat-native-{{ tomcat_native_ver }}-src/native"
  when: tomcat_native_library.stat.exists == False and ansible_distribution == 'RedHat' and ansible_distribution_major_version == '7' and jdk_version == 8


# Only configure if the Tomcat Native Library directory for the version didn't already exist
- name: Configure Tomcat Native Library (RHEL8)
  command: "./configure --with-java-home={{ JAVA_HOME }} --with-apr=/opt/apr/latest/bin/apr-1-config --with-ssl=yes --prefix=/opt/tomcat/apache-tomcat-{{ tomcat_ver }}"
  args:
    chdir: "/opt/tomcat/apache-tomcat-{{ tomcat_ver }}/bin/tomcat-native-{{ tomcat_native_ver }}-src/native"
  when: tomcat_native_library.stat.exists == False and ansible_distribution == 'RedHat' and ansible_distribution_major_version == '8'

# Only make if the Tomcat Native Library directory for the version didn't already exist
- name: Compile/install Tomcat Native Library
  shell: make && make install
  args:
      chdir: "/opt/tomcat/apache-tomcat-{{ tomcat_ver }}/bin/tomcat-native-{{ tomcat_native_ver }}-src/native"
  when: tomcat_native_library.stat.exists == False

# Cleanup
- name: Remove tomcat native source directory
  file:
    path: /opt/tomcat/apache-tomcat-{{ tomcat_ver }}/bin/tomcat-native-{{ tomcat_native_ver }}-src
    state: absent

```

## Setup Apache Tomcat Commons Daemon

This is where the Tomcat Commons Daemon (if not already installed) gets unpacked, configured, compiled, and installed.

``` yaml
- name: Check if Apache Tomcat Commons Daemon {{ commons_daemon_ver }} is already installed
  stat:
    path: /opt/tomcat/apache-tomcat-{{ tomcat_ver }}/bin/jsvc
  register: commons_daemon_jsvc

# Untar tomcat native library
- name: Unpack Tomcat Commons Daemon native library
  unarchive:
    src: "/opt/tomcat/apache-tomcat-{{ tomcat_ver }}/bin/commons-daemon-native.tar.gz"
    dest: "/opt/tomcat/apache-tomcat-{{ tomcat_ver }}/bin/"
    owner: tomcat
    group: tomcat
    remote_src: yes
  when: commons_daemon_jsvc.stat.exists == False

# Only configure if the Tomcat Native Library directory for the version didn't already exist
- name: Configure Tomcat Commons Daemon native library (JDK 11)
  command: "./configure --with-java={{ JAVA_HOME }}"
  args:
    chdir: "/opt/tomcat/apache-tomcat-{{ tomcat_ver }}/bin/commons-daemon-{{ commons_daemon_ver }}-native-src/unix"
  when: commons_daemon_jsvc.stat.exists == False and jdk_version == 11

# Only configure if the Tomcat Native Library directory for the version didn't already exist
- name: Configure Tomcat Commons Daemon native library (JDK 8)
  command: "./configure --with-java={{ JAVA_8_HOME }}"
  args:
    chdir: "/opt/tomcat/apache-tomcat-{{ tomcat_ver }}/bin/commons-daemon-{{ commons_daemon_ver }}-native-src/unix"
  when: commons_daemon_jsvc.stat.exists == False and jdk_version == 8

       
# Only make if the Tomcat Native Library directory for the version didn't already exist
- name: Compile Tomcat Commons Daemon native library
  shell: make
  args:
    chdir: "/opt/tomcat/apache-tomcat-{{ tomcat_ver }}/bin/commons-daemon-{{ commons_daemon_ver }}-native-src/unix"
  when: commons_daemon_jsvc.stat.exists == False

- name: Move jsvc file
  copy:
    src: "/opt/tomcat/apache-tomcat-{{ tomcat_ver }}/bin/commons-daemon-{{ commons_daemon_ver }}-native-src/unix/jsvc"
    dest: "/opt/tomcat/apache-tomcat-{{ tomcat_ver }}/bin/"
    remote_src: yes
    owner: root
    group: tomcat
    mode: 0755
  when: commons_daemon_jsvc.stat.exists == False


# Cleanup
- name: Remove commons-daemon source directory
  file:
    path: /opt/tomcat/apache-tomcat-{{ tomcat_ver }}/bin/commons-daemon-{{ commons_daemon_ver }}-native-src
    state: absent

```

## Configure Tomcat
This is where we take the template files that were created earlier for web.xml, server.xml, etc., and make sure they are copied over to the server.  If any are updated, it will notify Tomcat to restart when done.

This is basically just configuring the various files in /etc/tomcat.  To initially create these files - download copies of them from a default Tomcat install to your Ansible host's roles/apache-tomcat/templates directory and alter them as needed.  I will have copies of these files uploaded as well.

### To do - add links to the documents where these file changes are done.

``` yaml

- name: Setup catalina.properties
  template:
    src: cas6-catalina.properties.j2
    dest: /etc/tomcat/catalina.properties
    mode: 0640
    owner: root
    group: tomcat
  when: ("login" in inventory_hostname)
  notify: restart tomcat

- name: Setup context.xml
  template:
    src: cas6-context.xml.j2
    dest: /etc/tomcat/context.xml
    mode: 0640
    owner: root
    group: tomcat
  when: ("login" in inventory_hostname)
  notify: restart tomcat

- name: Setup server.xml
  template:
    src: cas6-server.xml.j2
    dest: /etc/tomcat/server.xml
    mode: 0640
    owner: root
    group: tomcat
  when: ("login" in inventory_hostname)
  notify: restart tomcat

- name: Setup web.xml
  template:
    src: cas6-web.xml.j2
    dest: /etc/tomcat/web.xml
    mode: 0640
    owner: root
    group: tomcat
  when: ("login" in inventory_hostname)
  notify: restart tomcat

```

## Post installation tasks
If the installation fails (errors, not warnings) at any point - it won't get to here, so it shouldn't move the symlink and restart tomcat unless the install completed.  It also won't 'notify' tomcat to restart if they didn't even have to create the symlink (or if the config files changed earlier)


``` yaml

- name: Setup Apache Tomcat {{ tomcat_ver }} symlink
  file:
    src: /opt/tomcat/apache-tomcat-{{ tomcat_ver }}
    dest: /opt/tomcat/latest
    owner: root
    group: root
    state: link
  notify: restart tomcat

# Cleanup
- name: Remove tomcat tarball
  file:
    path: /root/apache-tomcat-{{ tomcat_ver }}.tar.gz
    state: absent

```

