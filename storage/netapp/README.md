#NetApp Storage Modules
This directory contains modules that support the storage platforms in the NetApp portfolio.

##SANtricity Modules
The modules prefixed with *netapp\_e* are built to support the SANtricity storage platform.  They require the SANtricity 
WebServices Proxy.  The WebServices Proxy is free software available at the [NetApp Software Download site](http://mysupport.netapp.com/NOW/download/software/eseries_webservices/1.40.X000.0009/).
Starting with the E2800 platform (11.30 OS), the modules will work directly with the storage array.  Starting with this
platform, REST API requests are handled directly on the box.  This array can still be managed by proxy for large scale deployments.
The modules provide idempotent provisioning for volume groups, disk pools, standard volumes, thin volumes, LUN mapping, 
hosts, host groups (clusters), volume snapshots, consistency groups, and asynchronous mirroring.
### Prerequisites
| Software | Version |
| -------- |:-------:|
| SANtricity Web Services Proxy*|1.4 or 2.0|
| Ansible | 2.2** |
\* Not required for *E2800 with 11.30 OS*<br/>
\*\*The modules where developed with this version.  Ansible forward and backward compatibility applies.

###Questions and Contribution
Please feel free to submit pull requests with improvements.  Issues for these modules should be routed to @hulquest but 
we also try to keep an eye on the list for issues specific to these modules.  General questions can be made to our [development team](mailto:ng-hsg-engcustomer-esolutions-support@netapp.com)

### Examples
These examples are not comprehensive but are intended to help you get started when integrating storage provisioning into 
your playbooks.
```yml
- name: NetApp Test All Modules
  hosts: proxy20
  gather_facts: yes
  connection: local
  vars:
    storage_systems:
      ansible1:
        address1: "10.251.230.41"
        address2: "10.251.230.42"
      ansible2:
        address1: "10.251.230.43"
        address2: "10.251.230.44"
      ansible3:
        address1: "10.251.230.45"
        address2: "10.251.230.46"
      ansible4:
        address1: "10.251.230.47"
        address2: "10.251.230.48"
    storage_pools:
      Disk_Pool_1:
        raid_level: raidDiskPool
        criteria_drive_count: 11
      Disk_Pool_2:
        raid_level: raidDiskPool
        criteria_drive_count: 11
      Disk_Pool_3:
        raid_level: raid0
        criteria_drive_count: 2
    volumes:
        vol_1:
          storage_pool_name: Disk_Pool_1
          size: 10
          thin_provision: false
          thin_volume_repo_size: 7
        vol_2:
          storage_pool_name: Disk_Pool_2
          size: 10
          thin_provision: false
          thin_volume_repo_size: 7
        vol_3:
          storage_pool_name: Disk_Pool_3
          size: 10
          thin_provision: false
          thin_volume_repo_size: 7
        thin_vol_1:
          storage_pool_name: Disk_Pool_1
          size: 10
          thin_provision: true
          thin_volume_repo_size: 7
    hosts:
      ANSIBLE-1:
        host_type: 1
        index: 1
        ports:
          - type: 'fc'
            label: 'fpPort1'
            port: '2100000E1E191B01'

    netapp_api_host: 10.251.230.29
    netapp_api_url: http://{{ netapp_api_host }}/devmgr/v2
    netapp_api_username: rw
    netapp_api_password: rw
    ssid: ansible1
    auth: no
    lun_mapping: no
    netapp_api_validate_certs: False
    snapshot: no
    gather_facts: no
    amg_create: no
    remove_volume: no
    make_volume: no
    check_thins: no
    remove_storage_pool: yes
    check_storage_pool: yes
    remove_storage_system: no
    check_storage_system: yes
    change_role: no
    flash_cache: False
    configure_hostgroup: no
    configure_async_mirror: False
    configure_snapshot: no
    copy_volume: False
    volume_copy_source_volume_id:
    volume_destination_source_volume_id:
    snapshot_volume_storage_pool_name: Disk_Pool_3
    snapshot_volume_image_id: 3400000060080E5000299B640063074057BC5C5E
    snapshot_volume: no
    snapshot_volume_name: vol_1_snap_vol
    host_type_index: 1
    host_name: ANSIBLE-1
    set_host: no
    remove_host: no
    amg_member_target_array:
    amg_member_primary_pool:
    amg_member_secondary_pool:
    amg_member_primary_volume:
    amg_member_secondary_volume:
    set_amg_member: False
    amg_array_name: foo
    amg_name: amg_made_by_ansible
    amg_secondaryArrayId: ansible2
    amg_sync_name: foo
    amg_sync: no

  tasks:

    - name: Get array facts
      netapp_e_facts:
        ssid: "{{ item.key }}"
        api_url: "{{ netapp_api_url }}"
        api_username: "{{ netapp_api_username }}"
        api_password: "{{ netapp_api_password }}"
        validate_certs: "{{ netapp_api_validate_certs }}"
      with_dict: "{{ storage_systems }}"
      when: gather_facts

    - name:  Presence of storage system
      netapp_e_storage_system:
        ssid: "{{ item.key }}"
        state: present
        api_url: "{{ netapp_api_url }}"
        api_username: "{{ netapp_api_username }}"
        api_password: "{{ netapp_api_password }}"
        validate_certs: "{{ netapp_api_validate_certs }}"
        controller_addresses:
          - "{{ item.value.address1 }}"
          - "{{ item.value.address2 }}"
      with_dict: "{{ storage_systems }}"
      when: check_storage_system

    - name: Create Snapshot
      netapp_e_snapshot_images:
        ssid: "{{ ssid }}"
        api_url: "{{ netapp_api_url }}"
        api_username: "{{ netapp_api_username }}"
        api_password: "{{ netapp_api_password }}"
        validate_certs: "{{ netapp_api_validate_certs }}"
        snapshot_group: "ansible_snapshot_group"
        state: 'create'
      when: snapshot
      
    - name: Auth Module Example
      netapp_e_auth:
        ssid: "{{ ssid }}"
        current_password: 'Infinit2'
        new_password: 'Infinit1'
        set_admin: yes
        api_url: "{{ netapp_api_url }}"
        api_username: "{{ netapp_api_username }}"
        api_password: "{{ netapp_api_password }}"
      when: auth

    - name: No disk groups
      netapp_e_storagepool:
        ssid: "{{ ssid }}"
        name: "{{ item }}"
        state: absent
        api_url: "{{ netapp_api_url }}"
        api_username: "{{ netapp_api_username }}"
        api_password: "{{ netapp_api_password }}"
        validate_certs: "{{ netapp_api_validate_certs }}"
        remove_volumes: yes
      with_items:
        - Disk_Pool_1
        - Disk_Pool_2
        - Disk_Pool_3
      when: remove_storage_pool

    - name: Make disk groups
      netapp_e_storagepool:
        ssid: "{{ ssid }}"
        name: "{{ item.key }}"
        state: present
        api_url: "{{ netapp_api_url }}"
        api_username: "{{ netapp_api_username }}"
        api_password: "{{ netapp_api_password }}"
        validate_certs: "{{ netapp_api_validate_certs }}"
        raid_level: "{{ item.value.raid_level }}"
        criteria_drive_count: "{{ item.value.criteria_drive_count }}"
      with_dict: " {{ storage_pools }}"
      when: check_storage_pool

    - name: No thin volume
      netapp_e_volume:
        ssid: "{{ ssid }}"
        name: NewThinVolumeByAnsible
        state: absent
        thin_provision: yes
        log_path: /tmp/volume.log
        api_url: "{{ netapp_api_url }}"
        api_username: "{{ netapp_api_username }}"
        api_password: "{{ netapp_api_password }}"
        validate_certs: "{{ netapp_api_validate_certs }}"
      when: check_thins

    - name: Make a thin volume
      netapp_e_volume:
        ssid: "{{ ssid }}"
        name: NewThinVolumeByAnsible
        state: present
        thin_provision: yes
        thin_volume_repo_size: 7
        size: 10 
        log_path: /tmp/volume.log
        api_url: "{{ netapp_api_url }}"
        api_username: "{{ netapp_api_username }}"
        api_password: "{{ netapp_api_password }}"
        validate_certs: "{{ netapp_api_validate_certs }}"
        storage_pool_name: Disk_Pool_1
      when: check_thins

    - name: Remove standard/thick volumes
      netapp_e_volume:
        ssid: "{{ ssid }}"
        name: "{{ item.key }}"
        state: absent
        log_path: /tmp/volume.log
        api_url: "{{ netapp_api_url }}"
        api_username: "{{ netapp_api_username }}"
        api_password: "{{ netapp_api_password }}"
        validate_certs: "{{ netapp_api_validate_certs }}"
      with_dict: "{{ volumes }}"
      when: remove_volume

    - name: Make a volume
      netapp_e_volume:
        ssid: "{{ ssid }}"
        name: "{{ item.key }}"
        state: present
        storage_pool_name: "{{ item.value.storage_pool_name }}"
        size: "{{ item.value.size }}"
        thin_provision: "{{ item.value.thin_provision }}"
        thin_volume_repo_size: "{{ item.value.thin_volume_repo_size }}"
        log_path: /tmp/volume.log
        api_url: "{{ netapp_api_url }}"
        api_username: "{{ netapp_api_username }}"
        api_password: "{{ netapp_api_password }}"
        validate_certs: "{{ netapp_api_validate_certs }}"
      with_dict: "{{ volumes }}"
      when: make_volume

    - name: No storage system
      netapp_e_storage_system:
        ssid: "{{ item.key }}"
        state: absent
        api_url: "{{ netapp_api_url }}"
        api_username: "{{ netapp_api_username }}"
        api_password: "{{ netapp_api_password }}"
        validate_certs: "{{ netapp_api_validate_certs }}"
      with_dict: "{{ storage_systems }}"
      when: remove_storage_system

    - name: Update the role of a storage array
      netapp_e_amg_role:
        name: "{{ amg_name }}"
        role: primary
        force: true
        noSync: true
        ssid: "{{ ssid }}"
        api_url: "{{ netapp_api_url }}"
        api_username: "{{ netapp_api_username }}"
        api_password: "{{ netapp_api_password }}"
        validate_certs: "{{ netapp_api_validate_certs }}"
      when: change_role

    - name: Flash Cache
      netapp_e_flashcache:
        ssid: "{{ ssid }}"
        api_url: "{{ netapp_api_url }}"
        api_username: "{{ netapp_api_username }}"
        api_password: "{{ netapp_api_password }}"
        validate_certs: "{{ netapp_api_validate_certs }}"
        name: SSDCacheBuiltByAnsible
      when: flash_cache

    - name: Configure Hostgroup
      netapp_e_hostgroup:
        ssid: "{{ ssid }}"
        api_url: "{{ netapp_api_url }}"
        api_username: "{{ netapp_api_username }}"
        api_password: "{{ netapp_api_password }}"
        validate_certs: "{{ netapp_api_validate_certs }}"
        state: absent
        name: "ansible-host-group"
      when: configure_hostgroup

    - name: Configure Snapshot group
      netapp_e_snapshot_group:
        ssid: "{{ ssid }}"
        state: present
        api_url: "{{ netapp_api_url }}"
        api_username: "{{ netapp_api_username }}"
        api_password: "{{ netapp_api_password }}"
        validate_certs: "{{ netapp_api_validate_certs }}"
        base_volume_name: vol_3
        name: ansible_snapshot_group
        repo_pct: 20
        warning_threshold: 85
        delete_limit: 30
        full_policy: purgepit
        storage_pool_name: Disk_Pool_3
        rollback_priority: medium
      when: configure_snapshot

    - name: Copy volume
      netapp_e_volume_copy:
        ssid: "{{ ssid }}"
        api_url: "{{ netapp_api_url }}"
        api_username: "{{ netapp_api_username }}"
        api_password: "{{ netapp_api_password }}"
        status: present
        source_volume_id: "{{ volume_copy_source_volume_id }}"
        destination_volume_id: "{{ volume_destination_source_volume_id }}"
      when: copy_volume

    - name: Snapshot volume
      netapp_e_snapshot_volume:
        ssid: "{{ ssid }}"
        api_url: "{{ netapp_api_url }}"
        api_username: "{{ netapp_api_username }}"
        api_password: "{{ netapp_api_password }}"
        state: present
        storage_pool_name: "{{ snapshot_volume_storage_pool_name }}"
        snapshot_image_id: "{{ snapshot_volume_image_id }}"
        name: "{{ snapshot_volume_name }}"
      when: snapshot_volume

    - name: Remove hosts
      netapp_e_host:
        ssid: "{{ ssid }}"
        state: absent
        name: "{{ item.key }}"
        api_url: "{{ netapp_api_url }}"
        api_username: "{{ netapp_api_username }}"
        api_password: "{{ netapp_api_password }}"
        host_type_index: "{{ host_type_index }}"
      with_dict: "{{hosts}}"
      when: remove_host

    - name: Ensure/add hosts
      netapp_e_host:
        ssid: "{{ ssid }}"
        state: present
        api_url: "{{ netapp_api_url }}"
        api_username: "{{ netapp_api_username }}"
        api_password: "{{ netapp_api_password }}"
        name: "{{ item.key }}"
        host_type_index: "{{ item.value.index }}"
        ports:
          - type: 'fc'
            label: 'fpPort1'
            port: '2100000E1E191B01'
      with_dict: "{{hosts}}"
      when: set_host

    - name: Unmap a volume
      netapp_e_lun_mapping:
        state: absent
        ssid: "{{ ssid }}"
        lun: 2
        target: "{{ host_name }}"
        volume_name: "thin_vol_1"
        target_type: host
        api_url: "{{ netapp_api_url }}"
        api_username: "{{ netapp_api_username }}"
        api_password: "{{ netapp_api_password }}"
      when: lun_mapping

    - name: Map a volume
      netapp_e_lun_mapping:
        state: present
        ssid: "{{ ssid }}"
        lun: 16
        target: "{{ host_name }}"
        volume_name: "thin_vol_1"
        target_type: host
        api_url: "{{ netapp_api_url }}"
        api_username: "{{ netapp_api_username }}"
        api_password: "{{ netapp_api_password }}"
      when: lun_mapping

    - name: Update LUN Id
      netapp_e_lun_mapping:
        state: present
        ssid: "{{ ssid }}"
        lun: 2
        target: "{{ host_name }}"
        volume_name: "thin_vol_1"
        target_type: host
        api_url: "{{ netapp_api_url }}"
        api_username: "{{ netapp_api_username }}"
        api_password: "{{ netapp_api_password }}"
      when: lun_mapping

    - name: AMG removal
      netapp_e_amg:
        state: absent
        ssid: "{{ ssid }}"
        secondaryArrayId: "{{amg_secondaryArrayId}}"
        api_url: "{{ netapp_api_url }}"
        api_username: "{{ netapp_api_username }}"
        api_password: "{{ netapp_api_password }}"
        new_name: "{{amg_array_name}}"
        name: "{{amg_name}}"
      when: amg_create

    - name: AMG create
      netapp_e_amg:
        state: present
        ssid: "{{ ssid }}"
        secondaryArrayId: "{{amg_secondaryArrayId}}"
        api_url: "{{ netapp_api_url }}"
        api_username: "{{ netapp_api_username }}"
        api_password: "{{ netapp_api_password }}"
        new_name: "{{amg_array_name}}"
        name: "{{amg_name}}"
      when: amg_create

    - name: start AMG async
      netapp_e_amg_sync:
        name: "{{ amg_name }}"
        state: running
        ssid: "{{ ssid }}"
        api_url: "{{ netapp_api_url }}"
        api_username: "{{ netapp_api_username }}"
        api_password: "{{ netapp_api_password }}"
      when: amg_sync
```

## SolidFire Modules
The modules prefixed with *sf\_* are built to support the SolidFire storage platform.  

### Prerequisites

Note: These modules have been tested with Ansible (2.1.0.0) and solidfire-sdk-python (1.1.0.92)

* A physical or virtual SolidFire system
* solidfire-sdk-python

### Installation

Install solidfire-sdk-python

```sh
$ pip install solidfire-sdk-python
```

### Examples
These examples are not comprehensive but are intended to help you get started when integrating storage provisioning into your playbooks.
```yml

- name: SolidFire Sample Playbook
  hosts: localhost
  gather_facts: no
  connection: local
  vars:
    solidfire_hostname: <host_ip>
    solidfire_username: admin
    solidfire_password: <password>

  tasks:

   - name: Check connections to MVIP and SVIP
     sf_check_connections:
       hostname: "{{ solidfire_hostname }}"
       username: "{{ solidfire_username }}"
       password: "{{ solidfire_password }}"
       # Other configurable parameters
       # skip: - Skip checking connection to either SVIP or MVIP
       #         Valid options - mvip, svip
       # mvip: - Optionally check connection of a different MVIP.
       #         This is not needed to test the connection to the target cluster.
       # svip: - Optionally check connection of a different SVIP.
       #         This is not needed to test the connection to the target cluster.


# ----------- Account Manager ----------------


    - name: Create Account
      sf_account_manager:
       hostname: "{{ solidfire_hostname }}"
       username: "{{ solidfire_username }}"
       password: "{{ solidfire_password }}"
       action: create
       name: AnsibleUser
#        #
#        # Other configurable parameters:
#        # ------------------------------
#        # initiator_secret
#        # target_secret
#        # attributes

   - name: Modify Account
     sf_account_manager:
       hostname: "{{ solidfire_hostname }}"
       username: "{{ solidfire_username }}"
       password: "{{ solidfire_password }}"
       action: update
       account_id: 7
       name: AnsibleUser-Renamed
#        #
#        # Other configurable parameters that can be updated:
#        # --------------------------------------------------
#        # name
#        # status
#        # initiator_secret
#        # target_secret
#        # attributes

   - name: Delete Account
     sf_account_manager:
       hostname: "{{ solidfire_hostname }}"
       username: "{{ solidfire_username }}"
       password: "{{ solidfire_password }}"
       action: delete
       account_id: 7


# ----------- Volume Manager ----------------


   - name: Create Volume
     sf_volume_manager:
       hostname: "{{ solidfire_hostname }}"
       username: "{{ solidfire_username }}"
       password: "{{ solidfire_password }}"
       action: create
       name: AnsibleVol
       account_id: 3
       enable512e: False
       size: 1
       size_unit: gb
#
#        # Other configurable parameters
#        # qos: - Initial quality of service settings for this volume.
#        # attributes: - List of Name/Value pairs in JSON object format.

   - name: Update Volume
     sf_volume_manager:
       hostname: "{{ solidfire_hostname }}"
       username: "{{ solidfire_username }}"
       password: "{{ solidfire_password }}"
       action: update
       volume_id: 2
       access: readWrite
#
#        # Other configurable parameters
#
#        # account_id: - Account_id to which the volume is reassigned.
#        #               If none is specified, the previous account name is used.
#
#        # set_create_time: - Identify the time at which the volume was created.
#
#        # qos: - Initial quality of service settings for this volume.
#
#        # attributes: - List of Name/Value pairs in JSON object format.

   - name: Delete Volume
     sf_volume_manager:
       hostname: "{{ solidfire_hostname }}"
       username: "{{ solidfire_username }}"
       password: "{{ solidfire_password }}"
       action: delete
       volume_id: 2


# ----------- Volume Access Group Manager ----------------

   - name: Create Volume Access Group
     sf_volume_access_group_manager:
       hostname: "{{ solidfire_hostname }}"
       username: "{{ solidfire_username }}"
       password: "{{ solidfire_password }}"
       action: create
       name: AnsibleVolumeAccessGroup
       volumes: 7,8
        #
        # Other configurable parameters:
        # ------------------------------
        # initiators
        # virtual_network_id
        # virtual_network_tags
        # attributes

   - name: Modify Volume Access Group
     sf_volume_access_group_manager:
       hostname: "{{ solidfire_hostname }}"
       username: "{{ solidfire_username }}"
       password: "{{ solidfire_password }}"
       action: update
       volume_access_group_id: 1
       name: AnsibleVolumeAccessGroup-Renamed
#        #
#        # Other configurable parameters that can be updated:
#        # --------------------------------------------------
#        # name
#        # initiators
#        # virtual_network_id
#        # virtual_network_tags
#        # volumes
#        # attributes

   - name: Delete Volume Access Group
     sf_volume_access_group_manager:
       hostname: "{{ solidfire_hostname }}"
       username: "{{ solidfire_username }}"
       password: "{{ solidfire_password }}"
       action: delete
       volume_access_group_id: 1


# ----------- Snapshot Schedule Manager ----------------

   - name: Create Snapshot schedule
     sf_snapshot_schedule_manager:
       hostname: "{{ solidfire_hostname }}"
       username: "{{ solidfire_username }}"
       password: "{{ solidfire_password }}"
       action: create
       name: AnsibleSnapshotSchedule
       time_interval_days: 1
       starting_date: 2016--12--01T00:00:00Z
       volumes: 7

#         Other configurable parameters:
#         ------------------------------
#         time_interval_hours
#         time_interval_minutes
#         pause
#         recurring
#         snapshot_name
#         retention

   - name: Update Snapshot schedule
     sf_snapshot_schedule_manager:
       hostname: "{{ solidfire_hostname }}"
       username: "{{ solidfire_username }}"
       password: "{{ solidfire_password }}"
       action: update
       schedule_id: 6
       recurring: True
       snapshot_name: AnsibleSnapshots

   - name: Delete Snapshot schedule
     sf_snapshot_schedule_manager:
       hostname: "{{ solidfire_hostname }}"
       username: "{{ solidfire_username }}"
       password: "{{ solidfire_password }}"
       action: delete
       schedule_id: 6

#         Other configurable parameters:
#         ------------------------------
#         same as for the 'Create' example above.



# ----------- Cluster Pair Manager ----------------

   - name: Get Cluster Key
     sf_cluster_pair_manager:
       hostname: "{{ solidfire_hostname }}"
       username: "{{ solidfire_username }}"
       password: "{{ solidfire_password }}"
       action: get_key
     Note: Remember to run playbook in verbose mode

   - name: Create Cluster Pair
     sf_cluster_pair_manager:
       hostname: "{{ solidfire_hostname }}"
       username: "{{ solidfire_username }}"
       password: "{{ solidfire_password }}"
       action: create
       cluster_pair_key: {{ cluster pair key }}

   - name: Delete Cluster Pair
     sf_cluster_pair_manager:
       hostname: "{{ solidfire_hostname }}"
       username: "{{ solidfire_username }}"
       password: "{{ solidfire_password }}"
       action: delete
       cluster_pair_id: {{ cluster pair id }}



# ----------- Volume Pair Manager ----------------

   - name: Create Volume Pair
     sf_volume_pair_manager:
       hostname: "{{ solidfire_hostname }}"
       username: "{{ solidfire_username }}"
       password: "{{ solidfire_password }}"
       action: create
       first_volume_id: 7
       second_volume_id: 8
       mode:

   - name: Update Volume Pair
     sf_volume_pair_manager:
       hostname: "{{ solidfire_hostname }}"
       username: "{{ solidfire_username }}"
       password: "{{ solidfire_password }}"
       action: update
       pause: false
       first_volume_id: 7
       mode: SnapshotsOnly

   - name: Delete Volume Pair
     sf_volume_pair_manager:
       hostname: "{{ solidfire_hostname }}"
       username: "{{ solidfire_username }}"
       password: "{{ solidfire_password }}"
       action: delete
       first_volume_id: 7
       second_volume_id: 8

```

