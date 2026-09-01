# Colocalization Analysis Using the `coloc` R Package

This repository provides a pipeline to run colocalization analysis on QTL results using the [`coloc`](https://chr1swallace.github.io/coloc/) R package.

## Input Data Requirements

To run the analysis, provide the following input files:

### 1. GWAS Summary Statistics
A tab-delimited or comma-separated file containing the following columns:
* `SNP`: SNP IDs (must match the SNP IDs in the QTL file)
* `CHR`: Chromosome
* `POS`: Base-pair position
* `MAF`: Minor allele frequency
* `SE`: Standard error
* `BETA`: Effect size
* `N`: Sample size

### 2. QTL Summary Statistics
A QTL results file (e.g., from [MatrixEQTL](https://github.com/andreyshabalin/matrixeqtl)) with the following columns:
* `SNP`: SNP IDs (must match the SNP IDs in the GWAS file)
* `Gene`: Gene or CpG IDs
* `MAF`: Minor allele frequency
* `SE`: Standard error
* `BETA`: Effect size
* `N`: Sample size

### 3. Loci File
A file defining the genomic regions of interest for colocalization:
* `ID`: Region identifier (*optional*; automatically generated if omitted)
* `CHR`: Chromosome
* `START`: Genomic start position
* `END`: Genomic end position

### 4. Input List (`input_list.csv`)
A CSV file linking the input files and output prefixes for batch processing:
* `GWAS`: File name of the GWAS summary statistics
* `LOCI`: File name of the loci file
* `QTL`: File name of the QTL file
* `OUT`: Prefix for output files

---

## Usage and Configuration

The pipeline executes colocalization for each entry in `input_list.csv` using the Slurm submission script:

```bash
sbatch Main/Coloc.Permutation.Run.Slurm.sh
# or run directly:
bash Main/Coloc.Permutation.Run.Slurm.sh
```
Before running, configure the following parameters inside Main/Coloc.Permutation.Run.Slurm.sh:
* `input_list`: Path to the coloc input file. A colocalization analysis will be performed for each row in this file.
* `gwas_dir`: Directory containing the GWAS summary statistics files.
* `loci_dir`: Directory containing the loci files.
* `qtl_dir`: Directory containing the QTL files.
* `use_ld`: In each loci file, you can specify the lead SNPs' information instead of regions (ID, CHR, START, END). If you set this parameter to "yes", for all regions with a length < 2 (including lead SNPs), the linkage disequilibrium (LD) will be calculated based on reference genome genotype data. A region will then be defined based on an LD threshold and a distance (window size).
* `ref_genome_prefix`: PLINK binary prefix for the reference genome. Reference genome genotype data in binary PLINK format is required if you set use_ld to "yes".
* `ld_threshold`: LD threshold used to define the regions based on LD.
* `distance_`: A distance (in KB) to extend regions with a length < 2.
             If you set use_ld to "yes", this distance will be used as the "--ld-window-kb" option in the "plink --r2" command.
             If you set use_ld to "no", this distance will simply be used to define a region around the lead SNPs: [START - distance_, END + distance_].
* `type_QTL`: Type of trait in the QTL data ('quant' for a quantitative trait and 'cc' for binary). For more information, see the coloc package manual.
* `type_GWAS`: Type of trait in the GWAS data ('quant' for a quantitative trait and 'cc' for binary). For more information, see the coloc package manual.
* `use_permut`: If set to "yes", a permutation test will be performed on each Significant Coloc Result (SCR).
              An analysis is performed for each QTL-Region pair. For each SCR QTL-Region pair, 1,000 random regions with the same length (or processed in the same way for regions with length < 2) will be generated. After obtaining the results for all random regions, a P-value will be calculated using the following formula:
              P-value = (number of random SCRs) / 1000.
              A Significant Coloc Result (SCR) is a result containing at least one probability (H0-H4 or H3+H4) larger than coloc_P_threshold.
* `coloc_P_threshold`: A probability threshold to define Significant Coloc Results (default: 0.9).
* `n_permut`: Number of permutations (random regions) used to calculate a p-value based on the permutation analysis (default: 1000).
* `out_prefix`: Output files prefix.
* `ScriptDir`: Directory containing all dependent scripts.
* `Running_time`: The analysis will be executed using a Slurm array. Therefore, an estimated runtime is needed for the analysis (for each row in input_list).

