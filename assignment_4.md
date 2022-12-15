# Assignment 4: Compare Read Mapping with STAR and Salmon 

The goal of this assignment is to compare the read mapping statistics and read abundances from two different read mappers for RNA-Seq data, STAR, and Salmon. By comparing these two read mappers, we can understand the characteristics of each read mapper and the differences in their output. 

For this project, we'll be using 12 fastq files, each representing apricots of different cultivars, time periods, and biological replicates, and mapping them to a peach genome. 

A 'results table' is referenced throughout this assignment. It can be found at https://github.com/jacksonturnip/EPP_622/edit/main/assignment_4_results.xlsx . 
 
### FastQC

Log into ISAAC and fulfill the authentication requirements.
```
ssh [your user ID]@login.isaac.utk.edu
```

First, run FastQC on the files. FastQC allows us to visualize our read quality prior to further analysis. If FastQC reports generated from some of our data display unideal results, it may be worth considering excluding them from downstream analysis. 

Create a directory to perform FastQC, navigate to it, and populate it with our data.
```
mkdir 1_fastqc
cd 1_fastqc
ln -s /lustre/isaac/proj/UTK0208/test4/raw_data/*fastq* .
```

Run FastQC though ISAAC by making a shell script and executing it with `sbatch`

```
nano fastqc.qsh
```

Paste the following in to this shell script:
```
#!/bin/bash
#SBATCH -J fastqc
#SBATCH --nodes=1
#SBATCH --ntasks=1
#SBATCH -A ISAAC-UTK0208
#SBATCH -p condo-epp622
#SBATCH -q condo
#SBATCH -t 00:30:00

module load fastqc

fastqc *fastq
```

Execute the script using `sbatch`.
```
sbatch fastqc.qsh
```

You can view the status of jobs submitted to ISAAC with `squeue`
```
squeue -u [your user ID]
```

Eventually the FastQC runs will finish. Upon checking them, their read quality is good enough to proceed with mapping. 

### Mapping reads with STAR

STAR (Spliced Transcriptomes to A Reference) is a read mapper that uses a previously undescribed RNA-Seq aligment algorithm to map reads from RNA-Seq data. [1] Make the directory for this analysis, navitage to it, and populate it with the same fastq files as in the `1_fastqc` directory.
```
mkdir ../2_star
cd 2_star
ln -s /lustre/isaac/proj/UTK0208/test4/raw_data/*fastq* .
```

The `sbatch` script we'll create will require a task array since it contains 12+ sequences. To generate this task array, we'll create a text file with the names of all the fastq file inputs. 

```
for FILE in *.fastq; do ls $FILE >> fastq_array.txt; done
```

Now that the fastq array is made, we'll create a `qsh` file to send to ISAAC. This file has lots of parameters ISAAC's slurm system uses to run the job on its own automated queue system. The parameters of of this script that are used by ISAAC are the following: 
line 1: execute with bash
line 2: name of the job
line 3: number of nodes to use
line 4: total number of tasks required by the job
line 5: the ID of the project associated with the job
line 6: the condo of the job
line 7: use condo
line 8: length of time to run job
line 9: memory dedicated to job per cpu
line 10: use an array with 12 components (our 12 fastq files)
line 11: when to mail the results of the job
line 12: which address to mail the job

For the script itself, it creates a variable, `readonefile` from the file selected by the task array file we previously generated. For troubleshooting purposes, the name of this variable is repeated. This script then loads the `star` module, and executes the `STAR` command with the following parameters. The first is the indexing directory, which has already been provided by Dr. Meg Staton. The second lists the number of threads required for the job. The third shows the input fastq file for the command. The fourth provides the prefix for the output file, the fifth shows the output file type, and the sixth shows how you want to quantify the results.
For our purposes, we'll load in the current file at each step of the array (readonefile) and request STAR to provide read counts of mapped genes (GeneCounts). 

Create this script with `nano`.

```
nano star.qsh
```

Paste the following into the script and save it.

