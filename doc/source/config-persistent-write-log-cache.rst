=========================================
Configuring the Peristent Write Log Cache
=========================================

The Persistent Write Log Cache is simpler than the Immutable Object Cache
as it is implemented entirely within the RBD client libraries and needs
no extra packages, daemon running or client keyring to be installed.

As part of compute node preparation:

Assuming the spare disk available for a write cache is nvme2n1 for example:

.. code-block:: console

   # mkfs.ext4 /dev/nvme2n1     # create ext4 filesystem on disk
   # mkdir /rbd-write-log-cache
   # mount /dev/nvme2n1 /rbd-write-log-cache


The ceph_client ansible role will ensure that the directory permissions
are set correctly during deployment if
`ceph_persistent_write_log_cache_enabled: True` is set in
`/etc/openstack_deploy/group_vars/nova_compute.yml`. The variable
can be defined globally, or on a per group or per host basis.

To enable the Persistent Write Log Cache the following config must be
applied to the compute node also through group_vars, so that it is only
enabled on the nova_compute ceph clients.

Adjust the cache size based on the expected number of volumes mounted on
the compute node and the size of the cache device. The cache size is
allocated on the disk seperately for each pool/volume combination that is
active on the host.

.. code-block:: yaml

    ceph_client_ceph_conf_overrides:
      global:
        rbd_plugins = pwl_cache
        rbd_persistent_cache_mode = ssd
        rbd_persistent_cache_path = /rbd-write-log-cache
        rbd_persistent_cache_size = 10G       # size of cache used for each active rbd device

To see the activity within a write-log cache, use the following command
on the compute host `rbd status -n client.cinder <pool_name>/volume-<volume_uuid>`

Example:

.. code-block:: console

    # rbd status -n client.cinder cinder-volumes-nvme/volume-93f5a8fa-2e73-40c8-a9f1-bbeff3a3e6bc

    Watchers:
        watcher=10.51.1.134:0/2452041141 client.192434419 cookie=281466789599248
    Persistent cache state:
        host: compute1a01
        path: /rbd-write-log-cache/rbd-pwl.cinder-volumes-nvme.9058c8720de65b.pool
        size: 10 GiB
        mode: ssd
        stats_timestamp: Mon Apr  3 12:13:38 2023
        present: true    empty: false    clean: true
        allocated: 48 KiB
        cached: 24 KiB
        dirty: 0 B
        free: 1024 MiB
        hits_full: 6 / 0%
        hits_partial: 0 / 0%
        misses: 160340
        hit_bytes: 10 KiB / 0%
        miss_bytes: 20 GiB


When a new VM is created, a single 10GB file (whose name includes the ceph pool
name and a volume id) will be created in the `/rbd-write-log-cache` directory.
Note this is only used with new VMs created after caching was enabled.

If both Immutible Object Cache and Persistent Write Log are required to be
enabled on the same node then it is important to define the settings for
both in a single definiton of `ceph_client_ceph_conf_overrides`.
