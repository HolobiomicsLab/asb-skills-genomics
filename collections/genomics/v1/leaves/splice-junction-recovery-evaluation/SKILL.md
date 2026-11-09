---
name: splice-junction-recovery-evaluation
description: Use when benchmarking long-read Iso-seq annotation tools for splice-junction
  recovery under partial-reference-annotation conditions, or when quantifying sensitivity/precision
  of novel junction prediction against an incomplete GTF.
when_to_use_negative:
- The reference annotation is complete and the goal is de novo transcript reconstruction
  rather than junction-recovery benchmarking against partially deleted references
- Input data is short-read RNA-seq rather than PacBio Iso-seq full-length transcript
  FASTA, since TAGET's TransAnnot is designed for Iso-seq full-length reads
edam_operation: http://edamontology.org/operation_2928
edam_topics:
- http://edamontology.org/topic_0080
- http://edamontology.org/topic_3170
tools:
- name: TAGET
  role: Runs TransAnnot to align and annotate Iso-seq reads against incomplete references
    and emits splice-junction predictions ({sample_id}.annot.junction)
  repo: https://github.com/XiDsLab/TAGET
- name: SQANTI
  role: Baseline annotation tool producing transcript/junction predictions compared
    for sensitivity and precision
- name: StringTie2
  role: Transcript-reconstruction baseline for junction recovery benchmarking
- name: FLAIR
  role: Transcript-reconstruction baseline for junction recovery benchmarking
- name: IsoQuant
  role: Transcript-reconstruction baseline named in the comparison
- name: HISAT2
  role: Aligns Iso-seq transcripts to incomplete references (read length 100, overlap
    80, min read length 30); index built with hisat2-build
- name: minimap2
  role: Alternative aligner for Iso-seq transcripts against each incomplete reference
- name: samtools
  role: Alignment file handling within the TAGET TransAnnot pipeline
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-023-41649-0
    title: 'TAGET: a toolkit for analyzing full-length transcripts from long-read
      sequencing'
  source_content_hash: sha256:124699ddd0b0955ce07a2f25589ec0fc6b478fc443c5413076dff65e6feb87d5
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/XiDsLab/TAGET
  tools:
  - FLAIR
  - HISAT2
  - IsoQuant
  - SQANTI
  - StringTie2
  - TAGET
  - minimap2
  - samtools
  edam_operation: http://edamontology.org/operation_2928
  edam_topics:
  - http://edamontology.org/topic_0080
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 14
---

# splice-junction-recovery-evaluation

## Summary

A benchmarking skill that measures how well transcript-annotation tools (TAGET, SQANTI, StringTie2, FLAIR, IsoQuant) recover splice junctions when a known fraction (5–50%) of genes is deleted from the hg38 Ensembl reference, computing junction-recovery sensitivity and precision per tool per deletion level as in TAGET's Supplementary Fig. 5.

## When to use

Use when benchmarking long-read Iso-seq annotation tools for splice-junction recovery under partial-reference-annotation conditions, or when quantifying sensitivity/precision of novel junction prediction against an incomplete GTF.

## When NOT to use

- The reference annotation is complete and the goal is de novo transcript reconstruction rather than junction-recovery benchmarking against partially deleted references
- Input data is short-read RNA-seq rather than PacBio Iso-seq full-length transcript FASTA, since TAGET's TransAnnot is designed for Iso-seq full-length reads

## Inputs

- hg38 reference genome FASTA (hg38.fa, from the PKU disk link)
- hg38.ensembl.gtf Ensembl annotation
- Public Iso-seq full-length transcript FASTA
- Partially deleted reference GTFs (5%, 10%, 20%, 30%, 50% of genes removed)
- HISAT2 genome index (hisat2-build)

## Outputs

- Per-tool, per-deletion-level junction predictions (e.g., {sample_id}.annot.junction for TAGET)
- junction_sensitivity_precision.csv with one sensitivity and one precision value per tool per deletion level (5, 10, 20, 30, 50)

## How to apply

