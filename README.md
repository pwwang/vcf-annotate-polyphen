# vcf-annotate-polyphen
A tool to annotate human VCF files with PolyPhen-2 effect measures.
This tool only works on human variants,
collects ClinVar scores,
and assumes the VCF follows `hg19/GRCh37` conventions.

## Install
### via PyPi
```
$ pip install vcf-annotate-polyphen
```

### via Source Code
```
$ git checkout https://github.com/hammerlab/vcf-annotate-polyphen.git
$ cd vcf-annotate-polyphen/
$ python setup.py
```

## Usage
### As a library
```python
  import vap  # Vcf-Annotate-Polyphen (VAP)

  import sqlalchemy
  from sqlalchemy import create_engine
  engine = create_engine('sqlite:///polyphen-2.2.2-whess-2011_12.sqlite')
  conn = engine.connect()

  annotation = vap.annotate_variant(conn, 'chr14', 20344588, 'C', 'A')
  print ("Gene: {}; Protein: {}; Change: {}; "
           "HVar Prediction: {} (p: {}); HDiv Prediction: {} (p: {})") \
           .format(
            annotation.gene,
            annotation.protein,
            annotation.aa_change,
            annotation.hvar_pred,
            annotation.hvar_prob,
            annotation.hdiv_pred,
            annotation.hdiv_prob)
  # Gene: OR4K2; Protein: Q8NGD2; Change: H54Q;
  #  HVar Prediction: benign (p: 0.017); HDiv Prediction: benign (p: 0.008)
```

### Command line interface
After installing the package, you can invoke the command line utility as follows:

```
$ vcf-annotate-polyphen --help
Usage: vcf-annotate-polyphen polyphen.whess.sqlite input.vcf output.vcf

Options:
  -h, --help  show this help message and exit
```

As listed above in the help text, this tool expects three arguments from the user:

1. [PolyPhen-2 WHESS](ftp://genetics.bwh.harvard.edu/pph2/whess) in SQLite format
2. Input VCF to be annotated
3. Output VCF to be written with annotations

The output file should have an additional `INFO` field as described below:

```
##INFO=<ID=PP2,Number=1,Type=String,Description="PolyPhen2 annotations in the following order:Gene name; UniProt id; Amino acid change; ClinVar effect category; Strength of effect (probability)">
```

which manifests itself for each variant description:

```
...
2	165351172	.	T	A	.	PASS	SOMATIC;VT=SNP;PP2=.,.,.,.,.,.,.	GT:AD:BQ:DP:FA:SS	0:11,0:.:11:0.0:0	0/1:3,5:30.0:8:0.625:2
2	179247908	.	C	G	.	PASS	SOMATIC;VT=SNP;PP2=OSBPL6,Q9BZF3,N593K,probably damaging,0.998,probably damaging,1.0	GT:AD:BQ:DP:FA:SS	0:27,2:.:29:0.069:0	0/1:27,4:28.0:31:0.129:2
...
```

Here is an annotated VCF: [example/TCGA-55-6543.annotated.vcf](example/TCGA-55-6543.annotated.vcf).

### Example usage
```
$ cd example/
# The following file is ~7 GB!!!
$ wget "ftp://genetics.bwh.harvard.edu/pph2/whess/polyphen-2.2.2-whess-2011_12.sqlite.bz2"
$ bunzip2 polyphen-2.2.2-whess-2011_12.sqlite.bz2
$ vcf-annotate-polyphen ./polyphen-2.2.2-whess-2011_12.sqlite ./TCGA-55-6543.vcf ./TCGA-55-6543.annotated.vcf
$ less ./TCGA-55-6543.annotated.vcf
```
