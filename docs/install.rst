Installation
============

.. toctree::
   :maxdepth: 2
   :caption: Contents:

General usage
-------------
Choose ``[BACKEND_FILE]``, ``[BACKEND]``, ``[WDL]``, ``[PIPELINE]``, ``[CONDA_ENV]`` and ``[WORKFLOW_OPT]`` according to your platforms, kind of pipeline (``.wdl``) and presence of MySQL database and ``Docker``.

#. ``[BACKEND_FILE]`` (not required for DNANexus)
    * ``backends/backend.conf`` : backend conf. file for all backends.
    * ``backends/backend_db.conf`` : backend conf. file for all backends with MySQL DB.
#. ``[BACKEND]`` (not required for DNANexus)
    * ``Local`` : local (by default).
    * ``google`` : Google Cloud Platform.
    * ``sge`` : Sun GridEngine.
    * ``slurm`` : SLURM.
#. ``[PIPELINE]``
    * ``atac`` : ENCODE ATAC-Seq pipeline
    * ``chip`` : AQUAS TF/Histone ChIP-Seq processing pipeline
#. ``[WDL]``
    * ``atac.wdl`` : ENCODE ATAC-Seq pipeline
    * ``chip.wdl`` : AQUAS TF/Histone ChIP-Seq processing pipeline
#. ``[CONDA_ENV]`` (for systems without `Docker` support)
    * ``encode-atac-seq-pipeline`` : ENCODE ATAC-Seq pipeline
    * ``encode-chip-seq-pipeline`` : AQUAS TF/Histone ChIP-Seq processing pipeline
#. ``[DOCKER_CONTAINER]``
    * ``quay.io/encode-dcc/atac-seq-pipeline:v1`` : ENCODE ATAC-Seq pipeline
    * ``quay.io/encode-dcc/chip-seq-pipeline2:v1`` : AQUAS TF/Histone ChIP-Seq processing pipeline
#. ``[WORKFLOW_OPT]`` (not required for DNANexus)
    * ``docker.json`` : for systems with ``Docker`` support (Google Cloud, local, ...).
    * ``sge.json`` : Sun GridEngine (here you can specify your own queue and parallel environment).
    * ``slurm.json`` : SLURM (here you can specify your partition for ``sbatch -p`` or account for ``sbatch --account``).

DNANexus Platform
-----------------
#. Sign up for a new account on `DNANexus web site <https://www.dnanexus.com/>`_.
#. Create a project ``[DX_PRJ]``.
#. Install `DNANexus SDK <https://wiki.dnanexus.com/Downloads#DNAnexus-Platform-SDK>`_ on your local computer and login on that project::

    $ pip install dxpy
    $ dx login

#. Download the latest ``dxWDL``::

	$ wget https://github.com/dnanexus/dxWDL/releases/download/0.66.1/dxWDL-0.66.1.jar
	$ chmod +x dxWDL-0.66.1.jar

#. Convert WDL to a workflow on DNANexus web UI. Make sure that URIs in your ``input.json`` are valid (starting with ``dx://``) for DNANexus::

	$ java -jar dxWDL-0.66.1.jar compile [WDL] -f -folder /[DEST_DIR_ON_DX] -defaults input.json -extras workflow_opts/docker.json

#. Check if a new workflow is generated on a directory ``[DEST_DIR_ON_DX]`` on your project ``[DX_PRJ]``.
#. Click on a workflow, specify output directory and then launch it.

Google Cloud Platform
---------------------
#. Create a `Google Project <https://console.developers.google.com/project>`_.
#. Set up a `Google Cloud Storage bucket <https://console.cloud.google.com/storage/browser>`_ to store outputs.
#. Enable the following API's in your `API Manager <https://console.developers.google.com/apis/library>`_.
	* Google Compute Engine
	* Google Cloud Storage
	* Genomics API

#. Set quota for ``Google Compute Engine API`` on https://console.cloud.google.com/iam-admin/quotas per region. Increase quota for SSD/HDD storage, number of vCPUs to process more samples faster simulateneouly.
	* CPUs
	* Persistent Disk Standard (GB)
	* Persistent Disk SSD (GB)
	* In-use IP addresses
	* Networks

