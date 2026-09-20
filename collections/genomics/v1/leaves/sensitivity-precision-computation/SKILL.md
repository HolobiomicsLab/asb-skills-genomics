---
name: sensitivity-precision-computation
description: Use when evaluating how well transcript annotation/junction-prediction
  tools recover true splice junctions under incomplete references, needing per-tool
  sensitivity and precision values for comparison.
when_to_use_negative:
- Input reads are short-read RNA-seq rather than PacBio Iso-seq full-length transcripts;
  the benchmark and TransAnnot alignment settings target long-read Iso-seq data
- The goal is full transcript reconstruction or quantification (e.g., TPM expression
  matrices via TransAnnotMerge) rather than junction-recovery benchmarking
- A complete, unmodified reference annotation is used — sensitivity against deleted
  junctions requires partially deleted reference GTFs
edam_operation: http://edamontology.org/operation_3562
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0080
tools:
- name: TAGET
  role: Annotates Iso-seq transcripts against each incomplete reference and reports
    splice junctions in {sample_id}.annot.junction for sensitivity/precision computation
  repo: https://github.com/XiDsLab/TAGET
- name: SQANTI
  role: Independent long-read annotator benchmarked against TAGET for junction-recovery
    sensitivity and precision
- name: StringTie2
  role: Transcript-reconstruction baseline benchmarked for junction recovery
- name: FLAIR
  role: Transcript-reconstruction baseline benchmarked for junction recovery
- name: HISAT2
  role: Aligns full-length transcript reads to each incomplete reference using an
    index built with hisat2-build (read length 100, overlap 80, min read length 30)
- name: MINIMAP2
  role: Alternative aligner for mapping Iso-seq FASTA reads to each incomplete reference
- name: samtools
  role: Alignment file processing within the TAGET TransAnnot workflow
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-023-41649-0
    title: 'TAGET: a toolkit for analyzing full-length transcripts from long-read
      sequencing'
  source_content_hash: sha256:baea2c1f86b34fe8744f5483fac42adfc17da3dca8b9e50ee0abea1b65460313
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/XiDsLab/TAGET
  tools:
  - FLAIR
  - HISAT2
  - MINIMAP2
  - SQANTI
  - StringTie2
  - TAGET
  - samtools
  edam_operation: http://edamontology.org/operation_3562
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0080
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6363636363636364
n_claims: 11
---

# sensitivity-precision-computation

## Summary

Benchmark splice-junction recovery of transcript annotation tools by computing junction sensitivity (fraction of deleted reference junctions recovered) and precision (fraction of reported novel junctions that are true) across partial annotation-deletion simulations, as done for TAGET vs SQANTI, IsoQuant, StringTie2, and FLAIR.

## When to use

Use when evaluating how well transcript annotation/junction-prediction tools recover true splice junctions under incomplete references, needing per-tool sensitivity and precision values for comparison.

## When NOT to use

- Input reads are short-read RNA-seq rather than PacBio Iso-seq full-length transcripts; the benchmark and TransAnnot alignment settings target long-read Iso-seq data
- The goal is full transcript reconstruction or quantification (e.g., TPM expression matrices via TransAnnotMerge) rather than junction-recovery benchmarking
- A complete, unmodified reference annotation is used — sensitivity against deleted junctions requires partially deleted reference GTFs

## Inputs

- hg38 reference genome FASTA (hg38.fa) and hg38.ensembl.gtf annotation
- Incomplete reference GTFs with 5%, 10%, 20%, 30%, and 50% of genes deleted
- PacBio Iso-seq full-length transcript FASTA (public Iso-seq dataset)
- Per-tool splice-junction outputs (e.g., TAGET {sample_id}.annot.junction, plus SQANTI/StringTie2/FLAIR predictions)

## Outputs

- junction_sensitivity_precision.csv with one sensitivity and one precision value per tool (TAGET, SQANTI, StringTie2, FLAIR) per deletion level (5, 10, 20, 30, 50)
- Per-tool transcript/junction predictions per deletion level

