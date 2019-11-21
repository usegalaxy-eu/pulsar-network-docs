Pulsar configuration
====================

The RabbitMQ URL is mandatory for Pulsar endpoint configuration. Once obtained it is possible to configure Pulsar and turn it on.

Prerequisites: Ansible installation
-----------------------------------

To complete the pulsar configuration and turn it on, `Ansible <https://www.ansible.com>`_ is needed. Ansible can be easily installed following the `documentation <https://docs.ansible.com/ansible/latest/installation_guide/intro_installation.html>`_.

Prerequisites: hostname configuration
-------------------------------------

Add the following line to your `/etc/hosts` file:

::

  <Central-Manager-Public-IP-address> it02.pulsar.galaxyproject.eu <pulsar-endpoint-name>

Where the ``Central-Manager-Public-IP-address`` is the public IP address of the Central Manager VM and the ``pulsar-endpoint-name`` is the custom name of your endpoint.

For instance, on ``it02`` pulsar endpoint name is: ``it02.pulsar.galaxyproject.eu``

Pulsar configuration
--------------------

#. Clone the `Pulsar infrastructure playbook Github repository <https://github.com/usegalaxy-eu/pulsar-infrastructure-playbook>`_:

::

  git clone https://github.com/usegalaxy-eu/pulsar-infrastructure-playbook.git

#. Enter in the ``pulsar-infrastructure-playbook`` directory:

   ::

     cd pulsar-infrastructure-playbook

#. Change the inventory file, adding an entry for your Pulsar endpoint:

   ::

     [pulsarservers]
     ...
     <pulsar-endpoint-name> ansible_connection=ssh ansible_user=centos

   For example the it02 configuration is:

   ::

     [pulsarservers]
     ...
     it02.pulsar.galaxyproject.eu ansible_connection=ssh ansible_user=centos

#. Create job metrics file:

   ::

     cd pulsar-infrastructure-playbook/files

     mkdir <pulsar-endpoint-name>

     cp job_metrics_conf.xml <pulsar-endpoint-name>/

#. Create the host var file for your Pulsar endpoint ``pulsar-infrastructure-playbook/host_vars/<pulsar-endpoint-name`` with your favourite editor and add the following lines:

   ::

     message_queue_url : "<rabbit_mq_usegalaxy.eu_url>"
     
     copy_metrics_plugins: false
     
     apply_patches: true
     patches_to_apply:
       - { 'src': 'patches/uuid.patch', 'basedir': '/opt/pulsar/venv/lib/python2.7/site-packages', 'state': 'present', 'backup': 'yes' }\
       - { 'src': 'patches/params_submission.patch', 'basedir': '/opt/pulsar/venv/lib/python2.7/site-packages', 'state': 'present', 'backup': 'yes' }
       - { 'src': 'patches/hostname.patch', 'basedir': '/opt/pulsar/venv/lib/python2.7/site-packages', 'state': 'present', 'backup': 'yes' }

   where the ``<rabbit_mq_usegalaxy.eu_url>`` is the RabbitMQ url, retrieved from useGalaxy.eu.

#. Create a template yaml file for your endpoint:

   ::

     cp -r pulsar-infrastructure-playbook/de01.pulsar.galaxyproject.eu pulsar-infrastructure-playbook/<pulsar-endpoint-name>

#. It is now possibile to configure Pulsar using the Makefile. First of all, check if everything is fine with:

   ::

     $ make check

   and, finally, apply changes to Pulsare and turn it on:

   ::

     $ make apply
