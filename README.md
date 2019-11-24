Ansible Rook
=========

This role is responsible for deploying rook over a
Kubernetes Infrastructure. It can be executed locally,
connecting to an external kubernetes or can be run directly
on the kubernetes nodes.

Requirements
------------

This role uses the k8s module which depends on Openshift:

* openshift >= 0.9.2
* PyYAML >= 5.1.0

This role is actually capable of deploying Rook version: master

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
          owner: "{{ rook_storage_dir_owner | default('0') }}"
          group: "{{ rook_storage_dir_group | default('0') }}"
        loop: "{{ [ rook_storage_dir | default('') ] + [ rook_data_host_path | default('') ] }}"
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
        ## Define a storage pool
        ceph_block_storage_pools:
        - pool_name: replicated-pool
          pool_replicas: 3
        ## Optionally, define storage classses
        ## Currently, storageClassProvisioner should use CSI driver in favor of Flex driver (https://github.com/rook/rook/blob/master/Documentation/ceph-filesystem.md#provision-storage)
        rook_storage_classes:
        - name: rook-storage-class-block
          blockPoolName: replicated-pool
          storageClassProvisioner: rook-ceph.rbd.csi.ceph.com
        - name: rook-storage-class-filesystem
          fileSystemName: ceph_filesystem_name
          storageClassProvisioner: rook-ceph.cephfs.csi.ceph.com

        ## Alternatively to storage pool, a shared filesystems can be defined:
        ceph_file_systems:
        - name: ceph_filesystem_name
          pool_replicas:
            metadata: 3
            data: 3
          metadata_active: 1

        ## Optional to deploy ceph-mount pod to mount this filesystem on the host as well
        rook_mount_on_host:
        - name: mount-repl-shared-fs
          storageClassName: rook-ceph-filesystem-class
          enabled: true
          ## Optional for the ceph-mount pods, that mount this whole filesystem on the host
          hostPath: /mnt

      roles:
        - role: ansible-rook
          rook_osd_device_filter: ''
          rook_storage_dir: '/rook-storage'
          rook_mon_count: 1


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

Rook Ceph project examples
-------
https://github.com/rook/rook/tree/master/cluster/examples/kubernetes/ceph

License
-------

GPLv3

Author Information
------------------

Eric Baum, 2018

https://github.com/ericbaum
