Autoseq
=======

Autoseq is a command line tool and it has multiple pipelines used to analyse cancer genomics data from clinical samples. 

 `Pipelines`

 * Alassca
 * Liqbio
 * RNAseq

*Current version: [v0.6.0][dist]*


Installation Requirements
-------------------------

### Python modules

Autoseq is written in python. All requirements for the autoseq scripts can be found in Anchorage directory -- `/nfs/ALASCCA/autoseq-scripts/requirements.txt`

 * begins
 * click
 * pandas
 * pybedtools
 * pysam

### Required tools, packages, scripts

`Conda` is the Package, dependency and environment management for any language. Here we are using the conda to manage the tools and packages for python.

Find all conda packages used in autoseq in Anchorage directory -- `/nfs/ALASCCA/autoseq/conda-list.txt`

bcftools(1.2), bedtools(2.25.0), bwa(0.7.12), cffi(1.10.0), click(6.7), cnvkit(0.7.9), cryptography(1.8.1), fastqc(0.11.4), freebayes(1.0.1), htslib(1.6), igvtools(2.3.93), lofreq(2.1.2), matplotlib(2.0.2), nose(1.3.7), openpyxl(2.4.0), parallel(20160622), picard(2.5.0), pindel(0.2.5b9), pip(9.0.1), psycopg2(2.7.1), pycparser(2.17), pysam(0.8.4), pytest(3.1.2), pyvcf(0.6.8.dev0), r(3.3.2), bioconductor-variantannotation(1.20.3)(r3.3.2_0), r-data.table(1.10.0), r-devtools(1.12.0), r-getopt(1.20.0), r-ggplot2(2.2.0), r-httr(1.2.1), r-plyr(1.8.4), r-pscbs(0.60.0), r-rcurl(1.95_4.8), r-reshape(0.8.6), r-rjsonio(1.3_0), sambamba(0.5.9), samblaster(0.1.22), samtools(1.2), scalpel(0.5.1), skewer(0.1.126), star(2.4.2a), ucsc-gtftogenepred(332), vardict-java(1.4.3), vardict(2016.02.19), variant-effect-predictor(83), vcflib(1.0.0_rc0), vt(2015.11.10)

### Installation

 Clone the autoseq-scripts repo

```
git clone https://github.com/ClinSeq/autoseq-scripts
```
 Go to home folder and open .profile file in nano and add the autoseq-scripts cloned directory to the `$PATH` variable

```
nano .profile
```
 Add this line

```
export PATH="$PATH:/path/to/directory/autoseq-scripts/"
```

 Install autoseq core repos using latest git repo.


```
sudo pip install git+https://github.com/ClinSeq/pypedream.git

sudo pip install git+https://github.com/clinseq/autoseq.git
```


How it works
------------

``` CLI
Usage: autoseq [OPTIONS] COMMAND [ARGS]...

Options:
  --ref TEXT          json with reference files to use
  --job-params TEXT   JSON file specifying various pipeline job parameters.
  --outdir PATH       output directory
  --libdir TEXT       directory to search for libraries
  --runner_name TEXT  Runner to use.
  --loglevel TEXT     level of logging
  --jobdb TEXT        sqlite3 database to write job info and stats
  --dot_file TEXT     write graph to dot file with this name
  --cores INTEGER     max number of cores to allow jobs to use
  --scratch TEXT      scratch dir to use
  --help              Show this message and exit.

Commands:
  alascca
  liqbio
  liqbio_prepare
```

Running Pipelines
-----------------

### Alassca


### Liqbio

The Liquid Biopsy pipeline is invoked by

``` CLI
autoseq --ref ref.json --outdir /path/to/outdir --jobdb jobdb.json --cores 5 --runner_name slurmrunner --libdir 
/path/to/libdir liqbio sample.json

```

The sample.json file has the format (right side). In this file, a single tumor and normal sample is allowed, but multiple plasma samples. If no tumor or normal sample is avaialble, they can be set to `null`, but if no plasma samples are available, it should be set to `[]` (empty list), for example `"CFDNA": []`.

```
{
    "sdid": "NA12877",
    "panel": {
        "T": "NA12877-T-03098849-TD1-TT1",
        "N": "NA12877-N-03098121-TD1-TT1",
        "CFDNA": ["NA12877-CFDNA-03098850-TD1-TT1", "NA12877-CFDNA-03098850-TD2-TT1"]
    },
    "wgs": {
        "T": "NA12877-T-03098849-TD1-WGS",
        "N": "NA12877-N-03098121-TD1-WGS",
        "CFDNA": ["NA12877-CFDNA-03098850-TD1-WGS"]
    }
}
```

For the plasma samples, merging of libraries will take place before calling. On alignment, the `@RG` tag will be set as follows: 

* `ID` = `SDID-TYPE-SAMPLEID-PREPID-CAPTUREID`
* `LB` = `SDID-TYPE-SAMPLEID-PREPID`
* `SM` = `SDID-TYPE-SAMPLEID`

Of note is that the library tag (`LB`) does not include the `CAPTUREID` part, to ensure that PCR duplicates are removed correctly. 

If a single prepared samples is exposed to capture twice, to create the libraries `NA12877-T-49-TD1-TT1` and `NA12877-T-49-TD1-TT2` (note different digits in the capture id), read pairs being identical between the two libraries should be considered duplicates since the sample was split after the final PCR step. Therefore, the `LB` for these libraries is set to `NA12877-T-49-TD1`. After merging the bam files, removal of PCR duplicates is done using Picard MarkDuplicates, which will do the right thing. 



