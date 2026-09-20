---
name: cross-tool-expression-benchmarking
description: Use when you need to validate a long-read transcript quantification workflow
  by measuring replicate-to-replicate Pearson correlations (e.g. >0.971) and cross-tool
  agreement with an alternative quantifier such as LIQA (~0.83), as in Fig. 5g / Supplementary
  Table 4.
when_to_use_negative:
- Data are short-read RNA-seq rather than full-length Iso-seq/ONT transcript reads
  — TAGET is designed for full-length transcript data
- There are no biological replicates and no independent alternative quantification
  (e.g. LIQA) to compare against, so reproducibility and cross-tool correlation cannot
  be computed
- The goal is novel isoform or gene fusion discovery rather than expression quantification
  accuracy assessment
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0203
tools:
- name: TAGET
  role: aligns, annotates and TPM-quantifies isoform/gene expression per replicate
    via TransAnnot.py, fa2exp.py and TransAnnotMerge (-m TPM)
  repo: https://github.com/XiDsLab/TAGET
- name: minimap2
  role: one of the aligners used by TransAnnot to map full-length transcript FASTA
    reads to the hg38 genome
- name: HISAT2
  role: alternative aligner used by TransAnnot for transcript alignment against the
    indexed hg38 reference
- name: LIQA
  role: independent ONT quantification method whose per-replicate expression values
    are correlated against TAGET output (~0.83 Pearson)
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-023-41649-0
    title: 'TAGET: a toolkit for analyzing full-length transcripts from long-read
      sequencing'
  source_content_hash: sha256:948a1e47116ab119b3126b7c8f93c132d622531277427016f0495b8c07feb76a
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/XiDsLab/TAGET
  tools:
  - HISAT2
  - LIQA
  - TAGET
  - minimap2
  edam_operation: http://edamontology.org/operation_2939
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5384615384615384
n_claims: 13
---

# cross-tool-expression-benchmarking

## Summary

Benchmark the reproducibility and accuracy of long-read isoform/gene expression quantification by computing pairwise Pearson correlations across biological replicates within one tool (TAGET on ONT GM12878 replicates, expected >0.971) and cross-tool correlations against an alternative method (LIQA, expected ~0.83), writing results to gm12878_taget_correlations.csv.

## When to use

Use when you need to validate a long-read transcript quantification workflow by measuring replicate-to-replicate Pearson correlations (e.g. >0.971) and cross-tool agreement with an alternative quantifier such as LIQA (~0.83), as in Fig. 5g / Supplementary Table 4.

## When NOT to use

- Data are short-read RNA-seq rather than full-length Iso-seq/ONT transcript reads — TAGET is designed for full-length transcript data
- There are no biological replicates and no independent alternative quantification (e.g. LIQA) to compare against, so reproducibility and cross-tool correlation cannot be computed
- The goal is novel isoform or gene fusion discovery rather than expression quantification accuracy assessment

## Inputs

- full-length transcript FASTA files from ONT GM12878 cell-line replicates (3 replicates)
- hg38 reference genome (hg38.fa)
- Ensembl GTF annotation (hg38.ensembl.gtf) and its gtf_db_make.py pickle
- LIQA quantification of the same replicates (external reference expression values)

## Outputs

- {sample_id}.transcript.exp and {sample_id}.gene.exp per-replicate expression files
- multi-sample gene.exp and transcript.exp expression matrices
- gm12878_taget_correlations.csv containing replicate pairwise and TAGET-vs-LIQA Pearson correlations

## How to apply

