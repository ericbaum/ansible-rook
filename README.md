Ansible Rook
=========

This module is responsible for deploying rook over a
Kubernetes Infrastructure. It can be executed locally,
connecting to an external kubernetes or can be run directly
on the kubernetes nodes.

Requirements
------------

This modules uses the k8s_raw module which depends on Openshift:

* openshift == 0.4.3
* PyYAML >= 3.11

Role Variables
--------------

**TODO: Fill this up**

A description of the settable variables for this role should go here, including any variables that are in defaults/main.yml, vars/main.yml, and any variables that can/should be set via parameters to the role. Any variables that are read from other roles and/or the global scope (ie. hostvars, group vars, etc.) should be mentioned here as well.

Dependencies
------------

This role has no depencies

Example Playbook
----------------

Including an example of how to use your role (for instance, with variables passed in as parameters) is always nice for users too:

    - hosts: servers
      roles:
         - { role: username.rolename, x: 42 }

License
-------

BSD

Author Information
------------------

An optional section for the role authors to include contact information, or a website (HTML is not allowed).
