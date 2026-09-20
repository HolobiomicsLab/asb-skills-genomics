---
name: rpm-normalization-of-small-rna-counts
description: Use when comparing tncRNA abundance across sRNA-seq samples or species
  — e.g., to express reads mapped to a progenitor tRNA as RPM, or to build cross-sample/tissue
  matrices from tRF-5, tRF-3, tRF-1, 5'tRH, and 3'tRH counts.
when_to_use_negative:
- Reads have not yet been restricted to unique tRNA-derived mappings — normalizing
  raw or multi-mapped counts inflates the denominator and RPM values
- Comparing differential expression across stress conditions where dedicated DE models
  (not simple per-million scaling) are required
- Analyzing RNAs outside the 14–50 nt tncRNA size range, where the total-mapped-reads
  denominator of this pipeline does not apply
edam_operation: http://edamontology.org/operation_3435
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3170
tools:
- name: tncRNA Toolkit
  role: Pipeline that maps reads with bowtie1 (-v 0 -m 50) and produces the mapped
    read counts and total-mapped-reads denominator used for RPM computation
  repo: https://github.com/skbinfo/tncRNA-Toolkit
- name: pandas
  role: Loads unique mapped tncRNA read counts and computes RPM = reads mapped to
    progenitor tRNA / (total mapped reads / 1e6), writing CSV/TSV tables
- name: bowtie
  role: bowtie1 '--norc -v 0 -m 50 --best --strata' uniquely maps trimmed reads to
    the artificial tRNA genome, defining the counts and the per-million scaling factor
- name: Trim Galore
  role: Quality- and adapter-trims single-end small RNA reads before mapping, so total
    mapped reads entering the RPM denominator are from clean libraries
  repo: https://github.com/FelixKrueger/TrimGalore
provenance:
  source_task_ids:
  - task_006
  - task_001
  source_papers:
  - doi: 10.1016/j.csbj.2021.09.021
    title: 'Transfer RNA-derived non-coding RNAs (tncRNAs): Hidden regulation of plants''
      transcriptional regulatory circuits'
  source_content_hash: sha256:4e9744e69b151a368dfd84f6c2a43319cab17952f71c65aa2fbd672a9c9fa532
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/skbinfo/tncRNA-Toolkit
  repo_url: https://github.com/skbinfo/tncRNA-Toolkit
  tools:
  - HAMR
  - Trim Galore
  - biopython
  - bowtie
  - pandas
  - tRNAscanSE
  - tncRNA Toolkit
  edam_operation: http://edamontology.org/operation_3435
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# RPM normalization of small RNA counts

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Normalizes tncRNA (tRF/tRNA-half) read counts to reads per million (RPM) so that tncRNA abundances from different sRNA-seq libraries across the six angiosperms are comparable. Counts are expressed relative to the total number of reads mapped to the tRNA-derived (artificial chromosome) reference.

## When to use

Use when comparing tncRNA abundance across sRNA-seq samples or species — e.g., to express reads mapped to a progenitor tRNA as RPM, or to build cross-sample/tissue matrices from tRF-5, tRF-3, tRF-1, 5'tRH, and 3'tRH counts.

## When NOT to use

- Reads have not yet been restricted to unique tRNA-derived mappings — normalizing raw or multi-mapped counts inflates the denominator and RPM values
- Comparing differential expression across stress conditions where dedicated DE models (not simple per-million scaling) are required
- Analyzing RNAs outside the 14–50 nt tncRNA size range, where the total-mapped-reads denominator of this pipeline does not apply

## Inputs

- SAM/BAM alignments of uniquely mapped tncRNA reads to the artificial tRNA genome (bowtie1 -v 0 -m 50 output)
- FASTA of unique mapped tncRNA reads with per-read counts extracted from SAM flags 0 & 16
- Mature tRNA annotations (tRNAscan-SE output, score >= 50, pseudogenes removed) with 50 bp 5'/3' flanks
- Total mapped read count per sRNA-seq library (post Trim Galore QC)

