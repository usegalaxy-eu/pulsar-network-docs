RabbitMQ configuration
======================

In this step will be described how to make a PR to the `UseGalaxy.eu <https://github.com/usegalaxy-eu>`_ GitHub
repository to create a new RabbitMQ account for your Pulsar endpoint.

#. Fork the usegalaxy.eu GitHub `infrastructure-playbook <https://github.com/usegalaxy-eu/infrastructure-playbook>`_.

#. Clone the forked repository:

   ::

     git clone https://github.com/<user-name>/infrastructure-playbook.git

#. Edit the file ``infrastructure-playbook/group_vars/proxy-internal.yml`` with your favourite text editor:

   - in the `rabbitmq_users` section create a new user adding the following lines:

     ::

       rabbitmq_users:
       ...
         - user: <name_name>
           password: "{{ <rabbit_mq_password_for_your_user> }}"
           vhost: /pulsar/<user_name>

     For example the configuration for ``it02`` Pulsar endpoint is:

     ::

       rabbitmq_users:
       ...
         - user: galaxy_it02
           password: "{{ rabbitmq_password_galaxy_it02 }}"
           vhost: /pulsar/galaxy_it02


   - Add your virtual host, previously configured, to the ``rabbitmq_vhosts`` section:

     ::

       rabbitmq_vhosts:
       ...
         - /pulsar/<name_name>

     For example the configuration for ``it02`` virtual host is:

     ::

       rabbitmq_vhosts:
       ...
         - /pulsar/galaxy_it02

#. Make a Pull Request to the original repository and contact the usegalaxy.eu admins.
Once merged the useGalaxy.eu team will provide you the RabbitMQ queue URL by mail,
which needs to be added to your Pulsar configuration as described in the next step.

   .. figure:: _static/img/rabbitmq_PR.png
      :scale: 40%
      :align: center

   The queue URL will looks like this:

   ::

     pyamqp://galaxy_it02:*****@proxy.internal.galaxyproject.eu:5671//pulsar/galaxy_it02?ssl=1
