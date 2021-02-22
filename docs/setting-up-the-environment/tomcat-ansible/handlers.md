# Ansible Handlers setup for Tomcat

Handlers in Ansible are used to run operations based on a change.  For example, a template command in a playbook could have a 'notify: restart apache' section at the end.  This would be run (at the end of the playbook) if that template command *changes* something.  If nothing changes, with that specific template command at least, then that notify command wouldn't be triggered.


``` yaml
# handlers file for apache-tomcat
# All that's really here for now is just commands to start, stop, and restart tomcat.
- name: stop tomcat
  ansible.builtin.systemd:
    name: tomcat
    state: stopped

- name: start tomcat
  ansible.builtin.systemd:
    name: tomcat
    state: started

- name: restart tomcat
  ansible.builtin.systemd:
    name: tomcat
    state: restarted

```


## References
* [Ansible systemd module](https://docs.ansible.com/ansible/latest/collections/ansible/builtin/systemd_module.html)
* [Ansible - Handlers: running operations on change](https://docs.ansible.com/ansible/latest/user_guide/playbooks_handlers.html)