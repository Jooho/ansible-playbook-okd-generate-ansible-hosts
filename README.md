Ansible Role: Generate Ansible Hosts File for OKD Installation
=========

This role help generate 2 files: ansible hosts file or okd_param_official file

By default, this role expect there is okd_param file that specify okd parameters. This role combine the okd_file and host information that is from group hostvars

ansible hosts file is for OKD installation and okd_param_official file is generated based on official inventory file of OKD git repo.

*Details*
- Generate ansible hosts file named /etc/ansible/hosts_{{cluster_tag}} and /etc/ansible/hosts
- Generate okd_param_official file from official inventory

Requirements
------------
None

Role Variables
--------------

| Name                            | Default value      | Requird | Description                                                                     |
| ------------------------------- | ------------------ | ------- | ------------------------------------------------------------------------------- |
| okd_param_name                  | okd_param          | no      | okd param file name                                                             |
| okd_official_param_name         | okd_param_official | no      | okd official param file name that is converted from official ansible hosts file |
| okd_param_dir                   | undefined          | yes     | Directory path where okd param file is                                          |
| okd_official_ansible_hosts_name | okd_official_hosts | no      | official ansible hosts file name. It will be stored uder okd_param_dir          |
| cluster_tag                     | undefined          | yes     | For backup, need to specify tag (ex OKD0311)                                    |
| reformat_vars_to_hosts          | true               | no      | Set false if you want to convert official hosts file to varible style           |




Dependencies
------------

None


Common Prerequisites
--------------------

- masters/nodes/lb group must exist
- Each node must has openshift_node_group_name as a hostvar from OKD 3.10

Example Playbook - Generate ansible hosts file for OKD
----------------

Prerequisites:
- There must be okd_param file under {{ okd_param_dir }}

~~~
- name: Example Playbook
  hosts: localhost
  tasks:
    - import_role:
        name: ansible-role-generate-ansible-hosts-okd
      vars:
        okd_param_dir: /home/jooho/test
        cluster_tag: OKD0311
~~~



Example Playbook - Generate okd_param_official
----------------

Prerequisites:
- There must be {{ okd_official_ansible_hosts_name }} file under {{ okd_param_dir }}

~~~
- name: Example Playbook
  hosts: localhost
  tasks:
    - import_role:
        name: ansible-role-generate-ansible-hosts-okd
      vars:
        okd_param_dir: /home/jooho/test
        reformat_vars_to_hosts: false
~~~

Test Playbook - Generate ansible hosts file for OKD
-------------

Before you execute this, please check there is master/nodes group in ansible hosts file  that you are using now.
If so, please remove that and execute it.

```
- name: Example Playbook
  hosts: localhost
  pre_tasks:
    - name: create test okd_param
      copy:
        dest: /tmp/{{ okd_param_name }}
        content: >
          openshift_release: 3.11

    - name: add group - masters
      add_host:
        name: master1
        groups: masters
        openshift_node_group_name: master-group-test

    - name: add group -nodes
      add_host:
        name: node1
        groups: nodes
        openshift_node_group_name: node-group-test
  tasks:
    - import_role:
        name: ansible-role-okd-generate-ansible-hosts
      vars:
        okd_param_dir: /tmp
        cluster_tag: OKD0311


```
Check - /etc/ansible/hosts_OKD0311
Expected Result:
```
[OSEv3:children]
masters
etcd
nodes


[OSEv3:vars]
openshift_release=3.11



[masters]
master1

[etcd]
master1

[nodes]
node1   openshift_node_group_name='node-group-test'

```


Test Playbook - Generate okd_param_official
-------------

Before you execute this, please check there is master/nodes group in ansible hosts file  that you are using now.
If so, please remove that and execute it.

```
- name: Example Playbook
  hosts: localhost
  pre_tasks:
    - name: create test okd_param
      copy:
        dest: /tmp/{{ okd_param_name }}
        content: >
          openshift_release=3.11

    - name: add group - masters
      add_host:
        name: master1
        groups: masters
        openshift_node_group_name: master-group-test

    - name: add group -nodes
      add_host:
        name: node1
        groups: nodes
        openshift_node_group_name: node-group-test
  tasks:
    - import_role:
        name: ansible-role-okd-generate-ansible-hosts
      vars:
        okd_param_dir: /tmp
        reformat_vars_to_hosts: false



```
Check - /tmp/okd_param_official

Expected Result:
```
openshift_release: 3.11

```



License
-------

BSD/MIT

Author Information
------------------

This role was created in 2018 by [Jooho Lee](http://github.com/jooho).
