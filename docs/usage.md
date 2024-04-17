# nf-core/evoverse: Usage

## Table of contents

* [Table of contents](#table-of-contents)
* [Introduction](#introduction)
* [Running the pipeline](#running-the-pipeline)
  * [Updating the pipeline](#updating-the-pipeline)
  * [Reproducibility](#reproducibility)
* [Main arguments](#main-arguments)
  * [`-profile`](#-profile)
  * [`--reads`](#--reads)
  * [`--single_end`](#--single_end)
* [Reference genomes](#reference-genomes)
  * [`--genome` (using iGenomes)](#--genome-using-igenomes)
  * [`--fasta`](#--fasta)
  * [`--igenomes_ignore`](#--igenomes_ignore)
* [Job resources](#job-resources)
  * [Automatic resubmission](#automatic-resubmission)
  * [Custom resource requests](#custom-resource-requests)
* [AWS Batch specific parameters](#aws-batch-specific-parameters)
  * [`--awsqueue`](#--awsqueue)
  * [`--awsregion`](#--awsregion)
  * [`--awscli`](#--awscli)
* [Other command line parameters](#other-command-line-parameters)
  * [`--outdir`](#--outdir)
  * [`--email`](#--email)
  * [`--email_on_fail`](#--email_on_fail)
  * [`--max_multiqc_email_size`](#--max_multiqc_email_size)
  * [`-name`](#-name)
  * [`-resume`](#-resume)
  * [`-c`](#-c)
  * [`--custom_config_version`](#--custom_config_version)
  * [`--custom_config_base`](#--custom_config_base)
  * [`--max_memory`](#--max_memory)
  * [`--max_time`](#--max_time)
  * [`--max_cpus`](#--max_cpus)
  * [`--plaintext_email`](#--plaintext_email)
  * [`--monochrome_logs`](#--monochrome_logs)
  * [`--multiqc_config`](#--multiqc_config)

# Introduction

**evoverse** is a workflow to infer a tumour evolution model from whole-genome sequencing (WGS) data. 

Through the analysis of variant and copy-number calls, it reconstructs the evolutionary process leading to the observed tumour genome. Most of the analyses can be done at mutliple levels: single sample, multiple samples from the same patient (multi-region/longitudinal assays), and multiple patients from distinct cohorts.

# Running the pipeline

## Quickstart

The typical command for running the pipeline is as follows:

```bash
nextflow run nf-core/evoverse \
 -r <VERSION> \
 -profile <PROFILE> \
 --samples <INPUT CSV> \
 --publish_dir ./results
 --tools <TOOLS>
```

`-r <VERSION>` is optional but strongly recommended for reproducibility and should match the latest version.

`-profile <PROFILE>` is mandatory and should reflect either your own institutional profile or any pipeline profile specified in the [profile section](##-profile).

This documentation imply that any `nextflow run nf-core/evoverse` command is run with the appropriate `-r` and `-profile` commands.

This will launch the pipeline and perform variant calling with the tools specified in `--tools`, see the [parameter section]([https://github.com/caravagnalab/nf-core-evoverse/tree/dev]) for details on the available tools.

Unless running with the `test` profile, the paths of input files must be provided within the `<INPUT CSV>` file specified in `--samples`, see the [input section]([https://github.com/caravagnalab/nf-core-evoverse/tree/dev]) for input requirements. 

Note that the pipeline will create the following files in your working directory:

```bash
work            # Directory containing the nextflow working files
results         # Finished results (configurable, see below)
.nextflow_log   # Log file from Nextflow
# Other nextflow hidden files, eg. history of pipeline runs and old logs.
```

## Input: Sample sheet configurations

You will need to create a samplesheet with information about the samples you would like to analyse before running the pipeline. Use the parameter `--samples` to specify its location. It has to be a comma-separated file with at least 5 columns, and a header row as shown in the examples below.

It is recommended to use the absolute path of the files, but a relative path should also work.

For the joint analysis of multiple samples, a tumor BAM file is required for each sample, such that the number of reads of a private mutation can be retrieved for all the samples thorugh `mpileup`.

Multiple samples from the same patient must be specified with the same `dataset` ID, `patient` ID, and a different `sample` ID.

Multiple patients from the same dataset must be specified with the same `dataset` ID, and a different `patient` ID.

**evoverse** will output sample-specific results in a different directory for _each sample_, patient-specific results in a common directory for _each patient_, and dataset-specific results in a common directory for _each dataset_.

Output from different workflows, subworkflows and modules will be in a specific directory for each dataset, patient, sample and tool configuration.

### Overview: Samplesheet Columns

| Column    | Description                                                                                                                                                                                                                                                                                                                       |
| --------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `dataset` | **Dataset ID**; when sequencing data from multiple datasets is analysed, it designates the source dataset of each patient; must be unique for each dataset, but one dataset can contain samples from multiple patients. <br /> _Required_                                                                                      |
| `patient` | **Patient ID**; designates the patient/subject; must be unique for each patient, but one patient can have multiple samples (e.g. from multiple regions or multiple time points). <br /> _Required_                                                                                                                                |
| `sample`  | **Sample ID** for each sample; more than one sample for each subject is possible. <br /> _Required_                                              |
| `vcf`  | Full path to the vcf file. <br /> _Required_                                                                                                        |
| `vcf_tbi`  | Full path to the vcf `tabix` index file. <br /> _Required_                                                                                      |
| `cna_dir`  | Full path to the directory containing text files from copy-number calling. <br /> _Required_                                                    |
| `tumour_bam`  | Full path to the tumour bam file. <br /> _Required for `--step subclonal_multisample`_                                                       |
| `tumour_bai`  | Full path to the tumour bam index file. <br /> _Required for `--step subclonal_multisample_ `                                                |

An [example samplesheet](https://github.com/caravagnalab/nf-core-evoverse/blob/dev/test_input.csv) has been provided with the pipeline.

## Pipeline steps

### Quality control

This step can be started either from XXX files or XXXX. The CSV must contain at least the columns XXX.

The following parameters can be tuned for this step:

- XXX
- XXX
  
The available tools for this step are XXX.

**NB: When running this step, XXXX***

#### Examples

Minimal input file:

```bash
patient,sample,lane,fastq_1,fastq_2
patient1,test_sample,lane_1,test_1.fastq.gz,test_2.fastq.gz
```

In this example, the sample comes from multiple patients:

```bash
patient,sample,lane,fastq_1,fastq_2
patient1,test_sample,lane_1,test_L001_1.fastq.gz,test_L001_2.fastq.gz
patient1,test_sample,lane_2,test_L002_1.fastq.gz,test_L002_2.fastq.gz
patient1,test_sample,lane_3,test_L003_1.fastq.gz,test_L003_2.fastq.gz
```

### Variant Annotation

#### VEP

VEP (Variant Effect Predictor) is a Ensembl tool that determines the effect of your variants (SNPs, insertions, deletions, CNVs or structural variants) on genes, transcripts, and protein sequence.

This step can be started either from `vcf`or `txt` files. Can use compressed input files (gzipped).
The CSV samplesheet must contain at least the columns:

```bash
patient,sample,vcf
```

VEP requires cache files to read genomic data (need to be available for proper tool functioning).
To use these, specify `--cache`. Params `--dir_cache` specify the cache directory to use, `--cache_version` specify to use a different cache version than the assumed default (the VEP version).
By default all params are specified in `params.config` file.

Example of the values used for these parameters:

```bash
ref_genome_vep =  "$HOME/ref_genomes/Homo_sapiens/GATK/GRCh38/Sequence/WholeGenomeFasta/Homo_sapiens_assembly38.fasta"
vep_dir_cache  =  "$HOME/ref_genomes/VEP/"
vep_cache_version = "110"
vep_species = "homo_sapience" 
assembly = "GRCh38"
```

Using VEP plugins:
`--plugin` - plugin modules should be installed in the Plugin subdirectory of the VEP cache directory (defaults to "$HOME/.vep/").
To enable specify `--plugin SingleLetterAA`.


Output format options:
`--vcf`
`--tab`
`--json`
`--compress_output [gzip|bgzip]`


#### vcf2maf

Convert a VCF file into a MAF (Mutation annotation Format), where each variant must be mapped to only one of all possible gene transcripts/isoforms that it might affect.
vcf2maf is designed to work with VEP. 

Main "vcf2maf" arguments:

`--inhibit-vep` - if you want to skip running VEP;
`--tumor-id` - to fill columns 16 and 17 of the output MAF with tumor/normal sample IDs, and to parse out genotypes and allele counts from matched genotype columns in the VCF;
`--ref-fasta [file|dir]`;
`--ncbi-build` - assembly version;
`--vep-data` - VEP cache version;
`--species`


**NB: While VEP is tolerant of chromosome format mismatches (when the input .vcf file uses the UCSC format chrN and the reference fasta uses Ensembl/NCBI format N), vcf2maf is not. Make sure the reference fasta chromosome format matches that of your input.**

#### Maftools

This tool attempts to summarize, analyze, annotate and visualize MAF files. A MAF file can be gz compressed. 

`read.maf` function reads MAF files, summarizes it in various ways and stores it as an MAF object.
MAF files are read for the single patient following merging multiple MAFs to create a multisample MAF object for proper genomic data summary and visualization.

#### Examples of samplesheet 

Minimal input file:

```bash
dataset,patient,sample,vcf
CLL,patient1,sample1,file_name.vcf.gz
```

In this example, the sample comes from multiple patients:

```bash
dataset,patient,sample,vcf
CLL,patient1,test_sample1,test_L001.vcf.gz
CLL,patient1,test_sample2,test_L002.vcf.gz
CLL,patient2,test_sample1,test_L003.vcf.gz
```

### Subclonal Deconvolution

This step can be started either from XXX files or XXXX. The CSV must contain at least the columns XXX.

The following parameters can be tuned for this step:

- XXX
- XXX
  
The available tools for this step are XXX.

**NB: When running this step, XXXX***

#### Examples

Minimal input file:

```bash
patient,sample,lane,fastq_1,fastq_2
patient1,test_sample,lane_1,test_1.fastq.gz,test_2.fastq.gz
```

In this example, the sample comes from multiple patients:

```bash
patient,sample,lane,fastq_1,fastq_2
patient1,test_sample,lane_1,test_L001_1.fastq.gz,test_L001_2.fastq.gz
patient1,test_sample,lane_2,test_L002_1.fastq.gz,test_L002_2.fastq.gz
patient1,test_sample,lane_3,test_L003_1.fastq.gz,test_L003_2.fastq.gz
```

### Clone Tree Inference

This step can be started either from XXX files or XXXX. The CSV must contain at least the columns XXX.

The following parameters can be tuned for this step:

- XXX
- XXX
  
The available tools for this step are XXX.

**NB: When running this step, XXXX***

#### Examples

Minimal input file:

```bash
patient,sample,lane,fastq_1,fastq_2
patient1,test_sample,lane_1,test_1.fastq.gz,test_2.fastq.gz
```

In this example, the sample comes from multiple patients:

```bash
patient,sample,lane,fastq_1,fastq_2
patient1,test_sample,lane_1,test_L001_1.fastq.gz,test_L001_2.fastq.gz
patient1,test_sample,lane_2,test_L002_1.fastq.gz,test_L002_2.fastq.gz
patient1,test_sample,lane_3,test_L003_1.fastq.gz,test_L003_2.fastq.gz
```

### Signature Deconvolution

This step can be started from `rds` multisample CNAqc object. The CSV must contain at least the columns:

```bash
dataset,joint_table
```

#### SparseSignatures

This tool provides a set of functions to extract and visualize the mutational signatures that best explain the mutation counts of a large number of patients. In particular:
- reliably extracts mutational signatures and quantifies their activity;
- incorporates an explicit background model to improve the inference;
- exploits LASSO regularization to reduce the impact of overfitting;
- implements bi-cross-validation to select the best number of signatures

The following parameters can be tuned for this step:

- `K` - the candidate numbers of signatures (min.value = 2) to be fit to the dataset;
- `lambda_values_beta` - the range of values of the signature sparsity parameter;
- `cross_validation_repetitions` - the number of repetitions of the cross-validation procedure.

#### SigProfiler

`SigProfilerExtractor` is a python framework that allows de novo extraction of mutational signatures from data generated in a matrix format. The tool identifies the number of operative mutational signatures, their activities in each sample, and the probability for each signature to cause a specific mutation type in a cancer sample. The tool makes use of `SigProfilerMatrixGenerator` and `SigProfilerPlotting`, seamlessly integrating with other `SigProfiler` tools.

The following parameters can be tuned for this step:

- `minimum_signatures` - the minimum number of signatures to be extracted (default = 1); 
- `maximum_signatures` - the maximum number of signatures to be extracted (default = 25). 
  
The available tools for this step are:
- SparseSignatures (Bioconductor R package)
- SigProfilerMatrixGenerator (Python framework)
- SigProfilerExtractor (Python framework)
- SigProfilerPlotting
- CNAqc (R package)

**NB: When running this step, the mutation's information of the dataset of interest is extracted from multisample mCNAqc object and converted to `txt` format in order to import the constructed data file in R or Python. Mutation frequency count data is generated from point mutation data-frames. The optimal signature number and sparsity is determined by cross-validation (SparseSignatures), followed by discovering the signatures within the dataset. SparseSignatures requires sizable datasets (i.e., at least hundreds of samples, depending on the complexity/number of mut. signatures present in the dataset) to converge to meaningful results. Further, it requires a grid search to choose the best values of K and lambda-beta.**


#### Examples

Minimal input file:

```bash
dataset,joint_table
CLL,mcnaqc_miltisample.rds
```


### Updating the pipeline

When you run the above command, Nextflow automatically pulls the pipeline code from GitHub and stores it as a cached version. When running the pipeline after this, it will always use the cached version if available - even if the pipeline has been updated since. To make sure that you're running the latest version of the pipeline, make sure that you regularly update the cached version of the pipeline:

```bash
nextflow pull nf-core/evoverse
```

### Reproducibility

It's a good idea to specify a pipeline version when running the pipeline on your data. This ensures that a specific version of the pipeline code and software are used when you run your pipeline. If you keep using the same tag, you'll be running the same version of the pipeline, even if there have been changes to the code since.

First, go to the [nf-core/evoverse releases page](https://github.com/nf-core/evoverse/releases) and find the latest version number - numeric only (eg. `1.3.1`). Then specify this when running the pipeline with `-r` (one hyphen) - eg. `-r 1.3.1`.

This version number will be logged in reports when you run the pipeline, so that you'll know what you used when you look back in the future.

## Main arguments

### `-profile`

Use this parameter to choose a configuration profile. Profiles can give configuration presets for different compute environments.

Several generic profiles are bundled with the pipeline which instruct the pipeline to use software packaged using different methods (Docker, Singularity, Conda) - see below.

> We highly recommend the use of Docker or Singularity containers for full pipeline reproducibility, however when this is not possible, Conda is also supported.

The pipeline also dynamically loads configurations from [https://github.com/nf-core/configs](https://github.com/nf-core/configs) when it runs, making multiple config profiles for various institutional clusters available at run time. For more information and to see if your system is available in these configs please see the [nf-core/configs documentation](https://github.com/nf-core/configs#documentation).

Note that multiple profiles can be loaded, for example: `-profile test,docker` - the order of arguments is important!
They are loaded in sequence, so later profiles can overwrite earlier profiles.

If `-profile` is not specified, the pipeline will run locally and expect all software to be installed and available on the `PATH`. This is _not_ recommended.

* `docker`
  * A generic configuration profile to be used with [Docker](http://docker.com/)
  * Pulls software from dockerhub: [`nfcore/evoverse`](http://hub.docker.com/r/nfcore/evoverse/)
* `singularity`
  * A generic configuration profile to be used with [Singularity](http://singularity.lbl.gov/)
  * Pulls software from DockerHub: [`nfcore/evoverse`](http://hub.docker.com/r/nfcore/evoverse/)
* `conda`
  * Please only use Conda as a last resort i.e. when it's not possible to run the pipeline with Docker or Singularity.
  * A generic configuration profile to be used with [Conda](https://conda.io/docs/)
  * Pulls most software from [Bioconda](https://bioconda.github.io/)
* `test`
  * A profile with a complete configuration for automated testing
  * Includes links to test data so needs no other parameters

<!-- TODO nf-core: Document required command line parameters -->

### `--reads`

Use this to specify the location of your input FastQ files. For example:

```bash
--reads 'path/to/data/sample_*_{1,2}.fastq'
```

Please note the following requirements:

1. The path must be enclosed in quotes
2. The path must have at least one `*` wildcard character
3. When using the pipeline with paired end data, the path must use `{1,2}` notation to specify read pairs.

If left unspecified, a default pattern is used: `data/*{1,2}.fastq.gz`

### `--single_end`

By default, the pipeline expects paired-end data. If you have single-end data, you need to specify `--single_end` on the command line when you launch the pipeline. A normal glob pattern, enclosed in quotation marks, can then be used for `--reads`. For example:

```bash
--single_end --reads '*.fastq'
```

It is not possible to run a mixture of single-end and paired-end files in one run.

## Reference genomes

The pipeline config files come bundled with paths to the illumina iGenomes reference index files. If running with docker or AWS, the configuration is set up to use the [AWS-iGenomes](https://ewels.github.io/AWS-iGenomes/) resource.

### `--genome` (using iGenomes)

There are 31 different species supported in the iGenomes references. To run the pipeline, you must specify which to use with the `--genome` flag.

You can find the keys to specify the genomes in the [iGenomes config file](../conf/igenomes.config). Common genomes that are supported are:

* Human
  * `--genome GRCh37`
* Mouse
  * `--genome GRCm38`
* _Drosophila_
  * `--genome BDGP6`
* _S. cerevisiae_
  * `--genome 'R64-1-1'`

> There are numerous others - check the config file for more.

Note that you can use the same configuration setup to save sets of reference files for your own use, even if they are not part of the iGenomes resource. See the [Nextflow documentation](https://www.nextflow.io/docs/latest/config.html) for instructions on where to save such a file.

The syntax for this reference configuration is as follows:

<!-- TODO nf-core: Update reference genome example according to what is needed -->

```nextflow
params {
  genomes {
    'GRCh37' {
      fasta   = '<path to the genome fasta file>' // Used if no star index given
    }
    // Any number of additional genomes, key is used with --genome
  }
}
```

<!-- TODO nf-core: Describe reference path flags -->

### `--fasta`

If you prefer, you can specify the full path to your reference genome when you run the pipeline:

```bash
--fasta '[path to Fasta reference]'
```

### `--igenomes_ignore`

Do not load `igenomes.config` when running the pipeline. You may choose this option if you observe clashes between custom parameters and those supplied in `igenomes.config`.

## Job resources

### Automatic resubmission

Each step in the pipeline has a default set of requirements for number of CPUs, memory and time. For most of the steps in the pipeline, if the job exits with an error code of `143` (exceeded requested resources) it will automatically resubmit with higher requests (2 x original, then 3 x original). If it still fails after three times then the pipeline is stopped.

### Custom resource requests

Wherever process-specific requirements are set in the pipeline, the default value can be changed by creating a custom config file. See the files hosted at [`nf-core/configs`](https://github.com/nf-core/configs/tree/master/conf) for examples.

If you are likely to be running `nf-core` pipelines regularly it may be a good idea to request that your custom config file is uploaded to the `nf-core/configs` git repository. Before you do this please can you test that the config file works with your pipeline of choice using the `-c` parameter (see definition below). You can then create a pull request to the `nf-core/configs` repository with the addition of your config file, associated documentation file (see examples in [`nf-core/configs/docs`](https://github.com/nf-core/configs/tree/master/docs)), and amending [`nfcore_custom.config`](https://github.com/nf-core/configs/blob/master/nfcore_custom.config) to include your custom profile.

If you have any questions or issues please send us a message on [Slack](https://nf-co.re/join/slack).

## AWS Batch specific parameters

Running the pipeline on AWS Batch requires a couple of specific parameters to be set according to your AWS Batch configuration. Please use [`-profile awsbatch`](https://github.com/nf-core/configs/blob/master/conf/awsbatch.config) and then specify all of the following parameters.

### `--awsqueue`

The JobQueue that you intend to use on AWS Batch.

### `--awsregion`

The AWS region in which to run your job. Default is set to `eu-west-1` but can be adjusted to your needs.

### `--awscli`

The [AWS CLI](https://www.nextflow.io/docs/latest/awscloud.html#aws-cli-installation) path in your custom AMI. Default: `/home/ec2-user/miniconda/bin/aws`.

Please make sure to also set the `-w/--work-dir` and `--outdir` parameters to a S3 storage bucket of your choice - you'll get an error message notifying you if you didn't.

## Other command line parameters

<!-- TODO nf-core: Describe any other command line flags here -->

### `--outdir`

The output directory where the results will be saved.

### `--email`

Set this parameter to your e-mail address to get a summary e-mail with details of the run sent to you when the workflow exits. If set in your user config file (`~/.nextflow/config`) then you don't need to specify this on the command line for every run.

### `--email_on_fail`

This works exactly as with `--email`, except emails are only sent if the workflow is not successful.

### `--max_multiqc_email_size`

Threshold size for MultiQC report to be attached in notification email. If file generated by pipeline exceeds the threshold, it will not be attached (Default: 25MB).

### `-name`

Name for the pipeline run. If not specified, Nextflow will automatically generate a random mnemonic.

This is used in the MultiQC report (if not default) and in the summary HTML / e-mail (always).

**NB:** Single hyphen (core Nextflow option)

### `-resume`

Specify this when restarting a pipeline. Nextflow will used cached results from any pipeline steps where the inputs are the same, continuing from where it got to previously.

You can also supply a run name to resume a specific run: `-resume [run-name]`. Use the `nextflow log` command to show previous run names.

**NB:** Single hyphen (core Nextflow option)

### `-c`

Specify the path to a specific config file (this is a core NextFlow command).

**NB:** Single hyphen (core Nextflow option)

Note - you can use this to override pipeline defaults.

### `--custom_config_version`

Provide git commit id for custom Institutional configs hosted at `nf-core/configs`. This was implemented for reproducibility purposes. Default: `master`.

```bash
## Download and use config file with following git commid id
--custom_config_version d52db660777c4bf36546ddb188ec530c3ada1b96
```

### `--custom_config_base`

If you're running offline, nextflow will not be able to fetch the institutional config files
from the internet. If you don't need them, then this is not a problem. If you do need them,
you should download the files from the repo and tell nextflow where to find them with the
`custom_config_base` option. For example:

```bash
## Download and unzip the config files
cd /path/to/my/configs
wget https://github.com/nf-core/configs/archive/master.zip
unzip master.zip

## Run the pipeline
cd /path/to/my/data
nextflow run /path/to/pipeline/ --custom_config_base /path/to/my/configs/configs-master/
```

> Note that the nf-core/tools helper package has a `download` command to download all required pipeline
> files + singularity containers + institutional configs in one go for you, to make this process easier.

### `--max_memory`

Use to set a top-limit for the default memory requirement for each process.
Should be a string in the format integer-unit. eg. `--max_memory '8.GB'`

### `--max_time`

Use to set a top-limit for the default time requirement for each process.
Should be a string in the format integer-unit. eg. `--max_time '2.h'`

### `--max_cpus`

Use to set a top-limit for the default CPU requirement for each process.
Should be a string in the format integer-unit. eg. `--max_cpus 1`

### `--plaintext_email`

Set to receive plain-text e-mails instead of HTML formatted.

### `--monochrome_logs`

Set to disable colourful command line output and live life in monochrome.

### `--multiqc_config`

Specify a path to a custom MultiQC configuration file.