## How to apply

Generate incomplete reference GTFs by deleting 5%, 10%, 20%, 30%, and 50% of genes from hg38.ensembl.gtf. Align the public Iso-seq full-length transcript FASTA to each incomplete reference with minimap2 and HISAT2 (read length 100, read overlap 80, minimum read length 30, index built with hisat2-build as configured in TransAnnot.Config), then run TAGET's TransAnnot plus SQANTI, StringTie2, and FLAIR against the same incomplete references. Extract detected splice junctions per tool (e.g., TAGET's {sample_id}.annot.junction output) and compare them against the removed genes' junctions to compute, per tool and deletion level, sensitivity as the fraction of deleted junctions recovered and precision as the fraction of reported novel junctions that are true. Tabulate the results by tool and deletion level and check for the expected pattern: TAGET and SQANTI, which analyze long reads independently, attain higher sensitivity (with slightly lower precision) than reconstruction-based StringTie2 and FLAIR, as reported in Supplementary Fig. 5.

## Related tools

- **TAGET** (Annotates Iso-seq transcripts against each incomplete reference and reports splice junctions in {sample_id}.annot.junction for sensitivity/precision computation) — https://github.com/XiDsLab/TAGET
- **SQANTI** (Independent long-read annotator benchmarked against TAGET for junction-recovery sensitivity and precision)
- **StringTie2** (Transcript-reconstruction baseline benchmarked for junction recovery)
- **FLAIR** (Transcript-reconstruction baseline benchmarked for junction recovery)
- **HISAT2** (Aligns full-length transcript reads to each incomplete reference using an index built with hisat2-build (read length 100, overlap 80, min read length 30))
- **MINIMAP2** (Alternative aligner for mapping Iso-seq FASTA reads to each incomplete reference)
- **samtools** (Alignment file processing within the TAGET TransAnnot workflow)

## Examples

```
python TransAnnot.py -c TransAnnot.Config
```

## Evaluation signals

- junction_sensitivity_precision.csv contains exactly one sensitivity and one precision value per tool per deletion level (5, 10, 20, 30, 50)
- TAGET and SQANTI sensitivity values exceed those of StringTie2 and FLAIR at the deletion levels reported in Supplementary Fig. 5
- TAGET and SQANTI show slightly lower precision than StringTie2 and FLAIR, consistent with higher-sensitivity/lower-precision behavior for independent long-read analysis
- Each tool produced junction predictions for every deletion level, extracted from the correct output files (e.g., {sample_id}.annot.junction for TAGET)

## Limitations

- Higher sensitivity of independent long-read annotators (TAGET, SQANTI) comes with slightly lower precision than reconstruction-based tools
- Benchmark depends on which genes are randomly deleted (5–50% of genes); sensitivity is defined only against the removed genes' junctions
- Performance claims rely on supplementary figures (e.g., Supplementary Fig. 5) not present in the main text

## Evidence

- [results] In the deletion simulation (X% of genes removed, X = 5-50), TAGET and SQANTI achieved much higher junction-recovery sensitivities and slightly lower precisions than StringTie2 and FLAIR (Supplementary Fig 5).: [evidence span withheld — anchor status: not_found]
- [discussion] TAGET and SQANTI analyze long-reads independently and generally have higher sensitivity.: "TAGET and SQANTI analyze long-reads independently and generally have higher sensitivity."
- [readme] python gtf_db_make.py hg38.ensembl.gtf hg38.ensembl.v20200306.1.pickle: "python gtf_db_make.py hg38.ensembl.gtf hg38.ensembl.v20200306.1.pickle"
- [readme] {sample_id}.annot.junction: the information of splice junction file: "{sample_id}.annot.junction: the information of splice junction file"
- [readme] READ_LENGTH: [int], the read length used by HISAT2, default 100: "READ_LENGTH: [int],the read length used by HISAT2，default 100"
