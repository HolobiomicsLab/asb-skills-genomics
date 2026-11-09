---
name: splice-junction-support-validation
description: Use when you have long-read Iso-seq splice junction predictions from
  multiple aligners plus matched short-read RNA-seq data and need to quantify what
  fraction of predicted junctions (especially novel ones) are independently supported.
when_to_use_negative:
- You only have short-read RNA-seq and no long-read Iso-seq junction predictions to
  validate
- The goal is transcript quantification or DIU analysis rather than junction precision
  benchmarking (use TAGET's TransAnnotMerge/expression_V1.py workflow instead)
- No matched short-read data or reference annotation exists, so support cannot be
  assessed
edam_operation: http://edamontology.org/operation_3563
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0102
tools:
- name: TAGET
  role: Produces the Iso-seq splice junction predictions ({sample_id}.annot.junction)
    being validated; its alignment component generates junctions from full-length
    transcript FASTA
  repo: https://github.com/XiDsLab/TAGET
- name: HISAT2
  role: Aligns matched short-read RNA-seq reads to hg38 to provide spanning-read support
    evidence for each junction
  repo: https://github.com/XiDsLab/TAGET
- name: minimap2
  role: Baseline long-read aligner whose predicted junctions are benchmarked against
    TAGET in the support-rate comparison
- name: GMAP
  role: Baseline long-read aligner whose predicted junctions are benchmarked against
    TAGET in the support-rate comparison
- name: samtools
  role: Handles short-read alignment file processing when intersecting junctions with
    short-read alignments
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-023-41649-0
    title: 'TAGET: a toolkit for analyzing full-length transcripts from long-read
      sequencing'
  source_content_hash: sha256:b82443d95a0066adfc8388775cc1954112aeae11102cf7530e3033277e8114fc
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/XiDsLab/TAGET
  tools:
  - GMAP
  - HISAT2
  - TAGET
  - minimap2
  - samtools
  edam_operation: http://edamontology.org/operation_3563
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0102
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8181818181818182
n_claims: 11
---

# splice-junction-support-validation

## Summary

Validates splice junctions predicted from long-read Iso-seq alignments (TAGET, minimap2, GMAP) by counting how many matched short-read RNA-seq reads span each junction, computing overall and novel-only support fractions against a reference annotation. This benchmarks the precision of novel junction discovery across aligners.

## When to use

Use when you have long-read Iso-seq splice junction predictions from multiple aligners plus matched short-read RNA-seq data and need to quantify what fraction of predicted junctions (especially novel ones) are independently supported.

## When NOT to use

- You only have short-read RNA-seq and no long-read Iso-seq junction predictions to validate
- The goal is transcript quantification or DIU analysis rather than junction precision benchmarking (use TAGET's TransAnnotMerge/expression_V1.py workflow instead)
- No matched short-read data or reference annotation exists, so support cannot be assessed

## Inputs

- Per-aligner long-read splice junction files ({sample_id}.annot.junction) from TAGET, minimap2, and GMAP Iso-seq alignments
- Matched short-read RNA-seq FASTQ files
- hg38 reference genome FASTA (hg38.fa) and HISAT2 index built with hisat2-build
- Ensembl GTF annotation (hg38.ensembl.gtf) or its pickle form for known/novel junction classification

## Outputs

- Per-aligner junction support fractions (overall and novel-only) assembled into junction_support_fractions.csv
- Per-junction short-read spanning counts and known/novel classifications

## How to apply

Collect the per-aligner splice junction files ({sample_id}.annot.junction) produced by TAGET/HISAT2, minimap2, and GMAP from the long-read Iso-seq alignments. Align the matched short-read RNA-seq data to hg38 with HISAT2 using the prebuilt hisat2-build index, keeping the HISAT2 read-length default of 100 and read-overlap default of 80. Intersect each long-read junction against the short-read alignments to count spanning short reads, and classify each junction as known or novel using the Ensembl GTF (e.g. hg38.ensembl.gtf). For each aligner, compute the fraction of junctions supported by at least two short reads or present in the reference annotation, both over all junctions and restricted to novel junctions. Expect TAGET to outperform existing aligners on novel-junction precision (99.3%/75.2% vs 99.1%/48.5% for minimap2 and 99.1%/43.8% for GMAP), consistent with the paper's observation that many novel junctions from current aligners are non-canonical false positives.

## Related tools

- **TAGET** (Produces the Iso-seq splice junction predictions ({sample_id}.annot.junction) being validated; its alignment component generates junctions from full-length transcript FASTA) — https://github.com/XiDsLab/TAGET
- **HISAT2** (Aligns matched short-read RNA-seq reads to hg38 to provide spanning-read support evidence for each junction) — https://github.com/XiDsLab/TAGET
- **minimap2** (Baseline long-read aligner whose predicted junctions are benchmarked against TAGET in the support-rate comparison)
- **GMAP** (Baseline long-read aligner whose predicted junctions are benchmarked against TAGET in the support-rate comparison)
- **samtools** (Handles short-read alignment file processing when intersecting junctions with short-read alignments)

## Examples

```
python TransAnnot.py -f [fasta] -g hg38.fa -o [outdir] -a hg38.ensembl.gtf -p 8 --use_minimap2 1 --use_hisat2 [hisat2 index]
```

## Evaluation signals

- junction_support_fractions.csv contains one row per aligner with both overall and novel-only support fractions
- TAGET novel-junction support fraction (~75.2%) exceeds minimap2 (~48.5%) and GMAP (~43.8%), while overall support rates are all ~99%
- Each junction's support status is defined as at least two spanning short reads OR presence in the reference annotation
- Known/novel junction classification is consistent with the Ensembl GTF used for annotation
- All per-aligner support fractions fall in the valid [0, 1] range and are internally consistent with the per-junction support counts

## Limitations

- Over 40% of novel junctions predicted by current alignment tools are non-canonical, indicating many novel junctions are false positives — low novel-support fractions reflect this
- Support depends on availability of matched short-read RNA-seq data and a complete reference annotation; junctions absent from both may be true but unsupported
- HISAT2 short-read alignment uses fixed defaults (read length 100, read overlap 80) that may not suit all short-read library types

## Evidence

- [other] Junction support validation shows TAGET outperforms existing aligners: 99.3% of all junctions and 75.2% of novel junctions it predicts are supported: [evidence span withheld — anchor status: not_found]
- [intro] We observe that over 40% of the predicted novel junctions by the current alignment tools are non-canonical: "We observe that over 40% of the predicted novel junctions by the current alignment tools are non-canonical, indicating that many novel junctions are false positives."
- [readme] READ_LENGTH: [int]，the read length used by HISAT2，default 100: "* `READ_LENGTH`: `[int]`，the read length used by HISAT2，default 100
* `READ_OVERLAP`: `[int]` the read overlap used by HISAT2， default 80"
- [abstract] TAGET gives significantly more precise novel splice site prediction: "TAGET gives significantly more precise novel splice site prediction and enables more accurate novel isoform and gene fusion discoveries"
- [readme] HISAT2 and minimap2 need to index this reference: "Here we used hg38 to annotate transcripts,human reference hg38.fa and Ensemble gtf format files were needed. HISAT2 and minimap2 need to index this reference."
