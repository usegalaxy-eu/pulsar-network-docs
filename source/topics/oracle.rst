Deploying a Pulsar Endpoint on Oracle Cloud Infrastructure
##########################################################

This guide describes how to deploy a **Pulsar Endpoint** on `Oracle Cloud Infrastructure <https://www.oracle.com/cloud/>`_, adapting the instructions from the original deployment tutorial for Openstack.
It creates the necessary networks, routers, routing tables and instances.

.. note::

    The current Terraform configuration was validated using the Free Trial quota limits provided by Oracle Cloud Infrastructure (OCI) during the testing period.

Most steps follow the same logic as the original guide, with a few differences documented here.

.. warning::
    The Terraform configuration provisions a working cluster but should be considered a **working baseline**.
    It serves as a starting point and may require optimization for long-term usage.



OCI Deployment Steps
********************

Some variables differ from the OpenStack version; the most changed ones are documented below.
OCI provides a wide range of options for networking, instance configuration, and block storage.
For a deeper understanding and additional customization, refer to the remaining Terraform files, the `Terraform Registry <https://registry.terraform.io/providers/oracle/oci/latest/docs>`_, and the official OCI documentation.


.. note::
   All steps not explicitly mentioned here remain the same as in the OpenStack guide.


Requirements
============

The steps described in :doc:`../requirements` and :doc:`../rabbitmq` remain the same. 
Replace the steps involving OpenStack with the ones described in the :ref:`Setup <setup-label>` section.


Prerequisites
=============

Before starting, ensure the following additional prerequisites:

- An Oracle Cloud Infrastructure (or OCI) account with required permissions
- CLI tools installed:

  - `OCI-CLI` (refer to the official `installation guide <https://docs.oracle.com/en-us/iaas/Content/API/SDKDocs/cliinstall.htm#Quickstart>`_)
  - qemu-utils

``qemu-utils`` can be installed by running:

.. code-block:: bash

  # Debian/Ubuntu:
  sudo apt update 
  sudo apt install qemu-utils

  # RHEL-compatible distros:
  sudo dnf install -y epel-release  
  sudo dnf install -y qemu-img  


.. _setup-label:

Setup
=====

The Terraform configuration is stored in the same repository, in the branch `oci`. Before proceeding, switch to it:

.. code-block:: bash

  git clone https://github.com/usegalaxy-eu/pulsar-infrastructure.git
  cd pulsar-deployment
  git checkout oci
  cd tf

Some of the differences when deploying on Oracle Cloud Infrastructure are:

1. **Custom Images**: OCI uses a different image format, therefore the VGCN images need to be processed before they can be used.
2. **Variables**: e.g. `shape` instead of `flavor`. For all the variables see the file ``tf/vars.tf``.
3. **Networking**: Oracle Cloud Infrastructure manages the network and its security differently from OpenStack, you can supply your own preferred security groups and lists in `secgroups.tf`.
4. **Cloud-Init**: The cloud-init files can be found at ``tf/cloud_init_templates`` and ``tf/files/nfs_cloud_init.yaml``.

All the resources created are described :ref:`here <resources-in-the-oci-deployment>`.

Custom Images
-------------

Oracle Cloud Infrastructure (OCI) requires custom VM images to be in either QCOW2 or VMDK format. If your image is in a different format (e.g., raw, VDI, VHD), you can convert it using qemu-img. For example:

.. code-block:: bash

    qemu-img convert -f raw -O qcow2 vgcn-image.raw vgcn-image.qcow2
    # or
    qemu-img convert -f raw -O vmdk vgcn-image.raw vgcn-image.vmdk

Meanwhile create an OCI `bucket <https://docs.oracle.com/en-us/iaas/Content/Object/Tasks/managingbuckets_topic-To_create_a_bucket.htm#top>`_.
Once the conversion is completed and the image tested, it can be `uploaded into the bucket <https://docs.oracle.com/en-us/iaas/Content/Object/Tasks/managingobjects_topic-To_upload_objects_to_a_bucket.htm>`_ previously made.

Lastly, import the converted VGCN image following the official `Oracle Documentation <https://docs.oracle.com/en-us/iaas/Content/Compute/Tasks/custom-images-import.htm#listing-custom-images>`_.
Take note of the custom image OCID, it will be used while editing the ``tf/vars.tf`` file.

.. note::
    At the time of drafting, there are no VGCN images with pre-installed Oracle monitoring agents; therefore Oracle monitoring functionality will not be fully functional. 


Configuration
=============

Make and store in a safe location the following bash script filled with the OCI credentials. Source it before executing any Terraform command:

