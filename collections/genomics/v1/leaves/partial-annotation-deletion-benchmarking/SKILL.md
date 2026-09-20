---
name: partial-annotation-deletion-benchmarking
description: Use when evaluating how robustly a long-read transcript annotation or
  reconstruction tool recovers known splice junctions when the supplied reference
  annotation is incomplete, i.e. sensitivity/precision as a function of annotation
  completeness.
when_to_use_negative:
- Input transcripts are short-read RNA-seq reads rather than full-length Iso-seq (PacBio
  HiFi) transcripts, since this benchmark is designed for Iso-seq full-length transcript
  FASTA
- The goal is de novo junction discovery with a complete, trusted reference annotation
  rather than stress-testing recovery under annotation incompleteness
- The organism lacks a curated GTF annotation, so genes cannot be systematically deleted
  from a reference
edam_operation: http://edamontology.org/operation_2928
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0080
- http://edamontology.org/topic_3574
tools:
- name: TAGET
  role: TransAnnot aligns the full-length transcript FASTA to each incomplete reference
    and predicts splice junctions ({sample_id}.annot.junction) for sensitivity/precision
    evaluation
  repo: github.com__XiDsLab__TAGET
- name: SQANTI
  role: Comparison tool that annotates long-reads independently and, like TAGET, achieves
    high junction-recovery sensitivity on incomplete references
- name: StringTie2
  role: Transcript reconstruction baseline benchmarked for junction sensitivity/precision
    against incomplete references
- name: FLAIR
  role: Transcript reconstruction baseline benchmarked for junction sensitivity/precision
    against incomplete references
- name: HISAT2
  role: Aligner used within TransAnnot.Config with read length 100, read overlap 80,
    minimum read length 30; index built via hisat2-build
- name: MINIMAP2
  role: Long-read aligner used alongside HISAT2 in TransAnnot for each incomplete
    reference alignment
- name: samtools
  role: Dependency for handling alignments in the TAGET pipeline
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-023-41649-0
    title: 'TAGET: a toolkit for analyzing full-length transcripts from long-read
      sequencing'
  source_content_hash: sha256:85c3137599711bcd13d45931d4fb35e05b4d8efbd34e5a23eaaab6ac231dc019
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - FLAIR
  - HISAT2
  - MINIMAP2
  - SQANTI
  - StringTie2
  - TAGET
  - samtools
  edam_operation: http://edamontology.org/operation_2928
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0080
  - http://edamontology.org/topic_3574
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# partial-annotation-deletion-benchmarking

## Summary

Benchmark splice-junction recovery sensitivity and precision of transcript annotation tools (TAGET vs SQANTI, IsoQuant, StringTie2, FLAIR) by systematically deleting 5-50% of genes from the hg38.ensembl.gtf reference and measuring how much of the removed annotation each tool recovers from Iso-seq data.

## When to use

Use when evaluating how robustly a long-read transcript annotation or reconstruction tool recovers known splice junctions when the supplied reference annotation is incomplete, i.e. sensitivity/precision as a function of annotation completeness.

## When NOT to use

- Input transcripts are short-read RNA-seq reads rather than full-length Iso-seq (PacBio HiFi) transcripts, since this benchmark is designed for Iso-seq full-length transcript FASTA
- The goal is de novo junction discovery with a complete, trusted reference annotation rather than stress-testing recovery under annotation incompleteness
- The organism lacks a curated GTF annotation, so genes cannot be systematically deleted from a reference

## Inputs

- hg38 reference genome FASTA (hg38.fa) from the PKU disk link
- hg38.ensembl.gtf Ensembl annotation (GTF format)
- public Iso-seq full-length transcript FASTA
- incomplete reference GTFs with 5%, 10%, 20%, 30%, and 50% of genes deleted

## Outputs

- {sample_id}.annot.junction splice-junction file per tool and deletion level
- junction_sensitivity_precision.csv with one sensitivity and one precision value per tool per deletion level (5, 10, 20, 30, 50)

## How to apply

