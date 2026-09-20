---
name: rebase-motif-reference-comparison
description: Use when you have de novo enriched methylation motifs (from Nanoident_enrich.sh
  or nanodisco automatic mode) for bacteria with known REBASE motif references and
  need to quantify motif recovery and compare methods.
when_to_use_negative:
- When no REBASE reference motifs exist for the strain, so agreement cannot be scored
- When inputs are already typed/mapped motifs rather than de novo enriched motif sets
  needing reference matching
edam_operation: http://edamontology.org/operation_0306
edam_topics:
- http://edamontology.org/topic_3173
- http://edamontology.org/topic_3168
tools:
- name: Nanoident
  role: Provides the optimized de novo methylation motif enrichment module (Nanoident_enrich.sh)
    whose motifs are matched against REBASE references
  repo: https://github.com/cz-csu/Nanoident
- name: nanodisco
  role: 'Baseline method: automatic motif discovery mode run on the same preprocessed
    data for comparison'
  repo: https://github.com/fanglab/nanodisco
- name: REBASE database
  role: Source of reference methylation motifs used to score recovery by each method
- name: R
  role: Environment for data preprocessing and motif enrichment steps producing the
    motif lists
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/bioinformatics/btaf397
    title: Identifying DNA methylation types and methylated base positions from bacteria
      using nanopore sequencing with multi-scale neural network
  source_content_hash: sha256:6fec40b85d5b589220839180edd8411626ed4e407f5262be5497d5fb6f6383b6
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/cz-csu/Nanoident
  tools:
  - Nanoident
  - R
  - REBASE database
  - Singularity
  - bam-readcount
  - nanodisco
  edam_operation: http://edamontology.org/operation_0306
  edam_topics:
  - http://edamontology.org/topic_3173
  - http://edamontology.org/topic_3168
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4444444444444444
n_claims: 9
---

# rebase-motif-reference-comparison

## Summary

Matches de novo discovered methylation motifs from Nanoident's optimized enrichment module against REBASE reference motifs for each bacterium, providing a ground-truth-based comparison with nanodisco's automatic discovery mode.

## When to use

Use when you have de novo enriched methylation motifs (from Nanoident_enrich.sh or nanodisco automatic mode) for bacteria with known REBASE motif references and need to quantify motif recovery and compare methods.

## When NOT to use

- When no REBASE reference motifs exist for the strain, so agreement cannot be scored
- When inputs are already typed/mapped motifs rather than de novo enriched motif sets needing reference matching

## Inputs

- De novo enriched methylation motif lists from Nanoident_enrich.sh (motif-enrichment output)
- De novo motif lists from nanodisco automatic discovery mode on the same preprocessed data
- REBASE reference methylation motifs for each strain
- Strain difference RDS files (WGA minus native nanopore signal features) from Nanoident_preprocess.sh
- Strain reference FASTA files (e.g. NCBI accessions CP041693, CP041696, NC_008261.1)

## Outputs

- motif_enrichment_comparison.csv with per-strain motif counts (Nanoident vs nanodisco: 16 vs 12, 8 vs 1, 8 vs 8)
- Per-strain lists of discovered motifs matched to REBASE reference motifs with agreement recorded

## How to apply

Run Nanoident_enrich.sh per strain (bacterial name, strain's difference RDS file, motif-enrichment output path, strain's reference FASTA) to obtain Nanoident's de novo motifs, and run nanodisco's automatic motif discovery mode on the same preprocessed WGA/native difference data as the baseline. Count the de novo enriched methylation motifs discovered per strain by each method, then match discovered motifs against the REBASE reference motifs for each strain and record agreement. Write per-strain counts (Nanoident vs nanodisco) into motif_enrichment_comparison.csv. Rationale: the optimized Nanoident enrichment pipeline was designed to recover more known motifs than nanodisco's automatic mode, e.g. 16 vs 12 on H. pylori JP26 against 20 REBASE reference motifs.

## Related tools

- **Nanoident** (Provides the optimized de novo methylation motif enrichment module (Nanoident_enrich.sh) whose motifs are matched against REBASE references) — https://github.com/cz-csu/Nanoident
- **nanodisco** (Baseline method: automatic motif discovery mode run on the same preprocessed data for comparison) — https://github.com/fanglab/nanodisco
- **REBASE database** (Source of reference methylation motifs used to score recovery by each method)
- **R** (Environment for data preprocessing and motif enrichment steps producing the motif lists)

## Examples

```
chmod +x ./process/code/Nanoident_enrich.sh && bash ./process/code/Nanoident_enrich.sh <BACTERIUM> <strain_diff.RDS> <enrichment_out> <ref.fasta>
```

## Evaluation signals

- Produced motif counts per strain match the reported pairs: 16 vs 12 for H. pylori JP26, 8 vs 1 for N. gonorrhoeae FA 1090, 8 vs 8 for T. phaeum DSM 12270
- motif_enrichment_comparison.csv contains the three per-strain count pairs in the expected format
- Discovered motifs align to the 20 REBASE reference motifs for H. pylori JP26, with Nanoident recovering more references than nanodisco
- Both methods were run on the same preprocessed difference RDS data so the comparison is like-for-like

## Limitations

- Comparison is only meaningful for strains with REBASE reference motifs available (motifs were collected from REBASE for the studied bacteria)
- nanodisco originally handled only mean features; the comparison requires the update.sh-extended preprocessing (std, time, reference-base proportion, average quality score) for Nanoident's motifs
- Reference sequences must be available at NCBI (e.g. CP041693, CP041696) for the enrichment step

## Evidence

- [supplementary] REBASE reference used for validation: "Methylation motifs of those bacteria were collected from the REBASE database"
- [results] Baseline vs Nanoident counts on H. pylori JP26: [evidence span withheld — anchor status: not_found]
- [methods] Enrichment step is a defined script: "chmod +x ./process/code/Nanoident_enrich.sh"
- [readme] Preprocessing feature extension motivating method difference: "The original nanodisco could only handle mean features, but Nanoident can handle four additional features"
