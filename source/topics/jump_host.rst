Jump host
=========


The pulsar server setup described in this website, instructs you to assign a public IP to
your Pulsar server and expose then its ssh port through a public network, but if you are
in the lucky situation to have available a Jump host, you can avoid using the public IP.

A Jump host leverage a Secure Shell (SSH) technique that allows you to reach the Pulsar
server from your notebook through a gateway.
These kinds of hosts are placed at the boundary between the internal and the external
and act as a single point of entry, providing the transparent management of
devices within the internal network.

Terraform steps
---------------
There are 4 files that need to be modified to have Terraform creating a Pulsar setup without a public IP,
and they are:

 - main.tf
 - exec_nodes.tf
 - gpu_nodes.tf
 - output.tf

In particular, in the `main.tf` remove the first network:

  ::

    network {
        uuid = "${data.openstack_networking_network_v2.external.id}"
    }

in the `exec_nodes.tf` and `gpu_nodes.tf` change the network id from 1 to 0 in the CONDOR_HOST variable

  ::

    CONDOR_HOST = ${openstack_compute_instance_v2.central-manager.network.0.fixed_ip_v4}

and the same, in the `output.tf` for the value variable

  ::

    value = "${openstack_compute_instance_v2.central-manager.network.0.fixed_ip_v4}"

The pulsar setup fr01, created by GenOuest, is a working example of this configuration and the details
are available in the `pulsar-infrastructure`_ repo.

SSH configuration
-----------------
Now we show you how to create a simple jump with the following details:

 - Jump_IP (the jump host using a public IP)
 - Destination_IP (the pulsar server using a private IP only)

First, be sure to be able to SSH from your notebook into the Jump_IP and then, from
there to the Destination_IP. Done this, you can configure SSH on your notebook in this way:

    ::

      nano ~/.ssh/config


and paste the following configuration:

    ::

      Host pulsar_host
            Hostname Destination_IP
            User centos
            IdentityFile /path/to/your/private_ssh_key
            IdentitiesOnly yes
            PasswordAuthentication no
            Port 22
            ProxyCommand ssh -q -W %h:%p USERNAME@Jump_IP


and from now, you can easily access your Pulsar server from your notebook (through the Jump host) with the command:

    ::

      ssh pulsar_host

.. _pulsar-infrastructure: https://github.com/usegalaxy-eu/pulsar-infrastructure/commit/5cc6e2476c3a52df0f2ffb4532c3bc84240813db