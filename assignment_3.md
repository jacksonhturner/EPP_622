The objective of this assignment is to compare two assemblers, Flye and SmartDeNovo, using a set of nanopore dogwood chromosome 4 reads as an input. Different assemblers work better for different data, so it's important to review them before performing high-throughput assemblies. 

Set up your data.

LongQC.

Install flye using conda.
'''
conda create -n flye
conda activate flye
conda install -c bioconda flye
'''

Run flye. I have version 2.8 so it automatically scaffolds.
'''
flye --nano-raw microcitrus_australasica_nanopore.4.fastq --out-dir flye_assembly --threads 20
'''

Find sequencing statistics. 

Run BUSCO.

LongQC.

Run SmartDeNovo.

Run BUSCO.

Compare them.

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
