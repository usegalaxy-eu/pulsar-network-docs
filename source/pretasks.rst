Preparation
===========

This step will create several Pulsar needed resources on the cloud infrastructure, which will be used during the nodes configuration:

  - the virtual image
  - the private network
  - the router

This step can be skipped if the resources above are available or provided by your cloud infrastructure in other ways. More details below.

Pre-tasks
---------

.. warning::

   Source the tenant RC file (see section :doc:`requirements`) before to start the installation procedure, otherwise Terraform will not be able to perform resources creation and configuration.

#. Fork the usegalaxy.eu GitHub repository `pulsar-infrastructure <https://github.com/usegalaxy-eu/pulsar-infrastructure>`_.

#. Locally clone the forked repository:

   ::

     git clone https://github.com/<user-name>/pulsar-infrastructure.git

#. Navigate in the pulsar-infrastructure directory:

   ::

     cd pulsar-infrastructure

#. Execute the pre_tasks step using the Makefile.

   The Makefile exploits the `Terraform workspaces <https://www.terraform.io/docs/cloud/workspaces/index.html>`_, defining the workspace name using the environment variable ``WS``.

   To automatically setup your environment set ``WS`` before the make command, e.g. ``WS=<workspace-name> make pre_tasks``.
   A new directory is created, named ``<workspace_name>`` with the terraform files needed to run the pre_tasks configuration.


   Choose a label for your Terraform environment, for example `test01`:
   ::

     WS=test01 make pre_tasks

   it will create a directory named ``test01`` with the following files:

   ::

     $ ls test01
     ext_network.tf pre_tasks.tf providers.tf vars.tf


#. Edit the ``<workspace-name>/pre_tasks.tf`` file accordingly with your needs. It has three sections:

   - Upload the virtual machine image via OpenStack API. This block should be commented if the image is already available on your tenant or if you upload it via the dashboard interface.


   - Create private network. This block should be commented if network is already available.

   - Create a router to ensure the private network will be able to reach the Internet. This block should be commented if this feature is provided by the network

#. Edit the ``<workspace-name>/vars.tf`` file to configure the Pulsar endpoint.

   .. note::

      All the variables available in the ``vars.tf`` file are described in the :doc:`vars_tf` section.

#. Validate the terraform recipes configuration:

   ::

     WS=test01 make plan

#. Run the pre-stasks recipes:

   ::

     WS=test01 make apply


The resources described in to the ``pre_tasks.tf`` are now created on your Openstack tenant.
