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
* If you have a highly complex or fragmented metagenome, I suggest lowering the minimum contig size 
```
EukRep -i metagenome.fa -o euk_contigs.fa --min 1000
```

### Automated binning
* This step is important for separating multiple eukaryotic genomes in your sample.
* It is important to have genomes separated before gene prediction in order to obtain as high quality of gene predictions as possible. 
* Requires per-scaffold coverage information

Performed with CONCOCT:
```
concoct --coverage_file euk_contig_cov.txt --composition_file euk_contigs.fa
mkdir clusters
python /path/to/CONCOCT/scripts/extract_fasta_bins.py --output_path ./clusters/ euk_contigs.fa clustering_gt1000.csv
```
Performed with metabat:
```
metabat -a euk_contig_cov.txt -i euk_contigs.fa -o bin -t 6
```

### Filtering by bin size
* We find it useful to filter out any bins smaller than 2.5 Mbp at this stage. This filtering removes the majority of false positives. Especially useful if CONCOCT was used because CONCOCT will bin every scaffold, often generating many very small bins.

### Train GeneMark-ES
```
perl gmes_petap.pl --ES -min_contig 10000 --sequence bin_1.fa
```
* the -min_contig option specifies the minimum length of a contig used to train the gene prediction model for your bin. You do not need every contig of your bin used, however training may fail if too little of your contigs are above the threshold. Many bins from metagenomes can be quite fragmented so this option may need to be adjusted.

### Predict genes using the trained GeneMark-ES model and MAKER2
* MAKER uses control files. At a minimum, I suggest modifying them in the following way to use RepeatMasker and GeneMark-ES to predict genes:
```
Within the 'maker_opts.ctl' file:
keep_preds=1
gmhmm=/path/to/output/gmhmm.mod
```
* To then run MAKER with 6 cores, use the following commands:
```
maker -g bin_1.fa -c 6
cd *.maker.output
fasta_merge -d *_master_datastore_index.log -o bin_1
```
* To improve your gene predictions further, MAKER is capable of incorporating homologous proteins from reference genomes of related organisms, transcriptomic evidence, and other ab initio gene predictors such as AUGUSTUS. For obtaining high quality gene predictions, it is usually best to use as many of these lines of evidence as are available.
* For many metagenomic samples, performing ab initio gene predictions may be the only available option.  


### Run BUSCO 
```
python3 BUSCO.py -i *.maker.proteins.fasta -l eukaryota_odb9 -o bin_1 -m prot
```
* BUSCO will look for single copy orthologous genes (SCGs) within your bin, giving an estimate of completeness (and a rough estimation of contamination with duplicate single copy genes). 
* -l specifies the lineage set of SCGs to use. We generally use eukaryota_odb9 as it is the most general, however if and when you get a better idea of what type fo organisms your bin belongs to, it is possible to use a more specific lineage set.
