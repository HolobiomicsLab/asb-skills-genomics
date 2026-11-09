---
name: assembly-contiguity-metrics-computation
description: Use when you have ≥2 assemblies (contig FASTA files) — e.g. a modified
  assembler pipeline vs a baseline — and need reproducible contiguity (N50, longest
  contig) and quality (BUSCO, k-mer QV) comparisons on human long-read data.
when_to_use_negative:
- Input is raw reads or overlap files (e.g. Fedrann overlaps.tsv) rather than assembled
  contig FASTA files — this skill evaluates assemblies, not pre-assembly artifacts
- You only need overlap graph quality (precision/recall of candidate overlaps); use
  the article's benchmarking workflow with alignment filtering instead
- The reference lineage database or reference genome appropriate to the organism is
  unavailable — BUSCO against primates_odb10 and QV against T2T-CHM13 v2.0 are human-specific
  choices
edam_operation: http://edamontology.org/operation_3192
edam_topics:
- http://edamontology.org/topic_3697
tools:
- name: Fedrann-Shasta
  role: Pipeline variant under evaluation; its contig FASTA output supplies the metrics
    being computed
  repo: https://github.com/jzhang-dev/Fedrann-Shasta
- name: Shasta
  role: Baseline assembler whose contig FASTA output provides the comparison N50/longest-contig/BUSCO/QV
    values
- name: BUSCO
  role: Assesses assembly completeness (v5.8.0) against the primates_odb10 lineage
    database
- name: Python
  role: Used to compute N50 and longest contig from contig FASTA files and compile
    assembly_metrics.tsv
  repo: https://github.com/jzhang-dev/kNN-overlap-finder
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1093/gigascience/giag048
    title: 'FEDRANN: effective long-read overlap detection based on dimensionality
      reduction and approximate nearest neighbors'
  source_content_hash: sha256:11ff81bb32fc80249e09a05f9fe02dea7ebc23e7e3334463614cc23a21f2524d
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/jzhang-dev/Fedrann-Shasta
  tools:
  - BUSCO
  - Fedrann
  - Fedrann-Shasta
  - Python
  - Shasta
  - primates_odb10
  edam_operation: http://edamontology.org/operation_3192
  edam_topics:
  - http://edamontology.org/topic_3697
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# assembly-contiguity-metrics-computation

## Summary

Computes whole-genome assembly contiguity and quality metrics — contig N50, longest contig, BUSCO v5.8.0 completeness against primates_odb10, and k-mer quality value (QV) against T2T-CHM13 v2.0 — from contig FASTA files, to benchmark de novo assemblies such as Fedrann-Shasta vs native Shasta.

## When to use

Use when you have ≥2 assemblies (contig FASTA files) — e.g. a modified assembler pipeline vs a baseline — and need reproducible contiguity (N50, longest contig) and quality (BUSCO, k-mer QV) comparisons on human long-read data.

## When NOT to use

- Input is raw reads or overlap files (e.g. Fedrann overlaps.tsv) rather than assembled contig FASTA files — this skill evaluates assemblies, not pre-assembly artifacts
- You only need overlap graph quality (precision/recall of candidate overlaps); use the article's benchmarking workflow with alignment filtering instead
- The reference lineage database or reference genome appropriate to the organism is unavailable — BUSCO against primates_odb10 and QV against T2T-CHM13 v2.0 are human-specific choices

## Inputs

- Contig FASTA files from each assembly being compared (e.g. Fedrann-Shasta and native Shasta outputs for H4C and H4O HG002 datasets)
- T2T-CHM13 v2.0 reference assembly (for k-mer QV estimation)
- primates_odb10 lineage database (for BUSCO v5.8.0)
- Long-read input datasets: H4C (PacBio HiFi HG002, GIAB) and H4O (CycloneSEQ G400-ER HG002)

## Outputs

- assembly_metrics.tsv compiling contig N50, longest contig, BUSCO completeness, and k-mer QV for every assembly
- Per-assembly BUSCO completeness reports against primates_odb10
- Per-assembly k-mer QV estimates against T2T-CHM13 v2.0

## How to apply

For each assembly, run the modified and baseline assembler on the same input reads (e.g. Fedrann-Shasta and unmodified Shasta on H4C PacBio HiFi HG002 from GIAB and H4O CycloneSEQ G400-ER HG002, using k=30 for H4C and k=50 for H4O). Compute contig N50 and longest contig from each assembly's contig FASTA file using standard assembly statistics. Assess BUSCO v5.8.0 completeness against the primates_odb10 lineage database to gauge gene-space completeness. Estimate k-mer QV of each assembly against the T2T-CHM13 v2.0 reference to quantify base-level accuracy. Compile all four metrics for all assemblies into a single assembly_metrics.tsv table and compare modified-vs-baseline values within a reasonable tolerance (e.g. the reported H4O N50 of 17.07 Mb vs 5.73 Mb, a three-fold increase).

## Related tools

- **Fedrann-Shasta** (Pipeline variant under evaluation; its contig FASTA output supplies the metrics being computed) — https://github.com/jzhang-dev/Fedrann-Shasta
- **Shasta** (Baseline assembler whose contig FASTA output provides the comparison N50/longest-contig/BUSCO/QV values)
- **BUSCO** (Assesses assembly completeness (v5.8.0) against the primates_odb10 lineage database)
- **Python** (Used to compute N50 and longest contig from contig FASTA files and compile assembly_metrics.tsv) — https://github.com/jzhang-dev/kNN-overlap-finder

## Examples

```
python scripts/compute_assembly_metrics.py --fasta h4o_fedrann_shasta.fasta --fasta h4o_shasta.fasta --busco-db primates_odb10 --ref T2T-CHM13v2.0.fa --out assembly_metrics.tsv
```

## Evaluation signals

- Computed H4C contig N50 reproduces the reported values within tolerance: 44.45 Mb (Fedrann-Shasta) vs 45.07 Mb (Shasta)
- Computed H4O contig N50 reproduces 17.07 Mb (Fedrann-Shasta) vs 5.73 Mb (Shasta) — a roughly three-fold contiguity increase
- assembly_metrics.tsv contains all four metrics (N50, longest contig, BUSCO completeness, k-mer QV) for every assembly with no missing cells
- BUSCO runs complete against primates_odb10 with BUSCO v5.8.0 for all assemblies
- k-mer QV estimates are computable for all assemblies against T2T-CHM13 v2.0 without reference-mapping failures

## Limitations

- The Fedrann-Shasta results are only a proof-of-concept constrained by the internal architecture of established tools, so metric comparisons may reflect integration constraints rather than the upper limit of the method
- Reproducibility depends solely on GitHub repositories; no DOI or Zenodo archive of workflows is provided
- Metric values are dataset- and k-parameter-dependent (k=30 for H4C, k=50 for H4O); results do not automatically transfer to other read sets or assemblers

## Evidence

- [abstract] By integrating Fedrann into the Shasta assembler, we successfully reconstructed human whole genomes, achieving high assembly contiguity and quality: "By integrating Fedrann into the Shasta assembler, we successfully reconstructed human whole genomes, achieving high assembly contiguity and quality"
- [results] task card H4O N50 finding: [evidence span withheld — anchor status: not_found]
- [discussion] BUSCO lineage choice: "BUSCO v5.8.0 [43] against the primates_odb10 lineage database"
- [discussion] dataset k parameters: [evidence span withheld — anchor status: not_found]
