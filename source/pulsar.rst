Pulsar configuration
====================

In this step we will describe how to configure the Pulsar endpoint and turn it on.
The RabbitMQ URL, described in the step before (:doc:`rabbitmq`), is needed to proceed.

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

# is a fork needed?

#. Clone the `Pulsar infrastructure playbook Github repository <https://github.com/usegalaxy-eu/pulsar-infrastructure-playbook>`_:

   ::

     git clone https://github.com/usegalaxy-eu/pulsar-infrastructure-playbook.git

#. Creates several needed files

   Enter in the ``pulsar-infrastructure-playbook`` directory:

   ::

     cd pulsar-infrastructure-playbook

   and execute:

   ::

     make preparation FQDN=pulsar-fqdn-hostname

   "Make" makes these changes:

     - Updates the ``inventory`` file, adding an entry for your Pulsar endpoint
     - creates a job_metrics file into file/``pulsar-fqdn-hostname``
     - creates an host var file into host_vars/``pulsar-fqdn-hostname``
     - Create a template yaml file for your endpoint into templates/``pulsar-fqdn-hostname``


#. Revise/update all the files created in the step above accordingly with your setup requirements.

#. Configure the Pulsar endpoint using the Makefile.

   First of all, check if everything is fine with:

   ::

     $ make check

   and, finally, apply changes to Pulsar and turn it on:

   ::

     $ make apply