## Outputs

- RPM-normalized tncRNA abundance table per sample/species (tncRNA_counts.csv / TSV)
- Per-isoacceptor RPM values for cleavage-position nucleotide composition analysis
- Tissue x tncRNA RPM matrices (tissues with >= 10 samples) for t-SNE

## How to apply

After uniquely mapping trimmed sRNA reads to the artificial tRNA genome (bowtie1 '--norc -v 0 -m 50 --best --strata'), sum the reads mapped to each progenitor tRNA (extracted from SAM flags 0 and 16 for + and − strand with pandas) and compute the library scaling factor PMF = total mapped reads / 10^6. For each tncRNA, RPM = reads mapped to its progenitor tRNA / (total mapped reads / 1e6), i.e., per-million scaling against the tRNA-mapped fraction rather than raw library size. Apply this per sample/species before compiling subtype counts (tRF-5, tRF-3, tRF-1, 5'tRH, 3'tRH, leader-tRF, other tRF) into CSV/TSV tables with pandas, and before tissue matrices used for t-SNE clustering (tissues with at least 10 samples). Use the normalized values for downstream analyses such as cleavage-position nucleotide composition per isoacceptor (GluCTC, GlyTCC, LeuTAA, ProCGG) so abundance patterns reflect biology, not sequencing depth.

## Related tools

- **tncRNA Toolkit** (Pipeline that maps reads with bowtie1 (-v 0 -m 50) and produces the mapped read counts and total-mapped-reads denominator used for RPM computation) — https://github.com/skbinfo/tncRNA-Toolkit
- **pandas** (Loads unique mapped tncRNA read counts and computes RPM = reads mapped to progenitor tRNA / (total mapped reads / 1e6), writing CSV/TSV tables)
- **bowtie** (bowtie1 '--norc -v 0 -m 50 --best --strata' uniquely maps trimmed reads to the artificial tRNA genome, defining the counts and the per-million scaling factor)
- **Trim Galore** (Quality- and adapter-trims single-end small RNA reads before mapping, so total mapped reads entering the RPM denominator are from clean libraries) — https://github.com/FelixKrueger/TrimGalore

## Examples

```
RPM = reads_mapped_to_progenitor_tRNA / (total_mapped_reads / 1e6)  # pandas, per sample; PMF = total_mapped_reads/1e6
```

## Evaluation signals

- RPM values are finite and non-negative, and each equals reads-to-progenitor-tRNA divided by (total mapped reads / 1e6) when recomputed by hand
- Sum of RPM over all tncRNAs within a library is on the order of 10^6 divided by the number of distinct loci — i.e., scaling factor PMF = total mapped reads / 10^6 was applied consistently per sample
- All reported tncRNAs fall within the 14–50 nt size range identified from the 2,448 sRNA-seq samples
- All five subtype classes (tRF-5, tRF-3, tRF-1, 5'tRH, 3'tRH) are represented in the output count table
- Normalized per-isoacceptor cleavage-position nucleotide distributions are non-uniform and codon-dependent, consistent with specific rather than random tRNA cleavage

## Limitations

- RPM is computed against the tRNA-mapped read total (PMF = total mapped reads/10^6), so it is not comparable to miRNA RPM values normalized against all genome-mapped reads
- Conventional RNA-seq suffers from tRNA modification biases, which can distort mapped counts and hence RPM values; tRNA-seq would be needed to overcome them
- Experimental validation is still needed to distinguish bona fide tncRNAs, so RPM-abundant reads may include degradation artifacts

## Evidence

- [methods] RPM computed as reads mapped to progenitor tRNA / (total mapped reads / 1e6): [evidence span withheld — anchor status: not_found]
- [methods] PMF = total mapped reads/10^6: [evidence span withheld — anchor status: not_found]
