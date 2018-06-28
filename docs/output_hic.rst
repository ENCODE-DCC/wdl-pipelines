Output specification for ``hic.wdl``
=====================================

.. toctree::
   :maxdepth: 2
   :caption: Contents:

All output filenames keep prefixes from corresponding input filenames. 

#. ``Cromwell``: stores outputs for each task under ``cromwell-executions/[WORKFLOW_ID]/call-[TASK_NAME]/shard-[IDX]``.

.. csv-table:: 
    :header: Task name, File, Description
   
    align, \*.bam | \*.txt | \*.res.txt, Raw BAM | txt | res.txt
    merge, \*.bam, Merged BAM
    merge_sort, \*.txt, Merged txt
    align_qc,\*.json, Align QC JSON 
    dedup, \*.txt, Deduped txt
    merge_pairs, \*.txt, Merged and deduped txt
    create_hic, \*.hic, Final HiC
    create_tads,\*.bedpe, Final TADs
    strip_headers, \*.sam, SAM with no headers (for comparison)
    compare_md5sum, \*.tsv | \*.txt | \*.json, TSV | txt | JSON  for comparison


    