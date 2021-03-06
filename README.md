# PSASS

Current release: **2.1.0**.

To cite PSASS, please use the DOI below:

[![DOI](https://zenodo.org/badge/111808510.svg)](https://zenodo.org/badge/latestdoi/111808510)

## Overview

PSASS (Pooled Sequencing Analysis for Sex Signal) is a software to analyze Pooled-Sequencing data to look for sex signal. It is part of a general pipeline handling data processing ([PSASS-process](https://github.com/RomainFeron/PSASS-process)), analysis (PSASS), and visualisation ([PSASS-vis](https://github.com/RomainFeron/PSASS-vis)). PSASS was developed as part of a project by the [LPGP](https://www6.rennes.inra.fr/lpgp/) lab from INRA, Rennes, France.

Currently, PSASS uses as input a `.sync` file generated from a pileup file (`samtools mpileup`) with either `psass convert` or with the [Popoolation2](https://sourceforge.net/projects/popoolation2/) software. PSASS `analyze` computes the following metrics:

- The position of all bases with high male/female Fst
- The position of all sex-specific SNPs, defined as SNPs heterozygous in one sex and homozygous in the other
- Male / female Fst in a sliding window
- Number of male- and female-specific SNPs in a sliding window
- Absolute and relative coverage in the male and female pools in a sliding window
- Number of male- and female-specific SNPs, and absolute and relative coverage in the male and female pools for all genes in a provided GFF, with separate metrics for coding and noncoding elements (may not work with some GFFs as this format has no true standard)

## Installation

```
# Clone the repository
git clone https://github.com/RomainFeron/PSASS.git
# Navigate to the PSASS directory
cd PSASS
# Build PSASS
make
```

## Basic usage

Currently, PSASS has two commands:

- `convert` : convert output from samtools mpileup to a compact sync file
- `analyze` : analyze a popoolation2 sync file and output metrics

### Analyze

```
psass analyze [options] --input-file input_file.sync --output-prefix output_prefix
```

#### Options:

Flag | Type | Description | Default |
-----|------|-------------|---------|
--input-file         |  `string`  |  Input file (popoolation sync file)                                   |           |
--output-prefix      |  `string`  |  Full prefix (including path) for output files                        |           |
--pool1              |  `string`  |  ID of the first pool in the pileup file                              | "females" |
--pool2              |  `string`  |  ID of the second pool in the pileup file                             | "males"   |
--gff-file           |  `string`  |  GFF file for gene-specific output                                    | ""        |
--output-fst-pos     |  `bool`    |  If true, output fst positions (0/1)                                  | 1         |
--output-fst-win     |  `bool`    |  If true, output fst sliding window (0/1)                             | 1         |
--output-snps-pos    |  `bool`    |  If true, output snps positions (0/1)                                 | 1         |
--output-snps-win    |  `bool`    |  If true, output snps sliding window (0/1)                            | 1         |
--output-depth       |  `bool`    |  If true, output depth(0/1)                                           | 1         |
--min-depth          |  `int`     |  Minimum depth to process a site for FST / SNPs                       | 10        |
--min-fst            |  `float`   |  FST threshold to output bases with high FST                          | 0.25      |
--freq-het           |  `float`   |  Frequency of a sex-linked SNP in the heterogametic sex               | 0.5       |
--freq-hom           |  `float`   |  Frequency of a sex-linked SNP in the homogametic sex                 | 1         |
--range-het          |  `float`   |  Range of frequency for a sex-linked SNP in the heterogametic sex     | 0.1       |
--range-hom          |  `float`   |  Range of frequency for a sex-linked SNP in the homogametic sex       | 0.05      |
--window-size        |  `int`     |  Size of the sliding window (in bp)                                   | 100000    |
--output-resolution  |  `int`     |  Output resolution (in bp)                                            | 10000     |
--group-snps         |  `bool`    |  Group consecutive snps to count them as a single polymorphism (0/1)  | 0         |


### Output files

- **<prefix\>_fst_position.tsv** : a tabulated file with fields Contig, Position, Fst (between-pools Fst for this base)
- **<prefix\>_fst_window.tsv** : a tabulated file with fields Contig, Position (center of the window), Fst (between-pools Fst for this window)
- **<prefix\>_snps_position.tsv** : a tabulated file with fields Contig, Position, Pool, and nucleotide frequencies at this base in each pools (with I=Indel)
- **<prefix\>_snps_window.tsv** : a tabulated file with fields Contig, Position (center of the window), and number of pool-specific SNPs for each pool
- **<prefix\>_depth.tsv** : a tabulated file with fields Contig, Position (center of the window), and absolute and relative depth for each pool in this window
- **<prefix\>_genes.tsv** : a tabulated file with fields Contig, Start (gene starting position), End (gene ending position), ID (gene ID), Name (gene name), Product (gene product), and absolute and relative coverage for each pool as well as number of pool-specific SNPs in the entire gene, in coding regions, and in noncoding regions.


### How does PSASS work ?

In this section, we will briefly describe the process implemented in PSASS to compute FST and pool-specific SNPs.

The input of PSASS `analyze` is a file that contains nucleotide counts for each pool and each position in the reference genome. PSASS parses this input and computes:

* FST between the two pools for each position, using the formula implemented in [Popoolation2](https://academic.oup.com/bioinformatics/article/27/24/3435/306737).
* FST between pools in a sliding window, using the estimate defined in [Karlsson *et al.*, 2007](https://www.nature.com/articles/ng.2007.10) (in Supplementary information).
* The position of pool-specific SNPs. A pool-specific SNP is called at a given genomic position if, for any possible nucleotide, the frequency of this nucleotide is 1 in one pool and 0.5 in the other pool. Values for these thresholds can be adjusted with command-line arguments, and a range can be given; by default these values are: 1 - [0-0.05] in the homozygous pool, and 0.5 +/- 0.1 in the heterozygous pool).
* Number of pool-specific SNPs in a sliding window, using the definition from above.

Therefore, the definition of FST and pool-specific SNPs used in PSASS is only base on comparing the pools themselves; the reference used for the alignments is not used to call SNPs.
