Input JSON for ``hic.wdl``
===========================

.. toctree::
   :maxdepth: 2
   :caption: Contents:

Optional parameters and flags are marked with ``?``.

#. Reference genome

    Currently supported genomes:

    * hg38: ENCODE `GRCh38_no_alt_analysis_set_GCA_000001405 <https://www.encodeproject.org/files/GRCh38_no_alt_analysis_set_GCA_000001405.15/@@download/GRCh38_no_alt_analysis_set_GCA_000001405.15.fasta.gz>`_
    * mm10: ENCODE `mm10_no_alt_analysis_set_ENCODE <https://www.encodeproject.org/files/mm10_no_alt_analysis_set_ENCODE/@@download/mm10_no_alt_analysis_set_ENCODE.fasta.gz>`_

#. Input genome data files

    Choose any genome data type you want to start with and do not define others. 
    
    * ``"hic.fastq"``? : 3-dimensional array with FASTQ file path/URI. Next step is to be aligned and converted to bams.
        - 1st dimension: read number ID
        - 2nd dimension: sequencing run ID
        - 3rd dimension: library ID 
    * ``"hic.bams"``? : 3-dimensional array of raw (unfiltered) BAM file path/URI. Next step is to be merged.
        - 1st dimension: specific bam types ID
        - 2nd dimension: grouping of collisions, collisions_low_mapq, unmapped, mapq0, alignable
        - 3rd dimension: library ID
    * ``"hic.input_sort_files"``? : 2-dimensional array of not merged sort.txt files. Next step is to be merged.
        - 1st dimension: sequencing run ID
        - 2nd dimensions: library ID
    * ``"hic.input_merged_sort"``? : 1-dimensional array of merged sort.txt files. Next step is to be deduped.
        - 1st dimension: library ID
    * ``"hic.input_dedup_pairs"``? : 1-dimensional array of read pairs with no duplicates. Next step is to be merged.
        - 1st dimension: library ID
    * ``"hic.input_pairs"``? : Single file of merged read pairs with no duplicates. Next step is to create a .hic file.
    * ``"hic.chrsz"``? : Single chrom.sizes file. Used for alignment and creation of .hic file.
    * ``"hic.restriction_sites"``? : Single file containing locations of restriction sites in genome. Used for alignment.
    * ``"hic.reference_index"``? : Single reference sequence file. Used for alignment.


    ``"hic.restriction_sites"`` and ``"hic.reference_index"`` must only be defined if starting from fastq files.
  


