---
name: mummer-alignment-coordinates-extraction
description: Use when you need to validate that large de novo contigs align colinearly
  to a known reference genome, e.g. checking that ~342.5 kb and ~472.5 kb MeGAMerge
  contigs from HMP sample SRS022071 match the B. helcogenes chromosome.
when_to_use_negative:
- When the goal is de novo assembly or merging of contigs — use the MeGAMerge/Minimus2
  pipeline itself; nucmer alignment is only the downstream validation step
- When no reference genome is available — colinearity validation against a reference
  cannot be performed, and the article instead relies on read mapping, gene prediction,
  and similarity searches
- When contigs are expected to be chimeric — nucmer colinearity alone will not flag
  contigs that do not contiguously align to a single reference genome as chimeric
  without additional chimera-flagging analysis
edam_operation: http://edamontology.org/operation_3182
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_0080
tools:
- name: MUMmer
  role: Provides nucmer (whole-genome alignment of contigs to reference), delta-filter
    (best collinear alignment selection), and show-coords (tabular coordinate extraction);
    must be compiled 64-bit and visible to AMOS
  repo: https://github.com/LANL-Bioinformatics/MeGAMerge
- name: MeGAMerge
  role: Produces the merged contig set (MergedContigs.fasta) whose largest contigs
    are validated by this alignment workflow
  repo: https://github.com/LANL-Bioinformatics/MeGAMerge
- name: AMOS toolbox
  role: Minimus2 within AMOS invokes nucmer, delta-filter and show-coords; installation
    requires these MUMmer executables in PATH or NUCMER/DELTAFILTER/SHOWCOORDS configure
    variables
  repo: https://github.com/LANL-Bioinformatics/MeGAMerge
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/srep06480
    title: Improved Assemblies Using a Source-Agnostic Pipeline for MetaGenomic Assembly
      by Merging (MeGAMerge) of Contigs
  source_content_hash: sha256:5138abbbb8b29ff12e3de018288086b362ca68cbd9de4b153d810f4023c326af
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
  - AMOS toolbox
  - MUMmer
  - MeGAMerge
  - Nucmer
  edam_operation: http://edamontology.org/operation_3182
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_0080
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# mummer-alignment-coordinates-extraction

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Validates contiguity of large merged metagenomic contigs (e.g., MeGAMerge output) by aligning them with nucmer to a reference genome, filtering to the best collinear alignments with delta-filter, and extracting tabular query/reference coordinates with show-coords to confirm colinearity.

## When to use

Use when you need to validate that large de novo contigs align colinearly to a known reference genome, e.g. checking that ~342.5 kb and ~472.5 kb MeGAMerge contigs from HMP sample SRS022071 match the B. helcogenes chromosome.

## When NOT to use

- When the goal is de novo assembly or merging of contigs — use the MeGAMerge/Minimus2 pipeline itself; nucmer alignment is only the downstream validation step
- When no reference genome is available — colinearity validation against a reference cannot be performed, and the article instead relies on read mapping, gene prediction, and similarity searches
- When contigs are expected to be chimeric — nucmer colinearity alone will not flag contigs that do not contiguously align to a single reference genome as chimeric without additional chimera-flagging analysis

## Inputs

- Reference genome FASTA (e.g., Bacteroides helcogenes P 36-108 chromosome from NCBI)
- Query contigs in Multi-FASTA format (MergedContigs.fasta from the MeGAMerge output directory)

## Outputs

- Nucmer delta alignment file (.delta)
- delta-filtered delta file retaining best collinear alignments per query-reference pair
- Tabular coordinates table (show-coords output) with reference start/end and query start/end per aligned segment

## How to apply

Obtain a reference genome FASTA (e.g., Bacteroides helcogenes P 36-108 from NCBI) and the query contigs from the merged assembly (MergedContigs.fasta). Run nucmer from the MUMmer executables directory (built in 64-bit mode for large projects) to produce a delta alignment file, then apply delta-filter to retain the best collinear alignments per query-reference pair. Convert the filtered delta with show-coords into a tabular coordinates table listing reference and query start/end positions for each aligned segment. Judge success by confirming hits against the reference, query lengths near the expected sizes (e.g., 342.5 kb and 472.5 kb), and colinearity along the contig length with gaps explained by genomic islands or dissimilar regions, matching Supplementary Figures S4-S5. Note the README requires AMOS to locate nucmer, delta-filter and show-coords via PATH or NUCMER/DELTAFILTER/SHOWCOORDS configure variables.

## Related tools

- **MUMmer** (Provides nucmer (whole-genome alignment of contigs to reference), delta-filter (best collinear alignment selection), and show-coords (tabular coordinate extraction); must be compiled 64-bit and visible to AMOS) — https://github.com/LANL-Bioinformatics/MeGAMerge
- **MeGAMerge** (Produces the merged contig set (MergedContigs.fasta) whose largest contigs are validated by this alignment workflow) — https://github.com/LANL-Bioinformatics/MeGAMerge
- **AMOS toolbox** (Minimus2 within AMOS invokes nucmer, delta-filter and show-coords; installation requires these MUMmer executables in PATH or NUCMER/DELTAFILTER/SHOWCOORDS configure variables) — https://github.com/LANL-Bioinformatics/MeGAMerge

## Examples

```
nucmer --prefix bh_vs_merged B_helcogenes.fasta MergedContigs.fasta; delta-filter -1 bh_vs_merged.delta | show-coords -T > coords.tsv
```

## Evaluation signals

- The show-coords table records hits against the expected Bacteroides reference genome
- The two query contig lengths fall near 342.5 kb and 472.5 kb as expected for the largest SRS022071 MeGAMerge contigs
- The filtered alignments are colinear along the entire contig length, with gaps only where genomic islands or dissimilar regions intervene, matching Supplementary Figures S4-S5
- delta-filter retains the best collinear alignment per query-reference pair, so no contradictory or overlapping-inconsistent segments appear in the coordinates table
- MUMmer was compiled in 64-bit mode (make all CPPFLAGS="-O3 -DSIXTYFOURBITS") for large genome projects, per the README installation notes

## Limitations

- Alignments show gaps where genomic islands or dissimilar regions intervene, so exact full-length colinearity is not always observed even for valid contigs
- AMOS installation depends on MUMmer executables being reachable: nucmer, delta-filter and show-coords must be in PATH or specified via NUCMER, DELTAFILTER, and SHOWCOORDS ./configure variables
- For larger genome projects, MUMmer must be compiled in 64-bit mode or alignments may fail
- This validation covers colinearity to a reference; contig validity conclusions are otherwise supported by read mapping (BWA/SAMtools), gene prediction, and similarity searches per the article

## Evidence

- [other] Run nucmer from the MUMmer executables directory to align the two contigs against the reference genome, producing a delta alignment file.: [evidence span withheld — anchor status: not_found]
- [other] Filter the delta file with delta-filter to retain the best collinear alignments per query-reference pair.: [evidence span withheld — anchor status: not_found]
