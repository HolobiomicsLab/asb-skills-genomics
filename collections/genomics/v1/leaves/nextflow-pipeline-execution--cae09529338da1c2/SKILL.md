---
name: nextflow-pipeline-execution
description: Use when raw WES/WGS FASTQ samples from a family (e.g., a parent-offspring
  trio) must be run through the eDiVA Nextflow workflows to produce a multisample
  VCF and prioritized variant lists.
when_to_use_negative:
- Input is already a multisample VCF or ranked CSV — skip eDiVA-Predict and start
  at the Annotate/Prioritize steps instead
- CNV analysis is required — the article states CNV analysis is not yet available
  in eDiVA (integration pending)
- Samples are unrelated singletons with no pedigree/inheritance model — eDiVA's joint
  calling and inheritance filtering assume family structure such as trios
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3571
tools:
- name: eDiVA
  role: Pipeline whose Nextflow workflows implement Predict, Annotate/Score, and Prioritize
    stages
  repo: https://github.com/mbosio85/ediva
- name: Nextflow
  role: Workflow engine running eDiVA .nf scripts with local, SGE, or Docker executors
  repo: https://www.nextflow.io/
- name: bwa-mem
  role: Aligns FASTQ reads sample-wise to hg19 within the eDiVA-Predict stage
- name: GATK
  role: Performs variant calling (HaplotypeCaller) per GATK best practices inside
    eDiVA-Predict
  repo: https://software.broadinstitute.org/gatk/
- name: samtools
  role: Post-processes alignments (v1.5 required per README)
  repo: http://samtools.sourceforge.net/
- name: Picard
  role: Alignment post-processing tool (v1.119) used in eDiVA-Predict
  repo: https://broadinstitute.github.io/picard/
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1002/humu.23772
    title: eDiVA—Classification and prioritization of pathogenic variants for clinical
      diagnostics
  source_content_hash: sha256:30783a6d3a608c09b572e9aa100f0bd1e6070f0f3d0aae944986089bf844387f
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/mbosio85/ediva
  tools:
  - GATK
  - Nextflow
  - Picard
  - bwa-mem
  - eDiVA
  - samtools
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3571
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.46153846153846156
n_claims: 13
---

# nextflow-pipeline-execution

## Summary

Execute the eDiVA exome variant-analysis workflows (eDiVA-Predict, eDiVA_prepare_for_annotation, eDiVA-Annotate/Score, eDiVA-Prioritize) as Nextflow processes, locally or via SGE/Docker, to turn raw WES FASTQ reads into ranked, inheritance-filtered variant callsets.

## When to use

Use when raw WES/WGS FASTQ samples from a family (e.g., a parent-offspring trio) must be run through the eDiVA Nextflow workflows to produce a multisample VCF and prioritized variant lists.

## When NOT to use

- Input is already a multisample VCF or ranked CSV — skip eDiVA-Predict and start at the Annotate/Prioritize steps instead
- CNV analysis is required — the article states CNV analysis is not yet available in eDiVA (integration pending)
- Samples are unrelated singletons with no pedigree/inheritance model — eDiVA's joint calling and inheritance filtering assume family structure such as trios

## Inputs

- Paired-end FASTQ reads per sample (fastq.read1.gz, fastq.read2.gz)
- hg19 reference FASTA indexed for bwa
- dbSNP VCFs split into SNPs and indels (dbsnp_138.hg19.snps.vcf / .indels.vcf)
- Exome target BED file (kit regions extended 150 bp, sorted and merged)
- Concatenated sample_info.txt family config file
- pedigree.tree and HPO term list for prioritization

## Outputs

- Per-sample aligned/filtered BAM files
- Per-sample all_variants.vcf with SNP and INDEL calls
- sample_info.txt per sample
- Quality_control folder with FastQC results and quality metrics
- combined.variants.supplement.vcf multisample VCF and pedigree.tree
- combined.variants.supplement.ranked.csv annotated/scored variant table
- combined.variants.supplement.filtered.<inheritance>.csv and variant_prioritization_report.xlsx

## How to apply

