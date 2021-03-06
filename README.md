# GEO RNA-seq and microarray meta-analysis pipeline

Pipeline for processing and analysis of gene expression data. Currently consists of fallowing modules:
1) Downloading Series Matrices from GEO and gathering metadata to tsv tables. 
2) RNA-seq quantification with Kallisto(46.1). Currently:
    - RNA-seq samples filtered by metadata. GSE with 2-400 passing metadata filter samples are taken for quantification.
    - 100 rounds of bootstrap. Necessary for future downstream. 
3) Microarray preprocessing. 
4) WGCNA 
5) PCA
6) Ranked-list of genes statistical testing(GSEA, KS, wilcoxGST)


### NOTES:
 - On h5 files: 
 
    Most of RNA seq run on different 44.x 45.x 46.x versions with 0 bootsrap and
without SRRxxx.h5 file in folder because it doesn't make any sense to keep it having tsv and json output and not having
bootstrap in a run. Has to be replaced later for doing proper depressions with Sleuth. Simply remove all folders 
without h5 files. Little bit more for Rat as those files with 0 botstrap has to be identified based on info in json
files and then removed. Remember to "chmod +X" the directory as it's blocked by snakemake with temp() to prevent 
accidental removing. 
Version and number of bootstrap rounds doesn't affect anything. According to change log alignment algorithm hasn't 
been changed between this 45-46 versions. Bootstrap is used to estimate instrumental variance. So, basically 
this means that tsv output matrix is same for 44-46 versions with any number of bootstrap rounds. 
some comments on bootstrap: https://www.biostars.org/p/155032/
 - To speedup graph calculation comment all downstream rules of one you need and delete input section in first rule. 
 this would allow Snakemake to skip graph calculation for existing files. 

# Current input files:
1) GGS searches 04.12.2019

Pipeline quantifies SRR files and aggregates them first to GSM and then to GSM matrices.

### 1) Run docker container:
- Better run in tmux as interactive session:
```bash
bsub -Is -q docker-interactive  -a 'docker(biolabs/snakemake)' /bin/bash
```
### 2) Git clone snakemake repo to home directory:
```bash
git clone https://github.com/shpakb/gq-rnaseq-pipeline.git
```

### 3) Create input dir with:
### - Symlinks for reference genomes.
### - Search results from GEO to parse 
### see the config file for rest 

Example how to create symlink:
```bash
ln -s /gscmnt/gc2676/martyomov_lab/shpakb/Assemblies/rnor_v6/ index 
```

### 6) Create and activate conda env(when loacal) 
```bash 
conda env create --file ./envs/quantify.yaml --name snakemake && \
    source activate snakemake
```

Dry run(graph eval):
```bash
snakemake --dryrun
```

### 6) Run pipeline in test mode on cluster(when on cluster): 
```bash
snakemake --use-conda --profile lsf --jobs 50\
 --jobscript lsf_jobscript.sh\
 --resources download_res=4 writing_res=20
```

Add when script is more or less stable:
add: --restart-times 3 
remove: --notemp 
--verbose
 -pr- gives command arguments overview

 --keep-going - Might be useful but be cautious, downloading if pipeline stack on fastq-dump we will quickly run out of 
 disk space 

- resources parameter specifies amount of resources pipeline can use. In this particular case load 100 and 
each downloading job uses 50 "points" of load. So they can't be more than two downloading jobs simultaneously. 

- now pipeline outputs all the files right in to directory with scripts. No need to go through all steps with symlinks 
and copying. 

For test run:
snakemake -pr --notemp --use-conda --verbose

For large DAGs 
--batch myrule=1/3 

Omits jobs downstream of this target 
--omit-from srr_to_gsm 

For multiple run can try:
--cache RULE

Run test snakemake:
snakemake --snakefile=test.smk