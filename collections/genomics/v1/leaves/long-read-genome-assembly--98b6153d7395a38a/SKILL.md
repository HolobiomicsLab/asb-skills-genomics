---
name: long-read-genome-assembly
description: Use when you have long-read FASTQ/FASTA datasets (e.g. HG002 PacBio HiFi
  or CycloneSEQ G400-ER) and need a de novo whole-genome assembly benchmarked on contiguity
  and accuracy against a reference.
when_to_use_negative:
- You only need overlap detection or benchmarking of overlap quality (use Fedrann
  directly, not a full assembly pipeline)
- Input reads are short-read (Illumina) data; the overlap-layout-consensus workflow
  here targets long-read datasets
- A reference-guided assembly or variant calling against an existing reference is
  the goal rather than de novo reconstruction
edam_operation: http://edamontology.org/operation_0525
edam_topics:
- http://edamontology.org/topic_0193
- http://edamontology.org/topic_0622
tools:
- name: Fedrann
  role: Performs k-mer feature extraction, ICF/SRP dimensionality reduction, and NNDescent
    approximate nearest-neighbor search to produce candidate overlaps for assembly
  repo: https://github.com/jzhang-dev/fedrann
- name: Shasta
  role: Assembles contigs from the Fedrann-generated overlaps (Fedrann-Shasta pipeline)
    and serves as the unmodified baseline assembler
  repo: https://github.com/jzhang-dev/Fedrann-Shasta
- name: BUSCO
  role: Assesses assembly completeness against the primates_odb10 lineage database
    (v5.8.0)
- name: Python
  role: Implementation language of Fedrann; used with standard assembly statistics
    scripts to compute N50, longest contig, and k-mer QV
  repo: https://github.com/jzhang-dev/fedrann
- name: Jellyfish
  role: Efficient k-mer counting and filtering inside Fedrann, retaining only k-mers
    whose multiplicities exceed a predefined threshold
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1093/gigascience/giag048
    title: 'FEDRANN: effective long-read overlap detection based on dimensionality
      reduction and approximate nearest neighbors'
  source_content_hash: sha256:b193e5801d9691e30471dcb7f4c3e7ba906d3908e38d8b602c53246c0806dd3b
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/jzhang-dev/fedrann
  tools:
  - BUSCO
  - Fedrann
  - Jellyfish
  - Python
  - Shasta
  - primates_odb10
  edam_operation: http://edamontology.org/operation_0525
  edam_topics:
  - http://edamontology.org/topic_0193
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 12
---

# long-read-genome-assembly

## Summary

De novo assembly of whole genomes from long-read sequencing data (PacBio HiFi, ONT, BGI CycloneSEQ) via the overlap-layout-consensus paradigm, using Fedrann's k-mer feature extraction, IDF weighting, and sparse random projection with NNDescent to generate overlaps that feed Shasta assembly stages.

## When to use

Use when you have long-read FASTQ/FASTA datasets (e.g. HG002 PacBio HiFi or CycloneSEQ G400-ER) and need a de novo whole-genome assembly benchmarked on contiguity and accuracy against a reference.

## When NOT to use

- You only need overlap detection or benchmarking of overlap quality (use Fedrann directly, not a full assembly pipeline)
- Input reads are short-read (Illumina) data; the overlap-layout-consensus workflow here targets long-read datasets
- A reference-guided assembly or variant calling against an existing reference is the goal rather than de novo reconstruction

## Inputs

- Long-read sequencing reads in FASTQ/FASTA format (e.g. PacBio HiFi HG002 from GIAB, CycloneSEQ G400-ER HG002)
- Reference assembly for QV estimation (T2T-CHM13 v2.0)
- BUSCO lineage database (primates_odb10)

## Outputs

- Fedrann overlaps.tsv (candidate overlaps with query/target names, orientations, neighbor_rank, distance)
- Contig FASTA assemblies (Fedrann-Shasta and native Shasta per dataset)
- assembly_metrics.tsv with contig N50, longest contig, BUSCO completeness, and k-mer QV

## How to apply

