Get RabbitMQ credentials
========================

To be able to accept jobs and enable production in your Pulsar endpoint, you need to get an RabbitMQ key, which allows you to get jobs from the main pool.

In this step will be described how to make this request to the `UseGalaxy.eu <https://usegalaxy.eu>`_ server administrators to create a new RabbitMQ account for your Pulsar endpoint.

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

   The queue URL will looks like this:

   ::

      pyamqp://galaxy_it02:*****@mq.galaxyproject.eu:5671//pulsar/galaxy_it02?ssl=1
