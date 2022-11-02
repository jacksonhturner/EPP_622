The objective of this assignment is to compare two assemblers, Flye and SmartDeNovo, using a set of nanopore dogwood chromosome 4 reads as an input. Different assemblers work better for different data, so it's important to review them before performing high-throughput assemblies. 

Set up your data.

LongQC.

Install flye using conda.
```
conda create -n flye
conda activate flye
conda install -c bioconda flye
```

Run flye. I have version 2.8 so it automatically scaffolds.
```
flye --nano-raw microcitrus_australasica_nanopore.4.fastq --out-dir flye_assembly --threads 20
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