Run Fedrann on the long-read FASTQ/FASTA file with dataset-specific k-mer sizes (k=30 for PacBio HiFi HG002 'H4C' data, k=50 for CycloneSEQ G400-ER HG002 'H4O' data), using Jellyfish for k-mer counting with --kmer-min-multiplicity 2 to remove sequencing-error k-mers, sampling ~5% of k-mers (--kmer-sample-fraction 0.05), and applying inverse collection frequency (ICF) weighting fused with a sparse random projection matrix. Feed Fedrann's overlaps.tsv (query/target names, orientations, neighbor ranks, distances) into the Shasta assembler via the Fedrann-Shasta pipeline to build contigs. Assemble a baseline with unmodified Shasta on the same reads for comparison. Evaluate each assembly by computing contig N50 and longest contig from the contig FASTA, BUSCO v5.8.0 completeness against primates_odb10, and k-mer QV against T2T-CHM13 v2.0; expect H4C N50 near 44-45 Mb and H4O N50 near 17 Mb for Fedrann-Shasta versus 5.73 Mb for native Shasta on H4O. Compile all metrics into a single assembly_metrics.tsv for validation against reported values.

## Related tools

- **Fedrann** (Performs k-mer feature extraction, ICF/SRP dimensionality reduction, and NNDescent approximate nearest-neighbor search to produce candidate overlaps for assembly) — https://github.com/jzhang-dev/fedrann
- **Shasta** (Assembles contigs from the Fedrann-generated overlaps (Fedrann-Shasta pipeline) and serves as the unmodified baseline assembler) — https://github.com/jzhang-dev/Fedrann-Shasta
- **BUSCO** (Assesses assembly completeness against the primates_odb10 lineage database (v5.8.0))
- **Python** (Implementation language of Fedrann; used with standard assembly statistics scripts to compute N50, longest contig, and k-mer QV) — https://github.com/jzhang-dev/fedrann
- **Jellyfish** (Efficient k-mer counting and filtering inside Fedrann, retaining only k-mers whose multiplicities exceed a predefined threshold)

## Examples

```
fedrann -i reads.fasta.gz -o output/ -k 30 --kmer-sample-fraction 0.05 --seed 602 --kmer-min-multiplicity 2 --threads 32
```

## Evaluation signals

- Reproduce reported N50 values within reasonable tolerance: H4C ~44.45 Mb (Fedrann-Shasta) vs 45.07 Mb (Shasta); H4O 17.07 Mb (Fedrann-Shasta) vs 5.73 Mb (Shasta)
- overlaps.tsv contains the expected columns (query_name, query_orientation, target_name, target_orientation, neighbor_rank, distance) with smaller distances indicating higher similarity
- BUSCO completeness computed against primates_odb10 with BUSCO v5.8.0 for every assembly
- k-mer QV computed against T2T-CHM13 v2.0 for each assembly
- All four assemblies' metrics present in assembly_metrics.tsv (2 datasets x 2 assemblers)

## Limitations

- Fedrann's memory consumption is higher than most existing overlap detection tools; memory-efficient representations, shared-memory designs, or alternative ANN implementations remain future work
- Fedrann-Shasta results are a proof-of-concept rather than the upper limit, constrained by the internal architecture of established assemblers
- Integrating Fedrann into existing assemblers is non-trivial due to undocumented internal structures and tight coupling between pipeline stages
- No empirical validation of Fedrann for metagenomic binning, taxonomic classification, or other long-read applications yet
- Reproducibility depends solely on GitHub repositories; no DOI or Zenodo archive is provided

## Evidence

- [abstract] By integrating Fedrann into the Shasta assembler, we successfully reconstructed human whole genomes: "By integrating Fedrann into the Shasta assembler, we successfully reconstructed human whole genomes"
- [discussion] k=30 for H4C, k=50 for H4O: [evidence span withheld — anchor status: not_found]
- [discussion] Jellyfish k-mer filtering rationale: "The pipeline utilizes Jellyfish for efficient k-mer counting and filtering, retaining only k-mers whose multiplicities exceed a predefined threshold"
- [readme] Fedrann CLI with key parameters: "--kmer-sample-fraction 0.05 \
    --seed 602 \
    --kmer-min-multiplicity 2"
- [discussion] BUSCO version and lineage: "BUSCO v5.8.0 [43] against the primates_odb10 lineage database"
