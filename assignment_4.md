# This assignment is designed to fulfill option 1 for EPP 622 test 4. 

Option 1 - RNASEQ - I want to learn more about Isaac and RNA read mapping. The instructions to the assignment are shown below (to be deleted later):
 
The goal.
Map 12 fastq files to the peach genome with star and salmon and examine how reads are being assigned by each strategy.


```
for FILE in *.final.out; do ls $FILE >> STAR_uniquely_mapped_reads.txt; grep -E 'Uniquely mapped reads %' $FILE >> STAR_uniquely_mapped_reads.txt; done
for FILE in *.final.out; do ls $FILE >> STAR_multiple_loci_reads.txt; grep -E '% of reads mapped to multiple loci' $FILE >> STAR_multiple_loci_reads.txt; done
for FILE in *.final.out; do ls $FILE >> STAR_too_many_loci_reads.txt; grep -E '% of reads mapped to too many loci' $FILE >> STAR_too_many_loci_reads.txt; done
for FILE in *.final.out; do ls $FILE >> STAR_too_many_mismatches_reads.txt; grep -E '% of reads unmapped: too many mismatches' $FILE >> STAR_too_many_mismatches_reads.txt; done
for FILE in *.final.out; do ls $FILE >> STAR_too_short_reads.txt; grep -E '% of reads unmapped: too short' $FILE >> STAR_too_short_reads.txt; done
for FILE in *.final.out; do ls $FILE >> STAR_unmapped_other_reads.txt; grep -E '% of reads unmapped: other' $FILE >> STAR_unmapped_other_reads.txt; done
for FILE in *.final.out; do ls $FILE >> STAR_chimeric_reads.txt; grep -E '% of chimeric reads' $FILE >> STAR_chimeric_reads.txt; done
```

```
for FILE in slurm*; do grep -E 'readonefile is' $FILE >> salmon_mapping_table.txt; grep -E 'Number of mappings discarded because of alignment score' $FILE >> salmon_mapping_table.txt; grep -E 'Number of fragments entirely discarded because of alignment score' $FILE >> salmon_mapping_table.txt; grep -E 'Number of fragments discarded because they have only dovetail (discordant) mappings to valid targets' $FILE >> salmon_mapping_table.txt; grep -E 'Number of fragments discarded because they are best-mapped to decoys' $FILE >> salmon_mapping_table.txt; grep -E 'Number of fragments discarded because they have only dovetail' $FILE >> salmon_mapping_table.txt; grep -E 'Mapping rate' $FILE >> salmon_mapping_table.txt; done
```

```
for FOLDER in *_out; do echo $FOLDER >> salmon_gene_mapping.txt; grep -E 'Prupe.6G364900.1' $FOLDER/quant.sf >> salmon_gene_mapping.txt; grep -E 'Prupe.1G531100.1' $FOLDER/quant.sf >> salmon_gene_mapping.txt; grep -E 'Prupe.1G531100.2' $FOLDER/quant.sf >> salmon_gene_mapping.txt; grep -E 'Prupe.1G531400.1' $FOLDER/quant.sf >> salmon_gene_mapping.txt; grep -E 'Prupe.1G549600.1' $FOLDER/quant.sf >> salmon_gene_mapping.txt; grep -E 'Prupe.1G549600.2' $FOLDER/quant.sf >> salmon_gene_mapping.txt; grep -E 'Prupe.1G549600.3' $FOLDER/quant.sf >> salmon_gene_mapping.txt; done
```



The set up.
There are 12 fastq files in /lustre/isaac/proj/UTK0208/test4/raw_data
Experimental design: 2 apricot cultivars (Badami and Bergeron), 2 time points (400 chill hours and 800 chill hours), 3 biological reps (here labeled as “clones”)
This is single end read data, not paired
We will still use the peach genome (its close enough to apricot to work reasonably well)
Don’t forget indexing for star and salmon is already done, since we used this genome in lab
Work should be done on isaac
If you get a “bus error” for salmon try increasing ram to 8G instead of 4G as it was in the lab
I have NOT removed the nongene categories from the tsv files (“ambiguous”, “no feature”, etc)
 
Grading:
Results (15/25) - can be separate or embedded in the wiki documentation of reads for each library
 STAR: Build a table with the 12 libraries as rows and the following columns from the output files (grep and sed are useful friends here):
Uniquely mapped reads %
% of reads mapped to multiple loci
% of reads mapped to too many loci
% of reads unmapped: too many mismatches
% of reads unmapped: too short
% of reads unmapped: other
% of chimeric reads
Salmon: Build a table with the 12 libraries are rows and the following columns from the output files (grep and sed are useful friends here):
Number of mappings discarded because of alignment score 
Number of fragments entirely discarded because of alignment score
Number of fragments discarded because they are best-mapped to decoys
Number of fragments discarded because they have only dovetail (discordant) mappings to valid targets
Mapping rate
Look at 4 of Meg’s favorite genes
Prupe.6G364900 (FT, 1 transcript)
Prupe.1G531100 (misannotated DAM1-3, 2 transcipts)
Prupe.1G531400  (DAM4, 1 transcript)
Prupe.1G549600 (SEEDSTICK, 3 transcripts)
In a chart, put # of reads mapped by star to each gene and # of reads assigned to each transcript by salmon (grep and sed are useful friends here)
Analyze the pattern of reads mapped by STAR to single genes vs the reads mapped salmon to transcripts. Are more reads overall being mapped by STAR or Salmon? For the two genes with more than one transcript, is there a dominant transcript across samples? 
Results template - This is an example of the tables of results I am looking for
Documentation (10/25)
 Wiki only (no plain text. do not copy/paste command line output.)
 I want details, including each Isaac submission script and enough information about each command that I can follow along with why you are doing each step.
