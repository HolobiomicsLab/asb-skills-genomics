---
name: busco-completeness-assessment
description: Use when a newly assembled genome (e.g. from an overlap-layout-consensus
  long-read assembler) needs a quantitative completeness/quality score comparable
  across assemblers, using a curated ortholog lineage database.
when_to_use_negative:
- The sample belongs to a clade outside the primates_odb10 lineage — a different BUSCO
  lineage database matched to the organism must be selected
- You need base-level accuracy rather than gene-space completeness — k-mer QV against
  a reference (e.g. T2T-CHM13 v2.0) is the appropriate metric instead
- The assembly is only a partial/draft output still inside the pipeline rather than
  a final contig FASTA
tools:
- name: BUSCO
  role: Assesses assembly completeness (version 5.8.0) against the primates_odb10
    lineage database for each Shasta/Fedrann-Shasta human assembly
- name: Shasta
  role: Produces the contig FASTA assemblies (Fedrann-Shasta pipeline and unmodified
    Shasta baseline) that are scored by BUSCO
  repo: https://github.com/jzhang-dev/Fedrann-Shasta
- name: Fedrann
  role: Overlap detection stage whose integration into Shasta yields the Fedrann-Shasta
    assembly under evaluation
  repo: https://github.com/jzhang-dev/fedrann
- name: Python
  role: Used to compile per-assembly metrics (N50, longest contig, BUSCO completeness,
    k-mer QV) into assembly_metrics.tsv
  repo: https://github.com/jzhang-dev/kNN-overlap-finder
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1093/gigascience/giag048
    title: 'FEDRANN: effective long-read overlap detection based on dimensionality
      reduction and approximate nearest neighbors'
  source_content_hash: sha256:c56e9fcf7b210ffa9129bc5290fee3df42c9ab7dd9584dbf0df66e75f62ea16b
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/jzhang-dev/Fedrann-Shasta
  tools:
  - BUSCO
  - Fedrann
  - Python
  - Shasta
  - primates_odb10
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# busco-completeness-assessment

## Summary

Assess the completeness of de novo genome assemblies (e.g. Shasta and Fedrann-Shasta human assemblies) by running BUSCO v5.8.0 against the primates_odb10 lineage database, yielding a standard completeness metric for cross-assembler comparison.

## When to use

Use when a newly assembled genome (e.g. from an overlap-layout-consensus long-read assembler) needs a quantitative completeness/quality score comparable across assemblers, using a curated ortholog lineage database.

## When NOT to use

- The sample belongs to a clade outside the primates_odb10 lineage — a different BUSCO lineage database matched to the organism must be selected
- You need base-level accuracy rather than gene-space completeness — k-mer QV against a reference (e.g. T2T-CHM13 v2.0) is the appropriate metric instead
- The assembly is only a partial/draft output still inside the pipeline rather than a final contig FASTA

## Inputs

- Contig FASTA files from each assembly (Fedrann-Shasta and native Shasta outputs for H4C and H4O datasets)
- BUSCO v5.8.0 installation with the primates_odb10 lineage database

## Outputs

- BUSCO completeness metrics per assembly (short summary/report files)
- Rows in a compiled assembly_metrics.tsv containing N50, longest contig, BUSCO completeness, and k-mer QV for all assemblies

## How to apply

Take the contig FASTA produced by each assembly (here: Fedrann-Shasta and unmodified Shasta runs on the H4C PacBio HiFi HG002 and H4O CycloneSEQ G400-ER HG002 datasets) and run BUSCO v5.8.0 against the primates_odb10 lineage database, matching the lineage to the organism (primates for human HG002 assemblies). Record the completeness percentage per assembly and compile it alongside assembly statistics (contig N50, longest contig, k-mer QV against the T2T-CHM13 v2.0 reference) into an assembly_metrics.tsv table so Fedrann-Shasta and native Shasta results can be compared directly. Treat BUSCO completeness as one of four orthogonal quality axes: contiguity (N50/longest contig), gene-space completeness (BUSCO), and base-level accuracy (k-mer QV).

## Related tools

- **BUSCO** (Assesses assembly completeness (version 5.8.0) against the primates_odb10 lineage database for each Shasta/Fedrann-Shasta human assembly)
- **Shasta** (Produces the contig FASTA assemblies (Fedrann-Shasta pipeline and unmodified Shasta baseline) that are scored by BUSCO) — https://github.com/jzhang-dev/Fedrann-Shasta
- **Fedrann** (Overlap detection stage whose integration into Shasta yields the Fedrann-Shasta assembly under evaluation) — https://github.com/jzhang-dev/fedrann
- **Python** (Used to compile per-assembly metrics (N50, longest contig, BUSCO completeness, k-mer QV) into assembly_metrics.tsv) — https://github.com/jzhang-dev/kNN-overlap-finder

## Examples

```
busco -i fedrann_shasta_H4O_contigs.fasta -l primates_odb10 -m genome --out busco_H4O_fedrann
```

## Evaluation signals

- BUSCO runs to completion on all four assemblies (Fedrann-Shasta H4C/H4O, Shasta H4C/H4O) with the primates_odb10 lineage and reports a completeness percentage for each
- Compiled assembly_metrics.tsv contains BUSCO completeness for every assembly alongside N50, longest contig, and k-mer QV
- Reproduced metrics match reported values within reasonable tolerance, e.g. H4C N50 of 44.45 Mb (Fedrann-Shasta) vs 45.07 Mb (Shasta) and H4O N50 of 17.07 Mb vs 5.73 Mb
- BUSCO version and lineage database are recorded (v5.8.0 / primates_odb10) so the score is comparable across assemblers

## Limitations

- The Fedrann-Shasta whole-genome results are a proof-of-concept constrained by the internal architecture of the assembler, so BUSCO scores reflect that integration, not Fedrann's upper limit
- No DOI or Zenodo archive is provided for the workflows; reproducibility depends solely on the GitHub repositories
- BUSCO completeness measures gene-space completeness only and does not capture base-level errors; pair it with k-mer QV against T2T-CHM13 v2.0

## Evidence

- [discussion] BUSCO v5.8.0 [43] against the primates_odb10 lineage database: "BUSCO v5.8.0 [43] against the primates_odb10 lineage database"
- [intro] reconstructed human whole genomes with high contiguity and accuracy: "By integrating Fedrann into the Shasta assembly framework, we successfully reconstructed human whole genomes with high contiguity and accuracy."
- [other] H4O N50 of 17.07 Mb vs 5.73 Mb validation target: [evidence span withheld — anchor status: not_found]
