---
name: multi-mapped-read-filtering
description: Use when mapping quality-filtered, adapter-trimmed sRNA-seq reads to
  a tRNA-enriched (artificial-chromosome) genome index and ambiguous multi-locus alignment
  would otherwise inflate tncRNA counts or confuse subtype assignment.
when_to_use_negative:
- When the goal is to quantify multi-copy repeat-derived small RNAs where multi-mapping
  signal is biologically meaningful and fractional assignment (e.g. EM-based multi-mappers)
  is preferable
- When reads come from long RNA-seq or paired-end data with splice-aware mapping requirements
  — this Bowtie1-based small-RNA workflow assumes short single-end sRNA reads
- When the user needs base-modification-aware alignment of tRNAs (conventional RNA-seq
  suffers from tRNA modification biases; tRNA-seq is recommended instead)
edam_operation: http://edamontology.org/operation_3198
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3170
tools:
- name: Bowtie1
  role: Short-read aligner whose '-m 50' argument suppresses all alignments for reads
    mapping to more than 50 loci, discarding multi-mapped reads; bundled in the tncRNA
    Toolkit util directory (v1.3.0)
  repo: https://github.com/skbinfo/tncRNA-Toolkit
- name: tncRNA Toolkit
  role: Wraps Bowtie1 alignment and exposes the multi-mapping suppression threshold
    via the '-m' option (default 50) and mismatch allowance via '-v' (default 0)
  repo: https://github.com/skbinfo/tncRNA-Toolkit
- name: samtools
  role: Used to fetch mapped reads from the resulting SAM/BAM file (flags 0 & 16)
    after multi-mapped reads have been suppressed
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1016/j.csbj.2021.09.021
    title: 'Transfer RNA-derived non-coding RNAs (tncRNAs): Hidden regulation of plants''
      transcriptional regulatory circuits'
  source_content_hash: sha256:22896a5207769322684a8fa2f2e5aac2635491ff8fcf6c97dd1a10b73cd1fad6
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
  - Bowtie1
  - biopython
  - samtools
  - tncRNA Toolkit
  edam_operation: http://edamontology.org/operation_3198
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 6
---

# multi-mapped-read-filtering

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Suppress and discard reads that align to too many genomic loci when profiling tRNA-derived non-coding RNAs, using Bowtie1's -m option (default: discard alignments occurring more than 50 times) so that tncRNA calls and quantification (RPM) are not inflated by ambiguously mapped small RNAs.

## When to use

Use when mapping quality-filtered, adapter-trimmed sRNA-seq reads to a tRNA-enriched (artificial-chromosome) genome index and ambiguous multi-locus alignment would otherwise inflate tncRNA counts or confuse subtype assignment.

## When NOT to use

- When the goal is to quantify multi-copy repeat-derived small RNAs where multi-mapping signal is biologically meaningful and fractional assignment (e.g. EM-based multi-mappers) is preferable
- When reads come from long RNA-seq or paired-end data with splice-aware mapping requirements — this Bowtie1-based small-RNA workflow assumes short single-end sRNA reads
- When the user needs base-modification-aware alignment of tRNAs (conventional RNA-seq suffers from tRNA modification biases; tRNA-seq is recommended instead)

## Inputs

- Quality-filtered and adapter-trimmed single-end small RNA FASTQ reads
- Bowtie1 index of the artificial genome (masked genuine tRNA gene regions with 50 nt upstream/downstream as artificial chromosomes)
- Mature tRNA + 50 bp leader/trailer feature FASTA for the second-pass alignment

## Outputs

- Set of reads with suppressed/discarded multi-mapped alignments (reads aligning >50 loci removed)
- SAM/BAM of uniquely and permissibly mapped reads (samtools flags 0 & 16) suitable for tncRNA subtype classification
- FASTA of unique mapped reads with per-sequence counts feeding RPM quantification

## How to apply

After adapter trimming (e.g. Trim Galore) and genome-index building, align processed reads to the artificial genome index with Bowtie1 using '-v 2 --best' for error-tolerant alignment to the artificial genome, then re-align unique mapped reads to mature tRNA plus 50 bp leader/trailer features with '--norc -v 0 -m 50 --best --strata', keeping plus-strand zero-mismatch alignments. The '-m 50' argument suppresses all alignments for a read if more than 50 exist, so multi-mapped reads are discarded; the tncRNA Toolkit exposes this as the '-m' option (default 50) alongside '-v' for mismatches (default 0, recommended not to exceed 3). This prevents reads matching numerous tRNA copies or repetitive loci from being spuriously assigned a tncRNA subtype. Retained reads are then classified into subtypes (tRF-5, tRF-3, tRF-1/tsRNA, 5'/3' tRHs) and quantified as RPM against the total-mapped-reads Per Million Factor.

## Related tools

- **Bowtie1** (Short-read aligner whose '-m 50' argument suppresses all alignments for reads mapping to more than 50 loci, discarding multi-mapped reads; bundled in the tncRNA Toolkit util directory (v1.3.0)) — https://github.com/skbinfo/tncRNA-Toolkit
- **tncRNA Toolkit** (Wraps Bowtie1 alignment and exposes the multi-mapping suppression threshold via the '-m' option (default 50) and mismatch allowance via '-v' (default 0)) — https://github.com/skbinfo/tncRNA-Toolkit
- **samtools** (Used to fetch mapped reads from the resulting SAM/BAM file (flags 0 & 16) after multi-mapped reads have been suppressed)

## Examples

```
python3 tncRNAs.py -s <species name> -i <processed small RNA reads> -o <output dir> -v 0 -m 50
```

## Evaluation signals

- Every reported tncRNA maps exclusively to the artificial tRNA chromosomes (tRNA +/- 50 nt flank)
- The number of discarded reads equals the number of reads with more than 50 Bowtie1 alignments; no retained read exceeds the '-m' threshold
- Each surviving read carries a single tncRNA subtype assignment and passes the default read-count cut-off of 10
- Aligned reads are plus-strand, zero-mismatch ('-v 0') alignments to mature tRNA +/- 50 nt features
- RPM values are computed as reads mapped to progenitor tRNA / (total mapped reads / 10^6), using only post-filtering mapped reads

## Limitations

- Allowing a single mismatch to overcome sequencing errors may lead to base misidentification and inflation of false negatives; the toolkit defaults to zero mismatches and recommends not exceeding 3
- The multi-mapping threshold, mismatch allowance, and read-count cut-off are user-adjustable, so results depend on chosen parameters
- HAMR modification prediction (used downstream) requires python2, adding an environment constraint
- Genome FASTA headers must start with '>chr[Num]' (and '>chrMt'/'>chrPt' for organellar) for the automation to work

## Evidence
