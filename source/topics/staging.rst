Retries of the staging actions
==============================

The Pulsar setup described in these pages provides that the staging actions are carried out
by the Pulsar server itself.

Several control parameters have been added to the configuration to ensure reliable communication between the Galaxy server and the remote Pulsar server.
The aim of these parameters is to control the retrying of staging actions in the event of
a failure.

Parameters and default values are:

  ::

     preprocess_action_max_retries: 10
     preprocess_action_interval_start: 2
     preprocess_action_interval_step: 2
     preprocess_action_interval_max: 30
     postprocess_action_max_retries: 10
     postprocess_action_interval_start: 2
     postprocess_action_interval_step: 2
     postprocess_action_interval_max: 30

and are included in the `app.yml`_ file

For each action (input/preprocess and output/postprocess), you can specify:

- the maximum number of retries before giving up.
- how long start sleeping between retries (in seconds).
- by how much the interval is increased for each retry (in seconds).
- the maximum number of seconds to sleep between retries.

In the following box, as an example, we have the values used in a Pulsar server with a really problematic network connection:

  ::

        preprocess_action_max_retries: 30
        preprocess_action_interval_start: 2
        preprocess_action_interval_step: 10
        preprocess_action_interval_max: 300
        postprocess_action_max_retries: 30
        postprocess_action_interval_start: 2
        postprocess_action_interval_step: 10
        postprocess_action_interval_max: 300


.. _app.yml: https://github.com/usegalaxy-eu/pulsar-infrastructure-playbook/blob/master/templates/app.yml.j2