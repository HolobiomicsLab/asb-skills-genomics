---
name: pairwise-genetic-distance-calculation
description: Use when you have FASTA consensus sequences per specimen/marker (e.g.
  Nanopore rDNA consensus and Illumina reference consensus, or COI barcodes vs rDNA
  regions) and need a pairwise distance matrix partitioned into intra- vs interspecific
  classes.
when_to_use_negative:
- Input is raw Nanopore reads rather than polished consensus sequences — raw reads
  carry high error rates and must first be demultiplexed with MiniBar, consensus-called
  (e.g. Allele Wrangler with 30 reads-per-position coverage) and polished (RACON)
- The goal is phylogenetic tree inference or community composition (metabarcoding
  abundance) analysis rather than specimen-level pairwise distance quantification
edam_operation: http://edamontology.org/operation_0491
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_0080
tools:
- name: Edlib
  role: C/C++/Python library computing edit-distance (Levenshtein) sequence alignment
    used to derive pairwise genetic distances between consensus sequences
  repo: https://github.com/Martinsos/edlib
- name: MiniBar
  role: Upstream demultiplexing of dual-indexed Nanopore reads into per-sample FASTA
    files (with -T trimming, -F binning, -P prefix) that supply the consensus inputs;
    index edit distance also defines the comparison classes
  repo: https://github.com/calacademy-research/minibar
- name: Python 2.7
  role: Runtime language for minibar.py and for scripts computing and tabulating the
    pairwise distance matrices
provenance:
  source_task_ids:
  - task_005
  - task_002
  source_papers:
  - doi: 10.1093/gigascience/giz006
    title: Nanopore sequencing of long ribosomal DNA amplicons enables portable and
      simple biodiversity assessments with high phylogenetic resolution across broad
      taxonomic scale
  source_content_hash: sha256:f75cf9a80fd45bd21b107a9458dd70f005f9f0e85fd4da79d5b55eff05c18e85
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Martinsos/edlib
  tools:
  - Allele Wrangler
  - Edlib
  - MiniBar
  - Python 2.7
  edam_operation: http://edamontology.org/operation_0491
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_0080
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# pairwise-genetic-distance-calculation

## Summary

Computes pairwise genetic (edit/alignment) distances between consensus sequences — e.g. Nanopore rDNA consensus vs Illumina-based consensus, or COI vs rDNA distance matrices across specimens — to quantify consensus accuracy or compare taxonomic resolution between markers.

## When to use

Use when you have FASTA consensus sequences per specimen/marker (e.g. Nanopore rDNA consensus and Illumina reference consensus, or COI barcodes vs rDNA regions) and need a pairwise distance matrix partitioned into intra- vs interspecific classes.

## When NOT to use

- Input is raw Nanopore reads rather than polished consensus sequences — raw reads carry high error rates and must first be demultiplexed with MiniBar, consensus-called (e.g. Allele Wrangler with 30 reads-per-position coverage) and polished (RACON)
- The goal is phylogenetic tree inference or community composition (metabarcoding abundance) analysis rather than specimen-level pairwise distance quantification

## Inputs

- Per-specimen consensus FASTA sequences (Nanopore rDNA consensus, Illumina-based Peperomia consensus, COI 418 bp barcodes, long rDNA consensus sequences from the GigaScience repository deposit)
- Species/sample assignment table (e.g. 16 Hawaiian Tetragnatha species labels for the 47 specimens)
- Demultiplexing edit-distance class labels (index edit distances 2, 3, 4) for grouping consensus accuracy comparisons

## Outputs

- Pairwise genetic distance matrix (per marker: whole rDNA cluster, per rDNA gene region, COI)
- Per-specimen Nanopore-vs-Illumina distance table with averages (e.g. 0.165%, 0.187%, 2.072%)
- Mantel correlation statistics (R2 = 0.70, P < 0.001) and barcode-gap comparison (max intra- vs min interspecific distance per marker), written e.g. to tetragnatha_distance_correlation_results.tsv

## How to apply

Load the per-specimen consensus FASTA sequences (e.g. the 47 Hawaiian Tetragnatha COI 418 bp and long rDNA consensus sequences from the GigaScience deposit, or Peperomia Nanopore and Illumina rDNA consensus). Compute all pairwise distances using edit-distance alignment via the Edlib library (Python 2.7/3 bindings, Myers bit-vector algorithm, global/NW mode for full-length consensus comparison). For the Nanopore-vs-Illumina accuracy task, align each Nanopore rDNA consensus to its Illumina counterpart and express the distance as a percentage of alignment length, expecting reported averages of 0.165% (edit distance 2), 0.187% (edit distance 3), and 2.072% (edit distance 4). For the COI-vs-rDNA taxonomic-resolution task, partition pairs into intra- vs interspecific classes using the 16 species assignments, correlate the COI and rDNA distance matrices with a Mantel test (permutation-based P), and compare barcode-gap behavior via maximum intraspecific vs minimum interspecific distances, expecting R2 = 0.70 and P < 0.001.

## Related tools

- **Edlib** (C/C++/Python library computing edit-distance (Levenshtein) sequence alignment used to derive pairwise genetic distances between consensus sequences) — https://github.com/Martinsos/edlib
- **MiniBar** (Upstream demultiplexing of dual-indexed Nanopore reads into per-sample FASTA files (with -T trimming, -F binning, -P prefix) that supply the consensus inputs; index edit distance also defines the comparison classes) — https://github.com/calacademy-research/minibar
- **Python 2.7** (Runtime language for minibar.py and for scripts computing and tabulating the pairwise distance matrices)

## Examples

```
import edlib; d = edlib.align(nanopore_consensus, illumina_consensus, mode="NW")["editDistance"]
```

## Evaluation signals

- Nanopore-vs-Illumina average pairwise distances reproduce the reported 0.165%, 0.187%, and 2.072% values within rounding tolerance
- COI vs rDNA Mantel correlation yields R2 = 0.70 with P < 0.001, and pairwise correlation R2 = 0.70 with P < 0.001 as reported
- Every specimen consensus used meets the minimum 30 reads-per-position coverage threshold before distance computation
- Distance matrices are computed on the identical specimen set (47 specimens, 16 species) for both markers so matrices are directly comparable in the Mantel test
- Each computed distance is a valid edit/alignment distance (non-negative, zero for identical sequences) and intra-/interspecific class assignment covers all specimen pairs exactly once

## Limitations

- Consensus distance values depend on upstream demultiplexing edit distance; only quality-filtered, coverage-30, RACON-polished consensus sequences give the reported sub-0.2% accuracies
- Long rDNA amplicon sequencing recovers skewed diversity estimates for bulk community samples, and the source of this bias is undetermined, so distance-based taxonomic conclusions from bulk samples remain unreliable
- minibar.py is written for Python 2.7 (compatible with Python 3) and requires the edlib module; barcode-file column layout affects parsing

## Evidence

- [readme] A lightweight and super fast C/C++ library for sequence alignment using [edit distance]: [evidence span withheld — anchor status: external_doc]
- [readme] Supports multiple alignment methods: global(NW), prefix(SHW) and infix(HW): [evidence span withheld — anchor status: external_doc]
- [discussion] Other requirements: Edlib library module (https://github.com/Martinsos/edlib): "Other requirements: Edlib library module (https://github.com/Martinsos/edlib)"
- [other] Compute pairwise genetic distances between each Nanopore rDNA consensus and the corresponding Illumina-based Peperomia consensus: [evidence span withheld — anchor status: not_found]
