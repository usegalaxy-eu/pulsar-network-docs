Pre-tasks
=========

The pre_tasks recipes will create Pulsar needed resources, i.e. the virtual image, the private network and the router, which will be used during the Pulsar Network nodes configuration.

Prerequisites
*************

Make and Unzip are needed to configure a pulsar node, so you need to install them:

::

  # Ubuntu 16.04
  # apt-get install make unzip

  # CentOS 7
  # yum install make unzip

Pre-tasks configuration
***********************

.. warning::

   Source the tenant RC file before to start the installation procesure, otherwise Terraform will not be able to perform Resources creation and configuration.

#. Fork the usegalaxy.eu GitHub repoitory `pulsar-infrastructure-playbook <https://github.com/usegalaxy-eu/pulsar-infrastructure-playbook>`_.

#. Locally clone the forked repository:

   ::

     git clone https://github.com/<repository-name>/pulsar-infrastructure.git

#. Navigate in the pulsar-infrastructure directory:

   ::

     cd pulsar-infrastructure

#. Create a directory with the pre-tasks Terraform recipes using the Makefile. The Makefile exploits the `Terraform workspaces <https://www.terraform.io/docs/cloud/workspaces/index.html>`_, defining the workspace name using the environemnt variable ``WS``.

   To automatically setup your environment use the MakeFile and define ``WS`` before the make command, e.g. ``WS=<workspace-name> make pre_tasks``. A new directory is created, named ``<workspace_name>`` with the terraform files needed to run the pre_tasks configuration.

   For example the command:
   ::

     WS=test01 make pre_tasks

   will create a directory named ``test01`` with the following files:

   ::

     $ ls
     ext_network.tf pre_tasks.tf providers.tf vars.tf 

#. Edit the ``<workspace-name>/pre_tasks.tf`` recipe. It hase three sections:

   - Upload the virtual machine image via OpenStack API. This block should be commented if the image is already available on your tenant or if you upload it via the dashboard interface.

   - Create private network. This block should be commented if network is already available.

   - Create a router to ensure the private network will be able to reach the Internet. This block should be commented if this feature is provided by the network

#. Edit the ``<workspace-name>/vars.tf`` file to confiugre the Pulsar endpoint.

   .. note::

      All the variables available in the ``vars.tf`` file are described in the :doc:`vars_tf` section.

#. Validate the terraform recipes configuration: 

   ::

     WS=test01 make plan

#. Run the pre-stasks recipes:

   ::

     WS=test01 make apply

