.. ENCODE genomic pipelines documentation master file, created by
   sphinx-quickstart on Mon Apr 23 14:54:14 2018.
   You can adapt this file completely to your liking, but it should at least
   contain the root `toctree` directive.

ENCODE genomic pipelines
========================

.. toctree::
   :maxdepth: 2
   :caption: Contents:
   :hidden:

   install.rst
   input_json_chip.rst
   input_json_atac.rst
   input_hic.rst
   output_chip.rst
   output_atac.rst
   output_hic.rst

Pipelines

* `ATAC-Seq pipeline <https://github.com/ENCODE-DCC/atac-seq-pipeline>`_
* `Histone/TF ChIP-Seq pipeline <https://github.com/ENCODE-DCC/chip-seq-pipeline2>`_

Pipelines in development

* `(dev) RNA-Seq pipeline <https://github.com/ENCODE-DCC/rna-seq-pipeline>`_
* `(dev) HiC pipeline <https://github.com/ENCODE-DCC/hic-pipeline>`_
* `(dev) WGBS pipeline <https://github.com/ENCODE-DCC/wgbs-pipeline>`_

Installation
------------

`Cromwell <https://github.com/broadinstitute/cromwell>`_ is our main WDL runner and we additionally support `dxWDL <https://github.com/dnanexus/dxWDL>`_ for `DNANexus platform <https://platform.dnanexus.com>`_.

``Docker`` is a container solution for software dependencies. ``Conda`` is a software package manager working on a virtual environment.

Cromwell can work with either ``Docker`` or ``Conda``. You can choose between them on your local computer. But you need to use ``Docker`` for cloud platforms (DNANexus platform and Google Cloud) and ``Conda`` for cluster engines (SLURM and SGE).

* `DNANexus Platform <install.html#dnanexus-platform>`_
* `Google Cloud Platform <install.html#google-cloud-platform>`_
* `SLURM <install.html#slurm>`_
* `Sun GridEngine (SGE) <install.html#sun-gridengine-sge>`_
* `Local (with Docker) <install.html#local-computer-with-docker>`_
* `Local (without Docker) <install.html#local-computer-without-docker>`_

Input JSON
----------

Task-level variable definition is slightly different between dxWDL and Cromwell. For example of a sample name variable, dxWDL's input JSON file looks like ``[TASK].[VAR]``::

   {
      "qc_report.name" : "test sample"
   }

But Cromwell uses ``[WORKFLOW].[TASK].[VAR]``::

   {
      "atac.qc_report.name" : "test sample"
   }

* `ATAC-Seq pipeline <input_json_atac.html>`_
* `Histone/TF ChIP-Seq pipeline <input_json_chip.html>`_
* `HiC pipeline <input_json_hic.html>`_

Output specification
--------------------
* `ATAC-Seq pipeline <output_atac.html>`_
* `Histone/TF ChIP-Seq pipeline <output_chip.html>`_
* `HiC pipeline <output_hic.html>`_

Troubleshooting
---------------

Our pipelines use the same configuration for Conda dependencies as in genomic pipelines in Anshul Kundaje's lab at Stanford. If you have any issues about Conda and its dependencies, please take a look at the following pages before you post an issue on each pipeline's github repo.

* `General troubleshooting <https://kundajelab.github.io/bds_pipeline_modules/troubleshooting.html>`_
* `ATAC pipelines <https://github.com/kundajelab/atac_dnase_pipelines/issues?q=is%3Aissue>`_
* `AQUAS Histone/TF ChIP-Seq pipeline <https://github.com/kundajelab/chipseq_pipeline/issues?q=is%3Aissue>`_

For pipeline specific issues, post them on each pipeline's github issue page.

* `ATAC-Seq pipeline <https://github.com/ENCODE-DCC/atac-seq-pipeline/issues?q=is%3Aissue>`_
* `Histone/TF ChIP-Seq pipeline <https://github.com/ENCODE-DCC/chip-seq-pipeline2/issues?q=is%3Aissue>`_
* `(dev) RNA-Seq pipeline <https://github.com/ENCODE-DCC/rna-seq-pipeline/issues?q=is%3Aissue>`_
* `(dev) HiC pipeline <https://github.com/ENCODE-DCC/hic-pipeline/issues?q=is%3Aissue>`_
* `(dev) WGBS pipeline <https://github.com/ENCODE-DCC/wgbs-pipeline/issues?q=is%3Aissue>`_

Indices and tables
------------------

* :ref:`genindex`
* :ref:`modindex`
* :ref:`search`