Download hg38.fa from the PKU disk link and hg38.ensembl.gtf, then build the annotation pickle database with gtf_db_make.py. Generate five incomplete reference GTFs by deleting 5%, 10%, 20%, 30%, and 50% of genes. Align the public Iso-seq full-length transcript FASTA to each incomplete reference with minimap2 and HISAT2 (index built with hisat2-build; READ_LENGTH 100, READ_OVERLAP 80, MIN_READ_LENGTH 30 as configured in TransAnnot.Config). Run TAGET's TransAnnot on each alignment, and run SQANTI, StringTie2, and FLAIR against the same incomplete references. Extract splice junctions from each tool's output (e.g. {sample_id}.annot.junction for TAGET), compare against the deleted genes' junctions, and compute sensitivity (fraction of deleted junctions recovered) and precision (fraction of reported novel junctions that are true) per deletion level. The expected pattern, reported in Supplementary Fig 5, is that TAGET and SQANTI achieve much higher junction-recovery sensitivities with slightly lower precisions than StringTie2 and FLAIR.

## Related tools

- **TAGET** (TransAnnot aligns the full-length transcript FASTA to each incomplete reference and predicts splice junctions ({sample_id}.annot.junction) for sensitivity/precision evaluation) — github.com__XiDsLab__TAGET
- **SQANTI** (Comparison tool that annotates long-reads independently and, like TAGET, achieves high junction-recovery sensitivity on incomplete references)
- **StringTie2** (Transcript reconstruction baseline benchmarked for junction sensitivity/precision against incomplete references)
- **FLAIR** (Transcript reconstruction baseline benchmarked for junction sensitivity/precision against incomplete references)
- **HISAT2** (Aligner used within TransAnnot.Config with read length 100, read overlap 80, minimum read length 30; index built via hisat2-build)
- **MINIMAP2** (Long-read aligner used alongside HISAT2 in TransAnnot for each incomplete reference alignment)
- **samtools** (Dependency for handling alignments in the TAGET pipeline)

## Examples

```
python TransAnnot.py -f iso_seq.fa -g hg38.fa -o out_del5 -a hg38.ensembl.del5.gtf -p 8 --use_minimap2 1 --use_hisat2 hg38_index
```

## Evaluation signals

- junction_sensitivity_precision.csv contains exactly one sensitivity and one precision value per tool per deletion level (5, 10, 20, 30, 50)
- TAGET and SQANTI sensitivity values exceed those of StringTie2 and FLAIR at the deletion levels reported in Supplementary Fig 5
- Sensitivity generally decreases as the fraction of deleted genes increases from 5% to 50%
- Each incomplete GTF contains the correct proportion of removed genes (5%, 10%, 20%, 30%, 50%) before alignment
- TAGET junction extraction uses the expected {sample_id}.annot.junction output file produced by TransAnnot for every deletion level

## Limitations

- Higher junction-recovery sensitivity of TAGET and SQANTI comes with slightly lower precision than StringTie2 and FLAIR
- The benchmark depends on the choice of aligners and HISAT2 parameters (read length 100, read overlap 80, minimum read length 30) configured in TransAnnot.Config
- Full TransAnnot runs take up to ~1 hour with 8 cores on a Linux server per configuration, and five deletion levels multiplied by four comparison tools must all complete
- TAGET's Iso-seq CCS-read filtering step can discard ~80% of CCS reads (clusters with a single read), which limits detected genes and could affect sensitivity estimates

## Evidence

- [other] finding: deletion simulation sensitivity pattern: [evidence span withheld — anchor status: not_found]
- [readme] README: reference genome and gtf database: "Here we used hg38 to annotate transcripts,human reference hg38.fa and Ensemble gtf format files were needed. HISAT2 and minimap2 need to index this reference."
- [readme] README: pickle database generation: "python gtf_db_make.py hg38.ensembl.gtf hg38.ensembl.v20200306.1.pickle"
- [readme] README: junction output file: "* [{sample_id}.annot.junction]()  the information of splice junction file"
- [readme] README: HISAT2 parameters: "* `READ_LENGTH`: `[int]`，the read length used by HISAT2，default 100"
- [discussion] article: TAGET and SQANTI analyze long-reads independently: "TAGET and SQANTI analyze long-reads independently and generally have higher sensitivity."
