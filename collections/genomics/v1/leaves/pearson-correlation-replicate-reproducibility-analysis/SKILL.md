---
name: pearson-correlation-replicate-reproducibility-analysis
description: Use when you have expression quantification from ≥2 sequencing replicates
  (or two quantification tools on the same samples) and need to verify reproducibility
  (e.g., replicate Pearson r > 0.971) or cross-tool consistency (e.g., r ≈ 0.83).
when_to_use_negative:
- Input is raw read data with no per-sample expression matrices yet — annotation/quantification
  (TransAnnot, TransAnnotMerge) must precede correlation analysis
- Only a single sample/replicate is available, so pairwise replicate correlations
  cannot be computed
- You need differential isoform usage analysis (expression_V1.py with -r 0.05 / -p
  50 filters) rather than reproducibility assessment
edam_operation: http://edamontology.org/operation_3891
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0203
tools:
- name: TAGET
  role: Produces per-replicate transcript annotation and TPM expression matrices (TransAnnot.py,
    fa2exp.py, TransAnnotMerge) that are the input to the correlation analysis
  repo: https://github.com/XiDsLab/TAGET
- name: minimap2
  role: Aligner option (via --use_minimap2) for full-length transcript alignment against
    hg38 before quantification
- name: HISAT2
  role: Aligner option (via --use_hisat2 with a hisat2 index) for full-length transcript
    alignment against hg38 before quantification
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-023-41649-0
    title: 'TAGET: a toolkit for analyzing full-length transcripts from long-read
      sequencing'
  source_content_hash: sha256:99236f154a2cb0b7f4c9d5249c256e3df85b69868cc3b61789b7335b689cff76
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/XiDsLab/TAGET
  tools:
  - HISAT2
  - TAGET
  - minimap2
  edam_operation: http://edamontology.org/operation_3891
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 10
---

# pearson-correlation-replicate-reproducibility-analysis

## Summary

Computes pairwise Pearson correlations of TPM-based isoform and gene expression across sequencing replicates, and cross-tool correlations (e.g., TAGET vs LIQA), to quantify reproducibility and accuracy of long-read transcript quantification as done for GM12878 ONT Iso-seq replicates in the TAGET paper.

## When to use

Use when you have expression quantification from ≥2 sequencing replicates (or two quantification tools on the same samples) and need to verify reproducibility (e.g., replicate Pearson r > 0.971) or cross-tool consistency (e.g., r ≈ 0.83).

## When NOT to use

- Input is raw read data with no per-sample expression matrices yet — annotation/quantification (TransAnnot, TransAnnotMerge) must precede correlation analysis
- Only a single sample/replicate is available, so pairwise replicate correlations cannot be computed
- You need differential isoform usage analysis (expression_V1.py with -r 0.05 / -p 50 filters) rather than reproducibility assessment

## Inputs

- Full-length transcript FASTA files per replicate (e.g., GM12878 ONT Iso-seq replicates)
- hg38 reference genome (hg38.fa) and Ensembl GTF annotation (hg38.ensembl.gtf)
- Per-sample TAGET annotation outputs: {sample_id}.annot.stat, {sample_id}.annot.bed, {sample_id}.annot.db.pickle
- TPM-based expression matrices: transcript.exp and gene.exp per sample and merged multi-sample matrices
- Independent quantification of the same replicates (e.g., LIQA) for cross-tool correlation

## Outputs

- Pairwise Pearson correlation values of isoform and gene expression across replicates (expected >0.971)
- Cross-tool Pearson correlation between TAGET and LIQA quantification (expected ~0.83)
- Correlation summary table (gm12878_taget_correlations.csv)

## How to apply

After running TAGET's TransAnnot per replicate (alignment against hg38 with minimap2 and/or HISAT2 using the Ensembl GTF, ≥8 processes), compute TPM isoform expression with the --tpm parameter and extract expression from the transcript FASTA using fa2exp.py. Merge per-sample annot.stat, annot.bed, and annot.db.pickle files with TransAnnotMerge -m TPM to obtain per-sample {sample_id}.transcript.exp and {sample_id}.gene.exp plus multi-sample transcript.exp/gene.exp matrices. Then compute pairwise Pearson correlations of isoform and gene expression across the three replicates — TAGET on GM12878 ONT replicates yields r > 0.971 for all detected isoforms. Separately compute the Pearson correlation between TAGET-reported expression and an independent quantifier such as LIQA on the same replicates, expected around ~0.83 (Fig. 5g / Supplementary Table 4). Write all correlation values to a summary file (e.g., gm12878_taget_correlations.csv) and validate against the reported thresholds.

## Related tools

- **TAGET** (Produces per-replicate transcript annotation and TPM expression matrices (TransAnnot.py, fa2exp.py, TransAnnotMerge) that are the input to the correlation analysis) — https://github.com/XiDsLab/TAGET
- **minimap2** (Aligner option (via --use_minimap2) for full-length transcript alignment against hg38 before quantification)
- **HISAT2** (Aligner option (via --use_hisat2 with a hisat2 index) for full-length transcript alignment against hg38 before quantification)

## Examples

```
python fa2exp.py -f sample.fa -i sample_id -o sample.exp -p sample_out; python TranAnnotMerge -c MergeConfig -o outputdir -m TPM
```

## Evaluation signals

- All pairwise Pearson correlations of detected isoform expression across the three GM12878 replicates exceed 0.971
- TAGET-vs-LIQA Pearson correlation on the same replicates approximates 0.83, matching Fig. 5g / Supplementary Table 4
- Merged expression matrices contain {sample_id}.transcript.exp and {sample_id}.gene.exp per sample plus multi-sample gene.exp and transcript.exp
- Each per-replicate run produced the full annotation output set (annot.stat, annot.bed, annot.db.pickle) before merging
- Correlation values are recorded in gm12878_taget_correlations.csv for validation

## Limitations

- Cross-tool agreement is moderate (~0.83), not perfect, so exact concordance between TAGET and LIQA should not be expected
- Iso-seq CCS-read clustering filters ~80% of CCS reads (single-read clusters), many from true expressed isoforms, limiting gene detection and thus expression coverage
- Low-expression filtering defaults apply in DIU contexts (-r 0.05 transcripts, -p 50 genes) and should be kept in mind when interpreting correlation of low-TPM isoforms

## Evidence

- [other] TAGET's quantification on ONT GM12878 replicates was highly reproducible (pairwise Pearson correlation of all detected isoforms >0.971) and consistent with LIQA (Pearson correlation ~0.83).: [evidence span withheld — anchor status: not_found]
- [other] Merge per-sample annotation files with TransAnnotMerge using -m TPM to generate expression files.: [evidence span withheld — anchor status: not_found]
- [other] validation: replicate pairwise correlations must exceed 0.971 and the TAGET-vs-LIQA correlation must approximate 0.83 as reported in Fig. 5g / Supplementary Table 4.: [evidence span withheld — anchor status: not_found]
- [readme] The reads\transcript\gene expression can be caculated by the parameter of --tpm: "The reads\transcript\gene expression can be caculated by the parameter of --tpm"
- [readme] `python TransAnnot.py -f [fasta] -g [genome fasta] -o [output directory] -a[annot gtf] -p [process] --use_minimap2 [1] --use_hisat2 [hisat2 index]`: "python TransAnnot.py -f [fasta] -g [genome fasta] -o [output directory] -a[annot gtf] -p [process] --use_minimap2 [1]"
