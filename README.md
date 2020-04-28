Ansible Rook
=========

This role is responsible for deploying rook over a
Kubernetes Infrastructure. It can be executed locally,
connecting to an external kubernetes or can be run directly
on the kubernetes nodes.


Breaking changes
------------
Starting with the implementation of Rook v1.3.2, this ansible role will break conventions in configuration.
See the templates and sample vars file for new examples and specs.


Requirements
------------

This role uses the k8s module which depends on Openshift:

* openshift >= 0.9.2
* PyYAML >= 5.1.0

This role is actually capable of deploying Rook version: master


Role Variables
--------------
- [defaults/main.yml](defaults/main.yml)
- [vars/main.yml](vars/main.yml)


Dependencies
------------

This role has no depencies


Example Playbook
----------------

- Using block devices:
    ```yaml
    - hosts: kubernetes-master-node
      roles:
      - role: ansible-rook
        rook_osd_device_filter: 'sdb'
        rook_mon_count: 1
    ```
- A more detailed playbook:
    ```yaml
    - hosts: kubernetes-worker-nodes

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
          owner: "{{ rook_storage_dir_owner | default('0') }}"
          group: "{{ rook_storage_dir_group | default('0') }}"
        loop: "{{ [ rook_data_host_path | default('') ] }}"
        when: item | length
      - name: Install requirements
        pip:
          extra_args: -U
          name: "{{ item }}"
        loop: "{{ lookup('file', 'ansible-rook/requirements.txt').splitlines() }}"


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

      vars:
        ## See sample vars file in `vars/main.yml`
      roles:
        - role: ansible-rook
          rook_osd_device_filter: 'sdb'
          rook_mon_count: 1


    - hosts: kubernetes-worker-nodes

      tasks:
      - name: Restart Kubelet
        systemd:
          name: kubelet
          state: restarted
        tags: restart
    ```

> Note: nodes must be labeled beforehand with `ceph-role=all` or either **mon** or **mgr** or **osd**. This can be done automatically, as in the example playbook above.
> See also `defaults/main.yml`


Rook Ceph project examples
-------
https://github.com/rook/rook/tree/master/cluster/examples/kubernetes/ceph


License
-------

GPLv3


Authors Information
------------------

1. Eric Baum, 2018 - https://github.com/ericbaum
2. Revomatico - http://revomatico.com