Quantify isoform and gene expression per replicate with TAGET by running TransAnnot.py per sample (aligned with minimap2 and/or HISAT2 against hg38 with Ensembl GTF, ≥8 processes), then compute TPM with the --tpm parameter and extract isoform expression from the transcript FASTA using fa2exp.py. Merge per-sample {sample_id}.annot.stat, {sample_id}.annot.bed, and {sample_id}.annot.db.pickle files with TransAnnotMerge using -m TPM to produce {sample_id}.transcript.exp, {sample_id}.gene.exp and multi-sample transcript.exp/gene.exp matrices. Compute pairwise Pearson correlations of isoform expression across the three GM12878 replicates (TAGET reported >0.971) and the Pearson correlation between TAGET expression and LIQA quantification of the same replicates (~0.83). Write all correlation values to gm12878_taget_correlations.csv; treat replicate correlations below 0.971 or a large deviation from the ~0.83 cross-tool value as failure signals requiring inspection of alignment/annotation outputs.

## Related tools

- **TAGET** (aligns, annotates and TPM-quantifies isoform/gene expression per replicate via TransAnnot.py, fa2exp.py and TransAnnotMerge (-m TPM)) — https://github.com/XiDsLab/TAGET
- **minimap2** (one of the aligners used by TransAnnot to map full-length transcript FASTA reads to the hg38 genome)
- **HISAT2** (alternative aligner used by TransAnnot for transcript alignment against the indexed hg38 reference)
- **LIQA** (independent ONT quantification method whose per-replicate expression values are correlated against TAGET output (~0.83 Pearson))

## Examples

```
python TransAnnot.py -f {sample_id}.fasta -g hg38.fa -o outdir -a hg38.ensembl.gtf -p 8 --use_minimap2 1 --tpm
```

## Evaluation signals

- Pairwise Pearson correlation of all detected isoforms across the three GM12878 replicates exceeds 0.971
- Pearson correlation between TAGET expression and LIQA quantification of the same replicates approximates 0.83
- Per-replicate TransAnnot outputs {sample_id}.annot.stat, {sample_id}.annot.bed and {sample_id}.annot.db.pickle exist and merge cleanly with TransAnnotMerge
- gene.exp and transcript.exp expression matrices contain all three samples after TransAnnotMerge with -m TPM
- gm12878_taget_correlations.csv is written with all replicate-pairwise and cross-tool correlation values matching Fig. 5g / Supplementary Table 4

## Limitations

- The ~0.83 TAGET-vs-LIQA correlation indicates quantification agreement, not identity — cross-tool differences are expected and the validation threshold is approximate
- Iso-seq CCS-read cluster filtering removes ~80% of reads, some from true expressed isoforms, limiting detected gene numbers and thus quantification coverage
- Running time is under 1 hour with 8 cores on a Linux server; fewer processes will slow the per-replicate TransAnnot runs
- Performance numbers are specific to the GM12878 ONT dataset and hg38/Ensembl annotation; other datasets may not reproduce these correlation values

## Evidence

- [results] TAGET's quantification on ONT GM12878 replicates was highly reproducible (pairwise Pearson correlation of all detected isoforms >0.971) and consistent with LIQA (Pearson correlation ~0.83).: [evidence span withheld — anchor status: not_found]
- [intro] Quantification includes gene/isoform expression quantification, differential expression gene (DEG) analysis, and differential isoform usage (DIU) analysis.: "Quantification includes gene/isoform expression quantification, differential expression gene (DEG) analysis"
- [readme] The reads\transcript\gene expression can be caculated by the parameter of --tpm: "The reads\transcript\gene expression can be caculated by the parameter of --tpm"
- [readme] extract isoform expression from fasta file：
	 `python fa2exp.py -f [fa] -i [prefix] -o [exp] -p [taget output dictionary]`: "python fa2exp.py -f [fa] -i [prefix] -o [exp] -p [taget output dictionary]"
- [readme] python TranAnnotMerge -c MergeConfig -o outputdir -m [TPM/FLC/None]: "`-m`：the gene and transcript expression displayed by different methods,FLC: full length count，if none is not expression matrix"
- [methods] Here we used hg38 to annotate transcripts,human reference hg38.fa and Ensemble gtf format files were needed.: "Here we used hg38 to annotate transcripts,human reference hg38.fa and Ensemble gtf format files were needed"
