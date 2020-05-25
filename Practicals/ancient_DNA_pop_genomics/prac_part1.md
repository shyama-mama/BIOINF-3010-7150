# Practicals: Ancient DNA and population genomics

Bastien Llamas \(bastien.llamas@adelaide.edu.au\)
2020-05-26 and 2020-05-29

---
The two tutorials will be separated into:
1. Data handling (Tuesday 2020-05-26)
2. Population genomics applications (Friday 2020-05-29)

Icons are used to highlight sections of the tutorials:

:blue_book: Information

:computer: Hands-on task

:question: Question(s)

---
# Day 1: Data handling (Tuesday 2020-05-26)

## Tutorial outcomes

:blue_book: At the end of today's tutorial, you will know how to convert the information contained in a VCF file into other formats compatible with widely-used population genomics programs.

## Population genomics
:blue_book: In a nutshell, population genomics is the study of the genomic composition of populations and how evolution shaped it. Questions in population genomics classically target demographic history (population size through time), gene flow between populations, populations ancestry, or identification of conservation biology units.

:blue_book: While population genetics is usually restricted to a small set of genetic loci, population genomics leverages the large genomic datasets that have become available in recent years and uses up to millions of genetic loci at once.

:blue_book: We are not going to focus on the mathematical aspects of population genomics, but rather on how to manipulate genomic datasets and learn about a few popular approaches and tools. I encourage you to read Graham Coop's [course notes](https://github.com/cooplab/popgen-notes/blob/master/popgen_notes.pdf) if you are curious about the underlying mathematical theories.


