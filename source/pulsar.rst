Pulsar configuration
====================

In this step will be described how to configure the Pulsar endpoint and turn it on.
The RabbitMQ URL, described in the step before, is mandatory to keep going.

Prerequisites
-------------

hostname configuration
~~~~~~~~~~~~~~~~~~~~~~
We need to refer to a proper `FQDN <https://en.wikipedia.org/wiki/Fully_qualified_domain_name>`_ hostname for your Pulsar server, if it doesn't has it you can easily create one into your local machine in this way:

Add the following line to your `/etc/hosts` file:

::

  <Central-Manager-Public-IP-address> <pulsar-fqdn-hostname> <pulsar-endpoint-name>

Where the ``Central-Manager-Public-IP-address`` is the public IP address of the Central Manager VM, ``<pulsar-fqdn-hostname>`` is the FQDN hostname and the ``pulsar-endpoint-name`` is the custom name of your endpoint.

For example:

- ``it02`` is the ``pulsar-endpoint-name``
- ``it02.pulsar.galaxyproject.eu`` is the ``<pulsar-fqdn-hostname>``
- ``90.147.170.170`` is the ``Central-Manager-Public-IP-address``

Pulsar configuration
--------------------

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

#. Create job metrics file:

   ::

     cd pulsar-infrastructure-playbook/files

     mkdir <pulsar-endpoint-name>

     cp job_metrics_conf.xml <pulsar-endpoint-name>/

#. Create the host var file for your Pulsar endpoint ``pulsar-infrastructure-playbook/host_vars/<pulsar-endpoint-name`` with your favourite editor and add the following lines:

   ::

     message_queue_url : "<rabbit_mq_usegalaxy.eu_url>"

   where the ``<rabbit_mq_usegalaxy.eu_url>`` is the RabbitMQ url, retrieved from useGalaxy.eu. If you need to upload this file to a public git repository, take care to encrypt it before using `Ansible Vault <https://docs.ansible.com/ansible/latest/user_guide/vault.html>`_.

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

