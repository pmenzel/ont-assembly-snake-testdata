
## Test dataset for ont-assembly-snake and score-assemblies workflows

This repository contains sequencing data from ONT and Illumina for testing the [ont-assembly-snake](https://github.com/pmenzel/ont-assembly-snake) and [score-assemblies](https://github.com/pmenzel/score-assemblies) Snakemake workflows,
described in the preprint [Snakemake Workflows for Long-read Bacterial Genome Assembly and Evaluation](https://www.preprints.org/manuscript/202208.0191/v1).

The datasets are subsampled read sets from the ONT and Illumina sequencing data of NCBI BioSample [SAMN30015177](https://www.ncbi.nlm.nih.gov/sra?LinkName=biosample_sra&from_uid=30015177) of _Pandoraea commovens_,
from the publication [Outbreak of Pandoraea commovens Infections among Non–Cystic Fibrosis Intensive Care Patients, Germany, 2019–2021](https://wwwnc.cdc.gov/eid/article/29/11/23-0493_article).

### Setup

> [!IMPORTANT]  
> A working conda installation is required to run the workflows, see the [installation instructions for Miniconda](https://docs.conda.io/projects/miniconda/en/latest/index.html#quick-command-line-install).

To download the dataset, clone this repository with:
```
git clone https://github.com/pmenzel/ont-assembly-snake-testdata.git
cd ont-assembly-snake-testdata
```

It will contain two folders: `fastq-ont` and `fastq-illumina` with following files:
```
├── fastq-illumina
│   ├── example_R1.fastq.gz
│   └── example_R2.fastq.gz
└── fastq-ont
    └── example.fastq.gz
```

Next, clone the [ont-assembly-snake](https://github.com/pmenzel/ont-assembly-snake) and [score-assemblies](https://github.com/pmenzel/score-assemblies) repositories and create the respective conda-environments:

```
conda config --add channels bioconda

git clone https://github.com/pmenzel/ont-assembly-snake.git
conda env create -n ont-assembly-snake --file ont-assembly-snake/env/conda-main.yaml

git clone https://github.com/pmenzel/score-assemblies.git
conda env create -n score-assemblies --file score-assemblies/env/environment.yaml
```

### Run pipelines

The commands below will first download a reference genome assembly for _P. commovens_ and then run the ont-assembly-snake and score-assemblies workflows.
The assemblies that are to be generated are defined in the file `samples.yaml`.

```
# download assembly GCF_902459615.1 used as a reference for polishing and comparison
# using the NCBI datasets tool
mkdir -p references references-protein
wget https://ftp.ncbi.nlm.nih.gov/pub/datasets/command-line/LATEST/linux-amd64/datasets
chmod +x datasets
./datasets download genome accession GCF_902459615.1 --include genome
unzip -p -j ncbi_dataset.zip 'ncbi_dataset/data/GCF_902459615.1/GCF_902459615.1_LMG_31010_genomic.fna' > references/GCF_902459615.1.fa
unzip -p -j ncbi_dataset.zip 'ncbi_dataset/data/GCF_902459615.1/protein.faa' > references-protein/GCF_902459615.1.faa
rm datasets ncbi_dataset.zip

# run ont-assembly-snake
conda activate ont-assembly-snake
snakemake -s ont-assembly-snake/Snakefile --use-conda --cores 10 --configfile samples.yaml --config genome_size=5.9 --config medaka_model=r941_min_sup_g507

# run score-assemblies
conda activate score-assemblies
snakemake -s score-assemblies/Snakefile --use-conda --cores 20

```

The output files of `score-assemblies` are in the folder `score-assemblies-data`
and a summary HTML report is in `score-assemblies-report.html`.

