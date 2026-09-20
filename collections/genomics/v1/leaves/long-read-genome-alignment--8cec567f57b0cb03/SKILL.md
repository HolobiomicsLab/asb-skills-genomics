---
name: long-read-genome-alignment
description: Use when raw nanopore long-read FASTQ/BAM data must be aligned to a reference
  genome before variant calling, and the analysis goal is SV/CNV detection where aligner
  choice affects call sets.
when_to_use_negative:
- Input is short-read (Illumina) sequencing data — this skill is specific to long
  nanopore reads and long-read aligners (lra, minimap2, NGMLR)
- The task is coverage-based filtering of already-called SVs (deletions/duplications)
  — use disCoverage instead, which needs putative SV positions as input and is not
  an aligner or caller
- Reference build choice is already fixed and only one aligner is required — running
  the three-aligner comparison adds cost without benefit when the pipeline combination
  has already been decided
edam_operation: http://edamontology.org/operation_2928
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_0102
tools:
- name: lra
  role: Long-read aligner producing one of the three per-aligner BAMs used for SV
    calling comparison
- name: minimap2
  role: Long-read aligner producing one of the three per-aligner BAMs used for SV
    calling comparison
- name: NGMLR
  role: Long-read aligner producing one of the three per-aligner BAMs; its combination
    with Sniffles gave the largest consensus with aCGH-detected SVs
- name: nanopore_pipeline_21
  role: SLURM-based pipeline whose nanopore_slurm_pipeline.py handles alignment (and
    variant calling) driven by config files
  repo: https://github.com/javiercguard/nanopore_pipeline_21
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1016/j.jare.2022.10.012
    title: Improvement of large copy number variant detection by whole genome nanopore
      sequencing
  source_content_hash: sha256:9890329e52eb095a14f7c3ed05f82cb11574dcb2074bc8b833ea94851765abfa
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/javiercguard/disCoverage
  repo_url: https://github.com/javiercguard/nanopore_pipeline_21
  tools:
  - CuteSV
  - NGMLR
  - NanoVar
  - SVIM
  - Sniffles
  - disCoverage
  - lra
  - minimap2
  - nanopore_pipeline_21
  edam_operation: http://edamontology.org/operation_2928
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_0102
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9411764705882353
n_claims: 17
---

# long-read-genome-alignment

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Aligns whole-genome nanopore (PromethION) reads against a human reference genome (hg38) with multiple long-read aligners — lra, minimap2, and NGMLR — producing one BAM per aligner as the basis for downstream SV calling. Aligner choice and reference build directly determine the sensitivity of large (>50 kb) CNV detection.

## When to use

Use when raw nanopore long-read FASTQ/BAM data must be aligned to a reference genome before variant calling, and the analysis goal is SV/CNV detection where aligner choice affects call sets.

## When NOT to use

- Input is short-read (Illumina) sequencing data — this skill is specific to long nanopore reads and long-read aligners (lra, minimap2, NGMLR)
- The task is coverage-based filtering of already-called SVs (deletions/duplications) — use disCoverage instead, which needs putative SV positions as input and is not an aligner or caller
- Reference build choice is already fixed and only one aligner is required — running the three-aligner comparison adds cost without benefit when the pipeline combination has already been decided

## Inputs

- Whole-genome nanopore (PromethION) read set (e.g., NA19240 control sample)
- Human reference genome in hg38 (or hg19) FASTA format
- Edited pipeline configuration file (config/config.sh)

## Outputs

- One coordinate-sorted BAM (plus index) per aligner: lra, minimap2, NGMLR
- Per-aligner alignments suitable as input to SV variant callers (Sniffles, CuteSV, SVIM, NanoVar)

## How to apply

Align the nanopore reads against the chosen reference build (hg38 was concluded better for SV calling in this study) separately with each of the three long-read aligners — lra, minimap2, and NGMLR — yielding one BAM per aligner so that aligner-specific call sets can be compared. Run the alignments via the paper's SLURM-based pipeline, editing config/config.sh to match the input data so that nanopore_slurm_pipeline.py handles alignment and variant calling. Keep per-aligner BAMs separate because the paper evaluated all downstream combinations (e.g., NGMLR + Sniffles gave the largest consensus of 31 SVs with aCGH, while only 13.10%/27.38%/36.90% of aCGH SVs were recovered with lra/minimap2/NGMLR). Decide deliberately between hg19 and hg38: coordinate discrepancies were more evident with hg38, and 8 SVs with untranslatable coordinates were impossible to find in hg38. Feed each per-aligner BAM into variant callers (Sniffles, CuteSV, SVIM, NanoVar where compatible) rather than merging alignments.

## Related tools

- **lra** (Long-read aligner producing one of the three per-aligner BAMs used for SV calling comparison)
- **minimap2** (Long-read aligner producing one of the three per-aligner BAMs used for SV calling comparison)
- **NGMLR** (Long-read aligner producing one of the three per-aligner BAMs; its combination with Sniffles gave the largest consensus with aCGH-detected SVs)
- **nanopore_pipeline_21** (SLURM-based pipeline whose nanopore_slurm_pipeline.py handles alignment (and variant calling) driven by config files) — https://github.com/javiercguard/nanopore_pipeline_21

## Examples

```
python nanopore_slurm_pipeline.py  # alignment + variant calling driven by an edited config/config.sh
```

## Evaluation signals

- A BAM file exists for each of the three aligners (lra, minimap2, NGMLR) from the same input read set
- Alignments were made against the intended reference build (hg38), since reference version was a compared factor and hg38 was concluded better for SV calling
- Downstream steps succeed only if BAMs are valid: each should yield SV calls from the four nanopore callers and merge into one VCF per aligner via SURVIVOR
- Cross-aligner comparison shows detectable differences in SV call sets (e.g., 68.8/66.0/64.0% deletions for lra/minimap2/NGMLR against NA19240), confirming per-aligner outputs were kept separate
- Recall against external reference (e.g., ~13%–36% of aCGH SVs detected depending on aligner) is consistent with the paper's reported ranges

## Limitations

- NGMLR was slower and had some bugs at the time the manuscript was written
- No software versions were reported for any of the aligners or variant callers, so exact reproducibility of aligner behavior is not guaranteed
- Reference build choice matters: coordinate discrepancies were more evident with hg38, and SVs whose coordinates could not be translated (8 in the study) were impossible to find in hg38
- Aligner output compatibility limits downstream callers — NanoVar is applied only where compatible with the aligner output

## Evidence

- [readme] `nanopore_slurm_pipeline.py` handles alignment and variant calling. The .sh files handle the rest of the .slurm files in `slurmScripts/`.: [evidence span withheld — anchor status: external_doc]