.. code-block:: bash

    export TF_VAR_oracle_vars='{
    tenancy_ocid = "...",
    user_ocid = "...",
    fingerprint = "...",
    private_key_path = "...",
    region = "...",
    availability_domain = "...",
    compartment_id = "..."
    }'

These credentials are the same as those used by OCI-CLI. 


Terraform variables
-------------------

The following variables are found in ``tf/vars.tf`` and must be adjusted accordingly to the desired infrastructure.


``oracle-vars``
"""""""""""""""

  :Description:
      OCI credentials. This variable is determined by the environment variable ``TF_VAR_oracle_vars`` exported with the script above. It can be left unedited.



``shapes``
""""""""""

  :Description:
      Describes the shape of the instances, somewhat equivalent to OpenStack's flavors.
      The current configuration take advantage of the "VM.Standard.E3.Flex" shape. 
      To use different shapes, make sure to edit the ``shape_config`` attributes of all ``oci_core_instance`` resources accordingly.

  :Defaults:

      .. code-block::

        default = {
            "central-manager" = {
            shape         = "VM.Standard.E3.Flex"
            ocpus         = 2
            memory_in_gbs = 16
            },
            "nfs-server" = {
            shape         = "VM.Standard.E3.Flex"
            ocpus         = 2 
            memory_in_gbs = 32
            },
            "exec-node" = {
            shape         = "VM.Standard.E3.Flex"
            ocpus         = 4
            memory_in_gbs = 32
            },
            "gpu-node" = {
            shape         = "VM.Standard.E3.Flex"
            ocpus         = 8
            memory_in_gbs = 64
            }
        }
        }


``secgroups_cm``
""""""""""""""""

  :Description:
      The names of the security groups of the Central Manager.


``secgroups``
"""""""""""""

  :Description:
      The names of the security groups of the workers and NFS.


``main_vcn``
""""""""""""

  :Description:
      Main Virtual Cloud Network (VCN) configuration.
      The `cidr4` is also used to calculate distinct private and public subnet CIDR blocks. The `display_name` is a human-readable label for the VCN.

``private_network``
"""""""""""""""""""

  :Description:
      Configuration for the internal private subnet, used for communication within the cluster.

``public_network``
"""""""""""""""""""

  :Description:
      Configuration for a subnet with external internet access. Used by the Central Manager Node.
      Attach this network if you require public IPs for specific nodes.



.. warning::

    GPU worker nodes were excluded from testing due to GPU resources not being included in the Free Trial quota allowances.


Resources in the OCI Deployment
*******************************

.. _resources-in-the-oci-deployment:

Networking
==========

Networks and subnets
--------------------

`main`
""""""

  :Resource:
      oci_core_vcn

  :Description:
      Creates the main Virtual Cloud Network (VCN) with the specified CIDR block and DNS label "mainvcn".

  :File:
      ``pulsar-deployment/tf/int_network.tf``


`private_subnet`
""""""""""""""""

  :Resource:
      oci_core_subnet

  :Description:
      Defines a private subnet within the VCN, using the calculated CIDR. Public IPs are disabled, and it's associated with a private route table and security list.
      The NFS server and all the Pulsar nodes are attached to this subnet.

  :File:
      ``pulsar-deployment/tf/int_network.tf``


`public_subnet`
"""""""""""""""

  :Resource:
      oci_core_subnet

  :Description:
      Defines a public subnet within the VCN, allows public IPs, and is linked to an internet-facing route table and a corresponding security list.
      The Central Manager is attached to this subnet along with other VMs that need internet access and exposure.

  :File:
      ``pulsar-deployment/tf/int_network.tf``



Gateways
--------

`service_gateway`
"""""""""""""""""

  :Resource:
      oci_core_service_gateway

  :Description:
      A Service Gateway resource in OCI, which allows traffic between Virtual Cloud Network (VCN) and Oracle services without traversing the internet.

  :File:
      ``pulsar-deployment/tf/ext_network.tf``


`internet_gateway`
""""""""""""""""""

  :Resource:
      oci_core_internet_gateway

  :Description:
      Creates an Internet Gateway attached to the VCN, enabling public access for resources in the public subnet.

  :File:
      ``pulsar-deployment/tf/int_network.tf``


`nat_gw`
""""""""

  :Resource:
      oci_core_nat_gateway

  :Description:
      Defines a NAT Gateway for the VCN to allow instances in private subnets connections to the internet without being exposed.

  :File:
      ``pulsar-deployment/tf/int_network.tf``



Route Tables
------------

