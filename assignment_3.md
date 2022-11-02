The objective of this assignment is to compare two assemblers, Flye and SmartDeNovo, using a set of nanopore dogwood chromosome 4 reads as an input. Different assemblers work better for different data, so it's important to review them before performing high-throughput assemblies. 

Step 1: Loading in Data (and LongQC if it works)

First, create a directory to deposit the raw data.
```
mkdir 1_raw_data
cd 1_raw_data
```

Load in your data for this assignment by creating a soft link to the nanopore dogwood chromosome 4 reads. The path for these reads exists within the pickett_shared directory within this class's raw data. 

```
ln -s /pickett_shared/teaching/EPP622_Fall2022/raw_data/citrus_test2/microcitrus_australasica_nanopore.4.fastq .

```

LongQC goes here. At the moment it doesn't work for nanopore reads, but if you want to QC PacBio or any other read type you can!

Step 2: Perform Assembly With Flye

Create a new directory and navigate to it.
```
cd ..
mkdir 2_flye_assembly
cd 2_flye_assembly
```

This step utilizes Flye, a long-read assembly algorithm that builds repeat graphs using arbitrary, unknown repeat graphs [1]. Because this assembly algorithm is not native to the server, it will need to be installed. Since flye can be found in bioconda, it will be easiest to install using conda.

Install flye by creating a new conda environment and using bioconda to install the Flye package.
```
conda create -n flye
conda activate flye
conda install -c bioconda flye
```

Now that Flye is installed, double check the version number. 

```
conda list flye
```

Due to some quirk of my personal miniconda3, the following commands installed Flye version 2.8.1.

Next, perform the assembly using Flye, referencing its github (https://github.com/fenderglass/Flye) for whatever specifications needed for the data. Because I've installed Flye version 2.8.1, the assembler automatically scaffolds the assembly without a flag. This command lists using 10 threads, but use however many is appropriate for your situation. This will take a few minutes -- 

```
flye --nano-raw microcitrus_australasica_nanopore.4.fastq --out-dir flye_assembly --threads 10
```

Find sequencing statistics with bbmap.

```
Main genome scaffold total:             639
Main genome contig total:               644
Main genome scaffold sequence total:    35.366 MB
Main genome contig sequence total:      35.365 MB       0.001% gap
Main genome scaffold N/L50:             24/271.267 KB
Main genome contig N/L50:               26/270.297 KB
Main genome scaffold N/L90:             201/23.652 KB
Main genome contig N/L90:               206/23.615 KB
Max scaffold length:                    3.337 MB
Max contig length:                      3.337 MB
Number of scaffolds > 50 KB:            129
% main genome in scaffolds > 50 KB:     83.10%
```

Run BUSCO.


        --------------------------------------------------
        |Results from dataset embryophyta_odb10           |
        --------------------------------------------------
        |C:10.0%[S:9.3%,D:0.7%],F:1.4%,M:88.6%,n:1614     |
        |161    Complete BUSCOs (C)                       |
        |150    Complete and single-copy BUSCOs (S)       |
        |11     Complete and duplicated BUSCOs (D)        |
        |23     Fragmented BUSCOs (F)                     |
        |1430   Missing BUSCOs (M)                        |
        |1614   Total BUSCO groups searched               |
        --------------------------------------------------

LongQC.

Run SmartDeNovo.

Find sequencing statistics with bbmap.

```
Main genome scaffold total:             149
Main genome contig total:               149
Main genome scaffold sequence total:    27.007 MB
Main genome contig sequence total:      27.007 MB       0.000% gap
Main genome scaffold N/L50:             10/461.296 KB
Main genome contig N/L50:               10/461.296 KB
Main genome scaffold N/L90:             78/59.325 KB
Main genome contig N/L90:               78/59.325 KB
Max scaffold length:                    2.459 MB
Max contig length:                      2.459 MB
Number of scaffolds > 50 KB:            96
% main genome in scaffolds > 50 KB:     93.62%
```

Run BUSCO.

Compare them.


        --------------------------------------------------
        |Results from dataset embryophyta_odb10           |
        --------------------------------------------------
        |C:9.5%[S:9.3%,D:0.2%],F:1.0%,M:89.5%,n:1614      |
        |154    Complete BUSCOs (C)                       |
        |150    Complete and single-copy BUSCOs (S)       |
        |4      Complete and duplicated BUSCOs (D)        |
        |16     Fragmented BUSCOs (F)                     |
        |1444   Missing BUSCOs (M)                        |
        |1614   Total BUSCO groups searched               |
        --------------------------------------------------
