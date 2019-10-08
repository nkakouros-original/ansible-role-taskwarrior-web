[![Build
Status](https://travis-ci.com/nkakouros-original/ansible-role-taskwarrior-web.svg?branch=master)](https://travis-ci.com/nkakouros-original/ansible-role-taskwarrior-web)
[![Galaxy](https://img.shields.io/badge/galaxy-nkakouros.taskwarrior-web-blue.svg)](https://galaxy.ansible.com/nkakouros/taskwarrior-web/)

Role Name
=========

Installs [taskwarrior-web](https://github.com/AndresBott/ansible-autodoc).

Requirements
------------

A working installation of taskwarrior and taskserver on the server where
taskwarrior-web will be installed on. You can use the two roles below to
automate that part as well:

- [nkakouros.taskserver](https://github.com/AndresBott/ansible-autodoc)
- [kulla.taskwarrior](https://github.com/kulla/ansible-role-taskwarrior)

Role Variables
--------------

See the [defaults/main.yml](defaults/main.yml) file for a list of the role
variables with descriptions.

Dependencies
------------

This role uses
[geerlingguy.ruby](https://github.com/geerlingguy/ansible-role-ruby) to install
ruby and `taskwarrior-web`. This happens automatically, you don't need to take
additional steps.

Example Playbook
----------------

```
- hosts: servers
  roles:
    - nkakouros.taskserver
    - kulla.taskwarrior
    - nkakouros.taskwarrior-web
  vars:
    # nkakouros.taskserver configuration
    taskd_group: cert-readers
    taskd_server: tasks.example.com
    taskd_oranizations:
      - Private
    taskd_users:
      - name: My Name
        organization: Private
    taskd_selfsigned: true
    taskd_selfsigned_clients_download_dir: "/home/.task/taskd/"
    taskd_selfsigned_cn: tasks.example.com
    taskd_selfsigned_organization: me
    taskd_selfsigned_country: GR
    taskd_selfsigned_state: Athens
    taskd_selfsigned_locality: Athens
    taskd_taskwarrior_config_path: "/home/.task/taskd/taskd.rc"
    taskd_taskwarrior_user: "{{ taskwarrior_user_id }}"

    # kulla.taskwarrior role configuration
    taskwarrior_packages:
      - taskwarrior
    taskwarrior_user_id: taskwarrior
    taskwarrior_configuration: >-
      {{
        (
          lookup('file', taskd_selfsigned_clients_download_dir + '/taskd.rc')
        ).splitlines()[-2:]
        | join('
        ')
      }}
    taskwarrior_ca_certificate: >-
      {{ taskd_selfsigned_clients_download_dir }}/ca.cert.pem
    taskwarrior_client_certificate: >-
      {{ taskd_selfsigned_clients_download_dir }}/{{
        taskd_users[0].name | replace(' ', '_') }}.cert.pem
    taskwarrior_client_key: >-
      {{ taskd_selfsigned_clients_download_dir }}/{{
        taskd_users[0].name | replace(' ', '_') }}.key.pem
    taskserver_cron_files:
      - file: taskwarrior-sync
        user: "{{ taskserver_accounts[0].name }}"
        jobs:
          - name: sync taskwarrior
            minute: "*/10"
            job: task sync
            user: "{{ taskwarrior_user_id }}"

    # nkakouros.taskwarrior-web configuration
    taskweb_user: "{{ taskwarrior_user_id }}"
    taskweb_port: 45678
    taskweb_host: 127.0.0.1
```

License
-------

GPLv3

Author Information
------------------

Nikolaos Kakouros (nkak@kth.se)
