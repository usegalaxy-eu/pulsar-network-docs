Requirements
============

This framework exploits `HashiCorp Terraform <https://www.terraform.io/>`_ to perform the installation and configuration of a Pulsar Endpoint on OpenStack.

The Terraform script needs to access an OpenStack cloud via API to:

- upload a VM image of 4GB (preferred, but we support also a preloading via the Dashboard interface);
- access an ipv4 external network (means a network that can reach internet);
- the external network has a DHCP server enabled that can provide 1 public IP;
- create an ipv4 private network for the VMs (preferred, but we support also to use a pre-existent network of this kind);
- create a router to bridge the private network to the external network (optional, if this feature is provided by the Cloud network infrastructure);
- create one ``central manager`` where all services are installed;
- create one NFS server;
- create N worker nodes;
- attach a storage volume to the NFS server;
- create three secgroups;
- upload an ssh public key to access the Central Manager VM.

Software needed
---------------

Make and Unzip are needed to follow this instruction. You can install them on a Linux machine as described here:

On Ubuntu 18.04
::

  # apt-get install make unzip

On CentOS 7
::

  # yum install make unzip

OpenStack configuration
-----------------------

Access to an OpensStack tenant is needed to perform the Pulsar endpoint installation.

To allow Terraform to access the Tenant download the **OpenStack RC File v2.0 (v3)** from your OpenStack dashboard and source it.

.. figure:: _static/img/horizon_rc_file_download.png
   :scale: 20%
   :align: center

::

  $ source pulsar-network-openrc.sh 

Pulsar need both private and public network to properly work. Moreover, the private network needs to access to the internet.
This is needed to allow the Pulsar Compute Nodes to mount CVMFS repositories to get access to Galaxy reference data or Containers.

VGCN image
----------

The Pulsar network exploits a Virtual Image, named `VGCN (Virtual Galaxy Compute Node)<https://github.com/usegalaxy-eu/vgcn>`_,
with everything necessary to create a Pulsar Network node. This image must be available in your Tenant.

Depending on the OpenStack configuration you will be available to upload the image by URL or not. In the first case the image
can be installed straightforwardly using the :doc:`pretasks` recipes.

.. figure:: _static/img/horizon_image_upload.png
   :scale: 40%
   :align: center

Alternatively, the OpenStack Horizon Dashboard allows to upload an image by URL or a local image. 

.. note::

   The current VCGN image is located `here <https://usegalaxy.eu/static/vgcn/vggp-v31-j132-4ab83d5ffde9-master.raw>`_.

