The objective of this assignment is to compare two assemblers, Flye and SOMETHING, using a set of nanopore dogwood chromosome 4 reads as an input. Different assemblers work better for different data, so it's important to review them before performing high-throughput assemblies. 

Set up your data.

LongQC.

Install flye using conda.
'''
conda create -n flye
conda activate flye
conda install -c bioconda flye
'''

Run flye.
'''
flye --nano-raw microcitrus_australasica_nanopore.4.fastq --out-dir flye_assembly --threads 20
'''

Run BUSCO.

Run SOMETHING.

Run BUSCO.

Compare them.
