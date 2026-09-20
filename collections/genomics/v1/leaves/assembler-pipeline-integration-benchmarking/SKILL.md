---
name: assembler-pipeline-integration-benchmarking
description: Use when evaluating whether a new overlap detection or read-processing
  module can be swapped into an established assembler (e.g. Fedrann into Shasta) and
  comparing whole-genome assembly quality against the native baseline on identical
  long-read datasets.
when_to_use_negative:
- You only need to compare overlap graph quality (precision/recall) rather than end-to-end
  assembly quality — use the overlap-level benchmarking workflow in kNN-overlap-finder
  instead.
- The candidate module cannot be decoupled from the assembler's internal data structures;
  the article notes integration is non-trivial due to undocumented in-memory structures
  and tight stage coupling.
edam_operation: http://edamontology.org/operation_0525
edam_topics:
- http://edamontology.org/topic_0196
- http://edamontology.org/topic_3693
tools:
- name: Fedrann
  role: Replacement overlap detection stage whose output feeds the Shasta assembly
    stages
  repo: https://github.com/jzhang-dev/fedrann
- name: Shasta
  role: Assembly framework into which Fedrann is integrated and the baseline assembler
    for comparison
- name: BUSCO
  role: Assesses assembly completeness against the primates_odb10 lineage (v5.8.0)
- name: kNN-overlap-finder
  role: Reproducible workflows for benchmarking and evaluation of the integrated pipeline
  repo: https://github.com/jzhang-dev/kNN-overlap-finder
- name: Python
  role: Used to compute assembly statistics and compile metrics into assembly_metrics.tsv
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1093/gigascience/giag048
    title: 'FEDRANN: effective long-read overlap detection based on dimensionality
      reduction and approximate nearest neighbors'
  source_content_hash: sha256:f1526393183ddd2ec4a7df0b5cb44295f19cd9ae9061971ccb6bf5beb4e363d7
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/jzhang-dev/fedrann
  tools:
  - BUSCO
  - Fedrann
  - Python
  - Shasta
  - kNN-overlap-finder
  - primates_odb10
  edam_operation: http://edamontology.org/operation_0525
  edam_topics:
  - http://edamontology.org/topic_0196
  - http://edamontology.org/topic_3693
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6363636363636364
n_claims: 11
---

# assembler-pipeline-integration-benchmarking

## Summary

Substitutes a third-party overlap detection stage into an existing long-read assembler's pipeline and benchmarks the resulting assembly head-to-head against the unmodified assembler using contiguity (N50, longest contig), completeness (BUSCO), and accuracy (k-mer QV) metrics.

## When to use

Use when evaluating whether a new overlap detection or read-processing module can be swapped into an established assembler (e.g. Fedrann into Shasta) and comparing whole-genome assembly quality against the native baseline on identical long-read datasets.

## When NOT to use

- You only need to compare overlap graph quality (precision/recall) rather than end-to-end assembly quality — use the overlap-level benchmarking workflow in kNN-overlap-finder instead.
- The candidate module cannot be decoupled from the assembler's internal data structures; the article notes integration is non-trivial due to undocumented in-memory structures and tight stage coupling.

## Inputs

- H4C long-read dataset (PacBio HiFi HG002 from Genome in a Bottle)
- H4O long-read dataset (CycloneSEQ G400-ER HG002)
- T2T-CHM13 v2.0 reference assembly (for QV estimation)
- primates_odb10 BUSCO lineage database
- Unmodified Shasta assembler as baseline

## Outputs

- Fedrann-Shasta and native Shasta contig FASTA assemblies per dataset
- assembly_metrics.tsv with N50, longest contig, BUSCO completeness, and k-mer QV per assembly

## How to apply

Run the modified pipeline on each dataset — for Fedrann-Shasta, feed Fedrann's overlap detection stage (k=30 k-mer feature extraction for the PacBio HiFi H4C dataset, k=50 for the CycloneSEQ G400-ER H4O dataset, both HG002) into the Shasta assembly stages. Run unmodified Shasta on the same datasets as the baseline. Compute contig N50 and longest contig from each assembly's contig FASTA, assess BUSCO v5.8.0 completeness against the primates_odb10 lineage database, and estimate k-mer QV against the T2T-CHM13 v2.0 reference. Compile all metrics into a single table (assembly_metrics.tsv) and check that reported values are reproduced within tolerance, e.g. H4C N50 of 44.45 Mb (Fedrann-Shasta) vs 45.07 Mb (Shasta) and H4O N50 of 17.07 Mb vs 5.73 Mb.

## Related tools

- **Fedrann** (Replacement overlap detection stage whose output feeds the Shasta assembly stages) — https://github.com/jzhang-dev/fedrann
- **Shasta** (Assembly framework into which Fedrann is integrated and the baseline assembler for comparison)
- **BUSCO** (Assesses assembly completeness against the primates_odb10 lineage (v5.8.0))
- **kNN-overlap-finder** (Reproducible workflows for benchmarking and evaluation of the integrated pipeline) — https://github.com/jzhang-dev/kNN-overlap-finder
- **Python** (Used to compute assembly statistics and compile metrics into assembly_metrics.tsv)

## Examples

```
fedrann -i h4o_reads.fasta.gz -o fedrann_out/ -k 50 --kmer-sample-fraction 0.05 --seed 602 --kmer-min-multiplicity 2 --threads 32
```

## Evaluation signals

- H4C contig N50 reproduces ~44.45 Mb (Fedrann-Shasta) vs ~45.07 Mb (Shasta) within a reasonable tolerance
- H4O contig N50 reproduces ~17.07 Mb (Fedrann-Shasta) vs ~5.73 Mb (Shasta) — a three-fold contiguity gain
- BUSCO v5.8.0 completeness against primates_odb10 is computed for all four assemblies and consistent with reported values
- k-mer QV is estimated against T2T-CHM13 v2.0 for each assembly and matches reported quality
- Metrics for all assemblies (Fedrann-Shasta H4C, Shasta H4C, Fedrann-Shasta H4O, Shasta H4O) are present in assembly_metrics.tsv

## Limitations

- Fedrann-Shasta results are a proof-of-concept constrained by the internal architecture of the established assembler, not the upper limit of the module's potential.
- Integrating a module into existing assemblers is non-trivial, requiring deep expertise in undocumented internal structures.
- No DOI or Zenodo archive is provided for the workflows; reproducibility depends solely on the GitHub repositories.

## Evidence

- [abstract] By integrating Fedrann into the Shasta assembler, we successfully reconstructed human whole genomes, achieving high assembly contiguity and quality: "By integrating Fedrann into the Shasta assembler, we successfully reconstructed human whole genomes, achieving high assembly contiguity and quality"
- [discussion] BUSCO v5.8.0 against primates_odb10: "BUSCO v5.8.0 [43] against the primates_odb10 lineage database"
- [discussion] k=30 for the H4C dataset and k=50 for the H4O dataset: [evidence span withheld — anchor status: not_found]
- [discussion] Reproducible workflows repository: "Reproducible workflows for benchmarking and evaluation: https://github.com/jzhang-dev/kNN-overlap-finder"
- [discussion] Fedrann-Shasta proof-of-concept caveat: "we consider our current Fedrann-Shasta results a successful proof-of-concept rather than the upper limit of Fedrann’s potential"
- [readme] Fedrann output includes overlaps.tsv with candidate overlaps: "overlaps.tsv: This file lists all identified candidate overlaps with sequence names, orientations, and their similarity metrics."
