Get RabbitMQ credentials
========================

To be able to accept jobs and enable production in your Pulsar endpoint, you need to get RabbitMQ credentials, which allows you to get jobs from the main pool.

In this step will be described how to make this request to the `UseGalaxy.eu <https://usegalaxy.eu>`_ server administrators to create a new RabbitMQ account for your Pulsar endpoint.

Pull request on GitHub
----------------------

In order to retrieve the RabbitMQ credentials from usegalaxy-eu team, a Pull Request on their `Ìnfrastructure Playbook repository <https://github.com/usegalaxy-eu/infrastructure-playbook>`_ **is manadatory**

.. note::

   In this tutorial we will use ``vim`` as default editor. Please, if necessary replace it with the one you prefer.

.. note::

   We will call this test deployment test01-pulsar.

#. Fork the usegalaxy.eu GitHub repository `Ìnfrastructure Playbook repository <https://github.com/usegalaxy-eu/infrastructure-playbook>`_.

#. Locally clone the forked repository:

   ::

     git clone git@github.com:<your user name>/infrastructure-playbook.git

#. Navigate in the infrastructure-playbook directory: 

   ::

     cd infrastructure-playbook

#. First of all we need to add our (future) new Pulsar endpoint to the list of those available. So edit the YAML file:

   ::

     vim files/galaxy/config/user_preferences_extra_conf.yml

   Navigate to the ``distributed_compute`` section, and add your new endpoint at the end of the section, with a description of the deployment, here ``Test Pulsar deployment``, followed by its short name, ``test01-pulsar`` in this example.

   ::

     distributed_compute:
        description: Use distributed compute resources
        inputs:
            - name: remote_resources
              label: Remote resources id
              type: select
              required: False
              options:
                  - ["default - Galaxy will decide where to put your job", None]
                  ...
                  - ["Test Pulsar deployment", test01-pulsar]

   .. figure:: _static/img/HCMR_pulsar_preferences.png
      :scale: 20%
      :align: center

#. Next to this we need to add a new TPV destination. Edit the YAML file, in the form of a jinja template:

   ::

     vim files/galaxy/tpv/destinations.yml.j2

   Navigate to the ``Pulsar destinations`` section:

   ::

       #######################
       # PULSAR DESTINATIONS #
       #######################

       pulsar_default: # use for remote Pulsar nodes and ALWAYS overwrite the runner.
       ...

   and add the new pulsar destination to TPV configuratio:

   ::

     pulsar_test01_tpv:
       inherits: pulsar_default
       runner: pulsar_eu_test01
       max_accepted_cores: 8
       max_accepted_mem: 30
       min_accepted_gpus: 0
       max_accepted_gpus: 0
       scheduling:
         require:
           - test01-pulsar

   Give it a name, in this case ``pulsar_test01_tpv``, and configure it taking into account the resources you will make available:

   ``inherits``: this destination always inherits from ``pulsar_default``.

   ``runner``: this is the name of your destination, ``pulsar_eu_test01`` in this case.

   ``max_accepted_cores``: the maximum number of cores a job can require to run on this endpoint.

   ``max_accepted_mem``: the maximum number of GB of RAM a job can require to run on this endpoint.

   ``min_accepted_gpus``: the minumum number of GPUs a job can require to run on this endpoint.
   
   ``max_accepted_gpus``: the maximum number of GPUs a job can require to run on this endpoint.
   
   ``scheduling``: the short name you configured in the previous section, in this case ``test01-pulsar``.

   .. figure:: _static/img/HCMR_pulsar_destinations.png
      :scale: 20%
      :align: center

#. Then we add the user credentials, i.e. RabbitMQ username and password, for the new endpoint to the RabbitMQ list, editing the file:

   ::

     vim group_vars/mq.yml

   Navigate to the ``rabbitmq_users`` section and add your entry:

   ::

     # RabbitMQ
     rabbitmq_users:
       - user: mqadmin
         password: "{{ rabbitmq_password_admin }}"
         tags: administrator
         vhost: /
       - user: flower
         password: "{{ rabbitmq_password_flower }}"
         tags: administrator
         vhost: galaxy
       - user: galaxy
         password: "{{ rabbitmq_password_galaxy }}"
         vhost: galaxy
       ...
       - user: galaxy_test01
         password: "{{ rabbitmq_password_galaxy_test01 }}"
         vhost: /pulsar/galaxy_test01

   In this case, we added a user named ``galaxy_test01``, a password in the form of a jinja2 variable ``{{ rabbitmq_password_galaxy_test01 }}`` and a virtual host ``/pulsar/galaxy_test01``.

   .. figure:: _static/img/HCMR_pulsar_mq.png
      :scale: 20%
      :align: center

