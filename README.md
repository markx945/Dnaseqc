# Dnaseqc
Guideline for Quartet DNA QC Pipeline

# Quartet DNA Quality Control Evaluation

This repository contains the quality control (QC) evaluation pipelines for DNA sequencing data and DNA methylation sequencing data generated by the Quartet project. Our aim is to assess and ensure the quality of the sequencing data through comprehensive QC processes. The project is divided into two main parts: the DNA Data QC Pipeline and the DNA Methylation Data QC Pipeline.

## Table of Contents

- [Introduction](#introduction)
- [DNA Data QC Pipeline](#dna-data-qc-pipeline)
  - [Requirements](#requirements)
  - [Installation](#installation)
  - [Variant calling QC](#Variant-calling-QC)
  - [Generate QC report with dnaseqc](#Generate-QC-report-with-dnaseqc)

## Introduction

This project focuses on the quality control of DNA and DNA methylation sequencing data. Quality control is essential to validate the sequencing data for further analysis and research. Our pipelines leverage various software tools to analyze and generate QC reports, ensuring the reliability and accuracy of the data.

## DNA Data QC Pipeline

The DNA Data QC Pipeline starts with VCF files, using hap.py and VBT software to analyze and transform the data format, followed by dnaseqc to generate quality control reports.

### Requirements
- hap.py
- VBT
- dnaseqc
- multiqc
- rtgtools
### installation
- hap.py
(https://github.com/Illumina/hap.py)
- VBT
(https://github.com/sbg/VBT-TrioAnalysis)
- rtg-tools
（https://github.com/RealTimeGenomics/rtg-tools）
- Chinese Quartet
（https://chinese-quartet.org/#/reference-datasets/download）

### Variant calling QC
![image](https://github.com/markx945/Dnaseqc/assets/91772929/54c984fa-e915-444f-ac6d-c7f3087d7f34)

#### Performance assessment based on benchmark sets
Variants were compared with benchmark calls in benchmark regions.
We recommend that you input the four samples D5, D6, F7, and M8 from the same batch at one time.

The naming rules for VCF files should ideally follow the format {batch_name} + LCL5/6/7/8.
```bash
### truth.vcf is the Quartet reference vcf, confident.bed represents the high-confidence interval, and reference.fa is reference genome
hap.py truth.vcf query.vcf -f confident.bed -o output_prefix -r reference.fa

### WES data needs to be intersected with probes bed file first.
samtools intersect -a target.bed -b reference_dataset.bed > intersect.bed
hap.py truth.vcf query.vcf -f intersect.bed -o output_prefix -r reference.fa

### the output files are end with summary.csv

## Use MultiQC to integrate the D5, D6, F7, and M8, hap calculation results from the same batch
multiqc ./dir_to_four_summary_csv_file

## data integration scripts are in the extract_hap_result.ipynb file
## final output is variants.calling.qc.txt file
```

#### Performance assessment based on Quartet genetic built-in truth
We splited the Quartet family to two trios (F7, M8, D5 and F7, M8, D6) and then do the Mendelian analysis. A Quartet Mendelian concordant variant is the same between the twins (D5 and D6) , and follow the Mendelian concordant between parents (F7 and M8). Mendelian concordance rate is the Mendelian concordance variant divided by total detected variants in a Quartet family. Only variants on chr1-22,X are included in this analysis.

```bash
## Use rtgtools to merge the D5, D6, F7, and M8 files from the same batch for subsequent calculation of Mendelian heritability
rtg vcfmerge --force-merge-all -o ${project}.family.vcf.gz ${D5_vcf} ${D6_vcf} ${F7_vcf} ${M8_vcf}

## unzip the file for vbt input
gunzip ${project}.family.vcf.gz

## Run bvt.sh （The script needs to be modified according to the actual situation.）
## output ${family_name}.D5.txt、${family_name}.D6.txt and ${family_name}.consensus.txt files
bash vbt.sh

## Run merge_two_family_with_genotype.py to get mendelian result
python merge_two_family_with_genotype.py -LCL5 ${family_name}.D5.txt -LCL6 ${family_name}.D6.txt -genotype ${family_name}.consensus.txt -family {family_name}

```

#### output file

1.variants.calling.qc.txt文件示例
| Sample  | SNV number | INDEL number | SNV precision | INDEL precision | SNV recall | INDEL recall |
| :---: | :--: | :------: | :------:|  :------:|  :------:|  :------:|
| LCL5_UU_Illumina_D5_20230629_20171028_EATRISPLUS_UU_LCL5_hc  |  3855821  | 980430  | 99.73| 98.44 | 99.25 | 98.59 |
| LCL6_UU_Illumina_D6_20230629_20171028_EATRISPLUS_UU_LCL6_hc  |  3861023  | 976804  | 99.74| 98.51 | 99.38 | 98.68 |

2. ${family_name}.summary.txt

| Family  | Total_Variants | Mendelian_Concordant_Variants | Mendelian_Concordance_Rate |
| :---: | :--: | :------: | :------:|
| EATRISPLUS_UU.INDEL  |  1294054  | 1178598  | 0.910779611979|
| EATRISPLUS_UU.SNV  |  5034285  | 4868250   | 0.967019149691|


### Generate QC report with dnaseqc
Reminder: Analysis of the first two steps must be completed before generating the DNA QC report.
```R
## download and install dnaseqc
library(devtools)
devtools::install_github("markx945/Dnaseqc/dnaseqc")
library(dnaseqc)

## Read the F1 score calculation and Mendelian heritability calculation results
variant_qc <- system.file("example","variants.calling.qc.txt",package = "dnaseqc")
mendelian_qc <- system.file("example","EATRISPLUS_UU.summary.txt",package = "dnaseqc")

## Enter the sequencing type, "WGS" or "WES", to calculate the DNAseq QC metrics.
result = Dnaseqc(variant_qc_file = variant_qc, mendelian_qc_file = mendelian_qc, data_type = "WGS")

## Generate report 
### read report template from R packages
doc_path <- system.file("extdata","Quartet_temp.docx",package = "dnaseqc")
### generate QC report
GenerateDNAReport(DNA_result = result,doc_file_path = doc_path,output_path = './DNAseq/' )

```
#### output file
Quartet_DNA_report.docx




