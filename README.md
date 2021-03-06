## Combined Annotation Dependent Depletion (CADD)

CADD is a tool for scoring the deleteriousness of single nucleotide variants as well as insertion/deletions variants in the human genome (currently supported builds: GRCh37/hg19 and GRCh38/hg38).

Details about CADD, including features in the latest version, the different genome builds and how we envision the use case of CADD are described in our latest manuscript:
<blockquote>
    Rentzsch P, Witten D, Cooper GM, Shendure J, Kircher M. <br>
    <i>CADD: predicting the deleteriousness of variants throughout the human genome.</i><br>
    Nucleic Acids Res. 2018 Oct 29. doi: <a target="_blank" href="http://dx.doi.org/10.1093/nar/gky1016">10.1093/nar/gky1016</a>.<br>
    PubMed PMID: <a target="_blank" href="http://www.ncbi.nlm.nih.gov/pubmed/30371827">30371827</a>.
</blockquote>
The original manuscript describing the method and its features was published by Nature Genetics in 2014:
<blockquote>
Kircher M, Witten DM, Jain P, O'Roak BJ, Cooper GM, Shendure J. <br>
<i>A general framework for estimating the relative pathogenicity of human genetic variants.</i><br>
Nat Genet. 2014 Feb 2. doi: <a target="_blank" href="http://dx.doi.org/10.1038/ng.2892">10.1038/ng.2892</a>.<br>
PubMed PMID: <a target="_blank" href="http://www.ncbi.nlm.nih.gov/pubmed/24487276">24487276</a>.
</blockquote>

We provide pre-computed CADD-based scores (C-scores) for all 8.6 billion possible single nucleotide variants (SNVs) of the reference genome, as well as
all SNV and insertions/deletions variants (InDels) from population-wide whole genome variant releases and enable scoring of short InDels on our website.

Please check our [website for updates and further information](http://cadd.gs.washington.edu)

## Offline Installation

This section describes how users can setup CADD version 1.5 on their own system. Please note that this requires between 100 GB - 1 TB of disc space and at least 12 GB of RAM.

### Prerequisite

- conda
```bash
# can be installed like this
wget https://repo.continuum.io/miniconda/Miniconda2-latest-Linux-x86_64.sh
bash Miniconda2-latest-Linux-x86_64.sh -p $HOME/miniconda2 -b
export PATH=$HOME/miniconda2/bin:$PATH
```

*Note: You can also install CADD without conda by installing the dependencies otherwise. You can find the list of tools (VEP) and (python) libraries in the `src/environment.yml` file. In this case, you will also have to disable the line `source activate cadd-env` in `CADD.sh`*

*Note2: If you are using an existing conda installation, please make sure it is [a version >=4.4.0](https://github.com/conda/conda/issues/3200).* 

### Setup

- load/move the zipped CADD archive to its destination folder and unzip.

```bash
unzip CADD.zip
```

- from here, you can either install everything seperately or run the script `install.sh` to install using a brief installation dialog (see below).

#### Install script

This is the easier way of installing CADD, just run:

```
./install.sh
```

You first state which parts you want to install (the environment as well as at least one genome build including annotation tracks are neccessary for a quick start) and the script should manage loading and unpacking the neccessary files.

#### Manual installation

Running CADD depends on three big building blocks (plus the repository containing this README which we assume you already downloaded):

 - dependencies
 - genome annotations
 - prescored variants

**Installing dependencies**

As stated already in the Prerequisite you can install CADD dependencies without conda, although we heavily recommend doing so. This is because managing the various parts becomes very handy by relying it. To setup the neccessary environment, we only need to run the command:

```bash
conda env create -f src/environment.yml
```

After the installing process (which will take a few minutes), the CADD conda environment will be loaded (via `source activate cadd-env` automatically in the `CADD.sh` script) and CADD can run without further settings.

**Installing annotations**

Both version of CADD (for the different genome builds) rely on a big number of genomic annotations. Depending on which genome build you require you can get them from our website (be careful where you put them as these are really big files and have identical filenames) via:

```bash
# for GRCh37 / hg19
wget -c http://krishna.gs.washington.edu/download/CADD/v1.4/annotationsGRCh37.tar.gz
# for GRCh38 / hg38
wget -c http://krishna.gs.washington.edu/download/CADD/v1.5/annotationsGRCh38.tar.gz
```

As those files are about 100 and 200 GB in size, downloads can take long (depending on your internet connection). We recommend to setup the process in the background and using a tool (like `wget -c` mentioned above) that allows you to continue an interrupted download.

To make sure you downloaded the files correctly, we recommend downloading md5 hash files from our website (e.g. `wget http://krishna.gs.washington.edu/download/CADD/v1.4/GRCh37/MD5SUMs`) and checking for completness (via `md5sum -c`).

The annotation files are finally put in the folder `data/annotations` and unpacked:

```bash
cd data/annotations
tar -zxvf annotationsGRCh37.tar.gz
mv GRCh37 GRCh37_v1.4
tar -zxvf annotationsGRCh38.tar.gz
cd $OLDPWD
```

**Installing prescored files**

At this point you are ready to go, but if you want a faster version of CADD, you can download the prescored files from our website (see section Downloads for a list of available files). Please note that these files can be very big. The files are (together with their respective tabix indices) put in the folders `no_anno` or `incl_anno` depending on the file under `data/prescored/${GENOME_BUILD}_${VERSION}/` and will be automatically detected by the `CADD.sh` script.

### Running CADD

You run CADD via the script `CADD.sh` which technically only requieres an either vcf or vcf.gz input file as last argument. You can further specify the genome build via `-g`, CADD version via `-v`, request a fully annotated output (`-a` flag) and specify a seperate output file via `-o` (else inputfile name `.tsv.gz` is used). I.e:

```bash
./CADD.sh test/input.vcf

./CADD.sh -a -g GRCh37 -v v1.4 -o output_inclAnno_GRCh37.tsv.gz test/input.vcf
```

You can test whether your CADD is set up properly by comparing to the example files in the `test` directory.

### Update

Between versions 1.4 and 1.5, we adjusted the CADD repository slightly. If you used CADD before and obviously do not want to download all v1.4 files again, please proceed as follows:

1. update the repository (just overwriting is fine, however this dublicates some moved files so `git pull` is prefered)
2. rename the annotation (and prescored) folder from `data/annotations/$GENOMEBUILD` to `data/annotations/${GENOMEBUILD}_${VERSION}`

```
mv data/annotations/GRCh37 data/annotations/GRCh37_v1.4
mv data/annotations/GRCh38 data/annotations/GRCh38_v1.4

# if you have prescored files
mv data/prescored/GRCh37 data/prescored/GRCh37_v1.4
mv data/prescored/GRCh38 data/prescored/GRCh38_v1.4
```

## Copyright
Copyright (c) University of Washington, Hudson-Alpha Institute for
Biotechnology and Berlin Institute of Health 2013-2019. All rights reserved.

Permission is hereby granted, to all non-commercial users and licensees of CADD
(Combined Annotation Dependent Framework, licensed by the University of
Washington) to obtain copies of this software and associated documentation
files (the "Software"), to use the Software without restriction, including
rights to use, copy, modify, merge, and distribute copies of the Software. The
above copyright notice and this permission notice shall be included in all
copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR
IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY,
FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE
AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER
LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM,
OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE
SOFTWARE.