#. Finally, we need to add the new destination to the (use)Galaxy job configuration file:

   ::

     vim templates/galaxy/config/job_conf.yml

   and modify the ``galaxy_jobconf`` section:

   ::

     galaxy_jobconf:
       plugin_workers: 8
       handlers:
         count: "{{ galaxy_systemd_handlers }}"
         assign_with: db-skip-locked
         max_grab: 16
         ready_window_size: 32
       plugins:
       ...

  adding your endpoint configuration

  ::

    ...
    - id: pulsar_eu_test01
      load: galaxy.jobs.runners.pulsar:PulsarMQJobRunner
      params:
        amqp_url: "pyamqp://galaxy_test01:{{ rabbitmq_password_galaxy_hcmr01 }}@mq.galaxyproject.eu:5671//pulsar/galaxy_test01?ssl=1"
        galaxy_url: "https://usegalaxy.eu"
        manager: production
        amqp_acknowledge: "true"
        amqp_ack_republish_time: 300
        amqp_consumer_timeout: 2.0
        amqp_publish_retry: "true"
        amqp_publish_retry_max_retries: 60

  where:

  ``id``: is the name of the destiantion, in this case ``pulsar_eu_test01``

  ``amqp_url``, is the RabbitMQ url, customized  with the username and password. The queue URL will looks like this:

  ::

    pyamqp://galaxy_test01:*****@mq.galaxyproject.eu:5671//pulsar/galaxy_test01?ssl=1

   .. figure:: _static/img/HCMR_pulsar_jobconf.png
      :scale: 20%
      :align: center

#. Push them to the forked repository:

   ::

     Changes not staged for commit:
     (use "git add <file>..." to update what will be committed)
     (use "git restore <file>..." to discard changes in working directory)
   	modified:   files/galaxy/config/user_preferences_extra_conf.yml
   	modified:   files/galaxy/tpv/destinations.yml.j2
   	modified:   group_vars/mq.yml
   	modified:   templates/galaxy/config/job_conf.yml

   ::

     $ git add files/galaxy/config/user_preferences_extra_conf.yml files/galaxy/tpv/destinations.yml.j2 group_vars/mq.yml templates/galaxy/config/job_conf.yml

     $ git commit -m "add test deployment"
     [master c8bd1dbd] add test deployment
      4 files changed, 29 insertions(+), 3 deletions(-)
     
     $ git push
     Enumerating objects: 26, done.
     Counting objects: 100% (26/26), done.
     Delta compression using up to 2 threads
     Compressing objects: 100% (13/13), done.
     Writing objects: 100% (14/14), 1.68 KiB | 858.00 KiB/s, done.
     Total 14 (delta 10), reused 0 (delta 0), pack-reused 0
     remote: Resolving deltas: 100% (10/10), completed with 10 local objects.
     To github.com:mtangaro/infrastructure-playbook.git
        db718db4..c8bd1dbd  master -> master

The final pull request will looks like as the following:

.. figure:: _static/img/HCMR_pulsar_PR.png
   :scale: 20%
   :align: center

The ``secret_group_vars/pulsar.yml`` is edited by the usegalaxy-eu admin team, adding the RabbitMQ password.

Bring your own compute
----------------------

#. From the top-left `User` menu, navigate to `User -> Preferences -> Manage Information` panel.

   Here is possible to add Pulsar details to get your RabbitMQ credentials in the section:  

   ::

     Bring your own Pulsar endpoint to Galaxy. You can add here your Pulsar credentials and specifications.
     After 24 hours Galaxy's job scheduling systems will take your Pulsar into account and schedule appropriate jobs to your compute resources.
     This is an experimental feature. Contact us if you want to learn more about it.

#. Fill it taking into account your Pulsar endpoint specs:

   .. figure:: _static/img/esg_byoc.png
      :scale: 20%
      :align: center

#. The UseGalaxy.eu team will provide you the RabbitMQ queue URL by mail, which needs to be added to your Pulsar configuration as described in the next step.
