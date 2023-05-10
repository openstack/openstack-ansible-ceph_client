======================================
Configuring the Immutable Object Cache
======================================

If a compute node has fast local disks (such as NVMe or PMEM), ceph clients
such as Nova using the RBD interface for volumes can use these disks as a
local read-only cache for volumes created from snapshots, for example, when
a snapshot of a Glance image in the ceph cluster is made in order to create
a bootable Cinder volume.

The copy-on-write mechanism means that all volumes cloned from the same
snapshot on the same compute host can share this cache and may avoid the
need to repeatedly read from OSDs the same underlying blocks (i.e those from
the original Glance image). New data written to the volume will not be
cached with the Immutable Object Cache.

The immutable object cache runs a daemon on the client node and must be an
authorised user of the ceph cluster. To enable the immutable object cache
on Nova compute nodes, create the following config in
`/etc/openstack_deploy/group_vars/nova_compute.yml`, taking care to
consider any other use of `ceph_client_ceph_conf_overrides` in the
deployment as the definition should only appear once.

.. code-block:: yaml

    ceph_immutable_object_cache_enabled: true

    ceph_client_ceph_conf_overrides:
      global:
        rbd_plugins: parent_cache
        rbd_parent_cache_enabled: true
        immutable_object_cache_path: /ceph-immutable-object-cache
        immutable_object_cache_max_size: 1500G    # set max size appropriate to the cache disk capacity


As part of the pre-deployment configuration, operators must prepare a
suitable disk and mountpoint which defaults to `/ceph-immutable-object-cache`,
this can be changed by overriding the `ceph_immutable_object_cache_dir`
variable.

For ceph clusters which are not deployed using OpenStack-Ansible, a keyring
must be created for a immutable object cache client:

.. code-block:: console

    ceph auth get-or-create client.immutable-object-cache mon 'allow r' osd 'profile rbd-read-only'


When the service is deployed and correctly configured, a cache
directory structure will be created inside the cache directory
after a new VM is booted using a disk cloned from a Glance image
or Cinder snapshot. Existing VM will not be cached.
