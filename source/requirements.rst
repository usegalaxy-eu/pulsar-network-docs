Requirements
============

Software needed
---------------

Ansible, Make and Unzip are needed to follow this instruction.

`Ansible <https://www.ansible.com>`_  can be easily installed following the `documentation <https://docs.ansible.com/ansible/latest/installation_guide/intro_installation.html>`_.


Make and Unzip can be installed on a Linux machine as described here:

On Ubuntu 18.04
::

  # apt-get install make unzip

On CentOS 7
::

  # yum install make unzip

OpenStack configuration
-----------------------

Access to an OpensStack tenant is needed to perform the Pulsar endpoint installation.

To allow Terraform to access the Tenant download the **OpenStack RC File v2.0 (v3)** and source it.

.. figure:: _static/img/horizon_rc_file_download.png
   :scale: 20%
   :align: center

::

  $ source pulsar-network-openrc.sh 

Pulsar need both private and public network to properly work. Moreover, the private network needs to access to the internet, basically allowing the Pulsar Compute Nodes to mount CVMFS repositories.

VGCN image
----------

The Pulsar network exploits a Virtual Image, named `VGCN <https://github.com/usegalaxy-eu/vgcn>`_, with everything necessary to create a Pulsar Network node already inside, like:

    - HTCondor
    - Pulsar
    - NFS
    - CVMFS
    - Singularity
    - Docker

This image must be available in your Tenant.

Depending on the OpenStack configuration you will be available to upload the image by URL or not. In the first case the image can be installed straightforwardly using the :doc:`pretasks` recipes.

.. figure:: _static/img/horizon_image_upload.png
   :scale: 40%
   :align: center

Alternatively the OpenStack Horizon Dashboard allows to upload it by URL or uploading a local image. 

.. note::

   The current VCGN image is located `here <https://usegalaxy.eu/static/vgcn/vggp-v31-j132-4ab83d5ffde9-master.raw>`_.

