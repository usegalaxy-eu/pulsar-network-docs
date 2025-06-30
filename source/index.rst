.. Pulsar-Network documentation master file, created by
   sphinx-quickstart on Thu Nov 21 12:01:00 2019.
   You can adapt this file completely to your liking, but it should at least
   contain the root `toctree` directive.

Welcome to Pulsar-Network's documentation!
==========================================

The Pulsar Network is wide job execution system distributed across several
European datacenters, allowing to scale Galaxy instances computing power
over heterogeneous resources.

.. figure:: _static/img/eosc_esg_logo.svg
   :scale: 30%
   :align: center

Currently, the Pulsar Network is developed in the context of the `Horizon Europe
EuroScienceGateway project <https://galaxyproject.org/projects/esg/>`_,
enabling end-users to easily access and exploit remote compute resources,
and resource providers to offer compute resources for scientific
purposes through the Galaxy interface.

.. figure:: _static/img/esg_pulsar_network_contrib.png
   :scale: 40%
   :align: center

This documentation shows how to install and configure a Pulsar network
endpoint on an OpenStack Cloud infrastructure and how to connect it to
useGalaxy.* server. The same Pulsar endpoint can be associated to any Galaxy
instance, if properly configured.

When you reach to the end of this document, you'll have an operational Pulsar
endpoint running on your OpenStack tenant. The document covers from setting up of
OpenStack details, and required steps to install the endpoint with Terraform on
this created OpenStack space, using HTCondor as default Resource Manager system.
Nevertheless, any Pulsar enpoint, exploiting either Cloud or HPC resurces,
could be connected to the Pulsar Network.

.. toctree::
   :maxdepth: 2
   :caption: Pulsar Endpoint Configuration

   introduction
   requirements
   rabbitmq
   configuration
   build_pulsar_node
   vars_tf
   multi_pulsar

.. toctree::
   :maxdepth: 2
   :caption: Post Deployment

   post_deploy_pulsar_services
   rm_multi_pulsar

.. toctree::
   :maxdepth: 2
   :caption: Special Topics

   Benchmark <topics/benchmark>
   GPUs <topics/gpus>
   Staging actions <topics/staging>
   Jump host <topics/jump_host>
   Oracle Cloud Infrastructure <topics/oracle>

.. toctree::
   :maxdepth: 2
   :caption: About Project

   Partners <project/partners>
   Status <project/status>

Indices and tables
==================

* :ref:`genindex`
* :ref:`modindex`
* :ref:`search`
