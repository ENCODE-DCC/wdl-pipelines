Usage and configuration for running WDL pipelines with Cromwell
===================================================

# Usage

Choose `[BACKEND_FILE]`, `[BACKEND]`, `[WDL]`, `[PIPELINE]`, `[CONDA_ENV]` and `[WORKFLOW_OPT]` according to your platforms, kind of pipeline (`.wdl`) and presence of MySQL database and `Docker`.

* `[BACKEND_FILE]` (not required for DNANexus)
    - `backends/backend.conf` : backend conf. file for all backends.
    - `backends/backend_db.conf` : backend conf. file for all backends with MySQL DB.
* `[BACKEND]` (not required for DNANexus)
    - `Local` : local (by default).
    - `google` : Google Cloud Platform.
    - `sge` : Sun GridEngine.
    - `slurm` : SLURM.
* `[PIPELINE]`
    - `atac` : ENCODE ATAC/DNase-Seq pipeline
    - `chip` : AQUAS Transcription Factor and Histone ChIP-Seq processing pipeline
* `[WDL]`
    - `atac.wdl` : ENCODE ATAC/DNase-Seq pipeline
    - `chip.wdl` : AQUAS Transcription Factor and Histone ChIP-Seq processing pipeline
* `[CONDA_ENV]` (for systems without `Docker` support)
    - `encode-atac-seq-pipeline` : ENCODE ATAC/DNase-Seq pipeline
    - `encode-chip-seq-pipeline` : AQUAS Transcription Factor and Histone ChIP-Seq processing pipeline
* `[WORKFLOW_OPT]` (not required for DNANexus)
    - `docker.json` : for systems with `Docker` support (Google Cloud, local, ...).
    - `sge.json` : Sun GridEngine (here you can specify your own queue and parallel environment).
    - `slurm.json` : SLURM (here you can specify your partition or account).

## Running on DNANexus
Download the latest `dxWDL` first.
```
$ wget https://github.com/dnanexus/dxWDL/releases/download/0.60.2/dxWDL-0.60.2.jar
$ chmod +x dxWDL-0.60.2.jar
```
In order to run our pipeline on DNANexus, you first need to convert `[WDL]` to an equivalent workflow on DNANexus. A workflow on your current DNANexus project will be generated on `[DX_PRJ]/[DEST_DIR_ON_DX]` then specify an output directory and run it.
```
$ java -jar dxWDL-0.60.2.jar compile [WDL] -f -folder [DEST_DIR_ON_DX] -defaults input.json
```

## Running with Cromwell (single workflow mode)
Download the latest `cromwell` first.
```
$ wget https://github.com/broadinstitute/cromwell/releases/download/30.2/cromwell-30.2.jar
$ chmod +x cromwell-30.2.jar
```
Command line interface to run a single pipeline.
```
$ java -jar -Dconfig.file=[BACKEND_FILE] -Dbackend.default=[BACKEND] cromwell-30.2.jar run [WDL] -i input.json -o [WORKFLOW_OPT]
```

## Running with Cromwell (server mode)
Use [Cromwell server REST API](https://cromwell.readthedocs.io/en/develop/api/RESTAPI/#cromwell-server-rest-api) for submitting/monitoring/stopping your pipielines.
```
$ java -jar -Dconfig.file=[BACKEND_FILE] -Dbackend.default=[BACKEND] cromwell-30.2.jar server
```

## Running with Cromwell (server mode for Google Cloud)
Use [Cromwell server REST API](https://cromwell.readthedocs.io/en/develop/api/RESTAPI/#cromwell-server-rest-api) for submitting/monitoring/stopping your pipielines. All pipeline outputs will be stored on `[GC_BUCKET]`.
```
$ java -jar -Dconfig.file=backends/backend.conf -Dbackend.default=google -Dbackend.providers.google.config.project=[PROJ_NAME] -Dbackend.providers.google.config.root=[GC_BUCKET] cromwell-30.2.jar server
```

# Configuration for each platform

### DNANexus

1) Sign up for an account on [DNANexus](https://www.dnanexus.com).
2) Create a project `[DX_PRJ]`.
3) Install [DNANexus SDK](https://wiki.dnanexus.com/Downloads#DNAnexus-Platform-SDK) on your local computer and login on that project.
    ```
    $ pip install dxpy
    $ dx login
    ```
4) Convert WDL to a workflow on DNANexus web UI. Make sure that URIs in your `input.json` are valid (starting with `dx://`) for DNANexus.
    ```
    $ java -jar dxWDL-0.60.2.jar -f -folder [DEST_DIR_ON_DX] -defaults input.json
    ```
