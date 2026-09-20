---
name: bacterial-genome-chimera-detection-by-alignment
description: Use when validating large assembled contigs (e.g. 342.5 kb and 472.5
  kb MeGAMerge contigs) against a reference genome to confirm colinearity or detect
  chimeric breakpoints.
when_to_use_negative:
- Input contigs are already dereplicated/validated and the goal is only assembly size
  or N50/N90 reporting — the article notes total assembly size and N50/N90 are 'not
  very useful for comparing metagenome assembly performance'.
- No suitable single-genome reference exists for the organism represented by the contigs,
  since contiguity/chimera calling depends on contiguously aligning to one reference
  genome.
- The goal is de novo merging of contigs rather than post-hoc validation — use MeGAMerge
  itself (Minimus2/Newbler OLC merging) instead.
edam_operation: http://edamontology.org/operation_3194
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_0196
tools:
- name: MeGAMerge
  role: Source of the merged contigs (MergedContigs.fasta) that are validated; its
    OLC merging with Minimus2 can introduce new chimeras that this alignment step
    detects
  repo: https://github.com/LANL-Bioinformatics/MeGAMerge
- name: Nucmer
  role: Aligns query contigs against the reference genome to produce delta alignments
    used for colinearity and chimera assessment; also used to obtain reference genome
    coverage
- name: MUMmer
  role: Provides nucmer, delta-filter, and show-coords; must be compiled in 64-bit
    mode (make all CPPFLAGS="-O3 -DSIXTYFOURBITS") and on PATH for AMOS/MeGAMerge
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/srep06480
    title: Improved Assemblies Using a Source-Agnostic Pipeline for MetaGenomic Assembly
      by Merging (MeGAMerge) of Contigs
  source_content_hash: sha256:c5f7877af4b4e84f1d5ed6ba5820590259eea7d772b86908bfb7101c4a028a12
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
  edam_operation: http://edamontology.org/operation_3194
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_0196
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# bacterial-genome-chimera-detection-by-alignment

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Validates contig contiguity and flags chimeras by aligning assembled metagenomic contigs to a known bacterial reference genome with MUMmer's nucmer, filtering to collinear hits with delta-filter, and tabulating coordinates with show-coords; contigs that fail to align contiguously to a single reference are flagged as chimeric.

## When to use

Use when validating large assembled contigs (e.g. 342.5 kb and 472.5 kb MeGAMerge contigs) against a reference genome to confirm colinearity or detect chimeric breakpoints.

## When NOT to use

- Input contigs are already dereplicated/validated and the goal is only assembly size or N50/N90 reporting — the article notes total assembly size and N50/N90 are 'not very useful for comparing metagenome assembly performance'.
- No suitable single-genome reference exists for the organism represented by the contigs, since contiguity/chimera calling depends on contiguously aligning to one reference genome.
- The goal is de novo merging of contigs rather than post-hoc validation — use MeGAMerge itself (Minimus2/Newbler OLC merging) instead.

## Inputs

- Reference bacterial genome FASTA (e.g. Bacteroides helcogenes P 36-108 from NCBI)
- Query contigs in Multi-FASTA format (MergedContigs.fasta from a MeGAMerge assembly)
- MUMmer executables (nucmer, delta-filter, show-coords) in $PATH or $bindir

## Outputs

- Delta alignment file from nucmer
- Delta-filtered collinear alignment set per query-reference pair
- Tabular coordinates table (reference/query start-end positions per aligned segment from show-coords)
- Chimera flags with breakpoint locations for contigs not aligning contiguously to a single reference

## How to apply

Obtain the reference genome FASTA (e.g. Bacteroides helcogenes P 36-108 from NCBI) and the query contigs from the merged assembly (MergedContigs.fasta). Run nucmer from the MUMmer executables directory to align each query contig against the reference, producing a delta file, then run delta-filter to retain the best collinear alignments per query-reference pair. Convert the filtered alignments with show-coords into a tabular coordinates table of reference and query start/end positions per aligned segment. Confirm contiguity if a contig is colinear along its entire length (gaps may correspond to genomic islands or dissimilar regions), and flag any contig that did not contiguously align to a single reference genome as chimeric, recording the chimera locations. Check that query lengths match expectations (e.g. ~342.5 kb and ~472.5 kb) to match Supplementary Figures S4-S5.

## Related tools

- **MeGAMerge** (Source of the merged contigs (MergedContigs.fasta) that are validated; its OLC merging with Minimus2 can introduce new chimeras that this alignment step detects) — https://github.com/LANL-Bioinformatics/MeGAMerge
- **Nucmer** (Aligns query contigs against the reference genome to produce delta alignments used for colinearity and chimera assessment; also used to obtain reference genome coverage)
- **MUMmer** (Provides nucmer, delta-filter, and show-coords; must be compiled in 64-bit mode (make all CPPFLAGS="-O3 -DSIXTYFOURBITS") and on PATH for AMOS/MeGAMerge)

## Examples

```
nucmer --prefix=Bhel_vs_contigs B_helcogenes.fna MergedContigs.fasta; delta-filter -g Bhel_vs_contigs.delta > filt.delta; show-coords -T filt.delta > coords.tsv
```

## Evaluation signals

- show-coords table records hits against the expected Bacteroides reference and query contig lengths fall near 342.5 kb and 472.5 kb, matching Supplementary Figures S4-S5.
- The 342.5 kb contig shows colinearity along its entire length with a portion of the B. helcogenes chromosome, with gaps only where genomic islands or dissimilar regions intervene.
- delta-filter output retains only the best collinear alignments per query-reference pair (no spurious multi-fragment hits for a single reference).
- Contigs that fail to align contiguously to a single reference are flagged as chimeric, with breakpoint locations identified.
- MUMmer was compiled in 64-bit mode and nucmer, delta-filter, and show-coords are resolvable in $PATH (or via NUCMER/DELTAFILTER/SHOWCOORDS in AMOS configure).

## Limitations

- The merging process introduces new chimeras, even though MeGAMerge reduces the total number of chimeras input into the process.
- Removing contigs smaller than 2 kb (as in the MeGAMerge pipeline) impacts total assembly size and N50/N90 metrics, so chimera/alignment checks on the merged set exclude short error-prone contigs.
- Overlap-based consensus tools' run-times increase exponentially with increasing input dataset size, limiting scalability of the upstream merging whose outputs are validated here.
- Alignment-based validation requires reference genomes; contigs from organisms without a reference cannot be assessed this way.

## Evidence
