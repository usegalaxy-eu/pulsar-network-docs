
Manual Removal of Extra Pulsar Services
=======================================


To manually remove an extra Pulsar service, without recreating the central manager VM, ensure that all associated entities and resources are properly cleaned up. These include:

#. Directories
#. Configuration

   #. ``app.yml``
   #. ``server.ini``

#. Systemd Unit
#. Cron jobs


Connect to the central manager via SSH:

  ::

    ssh -i <private_ssh_key> <Central-Manager-Public-IP-address> -l centos

After establishing the connection, proceed with the following steps.


Remove Cron Jobs
^^^^^^^^^^^^^^^^

Open the crontab for editing:

  ::

    sudo crontab -e 

The text editor will open and the file should resemble the following example:

  ::

    #Ansible: delete successful directories
    21 */2 * * * find /data/share/staging/*/postprocessed -type f -amin +60 -exec dirname {} \; | xargs rm -rf 
    #Ansible: delete unsuccessful directories
    21 3 * * * find /data/share/staging/ -maxdepth 1 -mindepth 1 -type d -ctime +10 -exec rm -rf {} +
    #Ansible: restart Pulsar to take care of RabbitMQ shutdown issue
    11 6 * * * sudo systemctl restart pulsar
    #Ansible: condor_auto_approve
    0 * * * * /usr/bin/condor_token_request_auto_approve -netblock 172.18.41.0/24 -lifetime 3660
    #Ansible: delete successful <mq_id> directories
    21 */2 * * * find /data/share/<mq_id>/staging/*/postprocessed -type f -amin +60 -exec dirname {} \; | xargs rm -rf 
    #Ansible: delete unsuccessful <mq_id> directories
    21 3 * * * find /data/share/<mq_id>/staging/ -maxdepth 1 -mindepth 1 -type d -ctime +10 -exec rm -rf {} +
    #Ansible: restart pulsar_<mq_id> to take care of RabbitMQ shutdown issue
    11 6 * * * sudo systemctl restart pulsar_<mq_id>.service

Delete, or comment out, all entries containing <mq_id> for the service to be removed.
Save and exit the text editor.

Remove Systemd Units
^^^^^^^^^^^^^^^^^^^^

Replace <mq_id> with the specific identifier of the service to be removed and execute the following commands:

  ::
    
    sudo systemctl stop pulsar_<mq_id>.service
    sudo systemctl disable pulsar_<mq_id>.service
    sudo rm /etc/systemd/system/pulsar_<mg_id>.service
    sudo systemctl daemon-reload

In order, these commands will: 

#. Stop the service.
#. Disable the service to prevent it from restarting on boot.
#. Remove the service file.
#. Reload the systemd configuration to reflect changes.


  .. warning::
     Deleting the wrong systemd unit files can disrupt critical services. Make sure to thoroughly double-check everything before proceeding with deletion to prevent any potential issues.

Delete Directories
^^^^^^^^^^^^^^^^^^

Delete the directories associated with the service. Assuming default paths, run:

  ::

    sudo rm -rf /opt/pulsar/config_<mq_id>
    sudo rm -rf /data/share/<mq_id>

.. note::
    If ``/data/share`` is mounted on a NFS with root squashing, you may encounter a permission error. In that case re-execute the command as the owner of the directory, which is ``pulsar``. 

    ::

        sudo su pulsar

Then retry the removal command. 

    
    