5) Check if a new workflow is generated on a directory `[DEST_DIR_ON_DX]` on your project `[DX_PRJ]`.
6) Click on a workflow, specify output directory and then launch it.

### Google Cloud Platform

1) Create a [Google Project](https://console.developers.google.com/project).
2) Set up a [Google Cloud Storage bucket](https://console.cloud.google.com/storage/browser) to store outputs.
3) Enable the following API's in your [API Manager](https://console.developers.google.com/apis/library).
    * Google Compute Engine
    * Google Cloud Storage
    * Genomics API
4) Set quota for `Google Compute Engine API` on https://console.cloud.google.com/iam-admin/quotas per region. Increase quota for SSD/HDD storage, number of vCPUs to process more samples faster simulateneouly.
    * CPUs
    * Persistent Disk Standard (GB)
    * Persistent Disk SSD (GB)
    * In-use IP addresses
    * Networks
5) Set `default_runtime_attributes.zones` in `workflow_opts/docker_google.json` as your preferred Google Cloud zone.
    ```
    {
      "default_runtime_attributes" : {
        ...
        "zones": "us-west1-a us-west1-b us-west1-c",
        ...
    }
    ```
6) Set `default_runtime_attributes.preemptible` as `"0"` to disable preemptible instances. Pipeline defaults not to use [preemptible instances](https://cloud.google.com/compute/docs/instances/preemptible). If all retrial fails then the instance will be upgraded to a regular one. **Disabling it will cost you significantly more** but you can get your samples processed much faster and stabler. Preemptible instance is disabled by default for hard tasks like `bowtie2` and `bwa` since they can take longer than the limit (24 hours) of preemptible instances.
    ```
    {
      "default_runtime_attributes" : {
        ...
        "preemptible": "0",
        ...
    }
    ```
7) If you are already on a VM instance on your Google Project. Skip step 8 and 9.
8) Install [Google Cloud Platform SDK](https://cloud.google.com/sdk/downloads) and authenticate through it. You will be asked to enter verification keys. Get keys from the URLs they provide.
    ```
    $ gcloud auth login --no-launch-browser
    $ gcloud auth application-default login --no-launch-browser
    ```
9) Get on the Google Project.
    ```
    $ gcloud config set project [PROJ_NAME]
    ```
10) You don't have to repeat step 1-9 for next pipeline run. Credential information will be stored in `$HOME/.config/gcloud`. Go directly to step 11.
11) Run a pipeline. Make sure that URIs in your `input.json` are valid (starting with `gs://`) for Google Cloud. Use any string for `[SAMPLE_NAME]` to distinguish between multiple samples.
    ```
    $ java -jar -Dconfig.file=backends/backend.conf -Dbackend.default=google -Dbackend.providers.google.config.project=[PROJ_NAME] -Dbackend.providers.google.config.root=[OUT_BUCKET]/[SAMPLE_NAME] cromwell-30.2.jar run [WDL] -i input.json -o workflow_opts/docker_google.json
    ```

### Local computer with `Docker`

1) Install [dependencies](#dependency-installation) for installing genome data.
2) Install [genome data](#genome-data-installation).
3) Run a pipeline.
    ```
    $ java -jar -Dconfig.file=backends/backend.conf cromwell-30.2.jar run [WDL] -i input.json -o workflow_opts/docker.json
    ```

### Local computer without `Docker`

1) Install [dependencies](#dependency-installation).
2) Install [genome data](#genome-data-installation).
3) Run a pipeline.
    ```
    $ source activate [CONDA_ENV]
    $ java -jar -Dconfig.file=backends/backend.conf cromwell-30.2.jar run [WDL] -i input.json
    $ source deactivate
    ```

### Sun GridEngine (SGE)

Genome data have already been installed and shared on Stanford SCG4. You can skip step 3 on SCG4.
1) Set your parallel environment (`default_runtime_attributes.sge_pe`) and queue (`default_runtime_attributes.sge_queue`) in `workflow_opts/sge.json`. If there is no parallel environment on your SGE then ask your SGE admin to create one. `sge_queue` is optional.
    ```
    $ qconf -spl
    ```