#. Set ``default_runtime_attributes.zones`` in ``workflow_opts/docker.json`` as your preferred Google Cloud zone::

	{
	  "default_runtime_attributes" : {
	    ...
	    "zones": "us-west1-a us-west1-b us-west1-c",
	    ...
	}

#. Set ``default_runtime_attributes.preemptible`` as ``"0"`` to disable preemptible instances. Pipeline defaults not to use `preemptible instances <https://cloud.google.com/compute/docs/instances/preemptible>`_. If all retrial fails then the instance will be upgraded to a regular one. **Disabling it will cost you significantly more** but you can get your samples processed much faster and stabler. Preemptible instance is disabled by default for hard tasks like ``bowtie2``, ``bwa`` and ``spp`` since they can take longer than the limit (24 hours) of preemptible instances::

	{
	  "default_runtime_attributes" : {
	    ...
	    "preemptible": "0",
	    ...
	}

#. If you are already on a VM instance on your Google Project. Skip previous two steps.
#. Install `Google Cloud Platform SDK <https://cloud.google.com/sdk/downloads>`_ and authenticate through it. You will be asked to enter verification keys. Get keys from the URLs they provide::

	$ gcloud auth login --no-launch-browser
	$ gcloud auth application-default login --no-launch-browser

#. If you see permission errors at runtime, then unset environment variable ``GOOGLE_APPLICATION_CREDENTIALS`` or add it to your BASH startup scripts (``$HOME/.bashrc`` or ``$HOME/.bash_profile``)::
 
    $ unset GOOGLE_APPLICATION_CREDENTIALS

#. Get on the Google Project::

	$ gcloud config set project [PROJ_NAME]

#. Download the latest ``Cromwell``::

	$ wget https://github.com/broadinstitute/cromwell/releases/download/32/cromwell-32.jar
	$ chmod +x cromwell-32.jar

#. Run a pipeline. Make sure that URIs in your ``input.json`` are valid (starting with ``gs://``) for Google Cloud Platform. Use any string for ``[SAMPLE_NAME]`` to distinguish between multiple samples::

    $ java -jar -Dconfig.file=backends/backend.conf -Dbackend.default=google -Dbackend.providers.google.config.project=[PROJ_NAME] -Dbackend.providers.google.config.root=[OUT_BUCKET]/[SAMPLE_NAME] cromwell-32.jar run [WDL] -i input.json -o workflow_opts/docker.json

Local computer with ``Docker``
------------------------------
#. Install `genome data <#genome-data-installation>`_.
#. Set ``[PIPELINE].genome_tsv`` in ``input.json`` as the installed genome data TSV.
#. Run a pipeline::

    $ java -jar -Dconfig.file=backends/backend.conf cromwell-30.2.jar run [WDL] -i input.json -o workflow_opts/docker.json

Local computer without ``Docker``
---------------------------------
#. Install `dependencies <#dependency-installation>`_.
#. Install `genome data <#genome-data-installation>`_.
#. Set ``[PIPELINE].genome_tsv`` in ``input.json`` as the installed genome data TSV.
#. Run a pipeline::

    $ source activate [CONDA_ENV]
    $ java -jar -Dconfig.file=backends/backend.conf cromwell-30.2.jar run [WDL] -i input.json
    $ source deactivate

Sun GridEngine (SGE)
--------------------
.. note:: Genome data have already been installed and shared on Stanford Kundaje lab cluster. Use genome TSV files in ``genome/klab`` for your ``input.json``. You can skip step 4 on these clusters.

.. note:: If you are working on the OLD Stanford SCG4 cluster, try migrating to a new one based on SLURM. 

#. Set your parallel environment (``default_runtime_attributes.sge_pe``) and queue (``default_runtime_attributes.sge_queue``) in ``workflow_opts/sge.json``::

    {
      "default_runtime_attributes" : {
        "sge_pe": "YOUR_PARALLEL_ENV",
        "sge_queue": "YOUR_SGE_QUEUE (optional)"
    }

#. If there is no parallel environment on your SGE then ask your SGE admin to create one. ``sge_queue`` is optional::

    $ qconf -spl

#. Install `dependencies <#dependency-installation>`_.
#. Install `genome data <#genome-data-installation>`_.
#. Set ``[PIPELINE].genome_tsv`` in ``input.json`` as the installed genome data TSV.
#. Run a pipeline::

    $ source activate [CONDA_ENV]
    $ java -jar -Dconfig.file=backends/backend.conf -Dbackend.default=sge cromwell-30.2.jar run [WDL] -i input.json -o workflow_opts/sge.json
    $ source deactivate

#. If you want to run multiple (>10) pipelines, then run a Cromwell server on an interactive node. We recommend to use ``screen`` or ``tmux`` to keep your session alive and note that all running pipelines will be killed after walltime::

	$ qlogin ... # some qlogin command with some (>=2) cpu, enough memory (>=5G) and long walltime (>=2day)
	$ hostname -f # to get [CROMWELL_SVR_IP]
	$ source activate [CONDA_ENV]
	$ _JAVA_OPTIONS="-Xmx5G" java -jar -Dconfig.file=backends/backend/conf -Dbackend.default=sge cromwell-32.jar server

#. You can modify ``backend.providers.sge.concurrent-job-limit`` in ``backends/backend.conf`` to increase maximum concurrent jobs. This limit is **not per sample**. It's for all sub-tasks of all submitted samples.

#. On a login node, submit jobs to the cromwell server. You will get ``[WORKFLOW_ID]`` as a return value. Keep these workflow IDs for monitoring pipelines and finding outputs for a specific sample later::
    
    $ curl -X POST --header "Accept: application/json" -v "[CROMWELL_SVR_IP]:8000/api/workflows/v1" \
	-F workflowSource=@[WDL] \
	-F workflowInputs=@input.json \
	-F workflowOptions=@workflow_opts/sge.json

#. To monitor pipelines, see `Cromwell server REST API description <http://cromwell.readthedocs.io/en/develop/api/RESTAPI/#cromwell-server-rest-api>`_ for more details. ``qstat`` will not give enough information for monitoring per sample::

	$ curl -X GET --header "Accept: application/json" -v "[CROMWELL_SVR_IP]:8000/api/workflows/v1/[WORKFLOW_ID]/status"

SLURM
-----
.. note:: Genome data have already been installed and shared on Stanford Sherlock and SCG. Use genome TSV files in ``genome/scg`` or ``genome/sherlock`` for your ``input.json``. You can skip step 2 on these clusters.

#. Set your partition (``default_runtime_attributes.slurm_partition``) or account (``default_runtime_attributes.slurm_account``) in `workflow_opts/slurm.json`. Those two attibutes are optional according to your SLURM server configuration::

    {
      "default_runtime_attributes" : {
        "slurm_partition": "YOUR_SLURM_PARTITON (optional)",
        "slurm_account": "YOUR_SLURM_ACCOUNT (optional)"
      }
    }

  .. note:: Remove ``slurm_account`` on Sherlock and ``slurm_partition`` on SCG.

#. Install `dependencies <#dependency-installation>`_.
#. Install `genome data <#genome-data-installation>`_.
#. Set ``[PIPELINE].genome_tsv`` in ``input.json`` as the installed genome data TSV.
#. Run a pipeline::

    $ source activate [CONDA_ENV]
    $ java -jar -Dconfig.file=backends/backend.conf -Dbackend.default=slurm cromwell-30.2.jar run [WDL] -i input.json -o workflow_opts/slurm.json
    $ source deactivate

#. If you want to run multiple (>10) pipelines, then run a Cromwell server on an interactive node. We recommend to use `screen` or `tmux` to keep your session alive and note that all running pipelines will be killed after walltime::

	$ srun -n 2 --mem 5G -t 3-0 --qos normal --account [ACCOUNT] -p [PARTITION] --pty /bin/bash -i -l # some srun command with some (>=2) cpu, enough memory (>=5G) and long walltime (>=2day)
	$ hostname -f # to get [CROMWELL_SVR_IP]
	$ source activate [CONDA_ENV]
	$ _JAVA_OPTIONS="-Xmx5G" java -jar -Dconfig.file=backends/backend/conf -Dbackend.default=slurm cromwell-32.jar server

#. You can modify ``backend.providers.slurm.concurrent-job-limit`` in ``backends/backend.conf`` to increase maximum concurrent jobs. This limit is **not per sample**. It's for all sub-tasks of all submitted samples.

#. On a login node, submit jobs to the cromwell server. You will get ``[WORKFLOW_ID]`` as a return value. Keep these workflow IDs for monitoring pipelines and finding outputs for a specific sample later::
    
    $ curl -X POST --header "Accept: application/json" -v "[CROMWELL_SVR_IP]:8000/api/workflows/v1" \
	-F workflowSource=@[WDL] \
	-F workflowInputs=@input.json \
	-F workflowOptions=@workflow_opts/slurm.json

#. To monitor pipelines, see `Cromwell server REST API description <http://cromwell.readthedocs.io/en/develop/api/RESTAPI/#cromwell-server-rest-api>`_ for more details. ``squeue`` will not give enough information for monitoring per sample::

	$ curl -X GET --header "Accept: application/json" -v "[CROMWELL_SVR_IP]:8000/api/workflows/v1/[WORKFLOW_ID]/status"


Kundaje lab cluster with ``Docker``
-----------------------------------
.. note:: Jobs will run locally without being submitted to Sun GridEngine (SGE). Genome data have already been installed and shared. Use genome TSV files in ``genome/klab`` for your ``input.json``.

#. Run a pipeline::

    $ java -jar -Dconfig.file=backends/backend.conf -Dbackend.default=Local cromwell-30.2.jar run [WDL] -i input.json -o workflow_opts/docker.json

Kundaje lab cluster with SGE
----------------------------
.. note:: Jobs will be submitted to Sun GridEngine (SGE) and distributed to all server nodes. Genome data have already been installed and shared. Use genome TSV files in ``genome/klab`` for your ``input.json``.

#. Install `dependencies <#dependency-installation>`_.
#. Run a pipeline::

    $ source activate [CONDA_ENV]
    $ java -jar -Dconfig.file=backends/backend.conf -Dbackend.default=sge cromwell-30.2.jar run [WDL] -i input.json -o workflow_opts/sge.json
    $ source deactivate


Dependency installation
-----------------------
.. note:: WE DO NOT RECOMMEND RUNNING OUR PIPELINE WITHOUT ``DOCKER``! If you have ``Docker`` installed then skip this step. Use it with caution.

#. **Our pipeline is for BASH only. Set your default shell as BASH**.
#. For Mac OSX users, do not install dependencies and just install ``Docker`` and use our pipeline with it.
#. Remove any Conda (Anaconda Python and Miniconda) from your ``PATH``. PIPELINE WILL NOT WORK IF YOU HAVE OTHER VERSION OF CONDA BINARIES IN ``PATH``.
#. Install Miniconda3 for 64-bit Linux on your system. Miniconda2 will not work::

   $ wget https://repo.continuum.io/miniconda/Miniconda3-latest-Linux-x86_64.sh
   $ bash Miniconda3-latest-Linux-x86_64.sh -b -p [MINICONDA3_INSTALL_DIR]

#. Add ``PATH`` for our pipeline Python scripts and Miniconda3 to one of your bash startup scripts (``$HOME/.bashrc`` or ``$HOME/.bash_profile``).

  .. code-block:: bash

    export PATH=[WDL_PIPELINE_DIR]/src:$PATH # VERY IMPORTANT
    export PATH=[MINICONDA3_INSTALL_DIR]/bin:$PATH
    unset PYTHONPATH

#. Re-login.
#. Make sure that conda correctly points to ``[MINICONDA3_INSTALL_DIR]/bin/conda``::

   $ which conda

#. Install dependencies on Minconda3 environment. Java 8 JDK and Cromwell-29 are included in the installation::

   $ cd installers/
   $ source activate [CONDA_ENV]
   $ bash install_dependencies.sh
   $ source deactivate

#. **ACTIVATE MINICONDA3 ENVIRONMENT** and run a pipeline::
   
   $ source activate [CONDA_ENV]
   $ java -jar -Dconfig.file=backends/backend.conf -Dbackend.default=[BACKEND] cromwell-30.2.jar run [WDL] -i input.json
   $ source deactivate
   

Genome data installation
------------------------
On Google Cloud TSV files are already installed and shared on a bucket `gs://encode-chip-seq-pipeline-genome-data <https://console.cloud.google.com/storage/browser/encode-pipeline-genome-data?project=encode-dcc-1016>`_. On DNANexus platform TSV files are on `dx://project-FB7q5G00QyxBbQZb5k11115j <https://platform.dnanexus.com/projects/FB7q5G00QyxBbQZb5k11115j/data/>`_.

.. note:: **BUT WE RECOMMEND THAT YOU COPY THESE FILES TO YOUR OWN BUCKET OR DNANEXUS PROJECT TO PREVENT EGRESS TRAFFIC COST FROM BEING BILLED TO OUR SIDE EVERYTIME YOU RUN A PIPELINE.** You will need to modify URIs in all ``.tsv`` files to correctly point to genome data files on your own bucket or project.

Supported genomes:

  * hg38: ENCODE `GRCh38_no_alt_analysis_set_GCA_000001405 <https://www.encodeproject.org/files/GRCh38_no_alt_analysis_set_GCA_000001405.15/@@download/GRCh38_no_alt_analysis_set_GCA_000001405.15.fasta.gz>`_
  * mm10: ENCODE `mm10_no_alt_analysis_set_ENCODE <https://www.encodeproject.org/files/mm10_no_alt_analysis_set_ENCODE/@@download/mm10_no_alt_analysis_set_ENCODE.fasta.gz>`_
  * hg19: ENCODE `GRCh37/hg19 <http://hgdownload.cse.ucsc.edu/goldenpath/hg19/encodeDCC/referenceSequences/male.hg19.fa.gz>`_
  * mm9: `mm9, NCBI Build 37 <http://hgdownload.cse.ucsc.edu/goldenPath/mm9/bigZips/mm9.2bit>`_

A TSV file will be generated under ``[DEST_DIR]``. Use it for ``[PIPELINE].genome_tsv`` value in your ``input.json`` file.

.. note:: Do not install genome data on Stanford clusters (Sherlock, SCG and Kundaje lab). They already have all genome data installed and shared. Use ``genome/sherlock/[GENOME]_sherlock.tsv``, ``genome/scg/[GENOME]_scg.tsv`` or ``genome/klab/[GENOME]_klab.tsv`` as your TSV file.

If you don't have ``Docker`` on your system then use ``Conda`` to build genome data.

  #. For Mac OSX users, if `dependencies <#dependency-installation>`_ does not work then install ``Docker`` and try with the next method.
  #. Install `dependencies <#dependency-installation>`_.
  #. Install genome data::

     $ cd installers/
     $ source activate [CONDA_ENV]
     $ bash install_genome_data.sh [GENOME] [DEST_DIR]
     $ source deactivate

Otherwise, use the following command to build genome data with ``Docker``::

   $ cd installers/
   $ mkdir -p [DEST_DIR]
   $ cp -f install_genome_data.sh [DEST_DIR]
   $ docker run -v $(cd $(dirname [DEST_DIR]) && pwd -P)/$(basename [DEST_DIR]):/genome_data_tmp [DOCKER_CONTAINER] "cd /genome_data_tmp && bash install_genome_data.sh [GENOME] ."

Custom genome data installation
-------------------------------
You can also install genome data for any species if you have a valid URL for reference ``fasta`` (``.fa``, ``.fasta`` or ``.gz``)  or ``2bit`` file. Modfy ``installers/install_genome_data.sh`` like the following. If you don't have a blacklist file for your species then comment out the line ``BLACKLIST=``.

  .. code-block:: bash
    
	elif [[ $GENOME == "mm10" ]]; then
	  REF_FA="https://www.encodeproject.org/files/mm10_no_alt_analysis_set_ENCODE/@@download/mm10_no_alt_analysis_set_ENCODE.fasta.gz"
	  BLACKLIST="http://mitra.stanford.edu/kundaje/genome_data/mm10/mm10.blacklist.bed.gz"

	elif [[ $GENOME == "[YOUR_CUSTOM_GENOME_NAME]" ]]; then
	  REF_FA="[YOUR_CUSTOM_GENOME_FA_OR_2BIT_URL]"
	  BLACKLIST="[YOUR_CUSTOM_GENOME_BLACKLIST_BED]" # if there is no blacklist then comment this line out.

	fi

MySQL database configuration
----------------------------
There are several advantages (call-caching and managing multiple workflows) to use Cromwell with MySQL DB. Call-caching is disabled in ``[BACKEND_FILE]`` by default.

Find an initialization script directory ``[INIT_SQL_DIR]`` for MySQL database. It's located at ``docker_image/mysql`` on github repo of any ENCODE/Kundaje lab WDL pipelines. If you want to change username and password, make sure to match with those in the following command lines and ``[BACKEND_FILE]`` (``backends/backend_with_db.conf``).

Running MySQL server with ``Docker``
------------------------------------
Choose your destination directory ``[MYSQL_DB_DIR]`` for storing all data::

	$ docker run -d --name mysql-cromwell -v [MYSQL_DB_DIR]:/var/lib/mysql -v [INIT_SQL_DIR]:/docker-entrypoint-initdb.d -e MYSQL_ROOT_PASSWORD=cromwell -e MYSQL_DATABASE=cromwell_db --publish 3306:3306 mysql

To stop MySQL::

	$ docker stop mysql-cromwell

Running MySQL without ``Docker``
--------------------------------
Ask your DB admin to run ``[INIT_SQL_DIR]``. You cannot specify destination directory for storing all data. It's locally stored on ``/var/lib/mysql`` for most versions of MySQL by default.