Build the annotation pickle database from hg38.ensembl.gtf with gtf_db_make.py, then generate incomplete reference GTFs by deleting 5%, 10%, 20%, 30%, and 50% of genes. Align the public Iso-seq full-length transcript FASTA to each incomplete reference using minimap2 and HISAT2 (index via hisat2-build; TransAnnot.Config read length 100, read overlap 80, minimum read length 30), then run TAGET's TransAnnot per alignment while running SQANTI, StringTie2, and FLAIR against the same incomplete references. Extract detected splice junctions from each tool's output (for TAGET, the {sample_id}.annot.junction file) and compare against the junctions of the removed genes. Compute sensitivity (fraction of deleted junctions recovered) and precision (fraction of reported novel junctions that are true) per tool per deletion level. The rationale, per the article, is that TAGET and SQANTI analyze long reads independently and achieve higher junction-recovery sensitivity (with slightly lower precision) than transcript-reconstruction tools like StringTie2 and FLAIR.

## Related tools

- **TAGET** (Runs TransAnnot to align and annotate Iso-seq reads against incomplete references and emits splice-junction predictions ({sample_id}.annot.junction)) — https://github.com/XiDsLab/TAGET
- **SQANTI** (Baseline annotation tool producing transcript/junction predictions compared for sensitivity and precision)
- **StringTie2** (Transcript-reconstruction baseline for junction recovery benchmarking)
- **FLAIR** (Transcript-reconstruction baseline for junction recovery benchmarking)
- **IsoQuant** (Transcript-reconstruction baseline named in the comparison)
- **HISAT2** (Aligns Iso-seq transcripts to incomplete references (read length 100, overlap 80, min read length 30); index built with hisat2-build)
- **minimap2** (Alternative aligner for Iso-seq transcripts against each incomplete reference)
- **samtools** (Alignment file handling within the TAGET TransAnnot pipeline)

## Examples

```
python TransAnnot.py -f isoseq.fa -g hg38.fa -o out_dir -a hg38.ensembl.del20.gtf -p 8 --use_minimap2 1 --use_hisat2 hg38_index
```

## Evaluation signals

- junction_sensitivity_precision.csv contains exactly one sensitivity and one precision value per tool per deletion level (5, 10, 20, 30, 50)
- TAGET and SQANTI sensitivity values exceed those of StringTie2 and FLAIR at the deletion levels reported in Supplementary Fig. 5
- TAGET and SQANTI show slightly lower precision than StringTie2 and FLAIR, matching the reported pattern
- Each incomplete reference GTF contains the expected fraction of deleted genes (5%, 10%, 20%, 30%, 50%) relative to hg38.ensembl.gtf
- Each TAGET run produces the expected outputs, including {sample_id}.annot.junction and {sample_id}.annot.stat

## Limitations

- The deletion simulation covers only 5–50% gene removal; results may not extrapolate to complete-annotation or fully de novo settings
- Alignment-tool parameterization (HISAT2 read length 100, overlap 80, min read length 30) is fixed by TransAnnot.Config and may affect cross-tool comparability
- TAGET requires Linux, python3, R>=3.3, and specific Python (rpy2, pandas, numpy) and R (stringr, optparse, DEGseq) package dependencies

## Evidence

- [results] In the deletion simulation (X% of genes removed, X = 5-50), TAGET and SQANTI achieved much higher junction-recovery sensitivities and slightly lower precisions than StringTie2 and FLAIR: [evidence span withheld — anchor status: not_found]
- [discussion] TAGET vs transcript-reconstruction tools rationale: "TAGET and SQANTI analyze long-reads independently and generally have higher sensitivity."
- [discussion] recently developed computational tools, such as ESPRESSO39, FLAIR, StringTie2, and IsoQuant, use transcript reconstruction to improve transcript analyses: "recently developed computational tools, such as ESPRESSO39, FLAIR, StringTie2, and IsoQuant, use transcript reconstruction to improve transcript analyses"
- [readme] python gtf_db_make.py hg38.ensembl.gtf hg38.ensembl.v20200306.1.pickle: "python gtf_db_make.py hg38.ensembl.gtf hg38.ensembl.v20200306.1.pickle"
- [readme] the information of splice junction file: "[{sample_id}.annot.junction]()  the information of splice junction file"