`internet_route_table`
""""""""""""""""""""""

  :Resource:
      oci_core_route_table

  :Description:
      Associates a route table with the public subnet that directs all traffic (0.0.0.0/0) to the Internet Gateway.

  :File:
      ``pulsar-deployment/tf/int_network.tf``


`combined_private_route`
""""""""""""""""""""""""

  :Resource:
      oci_core_route_table

  :Description:
      Defines the route table for the private subnet with two rules:
        - Sending outbound traffic to the NAT Gateway;
        - Accessing OCI services via the Service Gateway.

  :File:
      ``pulsar-deployment/tf/int_network.tf``


Security lists
--------------

All the resources below can be edited accordingly to increase security strictness.


`pvt_security_list`
"""""""""""""""""""

  :Resource:
      oci_core_security_list

  :Description:
      Security list for the private subnet, allowing all traffic from the public subnet and permitting unrestricted outbound access.
      Can be edited to be more strict.

  :File:
      ``pulsar-deployment/tf/int_network.tf``


`pub_security_list`
"""""""""""""""""""

  :Resource:
      oci_core_security_list

  :Description:
      Security list for the public subnet, allowing all traffic from the private subnet and also permitting unrestricted outbound access.
      Can be edited to be more strict.
      
  :File:
      ``pulsar-deployment/tf/int_network.tf``



Security groups and rules
-------------------------

All the resources below can be edited accordingly to increase security strictness.

`cm_ingress_private`
""""""""""""""""""""

  :Resource:
      oci_core_network_security_group

  :Description:
      Creates a Network Security Group (NSG) for the Central Manager to allow any incoming traffic from members of the same group (i.e., other VMs in the private network).

  :File:
      ``pulsar-deployment/tf/secgroups.tf``


`cm_ingress_private_rule`
"""""""""""""""""""""""""

  :Resource:
      oci_core_network_security_group_security_rule

  :Description:
      Defines an ingress rule allowing all protocols and ports from the same NSG (``cm_ingress_private``), effectively allowing intra-group communication.

  :File:
      ``pulsar-deployment/tf/secgroups.tf``


`cm_egress_public`
""""""""""""""""""

  :Resource:
      oci_core_network_security_group

  :Description:
      Creates a NSG for the Central Manager to allow outbound communication.

  :File:
      ``pulsar-deployment/tf/secgroups.tf``



`cm_egress_public_rule`
"""""""""""""""""""""""

  :Resource:
      oci_core_network_security_group_security_rule

  :Description:
      Defines an egress rule allowing all outbound traffic to any destination (0.0.0.0/0), enabling the CM to reach the internet or other services.

  :File:
      ``pulsar-deployment/tf/secgroups.tf``


`public_ssh`
""""""""""""

  :Resource:
      oci_core_network_security_group

  :Description:
      Creates a NSG dedicated to allowing SSH access.

  :File:
      ``pulsar-deployment/tf/secgroups.tf``


`public_ssh_rule`
"""""""""""""""""

  :Resource:
      oci_core_network_security_group_security_rule

  :Description:
      Adds an ingress rule to allow SSH traffic on TCP port ``var.ssh-port``, from any IP address (0.0.0.0/0). It is intended for remote access to public-facing nodes.

  :File:
      ``pulsar-deployment/tf/secgroups.tf``


`ingress_private`
"""""""""""""""""

  :Resource:
      oci_core_network_security_group

  :Description:
      Defines a NSG for worker nodes to allow inbound traffic from other nodes within the same private group.

  :File:
      ``pulsar-deployment/tf/secgroups.tf``


`ingress_private_rule`
""""""""""""""""""""""

  :Resource:
      oci_core_network_security_group_security_rule

  :Description:
      Allows unrestricted ingress from other members of the same NSG (``ingress_private``), essential for internal protocols like NFS and HTCondor (9618).

  :File:
      ``pulsar-deployment/tf/secgroups.tf``


`egress_public`
"""""""""""""""

  :Resource:
      oci_core_network_security_group

  :Description:
      Creates a NSG for worker nodes to permit outbound connections.

  :File:
      ``pulsar-deployment/tf/secgroups.tf``
      

`egress_public_rule`
""""""""""""""""""""

  :Resource:
      oci_core_network_security_group_security_rule

  :Description:
      Allows all outbound traffic to any destination for nodes in the private network.

  :File:
      ``pulsar-deployment/tf/secgroups.tf``
      

`ingress_private_from_cm_rule`
""""""""""""""""""""""""""""""

  :Resource:
      oci_core_network_security_group_security_rule

  :Description:
      Grants ingress permission to worker nodes from the Central Manager\’s private NSG, enabling communication from CM to internal nodes.

  :File:
      ``pulsar-deployment/tf/secgroups.tf``
      