```
#!/bin/bash
#SBATCH -J star-map
#SBATCH --nodes=1 
#SBATCH --ntasks=2
#SBATCH -A ISAAC-UTK0208
#SBATCH -p condo-epp622
#SBATCH -q condo
#SBATCH -t 00:30:00
#SBATCH --mem-per-cpu=8G
#SBATCH --array=1-12
#SBATCH --mail-type=BEGIN,END,FAIL
#SBATCH --mail-user=jturne88@vols.utk.edu

readonefile=$(sed -n -e "${SLURM_ARRAY_TASK_ID} p" fastq_array.txt)
echo "readonefile is $readonefile"

module load star
for FILE in *.fastq; do  \
STAR \
--genomeDir /lustre/isaac/proj/UTK0208/rnaseq/raw_data/STAR_gtf_idx \
--runThreadN 2 \
--readFilesIn $readonefile \
--outFileNamePrefix $readonefile \
--outSAMtype BAM SortedByCoordinate \
--quantMode TranscriptomeSAM GeneCounts;
done
```

Run the script with `sbatch`.
```
sbatch star.qsh
```

While this job is running (you can check its status with `squeue` -- if it finishes quickly, something went wrong and you'll need to re-consult this input), we'll move on to running Salmon.

### Mapping reads with Salmon

Salmon, like STAR, is a read mapper that identifies the abundances of genes in RNA-seq data [2]. Notably, it's the first read mapper of its kind to correct for GC content bias, making it more accurate [2]. Make a directory for it, navigate to it, and populate it with the fastq files as we did for the `2_star` directory.

```
mkdir ../3_salmon
cd ../3_salmon
ln -s /lustre/isaac/proj/UTK0208/test4/raw_data/*fastq* .
```

Create the fastq array as we did for the same step in our STAR analysis. It's worth noting I gave this array a different name. 

```
for FILE in *.fastq; do ls $FILE >> salmon_array.txt; done
```

Now that the fastq array is completed for this analysis, we'll now build its script to submit to ISAAC. The first 12 lines have the same parameters as the script for the STAR analysis, but have those parameters changed slightly to use more cores to run the job over less time. The `readonefile` variable is also constructed the same way, but a new variable, `quantdir` is created instead and echoed in the same way for troubleshooting. This script will then run Salmon, which is installed in a nother directory that we path to. The  arguments for this command are as follows: The index for the command, which is provided by Dr. Meg Staton; the library type; the input reads (since this is single-end data, the `--unmatedReads` parameter is given); the output directory; and the number of threads to run the job.

Create this script with `nano`.

```
nano salmon.qsh
```

Paste the following into the script and save it.

```
#!/bin/bash
#SBATCH -J salmonTA
#SBATCH --nodes=1
#SBATCH --ntasks=4
#SBATCH -A ISAAC-UTK0208
#SBATCH -p condo-epp622
#SBATCH -q condo
#SBATCH -t 00:20:00
#SBATCH --mem-per-cpu=8G
#SBATCH --array=1-12
#SBATCH --mail-type=BEGIN,END,FAIL
#SBATCH --mail-user=jturne88@vols.utk.edu

readonefile=$(sed -n -e "${SLURM_ARRAY_TASK_ID} p" salmon_array.txt)
echo "readonefile is $readonefile"
quantdir=$(echo $readonefile | sed  's/.fastq/_out/')
echo "quantdir is $quantdir"

/lustre/isaac/proj/UTK0208/rnaseq/software/salmon-1.9.0_linux_x86_64/bin/salmon \
quant \
-i /lustre/isaac/proj/UTK0208/rnaseq/raw_data/salmon_transcripts_index \
-l IU \
--unmatedReads $readonefile \
--validateMappings \
-o $quantdir \
--threads 8
```

Submit the job to ISAAC with `sbatch`.
```
sbatch salmon.qsh
```

Eventually, these jobs will finish. When they do, we can populate the results table for this assignment.

### Populating Results Table

We'll be populating the table describing the results of the STAR and salmon analyses previously run. The finished table is in the github labeled `assignment_4_results.xlsx`. First, determine the number of reads per each fastq file. I do this by navigating to the `1_fastqc` directory and using the following script to generate a list of line lengths of each file. Because fastq files one read per every four lines, I divided the output from this script by 4 and populated each value in the table. 

```
cd ../1_fastqc
for FILE in *.fastq; do ls $FILE >> how_many_lines.txt; wc -l $FILE >> how_many_lines.txt; done
```

Next, the table asks for characteristics about mapped reads from the previously performed STAR analysis, such as the percentage of uniquely mapped reads, the percentage of reads mapped to multiple loci, and others. Navigate to the STAR directory and run the following scripts. Thess scripts create seperate files for each category described in the "STAR" section of the results table. It uses `grep -E` to capture an entire line of text within files generated by STAR with the `final.out` suffix using a query with each categories as defined in each section's columns. 
The results table has been populated with the values generated from these scripts. Use `less [file]` to view the contents of each.

```
cd ../2_STAR
for FILE in *.final.out; do ls $FILE >> STAR_uniquely_mapped_reads.txt; grep -E 'Uniquely mapped reads %' $FILE >> STAR_uniquely_mapped_reads.txt; done
for FILE in *.final.out; do ls $FILE >> STAR_multiple_loci_reads.txt; grep -E '% of reads mapped to multiple loci' $FILE >> STAR_multiple_loci_reads.txt; done
for FILE in *.final.out; do ls $FILE >> STAR_too_many_loci_reads.txt; grep -E '% of reads mapped to too many loci' $FILE >> STAR_too_many_loci_reads.txt; done
for FILE in *.final.out; do ls $FILE >> STAR_too_many_mismatches_reads.txt; grep -E '% of reads unmapped: too many mismatches' $FILE >> STAR_too_many_mismatches_reads.txt; done
for FILE in *.final.out; do ls $FILE >> STAR_too_short_reads.txt; grep -E '% of reads unmapped: too short' $FILE >> STAR_too_short_reads.txt; done
for FILE in *.final.out; do ls $FILE >> STAR_unmapped_other_reads.txt; grep -E '% of reads unmapped: other' $FILE >> STAR_unmapped_other_reads.txt; done
for FILE in *.final.out; do ls $FILE >> STAR_chimeric_reads.txt; grep -E '% of chimeric reads' $FILE >> STAR_chimeric_reads.txt; done

# to view a particular file (for this example, the STAR_uniquely_mapped_reads.txt file)
less STAR_uniquely_mapped_reads.txt
```

We'll move on to the section of the table describing how salmon mapped these reads. Below is a similar script that uses `grep -E` to grab each line and put it into one file instead of several seperate ones. Unlike the previous script, the relevant information for the results table is located in the individual slurm files. This structure is a better approach to gathering the relevant information and will be used henceforth for populating the results table. Navigate to the `3_salmon` directory and run the following script:
As with this script and for all scripts used to generate results for the table, the output of the following script was used to populate the results table.

```
cd ../3_salmon
for FILE in slurm*; 
do grep -E 'readonefile is' $FILE >> salmon_mapping_table.txt; 
grep -E 'Number of mappings discarded because of alignment score' $FILE >> salmon_mapping_table.txt; 
grep -E 'Number of fragments entirely discarded because of alignment score' $FILE >> salmon_mapping_table.txt; 
grep -E 'Number of fragments discarded because they have only dovetail (discordant) mappings to valid targets' $FILE >> salmon_mapping_table.txt; 
grep -E 'Number of fragments discarded because they are best-mapped to decoys' $FILE >> salmon_mapping_table.txt; 
grep -E 'Number of fragments discarded because they have only dovetail' $FILE >> salmon_mapping_table.txt; 
grep -E 'Mapping rate' $FILE >> salmon_mapping_table.txt; done

# look at the resulting file to populate the results table
less salmon_mapping_table.txt
```

Now we'll need to find the number of reads per gene in the STAR analysis. The following script uses `grep -E` as well to grab the entire line associated with each gene. It is important to note that the first column of this file was used to populate the table, as it represents the total number of recovered reads. Once this file is created, use `less` to view its contents and populate the results table.

```
cd ../2_star
for FILE in *ReadsPerGene.out.tab; do echo $FILE >> star_gene_mapping.txt;  
grep -E 'Prupe.6G364900' $FILE >> star_gene_mapping.txt;  
grep -E 'Prupe.1G531100' $FILE >> star_gene_mapping.txt;  
grep -E 'Prupe.1G531400' $FILE >> star_gene_mapping.txt;  
grep -E 'Prupe.1G549600' $FILE >> star_gene_mapping.txt; done

less star_gene_mapping.txt
```

The section detailing the number of reads per gene in the Salmon analysis will now be populated. Some of these genes have different numbers of transcripts that must be accounted for. This script uses `grep -E` like the others to grab the lines associated with each query. The last column in this generated file is what is used to populate the results table, as it contains the number of mapped reads per gene. Once this final file is created, use `less` to view its contents and populate the results table for the last time.

```
cd ../3_salmon
for FOLDER in *_out; do echo $FOLDER >> salmon_gene_mapping.txt; 
grep -E 'Prupe.6G364900.1' $FOLDER/quant.sf >> salmon_gene_mapping.txt; 
grep -E 'Prupe.1G531100.1' $FOLDER/quant.sf >> salmon_gene_mapping.txt; 
grep -E 'Prupe.1G531100.2' $FOLDER/quant.sf >> salmon_gene_mapping.txt; 
grep -E 'Prupe.1G531400.1' $FOLDER/quant.sf >> salmon_gene_mapping.txt; 
grep -E 'Prupe.1G549600.1' $FOLDER/quant.sf >> salmon_gene_mapping.txt; 
grep -E 'Prupe.1G549600.2' $FOLDER/quant.sf >> salmon_gene_mapping.txt; 
grep -E 'Prupe.1G549600.3' $FOLDER/quant.sf >> salmon_gene_mapping.txt; done
```

### Overall Conclusions

According to both read mappers, the 400CH group has a much higher number of reads mapped than the 800CH group across all genes and samples. This implies that the 400CH time point has higher levels of expression of the measured genes. The Badami cultivar also has a higher number of reads mapped than the Bergeron cultivar accross these same genes. Despite these samples being constructed from apricots, they map well to the provided peach genome, speaking to the relatedness of the two trees. 

STAR seems to have a higher mapping rate than Salmon and more mapped reads. Some genes are better mapped by STAR than Salmon and vice versa. The transcript Prupe.1G531100.1 is dominant to Prupe.1G531100.2, but Prupe.1G549600.1, Prupe.1G549600.2, and Prupe.1G549600.3 each are dominant in different taxa. The fact that some read mappers have different results than others demonstrates the importance of using multiple read mappers in RNA-Seq analysis. Each read mapper also displays different measurements of how well genes/samples mapped (or not) compared to others. The same is true for including different transcripts, as some are more dominant in some samples than others. While STAR seems to have higher overall mapping ability than Salmon, the ability to show the read abundances of multiple transcripts can be informative for some research questions. 


# References

1. Alexander Dobin, Carrie A. Davis, Felix Schlesinger, Jorg Drenkow, Chris Zaleski, Sonali Jha, Philippe Batut, Mark Chaisson, Thomas R. Gingeras, STAR: ultrafast universal RNA-seq aligner, Bioinformatics, Volume 29, Issue 1, January 2013, Pages 15–21, https://doi.org/10.1093/bioinformatics/bts635

2. Patro, R., Duggal, G., Love, M. et al. Salmon provides fast and bias-aware quantification of transcript expression. Nat Methods 14, 417–419 (2017). https://doi.org/10.1038/nmeth.4197