Install nextflow, then edit the provided nextflow.config to point env variables (REF, DBSNP, DBINDEL, BWA, GATK, PICARD, SAMTOOLS, BEDTOOLS, FASTQC, EXOME, PYTHON) at local tool installations; the EXOME parameter is a BED file of kit regions extended 150 bp per side, sorted and merged. Launch eDiVA-Predict.nf per sample with --NAME, --READ1/--READ2, --AFFECTED (1 for affected, default 0), --CPU 2, --OUTF and -w work dir, obtaining per-sample BAMs, all_variants.vcf, sample_info.txt and a Quality_control folder. After all family samples are called, concatenate their sample_info.txt files into a FAMILY_INFO file (verify the AFFECTED column) and run eDiVA_prepare_for_annotation.nf to jointly regenotype variant positions and emit combined.variants.supplement.vcf plus pedigree.tree. Subsequent eDiVA-Annotate.nf (which auto-runs eDiVA-Score) and eDiVA-Prioritize.nf accept the multisample VCF and ranked CSV with --MODE standard/strict, --FAMILY_TYPE trio, --INHERITANCE (dominant_inherited, recessive, dominant_denovo, Xlinked, compound), --HPO and --EXCLUSIONLIST parameters. Choose the SGE executor with memory/cpus/queue overrides for HPC runs, or use the Docker section with nextflow for containerized execution.

## Related tools

- **eDiVA** (Pipeline whose Nextflow workflows implement Predict, Annotate/Score, and Prioritize stages) — https://github.com/mbosio85/ediva
- **Nextflow** (Workflow engine running eDiVA .nf scripts with local, SGE, or Docker executors) — https://www.nextflow.io/
- **bwa-mem** (Aligns FASTQ reads sample-wise to hg19 within the eDiVA-Predict stage)
- **GATK** (Performs variant calling (HaplotypeCaller) per GATK best practices inside eDiVA-Predict) — https://software.broadinstitute.org/gatk/
- **samtools** (Post-processes alignments (v1.5 required per README)) — http://samtools.sourceforge.net/
- **Picard** (Alignment post-processing tool (v1.119) used in eDiVA-Predict) — https://broadinstitute.github.io/picard/

## Examples

```
nextflow run ~/ediva/edivatools-code/Predict/eDiVA-Predict.nf --NAME sample_name --READ1 fastq.read1.gz --READ2 fastq.read2.gz --AFFECTED 1 --CPU 2 --OUTF output_folder/ -w work_folder/
```

## Evaluation signals

- Per-sample output folder contains the BAM, all_variants.vcf, sample_info.txt, and a Quality_control folder with FastQC results
- combined.variants.supplement.vcf is valid VCF with genotype fields for all three trio samples and a variant count consistent with the ~138,705 variants reported for the CEPH trio
- The AFFECTED column in the concatenated FAMILY_INFO file is correct (0 for non-affected, 1 for affected) before joint calling
- Prioritization outputs include combined.variants.supplement.filtered.<inheritance>.csv sorted by eDiVA-Score and variant_prioritization_report.xlsx with one spreadsheet per inheritance type
- eDiVA-Annotate output CSV contains the annotation columns required by eDiVA-Score for on-the-fly score calculation

## Limitations

- CNV analysis is not yet available in eDiVA (integration pending), so this workflow covers SNVs and small indels only
- The EXOME BED must be built by extending kit specifications 150 bp on each side then sorting and merging intervals; wrong exome definitions will skew calling
- eDiVA-Predict output names are fixed ('all_variants.vcf'), so one output folder per sample is required to avoid overwrite
- Python 2.7 and specific package versions (e.g., mysql-connector==2.1.4) are required, which may conflict with modern environments

## Evidence

- [methods] The eDiVA‐Predict module performs sample‐wise variant calling according to the recent GATK (McKenna et al., 2010) best practices: "The eDiVA‐Predict module performs sample‐wise variant calling according to the recent GATK (McKenna et al., 2010) best practices"
- [methods] Reads in fastq format are aligned using bwa‐mem: "Reads in fastq format are aligned using bwa‐mem (H. Li, 2013)"
- [readme] this step takes care to perform the joint multisample calling of vairant positions, in order to gather all information for all samples: "this step takes care to perform the joint multisample calling of vairant positions, in order to gather all information for all samples"
- [readme] EXOME parameter is the bed file with the region of the genome covered by your kit. We normally extend the factory specifications by 150bp on each side, then we sort and merge the intervals.: "EXOME parameter is the bed file with the region of the genome covered by your kit. We normally extend the factory specifications by 150bp on each side, then we sort and merge the intervals."
- [readme] It is important here to double check the text file to verify the AFFECTED column is correct, 0 for non-affected samples and 1 for the affected ones: "It is important here to double check the text file to verify the AFFECTED column is correct, 0 for non-affected samples and 1 for the affected ones"
- [readme] The two steps are joined because eDiVA-Score requires eDiVA-Annotate input columns to be calculated on the fly.: "The two steps are joined because eDiVA-Score requires eDiVA-Annotate input columns to be calculated on the fly."