`cm_ingress_private_from_worker_rule`
"""""""""""""""""""""""""""""""""""""

  :Resource:
      oci_core_network_security_group_security_rule

  :Description:
      Grants ingress permission to the Central Manager from the worker nodes\’ NSG, allowing communication from workers to the CM.

  :File:
      ``pulsar-deployment/tf/secgroups.tf``
      

Misc
----

`all_oci_services`
""""""""""""""""""

  :Resource:
      oci_core_services

  :Description:
      A data source that fetches information about all Oracle Cloud Infrastructure services available in the Oracle Services Network.

  :File:
        ``pulsar-deployment/tf/ext_network.tf``




Storage
=======
Block Volumes
-------------

`volume_nfs_data`
"""""""""""""""""

  :Resource:
      oci_core_volume

  :Description:
      Creates a block volume in the specified availability domain for storing NFS-shared data. The size is defined via ``var.nfs_disk_size``.

  :File:
        ``pulsar-deployment/tf/nfs.tf``

Block Volumes Attachments
-------------------------

`nfs_data_attachment`
"""""""""""""""""""""

  :Resource:
      oci_core_volume_attachment

  :Description:
      Attaches the previously created block volume to the NFS server instance using paravirtualized attachment, enabling the instance to access the volume as a block device.

  :File:
        ``pulsar-deployment/tf/nfs.tf``




Instances
=========

Non-worker nodes
----------------

`nfs_server`
""""""""""""

  :Resource:
      oci_core_instance

  :Description:
      Provisions a compute instance to act as a NFS server with the following settings:

        - Shape and performance defined via the ``var.shapes["nfs-server"]`` map.
        - Boot configuration is set to paravirtualized.
        - Image is selected dynamically using data source.
        - Networking attaches the instance to the private subnet with security groups for private ingress and public egress.
        - Metadata injects a SSH key and a base64-encoded cloud-init script for automatic configuration.
        - Volume dependency ensures the instance is only created after the NFS volume is ready.


  :File:
     ``pulsar-deployment/tf/nfs.tf``


`central_manager`
"""""""""""""""""

  :Resource:
      oci_core_instance

  :Description:
      Creates the HTCondor Central Manager instance with:
        - Custom compute resources from ``var.shapes["central-manager"]``.
        - Public subnet networking with assigned public IP and attached to security groups allowing outbound traffic and SSH access.
        - Boot configuration is set to paravirtualized.
        - Metadata injects a SSH key and a base64-encoded cloud-init script for automatic configuration.

  :Network Attachment:
      `oci_core_vnic_attachment.manager_private_vnic`:
        - Attaches a second (private) VNIC to the Central Manager, enabling communication within the private subnet and the rest of the cluster.
        - Scoped to private security groups.
        - Triggers its replacement if the instance changes, ensuring network configuration stays aligned.

  :Provisions:
      `null_resource.provision_central_manager`:
          - Runs a local-exec provisioner to configure the instance using Ansible;
          - Network attachment dependency ensures the instance is only provisioned after being connected to the same network of the NFS server.
          - Triggers its replacement if the instance changes, ensuring the correct configuration.

  :File:
     ``pulsar-deployment/tf/main.tf``



Worker nodes
------------

`exec-node`
"""""""""""

  :Resource:
      oci_core_instance

  :Description:
      Provisions a configurable number (``var.exec_node_count``) of execution nodes with:
        - Custom CPU and memory from ``var.shapes["exec-node"]``.
        - Private subnet placement (no public IP) for internal-only access.
        - Security groups allowing internal ingress and external egress.
        - Metadata injects a SSH key and a base64-encoded cloud-init script for automatic configuration.
        - Dynamic display names incorporating count index for clarity (exec-node-0, exec-node-1, etc.).

  :File:
     ``pulsar-deployment/tf/exec_nodes.tf``


`gpu-node`
""""""""""

  :Resource:
      oci_core_instance

  :Description:
      Provisions a configurable number (``var.gpu_node_count``) of execution nodes with:
        - Custom CPU and memory from ``var.shapes["gpu-node"]``.
        - Private subnet placement (no public IP) for internal-only access.
        - Security groups allowing internal ingress and external egress.
        - Metadata injects a SSH key and a base64-encoded cloud-init script for automatic configuration.
        - Dynamic display names incorporating count index for clarity (gpu-node-0, gpu-node-1, etc.).

  :File:
     ``pulsar-deployment/tf/gpu_nodes.tf``

