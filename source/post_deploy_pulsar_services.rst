Setting up extra Pulsar services after deployment
=================================================

To add extra Pulsar services, after the initial deployment with Terraform, the ``post_deploy_pulsar_services.yml`` Ansible playbook can be used. 
The playbook adds only new services and required directories or configuration files to the central manager without recreating the VM.
Navigate into the ansible folder in the repository:

    ::

        cd pulsar-deployment/tf/ansible

Next, edit ``group_vars/vault.yml`` to define the extra message queues and their corresponding keys. These will be used to name directories and systemd units (see :doc:`multi_pulsar` for more information). Since these variables may contain sensitive data, encrypting the file with Ansible Vault is strongly recommended.

Here is an example configuration for the file:

    ::

        ex_mqs_dict: # Edit with the right key names and message queues
            key01: "pyamqp://galaxy_test01:password@mq.galaxyproject.eu:5671//pulsar/galaxy_test01?ssl=1"
            key02: "pyamqp://galaxy_test02:password@mq.galaxyproject.eu:5671//pulsar/galaxy_test02?ssl=1"

        message_queue_url: "<message_queue_used_during_deployment>"

Once you have updated the file, save it, exit, and encrypt it if using Ansible Vault.

``message_queue_url`` is the initial message queue url belonging to the main pulsar service. If defined, the playbook will check against ``ex_mqs_dict`` for duplicates to avoid conflicts. This parameter is not mandatory.

More details on the variables can be found in :ref:`here <multi-variables>`. 

To launch the playbook, from the current directory, use the commands:

    ::

        ansible-galaxy install -p roles -r requirements.yml
        ansible-playbook -u centos -b -i '<Central-Manager-Public-IP-address>,' --private-key <private_ssh_key> post_deploy_pulsar_services.yml --vault-password-file <path>

 .. note::
    This playbook uses the ``galaxyproject.pulsar`` role to set up the services, if it was already installed during previous runs the ``ansible-galaxy install`` command can be skipped.
