Ansible Rook
=========

This role is responsible for deploying rook over a
Kubernetes Infrastructure. It can be executed locally,
connecting to an external kubernetes or can be run directly
on the kubernetes nodes.

Requirements
------------

This role uses the k8s module which depends on Openshift:

* openshift >= 0.7.2
* PyYAML >= 3.11

This role is actually capable of deploying Rook version 0.9.3

Role Variables
--------------

[defaults/main.yml](defaults/main.yml)

[vars/main.yml](vars/main.yml)

Dependencies
------------

This role has no depencies

Example Playbook
----------------
- Using block devices:
    ```yaml
    - hosts: kubernetes-master-node
      roles:
         - { role: ansible-rook, rook_osd_device_filter: sdb, rook_mon_count: 1 }
    ```
- Using a directory:
    ```yaml
    - hosts: kubernetes-master-node
      roles:
         - { role: ansible-rook, rook_storage_dir: '/rook-storage', rook_mon_count: 3 }
    ```
- A more detailed playbook:
    ```yaml
    - hosts: nodes

      tasks:
      - name: Load ansible-rook defaults
        include_vars:
          dir: ansible-rook/defaults
      - name: Create storage directories
        file:
          path: "{{ item }}"
          state: directory
          recurse: yes
          mode: "{{ rook_storage_dir_permissions | default('0755') }}"
        loop: "{{ [ rook_storage_dir | default('') ] + [ rook_data_host_path | default('') ] }}"
        when: item | length


    - hosts: kubernetes-master-node

      pre_tasks:
      - name: Get Kubernetes Nodes
        k8s_facts:
          kubeconfig: "{{ kubeconfig_file_path | default(omit) }}"
          kind: 'Node'
        register: kubernetes_nodes
      - name: Load ansible-rook defaults
        include_vars:
          dir: ansible-rook/defaults
      - name: Label Kubernetes Nodes
        command: "kubectl label nodes {{ item.metadata.name }} {{ rook_node_label_key }}={{ rook_node_label_all }} --overwrite=true"
        register: label_return
        loop: "{{ kubernetes_nodes.resources }}"

      roles:
        - role: ansible-rook
          rook_osd_device_filter: ''
          rook_storage_dir: "{{ rook_storage_dir }}"
          rook_mon_count: "{{ rook_mon_count | default(1) }}"


    - hosts: nodes

      tasks:
      - name: Restart Kubelet
        systemd:
          name: kubelet
          state: restarted
        tags: restart
    ```

> Note: nodes must be labeled beforehand with `ceph-role=all` or either **mon** or **mgr** or **osd**.
> See `defaults/main.yml`

License
-------

GPLv3

Author Information
------------------

Eric Baum, 2018

https://github.com/ericbaum
