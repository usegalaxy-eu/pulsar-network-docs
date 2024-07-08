Configure the pulsar endpoint
=============================

This step will create several Pulsar needed resources on the cloud infrastructure, which is automatically done by `terraform`.
In that way, your Pulsar endpoint has its own resources and can be safely deleted and redeployed using `terraform` only.
The only required resource is a public network in your OpenStack tenant.

.. note::

   A SSH key pair is needed: the public key has to be configured in the vars.tf as public_key entry. The private key is needed in the terraform apply step. You can create a SSH key with the command ``ssh-keygen``.

Configuration
-------------

.. warning::

   Source the tenant RC file (see section :doc:`requirements`) before to start the installation procedure, otherwise Terraform will not be able to perform resources creation and configuration.

#. Clone the usegalaxy.eu GitHub repository `pulsar-deployment <https://github.com/usegalaxy-eu/pulsar-deployment>`_.

   ::

     git clone https://github.com/usegalaxy-eu/pulsar-deployment.git

#. Navigate in the pulsar-deployment directory:

   ::

     cd pulsar-deployment/tf

   we are going to edit ``vars.tf`` to customize your cluster's size, the images and naming.

#. Edit the ``vars.tf`` file to configure the Pulsar endpoint.

   .. note::

      All the variables available in the ``vars.tf`` file are described in the :doc:`vars_tf` section.

   .. list-table:: Title
      :widths: 25 25 50
      :header-rows: 1
   
      * - Variable
        - Default Value
        - Purpose
      * - image
        - ...
        - The name and the source url of the image to upload in your openstack environment
      * - name_prefix
        - vgcn-
        - Prefixed to the name of the VMs that are launched
      * - name_suffix
        - .usegalaxy.eu
        - This defaults to our domain, images do not need to be named as FQDNs but if you're using any sort of automated OpenStack DNS solution this can make things easier.
      * - flavors
        - ...
        - OpenStack flavors map that you will use to define resources of nova computing instances.
      * - exec_node_count
        - 2
        - Number of exec nodes.
      * - public_key
        - ...
        - SSH public key to use to access computing instances.
      * - secgroups_cm
        - ...
        - We have built some default rules for network access. Currently these are extremely broad, we may change that in the future.
      * - secgroups
        - <ingress-private>, <egress-public>
        - These are the secgroups for the exec nodes and the NFS node. <egress-public> will allow all egress to the entire internet. <ingress-private> allows ingress from other nodes with the same security group, only.
      * - public_network
        - public
        - The public network to connect the central manager to, terraform will **not** create this, so make sure to have it available in your OpenStack tenant.
      * - private_network
        - galaxy-net
        - The network to connect the nodes to, terraform will create this, but you need to specify a [private IP range](https://www.iana.org/help/private-addresses) in CIDR notation, that it can use on your tenant.
      * - nfs_disk_size
        - 3
        - NFS server disk size in GB.

   If you want to disable the built-in NFS server and supply your own, simply:

   #. Delete `nfs.tf`

   #. Change every autofs entry to point to your mount points and your NFS server's name/ip address.
