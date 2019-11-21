Variables configuration
=======================

Each pulsar endpoint can be configured editing the ``vars.tf``, located in the terraform worksapce directory.

Navigate in the Pulsar infrastructure directory:

   ::

     cd pulsar-infrastructure

and edit the file ``<workspace-name>/vars.tf`` with your favourite text ediitor.


Configuration options
*********************

-----------------
``nfs_disk_size``
-----------------

:Description: Size of the NFS storage drive. We suggest at least 300 GB of storage.

:Example:

	::

	  variable "nfs_disk_size" {
	    default = 300
	  }

-----------
``flavors``
-----------

:Description:
	Instance flavours names of the available flavours on your OpenStack tenant.
	``central-manager`` and ``nfs-server`` have minimal requirements, e.g. 4 virtual CPUs and 8 GB of RAM for each of them.
        The ``exec-node`` configures the HTCondor worker nodes. Its configuration depends on the availability on your Cloud provider. At least 16 vCPUs and 32 GB RAM are recommended. 
	The ``gpu-node`` identifies the size of the GPU nodes.

:Example:
	::

	  variable "flavors" {
	    type = "map"
	    default = {
	      "central-manager" = "m1.medium"
	      "nfs-server" = "m1.medium"
	      "exec-node" = "m1.medium"
	      "gpu-node" = "m1.medium"
	    }
	  }

-------------------
``exec_node_count``
-------------------

:Description:
	Set the number of HTCondor Compute Worker Nodes.

:Example:

	::
	  
	  variable "exec_node_count" {
	    default = 2
	  }

------------------
``gpu_node_count``
------------------

:Description:
        Set the number of HTCondor GPU Worker Nodes.

:Example:
	::

	  variable "gpu_node_count" {
	    default = 0
	  }
	 
---------
``image``
---------

:Description:
	Set the VGCN image to use. Terraform expects to find on OpenStack with the name specified in this field.
	More on VGCN image `here <https://github.com/usegalaxy-eu/vgcn>`_.
	This field can be left unchanged.

:Example:
	::
 
	  variable "image" {
	    type = "map"
	    default = {
	      "name" = "vggp-v31-j132-4ab83d5ffde9-master"
	      "image_source_url" = "https://usegalaxy.eu/static/vgcn/vggp-v31-j132-4ab83d5ffde9-master.raw"
	      "container_format" = "bare"
	      "disk_format" = "raw"
	     }
	  }

--------------
``public_key``
--------------

:Description:
	Defines a SSH public key, which will be inserted in the Central Manager VM, allowing the user to access it.
	The SSH public key has to be copied in the ``pubkey`` field.

:Example:	 
	::
 
	  variable "public_key" {
	    type = "map"
	    default = {
	      name = "key_label"
	      pubkey = "ssh-rsa blablablabla..."
	    }
	  }

---------------
``name_prefix``
---------------

:Description:
	Defines the name prefix for the resources created on OpenStack.
	This field can be left unchanged.

:Example:
	::
 
	  variable "name_prefix" {
	    default = "vgcn-"
	  }

-------------------
``name_suffix``
-------------------

:Description:
        Defines the name suffix for the resources created on OpenStack.
	This field can be left unchanged.

:Example:
	::
	  
	  variable "name_suffix" {
	    default = ".usegalaxy.eu"
	  }

-------------------
``secgroups_cm``
-------------------

:Description:
	Defines the security groups of the Central Manager VM.
	This field can be left unchanged.

:Example:
	::
	  
	  variable "secgroups_cm" {
	    type = "list"
	    default = [
	      "vgcn-public-ssh",
	      "vgcn-ingress-private",
	      "vgcn-egress-public",
	    ]
	  }

-------------
``secgroups``
-------------

:Description:
	Defines the security groups for NFS server, Compute Nodes nodes and GPU nodes.
	This field can be left unchanged.

:Example:
	::
	  
	  variable "secgroups" {
	    type = "list"
	    default = [
	      "vgcn-ingress-private",
	      "vgcn-egress-public",
	    ]
	  }

------------------
``public_network``
------------------

:Description:
	Defines the name of the public network, allowing to access the Internet. Depending on the Cloud Provider IaaS configuration, if the network is already existing, the ``default`` value should match the name of the public net.

:Example:
	::
	  
	  variable "public_network" {
	    default  = "public"
	  }

-------------------
``private_network``
-------------------

:Description:
        Defines the name of the private network among the nodes. Depending on the Cloud Provider IaaS configuration, if the network is already existing, the ``name`` should match the name of the private net and the ``subnet_name`` should match the name of the subnet. The associated subnet ``cidr4`` needs also to be configured to match the private_subnet range.

:Example:
	::
	  
	  variable "private_network" {
	    type = "map"
	    default  = {
	      name = "vgcn-private"
	      subnet_name = "vgcn-private-subnet"
	      cidr4 = "192.168.199.0/24"
	    }
	  }

------------
``ssh-port``
------------

:Description:
	Defines the SSH port. The default is set to ``22``.
	This field can be left unchanged.

:Example:
	::
	  
	  variable "ssh-port" {
	    default = "22"
	  }
