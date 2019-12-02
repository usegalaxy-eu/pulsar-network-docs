Benchmarking your Pulsar setup
==============================

To determine the performance of your Pulsar setup, you can use the
`GalaxyBenchmarker <https://github.com/usegalaxy-eu/GalaxyBenchmarker>`_.
This is a framework that allows for easy benchmarking of various Galaxy
job destinations.

As a start, clone the repository to your local workstation:

    ::

     git clone https://github.com/usegalaxy-eu/GalaxyBenchmarker.git

A docker-compose setup is already provided that includes InfluxDB and
Grafana for easy analysis of the metrics.

Configure the benchmarker
-------------------------

The benchmarker is configured using a yaml-configuration file. You can
find a basic example in ``benchmark_config.yml.usegalaxyeu``. As a start,
rename it to benchmark_config.yml and fill in the credentials for your
regular UseGalaxy.eu user (a deeper explanation of the configuration
can be found at https://github.com/usegalaxy-eu/GalaxyBenchmarker).
To later benchmark your Pulsar setup, you will need an additional user
for which UseGalaxy.eu is configured to only route jobs to your setup.
Also add the user key for this user to the config file under
``destinations``.

The benchmarker uses the test functionality
of `Planemo <https://github.com/galaxyproject/planemo>`_ to submit
workflows. The docker-compose setup already clones the examples from
https://github.com/usegalaxy-eu/workflow-testing. These are available at
the the path ``/workflow-testing`` inside the benchmarker container. Some
of the workflows there are already included in the example config file.
You can also add your own workflows: Simply add the workflows
somewhere in the root folder of the repository, which is mounted under
``/src`` inside the benchmarker container and mention it in the config file.

.. code-block:: yaml

    workflows:
      - name: YourCustomWorkflow
        type: Galaxy
        path: /src/custom-workflow-folder/custom-workflow.ga

The benchmarker can submit jobs to multiple job destinations to compare
the performance of each. As a start, we can stay with the one destination
already defined.

.. code-block:: yaml

    destinations:
      - name: YourDestinationName
        type: Galaxy
        galaxy_user_key: YOUR-USER-KEY-FOR-DESTINATION-USER

Now, we just need to define the actual benchmark that we want to perform.
Simply define the workflows that should be run and the destinations to
which the workflows should be submitted to.

.. code-block:: yaml

    benchmarks:
    - name: BenchmarkName
      type: DestinationComparison
      destinations:
        - YourDestinationName
      workflows:
        - ard
        - adaboost
        - gromacs
        - mapping_by_sequencing
      runs_per_workflow: 5
      warmup: true

Run the benchmark
-----------------

After everything has been set, we can start the actual benchmarking.
Simply run:

    ::

     docker-compose up

This will spin up a InfluxDB and Grafana container, together with a
container running the benchmarker. You can monitor the progress
in the console. If you need to re-run a benchmark, just do a
ctrl-c and run the ``docker-compose up`` command again.

.. warning::

    The data of InfluxDB is stored inside the container. Before
    running ``docker-compose down`` remember to back up your data!

Analyze the results
-------------------
After the benchmarking has been finished, you can view the results
using the Grafana Dashboards at http://localhost:3000
(username: admin, password: admin). These Dashboards are still
work in progress and you may need to alter them for your case,
however they may be a good starting point.
