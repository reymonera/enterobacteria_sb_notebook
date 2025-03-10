# Genomic surveillance of multidrug-resistant E. coli and Klebsiella in clinical and wastewater isolates from a pediatric hospital in Lima, Peru
This is a brief report on the commands used on the work [featured here](https://www.microbiologyresearch.org/content/journal/acmi/10.1099/acmi.0.001006.v1). This is the first version of this report, and any questions regarding the code should be made to the author of this repository/corresponding author.

## Assembly
Assembly was done using SPAdEs in its usual confirguration, executing it like this:
```bash
mlst *.fasta > results
```
## Quast
`Quast` was used for checking the quality of our assemblies. After performing Quast, we obtained 113 *E. coli* and 44 *Klebsiella* spp. genomes. 

## MLST
Identification was performed using `MLST` v. 2.22.0 executed from `conda` with the following version:
```
active environment : mlst
    active env location : /home/marlen/anaconda3/envs/mlst
            shell level : 2
       user config file : /home/marlen/.condarc
 populated config files : /home/marlen/.condarc
          conda version : 4.12.0
    conda-build version : 3.21.5
         python version : 3.9.7.final.0
       virtual packages : __linux=5.13.0=0
                          __glibc=2.31=0
                          __unix=0=0
                          __archspec=1=x86_64
       base environment : /home/marlen/anaconda3  (writable)
      conda av data dir : /home/marlen/anaconda3/etc/conda
```
`MLST` was executed with the following command:
```bash
mlst *.fasta > results
```
`results` contains all the identifications, and it is the main table regarding the identity of the isolates.

## fastANI
`fastANI` was also performed using the following command and using *E. coli* ASM584 as a reference:
```bash
fastANI --ql list_ecoli.txt -r ref_ecoli_ASM584v2_genomic.fna -o 
``` 
## Abricate
`Abricate` was used to obtain genes of interested in the obtained genomes. It was used with the following command:
```bash
abricate --db ncbi *.fasta > res_ecoli_ncbi.tab
```
The following database was used:
```bash
Using nucl database ncbi:  5386 sequences -  2021-Mar-27
```
## Kleborate
`Kleborate` was used to identify *Klebsiella* genomes with more precision, using the following commands:
```bash
ver: Kleborate v2.2.0
kleborate -o results_kleb.txt -a *.fasta
kleborate -o results_koxytoca.txt -a *.fasta
```
## SNPs based Phylogeny
We performed a SNP based phylogeny using CGI Phylogenomics, as a first version of our phylogeny. No parameters were changed, besides using the recommended use of FastTree.

## Core-genome based Phylogeny
We performed a core-genome based phylogeny first annotating our genomes using `Prokka`, with the following version:
```bash
$ prokka --version
prokka 1.14.6
```
The following slurm job was used to perform work with `Prokka`:
```bash
#!/bin/sh

module load miniconda
conda activate prokka

for k in *.fasta; do prokka $k --outdir "$k".prokka.output --prefix PROKKA_$k; echo $k; done

mkdir roary
find /home/ccastillo/sb_genomes/koxytoca -name '*.gff' -exec cp -t /home/ccastillo/sb_genomes/koxytoca/roary {} +

echo "All Done!"
```
With the obtained core-gene alignment, we used `RAxML` with the following version:
```bash
This is RAxML version 8.2.12 released by Alexandros Stamatakis on May 2018.
```
`RAxML` was applied using the following batch job:
```bash
#!/bin/sh
--ntasks=16 
--ntasks-per-node=2

module load miniconda
conda activate roary

roary -p 8 -i 90 -v -n -e -f roary_90 *.gff
roary -p 8 -v -n -e -f roary_95 *.gff

cd roary_90

conda deactivate
conda activate raxml

raxmlHPC-PTHREADS -f a -m GTRGAMMA -p 12345 -x 12345 -# 100 -s core_gene_alignment.aln -T 2 -n raxml_filog_ecoli_90

cd ..
cd roary_95
raxmlHPC-PTHREADS -f a -m GTRGAMMA -p 12345 -x 12345 -# 100 -s core_gene_alignment.aln -T 2 -n raxml_filog_ecoli_95

echo "Ya terminé!"
```