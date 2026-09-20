---
name: hybrid-genome-assembly-long-and-short-reads
description: Use when you have paired datasets of noisy third-generation long reads
  (PacBio/ONT) and accurate Illumina short reads from the same sample and need a fast,
  low-misasassembly de novo assembly of a genome up to human scale.
when_to_use_negative:
- Only one sequencing type is available (long reads alone or short reads alone) —
  HASLR is a hybrid tool requiring both
- A phased assembly of a diploid genome is required — the current HASLR version does
  not generate phased assemblies
- Ultra-long ONT reads are the primary data — ultra-long read support is listed as
  not yet implemented
edam_operation: http://edamontology.org/operation_0525
edam_topics:
- http://edamontology.org/topic_0196
- http://edamontology.org/topic_0622
tools:
- name: HASLR
  role: hybrid assembler orchestrating short-read contig anchoring, long-read threading,
    and consensus calling
  repo: https://github.com/vpc-ccg/haslr
- name: Minia
  role: assembling Illumina short reads into anchor contigs (k=49, filtered to >=250
    bp, no overlaps)
  repo: https://github.com/GATB/minia
- name: minimap2
  role: aligning short-read contigs onto long reads to build the backbone graph
  repo: https://github.com/lh3/minimap2
- name: SPOA
  role: consensus calling of long reads between anchors
  repo: https://github.com/rvaser/spoa
- name: fastutils
  role: FASTA/FASTQ manipulation utilities (stats, length filtering, subsampling)
    within the workflow
  repo: https://github.com/haghshenas/fastutils
- name: Bandage
  role: visualizing intermediate backbone graph GFA files
  repo: https://github.com/rrwick/Bandage
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1016/j.isci.2020.101389
    title: 'HASLR: Fast Hybrid Assembly of Long Reads'
  source_content_hash: sha256:2cacaecd9b2fa83275f8375bca6bdbb22c212bd328219523353bfeda2a7436be
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/vpc-ccg/haslr
  repo_url: https://github.com/vpc-ccg/haslr
  tools:
  - Bandage
  - HASLR
  - MaSuRCA
  - Minia
  - Oxford Nanopore
  - PBcR
  - Pilon
  - Racon
  - SPOA
  - fastutils
  - minimap2
  edam_operation: http://edamontology.org/operation_0525
  edam_topics:
  - http://edamontology.org/topic_0196
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.4375
n_claims: 16
---

# hybrid-genome-assembly-long-and-short-reads

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Assemble a genome de novo by combining error-prone long reads (PacBio or Oxford Nanopore) with high-quality Illumina short reads, following the HASLR strategy of short-read contigs anchoring a backbone graph onto which long reads are threaded and consensus-called for speed and low misassembly counts.

## When to use

Use when you have paired datasets of noisy third-generation long reads (PacBio/ONT) and accurate Illumina short reads from the same sample and need a fast, low-misasassembly de novo assembly of a genome up to human scale.

## When NOT to use

- Only one sequencing type is available (long reads alone or short reads alone) — HASLR is a hybrid tool requiring both
- A phased assembly of a diploid genome is required — the current HASLR version does not generate phased assemblies
- Ultra-long ONT reads are the primary data — ultra-long read support is listed as not yet implemented

## Inputs

- Illumina short reads (FASTQ, paired-end) from the same sample as the long reads
- PacBio or Oxford Nanopore long reads (FASTQ/FASTA)
- Approximate genome size (with k/m/g suffixes, e.g. -g 4.6m)
- Long-read technology preset choice (pacbio or nanopore)

## Outputs

- Assembled contigs (FASTA) with consensus sequences called by SPOA between short-read contig anchors
- Intermediate backbone graph files (GFA) after weak-edge, tip, simple-bubble, super-bubble, and small-bubble removal (e.g. backbone.02.weakEdge.gfa ... backbone.06.smallbubble.gfa)
- Selected longest 25x coverage long-read subset (e.g. lr25x.fasta)
- Step logs such as log_consensus.txt and log_coordinate.txt

## How to apply

First assemble the Illumina short reads with Minia using k=49 (yielding contigs like sr_k49_a3.contigs.nooverlap.250.fa, filtered to at least 250 bp without overlaps) to build unique short-read contig anchors. Align these short-read contigs onto the long reads with minimap2, applying a minimum alignment block length of 500 bp (the --aln-block default), and build a backbone graph, progressively cleaning it by removing weak edges, tips, simple bubbles, super bubbles, and small bubbles. Select the longest 25x coverage of long reads (the --cov-lr default of 25) to thread between anchors, compute long-read coordinates between anchors, and call consensus with SPOA to produce the assembly. Pass the long-read technology via -x {pacbio,nanopore} and the genome size via -g with k/m/g suffixes; optionally keep intermediate backbone graph GFA files to inspect connectivity with Bandage.

## Related tools

- **HASLR** (hybrid assembler orchestrating short-read contig anchoring, long-read threading, and consensus calling) — https://github.com/vpc-ccg/haslr
- **Minia** (assembling Illumina short reads into anchor contigs (k=49, filtered to >=250 bp, no overlaps)) — https://github.com/GATB/minia
- **minimap2** (aligning short-read contigs onto long reads to build the backbone graph) — https://github.com/lh3/minimap2
- **SPOA** (consensus calling of long reads between anchors) — https://github.com/rvaser/spoa
- **fastutils** (FASTA/FASTQ manipulation utilities (stats, length filtering, subsampling) within the workflow) — https://github.com/haghshenas/fastutils
- **Bandage** (visualizing intermediate backbone graph GFA files) — https://github.com/rrwick/Bandage

## Examples

```
haslr.py -t 64 -x nanopore -g 4.6m -l ont_reads.fastq.gz -s1 illumina_r1.fastq.gz -s2 illumina_r2.fastq.gz
```

## Evaluation signals

- QUAST metrics (NGA50, genome fraction, number of misassemblies) should show low misassembly counts with contiguity/accuracy on par with comparators like PBcR and MaSuRCA
- Wall-clock runtime should be markedly lower than comparator assemblers (e.g. CHM1 human assembled in less than 10 hours on 64 CPU threads)
- The longest 25x long-read subset (lr25x.fasta) should be produced per the default --cov-lr 25 setting
- Short-read contigs should meet the >=250 bp, no-overlap filter before backbone construction
- Backbone graph connectivity should be checked for multiple connected components, which arise when unique short-read contigs are missing

## Limitations

- HASLR usually has a higher indel rate compared with other tools, and mismatch/indel errors may require further polishing (e.g. with Pilon or Racon)
- HASLR sometimes produces lower genome fraction and/or NGA50 than other assemblers, most clearly versus Canu on the human genome
- Regions with high heterozygosity are not resolved because Minia short-read contigs there are fragmented and filtered out
- Backbone graph connectivity depends on the existence of unique short-read contigs; their absence yields multiple connected components
- Longer long reads are an important factor in assembly contiguity; short/fragmented long reads yield a less connected backbone graph

## Evidence
