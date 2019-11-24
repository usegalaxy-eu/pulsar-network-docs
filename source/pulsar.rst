Pulsar configuration
====================

In this step we will describe how to configure the Pulsar endpoint and turn it on.
The RabbitMQ URL, described in the step before (:doc:`rabbitmq`), is needed to proceed.

Prerequisites: Ansible installation
-----------------------------------

To complete the pulsar configuration and turn it on, `Ansible <https://www.ansible.com>`_ is needed. Ansible can be easily installed following
the `documentation <https://docs.ansible.com/ansible/latest/installation_guide/intro_installation.html>`_.

Prerequisites: hostname configuration
-------------------------------------
As we need to refer to a proper fqdn hostname for your Pulsar server, if it doesn't has it you can easily create one into your local machine in this way:

Add the following line to your `/etc/hosts` file:

::

  <Central-Manager-Public-IP-address> it02.pulsar.galaxyproject.eu <pulsar-endpoint-name>

Where the ``Central-Manager-Public-IP-address`` is the public IP address of the Central Manager VM and the ``pulsar-endpoint-name`` is the custom name of your endpoint.

For example, on ``it02`` pulsar endpoint the fqdn hostname is: ``it02.pulsar.galaxyproject.eu``

Pulsar configuration
--------------------

# is a fork needed?

#. Clone the `Pulsar infrastructure playbook Github repository <https://github.com/usegalaxy-eu/pulsar-infrastructure-playbook>`_:

::

  git clone https://github.com/usegalaxy-eu/pulsar-infrastructure-playbook.git

#. Enter in the ``pulsar-infrastructure-playbook`` directory:

   ::

     cd pulsar-infrastructure-playbook

#. Update the ``inventory`` file, adding an entry for your Pulsar endpoint:

   ::

     [pulsarservers]
     ...
     <pulsar-endpoint-name> ansible_connection=ssh ansible_user=centos

   For example the it02 configuration is:

   ::

     [pulsarservers]
     ...
     it02.pulsar.galaxyproject.eu ansible_connection=ssh ansible_user=centos

#. Create a job metrics file, to define metrices that Galaxy collects during job run-time:

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

## is the UUID patch still needed?

   where the ``<rabbit_mq_usegalaxy.eu_url>`` is the RabbitMQ url, retrieved from the useGalaxy.eu admins.

#. Create a template yaml file for your endpoint:

   ::

     cp -r pulsar-infrastructure-playbook/de01.pulsar.galaxyproject.eu pulsar-infrastructure-playbook/<pulsar-endpoint-name>

#. It is now possibile to configure Pulsar using the Makefile.

   First of all, check if everything is fine with:

   ::

     $ make check

   and, finally, apply changes to Pulsare and turn it on:

   ::

     $ make apply