2) Install [dependencies](#dependency-installation).
3) Install [genome data](#genome-data-installation).
4) Run a pipeline.
    ```
    $ source activate [CONDA_ENV]
    $ java -jar -Dconfig.file=backends/backend.conf -Dbackend.default=sge cromwell-30.2.jar run [WDL] -i input.json -o workflow_opts/sge.json
    $ source deactivate
    ```

### SLURM

Genome data have already been installed and shared on Stanford Sherlock. You can skip step 3 on Sherlock.
1) Set your partition (`default_runtime_attributes.slurm_partition`) or account (`default_runtime_attributes.slurm_account`) in `workflow_opts/slurm.json`. Those two attibutes are optional according to your SLURM server configuration.
2) Install [dependencies](#dependency-installation).
3) Install [genome data](#genome-data-installation).
4) Run a pipeline.
    ```
    $ source activate [CONDA_ENV]
    $ java -jar -Dconfig.file=backends/backend.conf -Dbackend.default=slurm cromwell-30.2.jar run [WDL] -i input.json -o workflow_opts/slurm.json
    $ source deactivate
    ```

### Kundaje lab cluster with `Docker`

Jobs will run locally without being submitted to Sun GridEngine (SGE). Genome data have already been installed and shared.
1) Run a pipeline. 
    ```
    $ java -jar -Dconfig.file=backends/backend.conf -Dbackend.default=Local cromwell-30.2.jar run [WDL] -i input.json -o workflow_opts/docker.json
    ```

### Kundaje lab cluster with Sun GridEngine (SGE)

Jobs will be submitted to Sun GridEngine (SGE) and distributed to all server nodes. Genome data have already been installed and shared.
1) Install [dependencies](#dependency-installation).
2) Run a pipeline.
    ```
    $ source activate [CONDA_ENV]
    $ java -jar -Dconfig.file=backends/backend.conf -Dbackend.default=sge cromwell-30.2.jar run [WDL] -i input.json -o workflow_opts/sge.json
    $ source deactivate
    ```

# Dependency installation

**WE DO NOT RECOMMEND RUNNING OUR PIPELINE WITHOUT `DOCKER`!** Use it with caution.
1) **Our pipeline is for BASH only. Set your default shell as BASH**.
2) For Mac OSX users, do not install dependencies and just install `Docker` and use our pipeline with it.
3) Remove any Conda (Anaconda Python and Miniconda) from your `PATH`. **PIPELINE WILL NOT WORK IF YOU HAVE OTHER VERSION OF CONDA BINARIES IN `PATH`**.
4) Install Miniconda3 for 64-bit Linux on your system. Miniconda2 will not work. If your system is 32-bit Linux then try with `x86_32`.
   ```
   $ wget https://repo.continuum.io/miniconda/Miniconda3-latest-Linux-x86_64.sh
   $ bash Miniconda3-latest-Linux-x86_64.sh -b -p [MINICONDA3_INSTALL_DIR]
   ```
5) Add `PATH` for our pipeline Python scripts and Miniconda3 to one of your bash startup scripts (`$HOME/.bashrc`, `$HOME/.bash_profile`...). 
   ```
   unset PYTHONPATH
   export PYTHON_EGG_CACHE=/tmp

   export PATH=[WDL_PIPELINE_DIR]/src:$PATH
   export PATH=[MINICONDA3_INSTALL_DIR]/bin:$PATH
   ```
6) Re-login.
7) Make sure that conda correctly points to `[MINICONDA3_INSTALL_DIR]/bin/conda`.
   ```
   $ which conda
   ```
8) Install dependencies on Minconda3 environment. Java 8 JDK and Cromwell-29 are included in the installation.
   ```
   $ cd installers/
   $ source activate [CONDA_ENV]
   $ bash install_dependencies.sh
   $ source deactivate
   ```
9) **ACTIVATE MINICONDA3 ENVIRONMENT** and run a pipeline.
   ```
   $ source activate [CONDA_ENV]
   $ java -jar -Dconfig.file=backends/backend.conf -Dbackend.default=[BACKEND] cromwell-30.2.jar run [WDL] -i input.json
   $ source deactivate
   ```

