.. meta::
   :description: A guide to debugging the RCCL library of multi-GPU and multi-node collective communication primitives optimized for AMD GPUs
   :keywords: RCCL, ROCm, library, API, debug

.. _debugging-rccl:

*********************
RCCL debugging guide
*********************

This guide explains the steps to triage and debug functional and performance issues with RCCL.
While debugging, collect the output from the commands in this guide for
support or field personnel. To debug RCCL, follow these steps.

.. _debugging-system-info:

Collecting system information
=============================

Collect information about the ROCm version, GPU/accelerator, platform, and configuration and provide this
information to the support team.

*  Verify the ROCm version. This might be a release version or a
   mainline or staging version. Use this command to display the version:

   .. code:: shell

      cat /opt/rocm/.info/version

*  The problem might be a general issue or specific to the architecture or system.
   To narrow down the issue, collect information about the GPU or accelerator, along with other
   details about the platform and system. Some issues to consider include:

   *  Is ROCm running on a bare-metal setup, in a Docker container, in a SR-IOV virtualized
      environment, or in some combination of these configurations? If ROCm is running in a Docker
      container, provide the name of the Docker image.
   *  Is the problem only seen on a specific GPU architecture?
   *  Is it only seen on a specific system type?
   *  Is it happening on a single node or multinode setup?
  
   Run the following command and collect the output:

   .. code:: shell

      rocm_agent_enumerator

   Also collect the name of the GPU or accelerator:

   .. code:: shell

      rocminfo

*  Run the ``rocm-smi`` command to display the system topology.

   .. code:: shell

      rocm-smi
      rocm-smi --showtopo

*  Determine the values of the ``PATH`` and ``LD_LIBRARY_PATH`` environment variables.

   .. code:: shell

      echo $PATH
      echo $LD_LIBRARY_PATH

*  Collect the HIP configuration.

   .. code:: shell

      /opt/rocm/bin/hipconfig --full

*  Verify the network settings and setup using the ``ibv_devinfo`` command. 
   This command displays information about the available RDMA devices available and determines 
   whether they are installed and functioning properly.

   .. code:: shell

      ibv_devinfo

*  Determine the BKC version, if this information is known.

.. _collecting-rccl-info:

Collecting RCCL information
=============================

Collect the following information about the RCCL installation and configuration.

*  Run the ``ldd`` command to list any dynamic dependencies for RCCL.

   .. code:: shell

      ldd <specify-path-to-librccl.so>

*  Determine the RCCL version. This might be the pre-packaged component in
   ``/opt/rocm/lib`` or a version that was built from source. To verify the RCCL version,
   enter the following command, then run either rccl-tests or an e2e application.

   .. code:: shell

      export NCCL_DEBUG=VERSION

   To run rccl-tests, use these commands:

   .. code:: shell

      chmod +x run_rccl-tests.sh
      ./run_rccl-tests.sh

   The results are displayed using the following format:

   .. code:: shell

      RCCL version 2.20.5+hip6.2 develop:eb562e7

   .. note::
   
      For more information on how to build and run rccl-tests, see the
      `rccl-tests GitHub <https://github.com/ROCm/rccl-tests/blob/develop/README.md>`_.

*  Collect the RCCL logging information. Enable the debug logs, 
   then run rccl-tests or any e2e workload to collect the logs. Use the 
   following command to enable the logs.

   .. code:: shell

      export NCCL_DEBUG=INFO

Troubleshooting
=============================

Use the following troubleshooting techniques to potentially isolate the issue.

*  Build or run the develop branch version of RCCL and see if the problem persists.
*  Try an earlier minor or major releases of RCCL.
*  If you recently made changes to the ROCm runtime configuration, KFD/driver or compiler,
   run the test again with the previous configuration.

.. _analyze-performance-info:

Analyzing performance issues
=============================

If the issues involve performance issues and occur in an e2e workload, try the following 
microbenchmarks and collect the results. Follow the instructions in the subsequent sections
to run these benchmarks and provide the results to the support team.

*  TransferBench
*  RCCL Unit Tests
*  rccl-tests
  
Collect the TransferBench data
---------------------------------

TransferBench allows you to benchmark simulataneous copies between
user-specified devices. For more information, 
see :doc:`the TransferBench documentation <transferbench:index>`.

To collect the TransferBench data, follow these steps:

#. Clone the TransferBench Git repository.

   .. code:: shell

      git clone https://github.com/ROCm/TransferBench.git 

#. Change to the new directory and build the component.

   .. code:: shell

      cd TransferBench
      make

#. Run the TransferBench utlity with the following parameters and save the results.

   .. code:: shell

      USE_FINE_GRAIN=1 GFX_UNROLL=2 ./TransferBench a2a 64M 8

Collect the RCCL microbenchmark data
-------------------------------------

To use the RCCL tests to collect the RCCL benchmark data, follow these steps:

#. Disable NUMA auto-balancing using the following command:

   .. code:: shell

      sudo sysctl kernel.numa_balancing=0

   Run the following command to verify the setting. The expected output is ``0``.

   .. code:: shell

      cat /proc/sys/kernel/numa_balancing

#. Build MPI, RCCL, and rccl-tests.

   .. code:: shell

      chmod +x build_mpich_rccl_rccl-tests.sh
      ./build_mpich_rccl_rccl-tests.sh

#. Run rccl-tests with MPI using the following script and collect the performance numbers.

   .. code:: shell

      chmod +x run_rccl-tests.sh
      ./run_rccl-tests.sh

RCCL and NCCL comparisons
=============================

If you are also using NVIDIA HW or NCCL and notice a performance gap between the two systems,
collect the system and performance data on the NVIDIA platform as well. 
Provide both sets of data to the support team.
