Adding GPUs to your Pulsar setup
================================

GPU's devices are presently widely adopted to accelerate high-intensive computational tasks, leveraging the intrinsic
parallel computation capability of this kind of hardware.  If your Cloud provider
makes GPUs available to your tenant, you can effectively apply them in many scientific contexts like the
molecular docking, prediction and searching of molecular structures or machine learning applications.


In the following steps, we describe how to add a GPU device to the computation
cluster created following the instructions provided in the section above.

Prerequisites
-------------
You know the **name** of the OpenStack's flavor that can be used to instantiate a VM with one or more GPU
devices connected and the **number** of VMs that can be created.

Software provided
-----------------
The VGCN image provides all the software need to enable an **NVIDIA** GPU to submit a GPU job to the
**HTCondor** queue manager, also through a **Docker** container.

The current `VGCN image`_ provides the following packages to your VMs:

- cuda toolkit 10.1
- Docker version 19.03.8
- `NVIDIA Container toolkit`_ 1.1.1



Configuration
-------------
In the `preparation step`_, you have created a directory named ``<workspace_name>`` and inside,
you have a ``vars.tf`` file with all the parameters to configure the Pulsar endpoint.

Edit the variable ``flavors`` and ``gpu_node_count`` in ``<workspace-name>/vars.tf``, replacing
the default values with your own details.

:Example:
	::

	  variable "flavors" {
	    type = "map"
	    default = {
	      "central-manager" = "m1.medium"
	      "nfs-server" = "m1.medium"
	      "exec-node" = "m1.medium"
	      "gpu-node" = "gpu_flavor_name"  <--
	    }
	  }

	  variable "gpu_node_count" {
	    default = 10                      <--
	  }


Now you can validate the new terraform configuration:

   ::

     WS=<workspace-name> make plan

and if the previous step doesn't show any error, you can go forward applying the new configuration.

   ::

     WS=<workspace-name> make apply

Test your setup
---------------

Access one of your new shiny workers with a GPU enabled and digit:

  ::

     nvidia-smi

You will receive  a message like this:

  ::

    $ nvidia-smi
    Tue May 19 17:51:12 2020
    +-----------------------------------------------------------------------------+
    | NVIDIA-SMI 440.64.00    Driver Version: 440.64.00    CUDA Version: 10.2     |
    |-------------------------------+----------------------+----------------------+
    | GPU  Name        Persistence-M| Bus-Id        Disp.A | Volatile Uncorr. ECC |
    | Fan  Temp  Perf  Pwr:Usage/Cap|         Memory-Usage | GPU-Util  Compute M. |
    |===============================+======================+======================|
    |   0  Tesla T4            Off  | 00000000:00:05.0 Off |                    0 |
    | N/A   37C    P0    21W /  70W |      0MiB / 15109MiB |      0%      Default |
    +-------------------------------+----------------------+----------------------+

    +-----------------------------------------------------------------------------+
    | Processes:                                                       GPU Memory |
    |  GPU       PID   Type   Process name                             Usage      |
    |=============================================================================|
    |  No running processes found                                                 |
    +-----------------------------------------------------------------------------+


and the same with the latest CUDA docker image:

  ::

     $ docker run --gpus all nvidia/cuda:10.1-base nvidia-smi
        Tue May 19 16:08:27 2020
        +-----------------------------------------------------------------------------+
        | NVIDIA-SMI 440.64.00    Driver Version: 440.64.00    CUDA Version: 10.2     |
        |-------------------------------+----------------------+----------------------+
        | GPU  Name        Persistence-M| Bus-Id        Disp.A | Volatile Uncorr. ECC |
        | Fan  Temp  Perf  Pwr:Usage/Cap|         Memory-Usage | GPU-Util  Compute M. |
        |===============================+======================+======================|
        |   0  Tesla T4            Off  | 00000000:00:05.0 Off |                    0 |
        | N/A   37C    P0    20W /  70W |      0MiB / 15109MiB |      0%      Default |
        +-------------------------------+----------------------+----------------------+

        +-----------------------------------------------------------------------------+
        | Processes:                                                       GPU Memory |
        |  GPU       PID   Type   Process name                             Usage      |
        |=============================================================================|
        |  No running processes found                                                 |
        +-----------------------------------------------------------------------------+



.. _preparation step: ../pretasks.html#pre-tasks
.. _VGCN image: https://github.com/usegalaxy-eu/pulsar-infrastructure/blob/ef3ba68abb9fef79a8d159687b912d25c901d90d/tf/vars.tf#L26
.. _NVIDIA Container toolkit: https://github.com/NVIDIA/nvidia-container-runtime