# Genome data installation

**WE DO NOT RECOMMEND RUNNING OUR PIPELINE WITH LOCALLY INSTALLED/BUILT GENOME DATA!** Use it with caution. **We will provide an official downloader for all genome data later**. Cromwell is planning to support AWS buckets (`s3://`). Until then, use this installer.
**On Google Cloud TSV** files are already installed and shared on a bucket `gs://encode-chip-seq-pipeline-genome-data`.

Supported genomes:

  * hg38: ENCODE [GRCh38_no_alt_analysis_set_GCA_000001405](https://www.encodeproject.org/files/GRCh38_no_alt_analysis_set_GCA_000001405.15/@@download/GRCh38_no_alt_analysis_set_GCA_000001405.15.fasta.gz)
  * mm10: ENCODE [mm10_no_alt_analysis_set_ENCODE](https://www.encodeproject.org/files/mm10_no_alt_analysis_set_ENCODE/@@download/mm10_no_alt_analysis_set_ENCODE.fasta.gz)
  * hg19: ENCODE [GRCh37/hg19](http://hgdownload.cse.ucsc.edu/goldenpath/hg19/encodeDCC/referenceSequences/male.hg19.fa.gz)
  * mm9: [mm9, NCBI Build 37](http://hgdownload.cse.ucsc.edu/goldenPath/mm9/bigZips/mm9.2bit)

A TSV file will be generated under `[DEST_DIR]`. Use it for `[PIPELINE].genome_tsv` value in pipeline's input JSON file.

1) Do not install genome data on Stanford clusters (Sherlock-2 and SCG4). They already have all genome data installed. Use `genome/[GENOME]_sherlock.tsv` or `genome/[GENOME]_scg4.tsv` as your TSV file.
2) For Mac OSX users, if [dependency installation](#dependency-installation) does not work then post an issue on the repo.
3) Install [dependencies](#dependency-installation) first.
4) Install genome data.
   ```
   $ cd installers/
   $ source activate [CONDA_ENV]
   $ bash install_genome_data.sh [GENOME] [DEST_DIR]
   $ source deactivate
   ```

### Custom genome data installation

You can also install genome data for any species if you have a valid URL for reference `fasta` or `2bit` file. Modfy `installers/install_genome_data.sh` like the following.
```
...
elif [[ $GENOME == "mm10" ]]; then
  REF_FA="https://www.encodeproject.org/files/mm10_no_alt_analysis_set_ENCODE/@@download/mm10_no_alt_analysis_set_ENCODE.fasta.gz"
  BLACKLIST="http://mitra.stanford.edu/kundaje/genome_data/mm10/mm10.blacklist.bed.gz"

elif [[ $GENOME == "[YOUR_CUSTOM_GENOME_NAME]" ]]; then
  REF_FA="[YOUR_CUSTOM_GENOME_FA_OR_2BIT_URL]"
  BLACKLIST="[YOUR_CUSTOM_GENOME_BLACKLIST_BED]" # if there is no blacklist then comment this line out.

fi
...
```

# MySQL database configuration

There are several advantages (call-caching and managing multiple workflows) to use Cromwell with MySQL DB. Call-caching is disabled in `[BACKEND_FILE]` by default.

Find an initialization script directory `[INIT_SQL_DIR]` for MySQL database. It's located at `/docker_image/mysql` on github repo of any ENCODE/Kundaje lab WDL pipelines. If you want to change username and password, make sure to match with those in the following command lines and `[BACKEND_FILE]` (`backends/backend_with_db.conf`).

## Running MySQL server with `Docker`

Choose your destination directory `[MYSQL_DB_DIR]` for storing all data.
```
$ docker run -d --name mysql-cromwell -v [MYSQL_DB_DIR]:/var/lib/mysql -v [INIT_SQL_DIR]:/docker-entrypoint-initdb.d -e MYSQL_ROOT_PASSWORD=cromwell -e MYSQL_DATABASE=cromwell_db --publish 3306:3306 mysql
```
To stop MySQL
```
$ docker stop mysql-cromwell
```

## Running MySQL without `Docker`

Ask your DB admin to run `[INIT_SQL_DIR]`. You cannot specify destination directory for storing all data. It's locally stored on `/var/lib/mysql` for most versions of MySQL by default.

