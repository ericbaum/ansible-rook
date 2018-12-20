Ansible Rook
=========

This role is responsible for deploying rook over a
Kubernetes Infrastructure. It can be executed locally,
connecting to an external kubernetes or can be run directly
on the kubernetes nodes.

Requirements
------------

This role uses the k8s module which depends on Openshift:

* openshift == 0.7.2
* PyYAML >= 3.11

This role is actually capable of deploying Rook version 0.8.3

Role Variables
--------------

[defaults/main.yml](defaults/main.yml)

[vars/main.yml](vars/main.yml)

Dependencies
------------

This role has no depencies

Example Playbook
----------------

    - hosts: kubernetes-master-node
      roles:
         - { role: ansible-rook, rook_osd_device_filter: sdb, rook_mon_count: 1 }

License
-------

GPLv3

Author Information
------------------

Eric Baum, 2018

https://github.com/ericbaum
