# EukRep_Pipeline

An example workflow for binning eukaryotic genomes from a metagenome incorporating EukRep. 
Included is an example bash script, euk_pipeline.sh, that incorporates all of the steps below.

## Requirements:
* Pre-assembled shotgun metagenomic sample with per-scaffold coverage information
* EukRep
* CONCOCT or metabat
* genemark-ES
* MAKER2
* BUSCO

## Optional (but recommended):
* pyenv

-----

### Classify with EukRep
* Run EukRep on the pre-assembled shotgun metagenomic sample
```
EukRep -i metagenome.fa -o euk_contigs.fa 
```

### Automated binning
* Perform automated binning on the predicted eukaryotic contigs. This step is important for cases where there are multiple eukaryotic genomes in your sample and to help remove remaining prokaryotic scaffolds from any eukaryotic bin
* This step requires per-scaffold coverage information
Performed with CONCOCT:
```
concoct --coverage_file euk_contig_cov.txt --composition_file euk_contigs.fa
```
Performed with metabat:
```
metabat -a euk_contig_cov.txt -i euk_contigs.fa -o bin -t 6
```

### Filtering by bin size

### Train GeneMark-ES

### Run MAKER2 gene prediction

### Run BUSCO 
