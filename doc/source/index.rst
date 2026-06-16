=============================
OpenStack-Ansible Ceph client
=============================

.. toctree::
   :maxdepth: 2

   configure-ceph.rst
   config-from-file.rst
   config-immutable-object-cache.rst
   config-persistent-write-log-cache.rst

This Ansible role installs the Ceph operating system
packages used to interact with a Ceph cluster.

To clone or view the source code for this repository, visit the role repository
for `ceph_client <https://opendev.org/openstack/openstack-ansible-ceph_client>`_.

Ceph releases and OS recommendations
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Information on current and past Ceph releases, including active and
end-of-life versions, can be found in the
`Ceph releases overview <https://docs.ceph.com/en/latest/releases/>`_.

For supported operating systems and kernel version requirements per Ceph
release, refer to the
`Ceph OS recommendations <https://docs.ceph.com/en/latest/start/os-recommendations/>`_.

Default variables
~~~~~~~~~~~~~~~~~

.. literalinclude:: ../../defaults/main.yml
   :language: yaml
   :start-after: under the License.

Example playbook
~~~~~~~~~~~~~~~~

.. literalinclude:: ../../examples/playbook.yml
   :language: yaml
