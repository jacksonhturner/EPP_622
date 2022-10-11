## EPP 622 Assignment 2

The purpose of this exercise is to call variants and generate some basic statistics for four samples. 

# Step 1: FastQC

We’re analyzing four different fastq files that are listed in the assignment, SRR6922141_1, SRR6922185_1, SRR6922187_1, and SRR6922236_1. We want to know first if these sequences are high quality enough to analyze. If they’re fragmented or inaccurate, then we would normally exclude them from data analysis. To check to see if they are usable,  use FastQC (version 0.11.9) to examine their quality and then download them onto our personal computers to examine them ourselves. To do this, first create a directory for FastQC and populate it with symbolic links to our fastq files.

```
mkdir 1_fastqc
cd 1_fastqc
ln -s /pickett_shared/teaching/EPP622_Fall2022/raw_data/solenopsis_invicta_test2/*fastq .
spack load fastqc
for FILE in *; do fastqc $FILE; done
scp jturne88@sphinx.ag.utk.edu:/pickett_shared/teaching/EPP622_Fall2022/analysis_test2/jturne88/1_fastqc/*html ./
```

Now, the html files can be viewed – these provide several informative metrics about the quality of our fastq files. 

SRR6922141_1:
![image](https://user-images.githubusercontent.com/80480626/195209247-16376f67-a2b6-48da-9cfc-0654c047b203.png)

There isn’t anything egregiously wrong with our samples based on the FastQC reports, so go ahead and move forward with them! 

# Step 2: Skewer

It’s standard practice to trim sequences to remove adapters, fragments, or other artifacts of the sequencing process. To do this, run skewer on each of our samples to optimally trim them.

```
mkdir ../2_skewer
cd ../2_skewer
ln -s /pickett_shared/teaching/EPP622_Fall2022/raw_data/solenopsis_invicta_test2/*.fastq .
for FILE in *.fastq; do /sphinx_local/software/skewer/skewer -t 2 -l 95 -x AGATCGGAAGAGCGGTTCAGCAGGAATGCCGAGACCGATCTCGTATGCCGTCTTCTGCTTG -Q 30 $FILE -o $FILE.trimmed; done
```

BC is a basic calculator that can be used to calculate the number of reads. Because there are four lines per read in a fastq file, the number of reads can be calculated by dividing the number of lines in a fastq file by four. Use BC in this way to determine the number of reads per fastq file and trimmed fastq file.

```
spack load bc%gcc@8.4.1
echo $(cat SRR6922141_1.fastq-trimmed.fastq | wc -l)/4 | bc
echo $(cat SRR6922141_1.fastq | wc -l)/4 | bc
echo $(cat SRR6922185_1.fastq-trimmed.fastq | wc -l)/4 | bc
echo $(cat SRR6922185_1.fastq | wc -l)/4 | bc
echo $(cat SRR6922187_1.fastq-trimmed.fastq | wc -l)/4 | bc
echo $(cat SRR6922187_1.fastq | wc -l)/4 | bc
echo $(cat SRR6922236_1.fastq-trimmed.fastq | wc -l)/4 | bc
echo $(cat SRR6922236_1. fastq | wc -l)/4 | bc
```

Before trimming, SRR6922141_1 had 1,148,398 reads -- after trimming, SRR6922141_1 had 1,068,311 reads. Before trimming, SRR6922185_1 had 1,315,096 reads before trimming and 1,254,903 reads after. Before trimming, SRR6922187_1 had 1,081,747 reads before trimming and 993,372 reads after. Before trimming, SRR6922236_1 had 977,855 reads before trimming and 928,913 reads after. Many of these removed reads were probably adapter sequences and other artifacts we would want to exclude from analysis. 

# Step 3: BWA

Now, map these reads to their respective genome. To do this, use BWA version 0.7.17, which aligns reads from a query to a database.  go ahead and make a directory for this step and copy all the information associated with the reference genome into a seperate directory. 

```
mkdir ../3_bwa
cd ../3_bwa
mkdir solenopsis_gene_index
cd solenopsis_gene_index
ln -s ../../../raw_data/solenopsis_invicta/genome/UNIL_Sinv_3.0.fasta* .
cd ..
```

Now, perform the read mapping using a for loop. This will create a sorted bam file to use in later analysis. Samtools (version 1.9) contains a suite of software used to identify variation between sequences. First, load samtools version @1.9%gcc@8.4.1 and BWA.

```
spack load bwa
spack load samtools@1.9%gcc@8.4.1 
for f in *fastq-trimmed.fastq; do FILE=`basename ${f%%.*}`; echo ${FILE}; bwa mem -t 6 solenopsis_genome_index/UNIL_Sinv_3.0.fasta ${FILE}.fastq-trimmed.fastq  | samtools view -bh | samtools sort -@ 3 -m 4G -o ${FILE}_sorted.bam; done
for FILE in *bam*; do samtools flagstat $FILE > $FILE.stat.txt; done
```

These commands produced sorted bam files as outputs. A bam file describes all the variation of an alignment between query and reference sequences, but is compressed and is not human-readable. It’s generally used in downstream analyses since it saves space compared to the human-readable but large sam files that are used to produce it. Use the cat command to view the stats for our files.

```
cat SRR6922141_1_sorted_bam.stat.txt
cat SRR6922185_1_sorted_bam.stat.txt
cat SRR6922187_1_sorted_bam.stat.txt
cat SRR6922236_1_sorted_bam.stat.txt
```

Looking at the stats file for SRR6922141_1, it contains 1,051,910 reads that were mapped while 3,160 were supplementary. For SRR6922236_1, 922,747 reads were mapped and 1,446 were supplementary. For SRR6922185_1, 1,225,386 reads were mapped and 1,829 were supplementary. For SRR6922187_1, 986,550 reads were mapped and 2,867 were supplementary.

Now  add read groups to the bam files using samtools and then index them -- these read groups are the set of reads that come from each run of a sequencing instrument.

```
spack load openjdk@11.0.8_10%gcc@8.4.1
spack load samtools@1.9
for f in *_sorted.bam; do FILE=`basename ${f%%_*}`; echo ${FILE}_1; java -jar /pickett_shared/software/picard-2.27.4/picard.jar AddOrReplaceReadGroups I=${FILE}_1_sorted.bam O=${FILE}_1_sorted.RG.bam RGSM=$FILE RGLB=$FILE RGPL=illumina RGPU=$FILE ; samtools index ${FILE}_1_sorted.RG.bam; done
```

The read groups have now been added to the bam files!

# Step 4: GATK

Now that read groups have been added to our bam files,  go ahead and run GATK on them. GATK (version 4.2.6.1) calls the variation between the reference genome and the four samples by identifying SNPs and indels. Make a directory for this step and add the files needed for our analysis. GATK will use these files to call the variants for analysis.

```
mkdir ../4_gatk
for FILE in *RG*; do ln -s /pickett_shared/teaching/EPP622_Fall2022/analysis_test2/jturne88/3_bwa/$FILE /pickett_shared/teaching/EPP622_Fall2022/analysis_test2/jturne88/4_gatk/$FILE; done
cd ../4_gatk
ln -s /pickett_shared/teaching/EPP622_Fall2022/raw_data/solenopsis_invicta/genome/UNIL_Sinv_3.0.fasta
```

Since the GATK directory has been created and all its requisite files have been added,  go ahead and index our reference genome so GATK can use it.

```
/pickett_shared/software/gatk-4.2.6.1/gatk CreateSequenceDictionary -R UNIL_Sinv_3.0.fasta
samtools faidx UNIL_Sinv_3.0.fasta
```

Since running GATK could take a while, I’m choosing to run it in screen. Then, use samtools to run GATK HaplotypeCaller for all of our read group-sorted files using the following script. HaplotypeCaller is software within GATK that calls variant sites and stores them as gvcf files. 

```
screen -s run_GATK
spack load openjdk@11.0.8_10%gcc@8.4.1
spack load samtools@1.9
for f in *_sorted.RG.bam
do
        BASE=$( basename $f | sed 's/-trimmed_sorted.RG.bam*//g' )
        echo "BASE $BASE"
        /pickett_shared/software/gatk-4.2.6.1/gatk HaplotypeCaller \
        -R UNIL_Sinv_3.0.fasta \
        -I $f \
        -O ${BASE}.g.vcf \
        -ERC GVCF \
        -bamout ${BASE}_sorted.RG.realigned.bam
done
exit
```

This script generated gvcf files for each sequence which display all the SNPs and indels between our reference and query sequences. These are human readable (similar to vcf files but with records for all sites and not those just with variation) and are used to identify variation among these sequences. Now, determine the number of SNPs and indels of each sample using BCFtools version 1.10.2. 

```
for FILE in *.vcf; do bcftools stats $FILE > $FILE.stats.txt; done
head -n 40 SRR6922141_1_sorted.RG.bam.g.vcf.stats.txt
head -n 40 SRR6922185_1_sorted.RG.bam.g.vcf.stats.txt
head -n 40 SRR6922187_1_sorted.RG.bam.g.vcf.stats.txt
head -n 40 SRR6922236_1_sorted.RG.bam.g.vcf.stats.txt
```

This for loop generates stats files of each sample, where the number of SNPs and indels is displayed. The number of SNPs and indels are then viewed using the head command, where the first 40 lines of the sample stats files are displayed. There are 330375 SNPs and 4093 indels in SRR6922141_1, 20,326 SNPs and 3,408 indels in SRR6922185_1, 26,256 SNPs and 3,698 indels in SRR6922187_1, and 18,638 SNPs and 3,189 indels in SRR6922236_1.

Visualize some of this variaiton with IGV! IGV stands for Integrated Genomics Viewer – it can display different genome alignments together for easy comparison. First, download the requisite files to use IGV onto a personal computer. The reference genome, its index, a bam file, and an indexed bam file are necessary for this step. I’ll be using SRR6922141_1 for this exercise. Now, download these files.

```
scp jturne88@sphinx.ag.utk.edu:/pickett_shared/teaching/EPP622_Fall2022/analysis_test2/jturne88/4_gatk/UNIL_Sinv_3.0.fasta.fai ./
scp jturne88@sphinx.ag.utk.edu:/pickett_shared/teaching/EPP622_Fall2022/analysis_test2/jturne88/4_gatk/UNIL_Sinv_3.0.fasta ./
scp jturne88@sphinx.ag.utk.edu:/pickett_shared/teaching/EPP622_Fall2022/analysis_test2/jturne88/4_gatk/SRR6922141_1_sorted.RG.bam.bai ./
scp jturne88@sphinx.ag.utk.edu:/pickett_shared/teaching/EPP622_Fall2022/analysis_test2/jturne88/4_gatk/SRR6922141_1_sorted.RG.bam ./
```

Once these files have been downloaded, use the gvcf file to select which variant that needs to be called. After some scrolling, I found this line in the corresponding gvcf file representing a SNP. A vcf file could be used instead of a gvcf file to examine only positions in the genome with variation. 

```
NC_052664.1     1385298 .       C       A,<NON_REF>     117.83  .       DP=3;ExcessHet=0.0000;MLEAC=1,0;MLEAF=0.500,0.00;RAW_MQandDP=10800,3    GT:AD:DP:GQ:PL:SB       1/1:0,3,0:3:9:131,9,0,131,9,131:0,0,0,3
```

From this line, its position (NC_052664.1:1385298) in the reference represents an adenine where there should be a cytosine. Now, navigate using a web browser to ivg.gov/app and user its browser application to view our variant. 

![image](https://user-images.githubusercontent.com/80480626/195210732-e04dacbe-5847-4ac4-951d-34a5d73afb5e.png)

As we can see, there’s a SNP in this section of the genome that the vcf file claimed there was. This evidence supports the genotypes called in the VCF file, so they contain at least some information that’s grounded in reality! However, it's good practice to examine multiple SNPs and indels in your data with IGV to double check its validity. 

# Step 5: GATK Combine

Now that these variants with GATK have been called, combine these SNPs across all our samples for analysis. First, create a directory for this space.

```
mkdir ../5_gatk_combine
cd ../5_gatk_combine/
```

Add the necessary genome fasta, fasta index, and dict file to this directory so we can combine our variants across samples. The vcf files we previously generated will also be added to this directory.

```
ln -s /pickett_shared/teaching/EPP622_Fall2022/raw_data/solenopsis_invicta/genome/UNIL_Sinv_3.0.fasta
ln -s /pickett_shared/teaching/EPP622_Fall2022/analysis_test2/jturne88/4_gatk/UNIL_Sinv_3.0.fasta.fai
ln -s /pickett_shared/teaching/EPP622_Fall2022/analysis_test2/jturne88/4_gatk/UNIL_Sinv_3.0.dict
ln -s /pickett_shared/teaching/EPP622_Fall2022/analysis_test2/jturne88/4_gatk/*g.vcf .
```

Now, create a shell script to run so that we can combine the variants using the CombineGVCFs feature of GATK.

```
echo "/pickett_shared/software/gatk-4.2.6.1/gatk CombineGVCFs \\
-R UNIL_Sinv_3.0.fasta \\" >> combine_SNPS.sh
for f in *g.vcf ; do echo "-variant $f \\" >> combine_SNPS.sh; done
echo "-O solenopsis_combined.g.vcf.gz" >> combine_SNPS.sh
```

Now that the shell script is created, go ahead and run it to combine our variants.

```
sh combine_SNPS.sh
```

This script combined all the SNPs and indels of each of its constituent files into one. Now that it merged all our variants into one file, recall them to obtain the vcf file.

```
/pickett_shared/software/gatk-4.2.6.1/gatk --java-options "-Xmx10g" GenotypeGVCFs \
   -R UNIL_Sinv_3.0.fasta  \
   -V solenopsis_combined.g.vcf.gz \
   -O solenopsis_combined.vcf.gz
```

Then, view the number of SNPs found using BCFtools.

```
spack load bcftools
bcftools stats solenopsis_combined.vcf.gz > solenopsis_combined.vcf.stats.txt
head -n 40 solenopsis_combined.vcf.stats.txt
```

We have 52,557 SNPs across these four samples. Now, to filter our SNPs to capture only high quality ones. First,  decompress the combined vcf file before applying our filters.

```
gunzip solenopsis_combined.vcf.gz
```

Now, filter our file using GATK’s selected parameters.

```
/pickett_shared/software/gatk-4.2.6.1/gatk VariantFiltration \
        -R UNIL_Sinv_3.0.fasta  \
        -V solenopsis_combined.vcf \
        -O solenopsis_combined.GATKfilters.vcf \
        -filter-name "QD_filter" -filter "QD < 2.0" \
        -filter-name "FS_filter" -filter "FS > 60.0" \
        -filter-name "MQ_filter" -filter "MQ < 40.0" \
        -filter-name "SOR_filter" -filter "SOR > 4.0" \
        -filter-name "MQRankSum" -filter "MQRankSum < -12.5" \
        -filter-name "ReadPosRankSum" -filter "ReadPosRankSum < -8.0"
```

The results are filtered. Yay! Now, compare them to the unfiltered vcf file.

```
bcftools stats -f "PASS,." solenopsis_combined.vcf >solenopsis_combined.vcf.stats.txt
bcftools stats -f "PASS,." solenopsis_combined.GATKfilters.vcf > solenopsis_combined.GATKfilters.vcf.stats.txt
grep 'number of SNPs:' *stats.txt
```

There were 52,557 SNPs before filtering and 36,851 after.

# Answers to selected questions on the assignment prompt:

a.) SRR6922141_1:
![image](https://user-images.githubusercontent.com/80480626/195209247-16376f67-a2b6-48da-9cfc-0654c047b203.png)

b.) Before trimming, SRR6922141_1 had 1,148,398 reads -- after trimming, SRR6922141_1 had 1,068,311 reads. Before trimming, SRR6922185_1 had 1,315,096 reads before trimming and 1,254,903 reads after. Before trimming, SRR6922187_1 had 1,081,747 reads before trimming and 993,372 reads after. Before trimming, SRR6922236_1 had 977,855 reads before trimming and 928,913 reads after.

c.) Looking at the stats file for SRR6922141_1, it contains 1,051,910 reads that were mapped while 3,160 were supplementary. For SRR6922236_1, 922,747 reads were mapped and 1,446 were supplementary. For SRR6922185_1, 1,225,386 reads were mapped and 1,829 were supplementary. For SRR6922187_1, 986,550 reads were mapped and 2,867 were supplementary.

d.) The number of SNPs and indels are then viewed using the head command, where the first 40 lines of the sample stats files are displayed. There are 330375 SNPs and 4093 indels in SRR6922141_1, 20326 SNPs and 3408 indels in SRR6922185_1, 26,256 SNPs and 3,698 indels in SRR6922187_1, and 18,638 SNPs and 3,189 indels in SRR6922236_1.

e.) ![image](https://user-images.githubusercontent.com/80480626/195210732-e04dacbe-5847-4ac4-951d-34a5d73afb5e.png)

As we can see, there’s a SNP in this section of the genome that the vcf file claimed there was. This evidence supports the genotypes called in the VCF file, so they contain at least some information that’s grounded in reality!

extra credit: There were 52,557 SNPs before filtering and 36,851 after.
