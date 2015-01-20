Microbiome Helper
=================

An assortment of scripts to help process and automate various microbiome and metagenomic bioinformatic tools.

As I analyze various microbiome datasets (16S, metagenomics, etc.), I tend to write scripts to automate processes or to convert file formats between tools. 

These are the collection of such tools. 

STAMP
-----

I find STAMP to be a very useful tool for creating figures and doing statistical analyses. Here are scripts to covert tables from different tools into STAMP input profile files.

**metaphlan_to_stamp.pl**: This is a simple script that converts a merged metaphlan output file to a STAMP profile file.

**biom_to_stamp.py**: STAMP has built-in BIOM conversion, but depending on where the BIOM file comes from there can be slight format problems with STAMP. Specifically, this script handles PICRUSt BIOM output (both KOs and KEGG Pathways) and QIIME 16S OTU tables (with metadata 'taxonomy').

MetaPhlan
---------

**run_metaphlan.pl**: Wraps the metaphlan.py package and handles running multiple samples at once as well as handling paired end data a bit more cleaner. It also runs each sample in parallel and merges the results into a single output file.


Metagenomics workflow (starting with demultiplexed MiSeq fastq files)
---------------------

1. Run fastqc to allow manual inspection of the quality of sequences (optional)

        mkdir fastqc_out
        fastqc -t 4 raw_miseq_data/* -o fastqc_out/

2. Stich paired end reads together

        run_pear.pl -p 4 -o stitched_reads raw_miseq_data/*

3. Filter sequences for human contamination (this is a bit slow and the following bowtie2 method is quicker)
    
        run_deconseq.pl -p 4 -o screened_reads ./stitched_reads/*.assembled.*

4. Run bowtie2 to screen out human sequences
    
        run_human_filter.pl -p 4 -o screened_reads/ stitched_reads/*.assembled*

5. Run Metaphlan for taxanomic composition

        run_metaphlan.pl -p 4 -o metaphlan_taxonomy.txt screened_reads/*

6. Convert from metaphlan to stamp profile file

        metaphlan2stamp.pl metaphlan_taxonomy.txt > metaphlan_taxonomy.spf

7. Run pre-humann (diamond search)

        run_pre_humann.pl -p 4 -o pre_humann/ screened_reads/*

8. Run humann (link files to humann "input" directory and then run humann with 4 threads)

        ln -s $PWD/pre_humann/* ~/programs/humann-0.99/input/
        cd ~/programs/human-0.99/
        scons -j 4

9. Convert humann output to stamp format

        humann_to_stamp.py 04b-hit-keg-mpm-cop-nul-nve-nve.txt > hummann_modules.spf
        humann_to_stamp.py 04b-hit-keg-mpt-cop-nul-nve-nve.txt > hummann_pathways.spf
		humann_to_stamp.py 01b-hit-keg-cat.txt > hummann_kos.spf
