---
name: metagenomic-contig-colinearity-analysis
description: Use when a post-assembly contig set (e.g., MergedContigs.fasta from MeGAMerge)
  contains very large contigs (hundreds of kb) whose contiguity must be confirmed
  by whole-contig colinear alignment to a single reference genome, such as Supplementary
  Figures S4-S5 validation.
when_to_use_negative:
- The goal is de novo assembly or contig merging rather than validation of already-produced
  contigs (use MeGAMerge itself, not a colinearity check).
- Contigs are short (<2 kb) fragments where full-length colinear alignment to a single
  reference is not the relevant validity criterion; the paper removes very small contigs
  before analysis.
- No reference genome of sufficient similarity is available, so nucmer alignment would
  yield no meaningful hits and colinearity cannot be judged.
edam_operation: http://edamontology.org/operation_3182
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3174
tools:
- name: MeGAMerge
  role: Produces the merged contig set (MergedContigs.fasta) whose largest contigs
    are validated by colinear alignment
  repo: https://github.com/LANL-Bioinformatics/MeGAMerge
- name: Nucmer
  role: Aligns the two largest MeGAMerge contigs against the Bacteroides helcogenes
    reference genome, producing the delta alignment file
  repo: https://github.com/LANL-Bioinformatics/MeGAMerge
- name: MUMmer
  role: Tool suite providing nucmer, delta-filter, and show-coords used for alignment,
    filtering, and coordinate-table generation
  repo: https://github.com/LANL-Bioinformatics/MeGAMerge
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/srep06480
    title: Improved Assemblies Using a Source-Agnostic Pipeline for MetaGenomic Assembly
      by Merging (MeGAMerge) of Contigs
  source_content_hash: sha256:830e0fa023402c1f7b7e53cd51d5ec572bc3b98a169213b520e842199f1cdfae
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: cc-by-nc-sa
    url: https://github.com/LANL-Bioinformatics/MeGAMerge
  repo_url: https://github.com/LANL-Bioinformatics/MeGAMerge
  tools:
  - MUMmer
  - MeGAMerge
  - Nucmer
  edam_operation: http://edamontology.org/operation_3182
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 7
---

# metagenomic-contig-colinearity-analysis

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Validates the contiguity and validity of large metagenomic assembly contigs (e.g., the 342.5 kb and 472.5 kb largest MeGAMerge contigs from HMP sample SRS022071) by aligning them colinearly against a known reference genome with Nucmer/MUMmer, as done against the Bacteroides helcogenes P 36-108 chromosome in Scholz et al. 2014 (Scientific Reports 4, 6480).

## When to use

Use when a post-assembly contig set (e.g., MergedContigs.fasta from MeGAMerge) contains very large contigs (hundreds of kb) whose contiguity must be confirmed by whole-contig colinear alignment to a single reference genome, such as Supplementary Figures S4-S5 validation.

## When NOT to use

- The goal is de novo assembly or contig merging rather than validation of already-produced contigs (use MeGAMerge itself, not a colinearity check).
- Contigs are short (<2 kb) fragments where full-length colinear alignment to a single reference is not the relevant validity criterion; the paper removes very small contigs before analysis.
- No reference genome of sufficient similarity is available, so nucmer alignment would yield no meaningful hits and colinearity cannot be judged.

## Inputs

- Reference genome FASTA (e.g., Bacteroides helcogenes P 36-108 chromosome from NCBI)
- Query contigs in Multi-FASTA format (e.g., MergedContigs.fasta, the MeGAMerge output for SRS022071)
- MUMmer executables directory (nucmer, delta-filter, show-coords)

## Outputs

- Nucmer delta alignment file (query vs. reference)
- delta-filter output retaining best collinear alignments per query-reference pair
- Tabular show-coords coordinates table with reference and query start/end positions for each aligned segment
- Colinearity assessment confirming/denying full-length alignment of the 342.5 kb and 472.5 kb contigs (as in Supplementary Figures S4-S5)

## How to apply

Obtain the reference genome FASTA (e.g., Bacteroides helcogenes P 36-108 from NCBI) and the query contigs from the merged assembly output (MergedContigs.fasta). Run nucmer from the MUMmer executables directory to align each query contig against the reference, producing a delta alignment file; the README notes MUMmer should be compiled in 64-bit mode ('make all CPPFLAGS="-O3 -DSIXTYFOURBITS"') for larger projects. Filter the delta file with delta-filter to retain the best collinear alignments per query-reference pair, then convert to a tabular coordinates table with show-coords listing reference and query start/end positions per aligned segment. Judge success by confirming the table records hits against the expected Bacteroides reference, that the two query lengths fall near 342.5 kb and 472.5 kb, and that alignments are colinear along the full contig length (gaps permitted where genomic islands or dissimilar regions intervene), matching Supplementary Figures S4-S5. Rationale: contigs that do not contiguously align to a single reference genome were flagged as chimeric in the paper, so full-length colinearity is the criterion for a non-chimeric, valid contig.

## Related tools

- **MeGAMerge** (Produces the merged contig set (MergedContigs.fasta) whose largest contigs are validated by colinear alignment) — https://github.com/LANL-Bioinformatics/MeGAMerge
- **Nucmer** (Aligns the two largest MeGAMerge contigs against the Bacteroides helcogenes reference genome, producing the delta alignment file) — https://github.com/LANL-Bioinformatics/MeGAMerge
- **MUMmer** (Tool suite providing nucmer, delta-filter, and show-coords used for alignment, filtering, and coordinate-table generation) — https://github.com/LANL-Bioinformatics/MeGAMerge

## Examples

```
/usr/local/mummer/bin/nucmer --maxmatch -p out B_helcogenes.fasta MergedContigs.fasta; delta-filter -r -q out.delta > out.filter; show-coords -T out.filter > out.coords
```

## Evaluation signals

- The show-coords table records alignment hits against the expected Bacteroides helcogenes reference genome.
- Query contig lengths in the coordinates table fall near 342.5 kb and 472.5 kb, matching the two largest MeGAMerge contigs from SRS022071.
- Alignments are colinear along the entire length of each query contig, with gaps only where genomic islands or dissimilar regions intervene (as displayed in Supplementary Figures S4-S5).
- delta-filter output retains the best collinear alignment(s) per query-reference pair rather than fragmented or redundant hits.
- Contigs failing to align contiguously to the single reference should be flagged as chimeric, per the paper's chimera-identification criterion.

## Limitations

- Colinearity may be interrupted by genomic islands or regions of low sequence similarity, so gaps in the alignment do not by themselves invalidate a contig but require manual interpretation.
- Overlap-based consensus tools, including the MUMmer/AMOS components used in this workflow, have run-times that increase exponentially with increasing input dataset size.
- MUMmer must be compiled in 64-bit mode for larger genome projects, and AMOS must be configured to locate nucmer, delta-filter, and show-coords, or the workflow will fail to run.

## Evidence

- [results] The 342.5 kb small-Kmer MeGAMerge contig was colinear along its entire length with a portion of the B. helcogenes chromosome, with gaps where genomic islands or dissimilar regions intervened.: [evidence span withheld — anchor status: not_found]
