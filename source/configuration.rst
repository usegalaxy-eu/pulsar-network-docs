Configure the pulsar endpoint
=============================

This step will create several Pulsar needed resources on the cloud infrastructure, which will be used during the nodes configuration:

  - the virtual image
  - the private network
  - the router

This step can be skipped if the resources above are available or provided by your cloud infrastructure in other ways. More details below.

.. note::

   A SSH key pair is needed: the public key has to be configured in the vars.tf as public_key entry. The private key is needed in the terraform apply step. You can create a SSH key with the command ``ssh-keygen``.

Configuration
-------------

.. warning::

   Source the tenant RC file (see section :doc:`requirements`) before to start the installation procedure, otherwise Terraform will not be able to perform resources creation and configuration.

#. Clone the usegalaxy.eu GitHub repository `pulsar-deployment <https://github.com/usegalaxy-eu/pulsar-deployment>`_.

   ::

     git clone https://github.com/usegalaxy-eu/pulsar-infrastructure.git

#. Navigate in the pulsar-deployment directory:

   ::

     cd pulsar-deployment/tf

   we are going to edit two files: ``pre_tasks.tf`` and ``vars.tf``

#. Edit the ``pre_tasks.tf`` file accordingly with your needs. It has three sections:

   - Upload the virtual machine image via OpenStack API. This block should be commented if the image is already available on your tenant or if you upload it via the dashboard interface.

   - Create private network. This block should be commented if network is already available.

   - Create a router to ensure the private network will be able to reach the Internet. This block should be commented if this feature is provided by the network

#. Edit the ``vars.tf`` file to configure the Pulsar endpoint.

   .. note::

      All the variables available in the ``vars.tf`` file are described in the :doc:`vars_tf` section.

   |Variable          | Default Value          | Purpose|
   |--------          | -------------          | -------|
   |image             | ...                    | The name and the source url of the image to upload in your openstack environment
   |`name_prefix`     | `vgcn-`                | Prefixed to the name of the VMs that are launched
   |`name_suffix`     | `.usegalaxy.eu`        | This defaults to our domain, images do not need to be named as FQDNs but if you're using any sort of automated OpenStack DNS solution this can make things easier.
   |`flavors`         | ...                    | OpenStack flavors map that you will use to define resources of nova computing instances.
   |`exec_node_count` | `2`                    | Number of exec nodes.
   |`public_key`      | ...                    | SSH public key to use to access computing instances.
   |`secgroups`       | ...                    | We have built some default rules for network access. Currently these are extremely broad, we may change that in the future. Alternatively you can supply your own preferred security groups here.
   |`network`         | `galaxy-net`           | The network to connect the nodes to.
   |`nfs_disk_size`   | `3`                    | NFS server disk size in GB.

If you want to disable the built-in NFS server and supply your own, simply:

1. Delete `nfs.tf`
2. Change every autofs entry to point to your mount points and your NFS
   server's name/ip address.

#. Validate the terraform recipes configuration:

   ::

     WS=test01 make plan

#. Run the pre-stasks recipes:

   ::

     WS=test01 make apply


The resources described in to the ``pre_tasks.tf`` are now created on your Openstack tenant.