## VCF format: a reminder
:blue_book: You have previously learnt about several raw or processed high throughput sequencing data formats (e.g., FASTQ, SAM/BAM, VCF). In particular, you should now know that [VCF](https://samtools.github.io/hts-specs/VCFv4.2.pdf) files contain information about variant calls found at specific positions in a reference genome.

:computer: We will use a VCF file of human chromosome 22 from the 1000 Genomes Project (1kGP) that we will save into a working directory in your home directory:
```bash
# Create working directory
mkdir ~/BIOINF_Tuesday
cd ~/BIOINF_Tuesday
# Download VCF file and its index from the 1kGP public FTP site (VCF file size: 214453750 bytes)
curl ftp://ftp.ncbi.nlm.nih.gov/1000genomes/ftp/release/20130502/ALL.chr22.phase3_shapeit2_mvncall_integrated_v5a.20130502.genotypes.vcf.gz	> 1kGP_chr22.vcf.gz
curl ftp://ftp.ncbi.nlm.nih.gov/1000genomes/ftp/release/20130502/ALL.chr22.phase3_shapeit2_mvncall_integrated_v5a.20130502.genotypes.vcf.gz.tbi	> 1kGP_chr22.vcf.gz.tbi
```

:computer: Although you could use your own scripts to parse VCF files and analyse variant calls, several tools have already been developed for your convenience. In particular, [BCFtools](http://samtools.github.io/bcftools/bcftools.html) is a set of useful utilities to manipulate variant calls in VCF files. Install it easily with the conda package management system.
```bash
conda config --add channels bioconda
conda config --add channels conda-forge
conda install -c bioconda bcftools
```

#### VCF meta-information and header lines
:computer: Have a look at the VCF file using `zless`. 

:blue_book: Meta-information lines start with `##` and contain various metadata. The header line starts with `#` and is tab separated. It contains 9 columns of information about the variant calls, and then one column per sample name:

||Name|Brief description (from [Wikipedia](https://en.wikipedia.org/wiki/Variant_Call_Format#The_columns_of_a_VCF))|
|:-|:-|:-|
1|	CHROM|	The name of the sequence (typically a chromosome) on which the variation is being called. This sequence is usually known as 'the reference sequence', i.e. the sequence against which the given sample varies.
2|	POS|	The 1-based position of the variation on the given sequence.
3|	ID|	The identifier of the variation, e.g. a dbSNP rs identifier, or if unknown a ".". Multiple identifiers should be separated by semi-colons without white-space.
4|	REF|	The reference base (or bases in the case of an indel) at the given position on the given reference sequence.
5|	ALT|	The list of alternative alleles at this position.
6|	QUAL|	A quality score associated with the inference of the given alleles.
7|	FILTER|	A flag indicating which of a given set of filters the variation has passed.
8|	INFO|    	An extensible list of key-value pairs (fields) describing the variation. See below for some common fields. Multiple fields are separated by semicolons with optional values in the format: <key>=<data>[,data].
9|	FORMAT|	An (optional) extensible list of fields for describing the samples. See below for some common fields.
+|	SAMPLE|	For each (optional) sample described in the file, values are given for the fields listed in FORMAT

:computer: Have a closer look at how the information in the [INFO](https://en.wikipedia.org/wiki/Variant_Call_Format#Common_INFO_fields) and [FORMAT](https://en.wikipedia.org/wiki/Variant_Call_Format#Common_FORMAT_fields) fields is commonly coded. The 1kGP VCF datasets also contain some project-specific keys explained in a file that can be downloaded.
```bash
wget ftp://ftp.ncbi.nlm.nih.gov/1000genomes/ftp/release/20130502/README_vcf_info_annotation.20141104
```

#### VCF body
:blue_book: The body of the VCF file is tab separated. Each line represents a unique variant site.

---
Before we move forward, let's see if you can retrieve basic information from a 1kGP VCF file that will be useful for population genomic analyses.

#### :question: *Questions*
1. Using `bcftools view` or bash commands, determine how many variant sites are recorded in the VCF file.
2. Using `bcftools query` or bash commands, determine how many samples are recorded in the VCF file.
3. The INFO fields contain a lot of information. In particular for the first variant position in the file: determine how many samples have data, how many ALT alleles are reported,  what the frequency of the ALT allele is globally, and what the frequency of the ALT allele is in East Asians.
4. Same as question 3 for variant position 16051249 (see the [BCFtools manual](http://samtools.github.io/bcftools/bcftools.html) for region or target formatting).
5. How many alternative alleles are observed at position 16050654?
6. Looking at the information contained in the FORMAT field in the body of the VCF file, what kind of data is stored in the VCF file for each sample?
---

#### Other useful 1kGP metadata
:computer: Download sample details from the 1kGP FTP site to learn about population of origin and sex of each individual.
```bash
wget ftp://ftp.ncbi.nlm.nih.gov/1000genomes/ftp/release/20130502/integrated_call_samples_v3.20130502.ALL.panel
```

---
#### :question: *Questions*
7. Using bash commands on the panel file you just downloaded, determine how many different populations and super-populations are represented in the 1kGP dataset.
8. How many individuals are in each super-population?
---

## Converting VCF files into population genomics formats
### Rationale
:blue_book: A VCF file may contain a lot of information (e.g. variant annotation) that can be very useful for clinical genomics. This was the case when you looked at a trio data with Jimmy Breen. However, population genomics applications only need a subset of the information in VCF file, i.e., variant genomic coordinates, variant ID, reference (REF) and alternative (ALT) alleles, and sample genotypes. This is what you typically find in the 1kGP data.

:blue_book: Genotypes can be coded differently depending on ploidy (in humans, diploid for autosomes or chrX in females, haploid for mitochondrial genomes and chrY), the number of alternate alleles, whether the genomes are phased (i.e., alleles on maternal and paternal chromosomes are identified) or not, and homo/heterozygosity. For convenience, 0 is used to code REF, and 1, 2, 3, etc are used to code ALT.
||Example|
|:-|:-|
|**Haploid, 2 alleles**|2 genotypes: 0, 1|
|**Diploid, not phased, 2 alleles**|3 genotypes: 0/0, 0/1, 1/1|
|**Diploid, phased, 2 alleles**|4 genotypes: 0\|0, 0\|1, 1\|0, 1\|1|
|**Haploid, n alleles**|n genotypes: e.g., 0, 1, 2, 7|
|**Diploid, not phased, n alleles**|n! genotypes: e.g., 0/0, 1/4, 0/2, 3/3|
|**Diploid, phased, n alleles**|n<sup>2</sup> genotypes: e.g., 0/0, 1/4, 0/2, 3/3|

---
:computer: Have a look at the first variant in the VCF file.

#### :question:*Questions*
9. What are the REF and ALT alleles?
10. Given REF and ALT alleles found when answering question 9, and knowing that the genotypes are phased, what are the possible genotypes with nucleotides and 1kGP coding?
---

:blue_book: You saw that the VCF genotype information can be very detailed. However, all we need usually for population genomics is a table of samples and variant calls, where the genotype information is coded so it can be parsed easily and file size remains as small as possible (imagine storing and parsing whole genome variation data for >100k individuals). 

### The PLINK format
:blue_book: [PLINK](https://www.cog-genomics.org/plink/1.9/) is a set of utilities that allows converting VCF files into more practical formats and manipulating variant calls. It also performs many operations on variant calls, such as calculating basic statistics or linkage desiquilibrium, for example.

:blue_book: The PLINK online manual is extremely detailed but also not easy to follow. Alternatively, you may want to have a look (not now though) at a [PLINK tutorial](http://zzz.bwh.harvard.edu/plink/tutorial.shtml) from Harvard University or a recent [book chapter](https://link.springer.com/protocol/10.1007/978-1-0716-0199-0_3) by Christopher C Chang.

:blue_book: Although PLINK can generate many different [file outputs](https://www.cog-genomics.org/plink/1.9/formats), the default outputs are as follows:
* .bed: binary file that contains genotype information.
* .bim: tab-delimited text file that always accompanies a .bed genotype file. It contains variant information, has no header line, and one line per variant with the following six fields:
  * Chromosome code (either an integer, or 'X'/'Y'/'XY'/'MT'; '0' indicates unknown) or name
  * Variant identifier
  * Position in morgans or centimorgans (safe to use dummy value of '0')
  * Base-pair coordinate (1-based)
  * Allele 1 (usually minor)
  * Allele 2 (usually major)
* .fam: tab-delimited text file that always accompanies a .bed genotype file. It contains sample information, has no header line, and one line per sample with the following six fields:
  * Family ID ('FID')
  * Within-family ID ('IID'; cannot be '0')
  * Within-family ID of father ('0' if father isn't in dataset)
  * Within-family ID of mother ('0' if mother isn't in dataset)
  * Sex code ('1' = male, '2' = female, '0' = unknown)
  * Phenotype value ('1' = control, '2' = case, '-9'/'0'/non-numeric = missing data if case/control)

:computer: Convert the VCF file to PLINK files.
```bash
plink \
  --vcf 1kGP_chr22.vcf.gz \
  --out plink_temp
```

---
:computer: Have a look at the newly created files.

#### :question:*Questions*
11. How many files have been generated, and what are their extensions?
12. If you look at the content of the PLINK variant file, you will notice that some variants are not bi-allelic SNPs. Provide an example of at most 2 other types of variations (tell what variations you observe and report the whole line for each example).
13. Is the information stored in the panel file (integrated_call_samples_v3.20130502.ALL.panel) downloaded from the 1kGP FTP site reported in the PLINK sample file?
---

:blue_book: The VCF file does not contain information about each sample's population of origin or sex. That information is stored in the panel file. Thus we need to create a file that will be used to generate the .fam output when we convert the VCF file into PLINK files. For this, we simply have to follow instructions from the [PLINK online manual](http://www.cog-genomics.org/plink/1.9/data#update_indiv). 

:computer: By default, PLINK assigns the sample ID from the VCF file as both family ID and within-family ID. What we want instead is keep track of the population (pop) and super-population (super_pop) information stored in the panel file for future analyses. We will simply store that information using `_` in the family ID field, and store the sample ID in the within-family ID field. Also by default, PLINK assign missing sex information (`0`) to all samples, unless you provide that information. Create a tab-delimited updated ID file with one line per sample and the following 5 fields:
* Old family ID
* Old within-family ID
* New family ID
* New within-family ID
* sex information (1 or M = male, 2 or F = female, 0 = missing)
```bash
# Check that the panel file only contains "male" or "female" info, and does not have missing sex information (total should be 2504)
tail -n+2 integrated_call_samples_v3.20130502.ALL.panel | cut -f4 | sort | uniq -c
# Generate updateFields file containing the 5 fields described above
awk -v \
 'OFS=\t' \
 'NR>1 {print $1, $1, $3"_"$2, $1, toupper(substr($4, 1, 1))}' \
 integrated_call_samples_v3.20130502.ALL.panel \
 > updateFields
# Check that the updateFields file contains 2504 records
wc -l updateFields
```

:computer: Convert the VCF file into PLINK files.
```bash
# Update sex first (sex and IDs cannot be updated at the same time)
plink \
  --vcf 1kGP_chr22.vcf.gz \
  --snps-only \
  --biallelic-only strict \
  --keep-allele-order \
  --vcf-filter \
  --maf 0.10 \
  --update-sex updateFields 3 \
  --make-bed \
  --out 1kGP_chr22
# Then update IDs
plink \
  --bfile 1kGP_chr22 \
  --update-ids updateFields \
  --make-just-fam \
  --out 1kGP_chr22
```


### The Eigenstrat format

Prune LD
