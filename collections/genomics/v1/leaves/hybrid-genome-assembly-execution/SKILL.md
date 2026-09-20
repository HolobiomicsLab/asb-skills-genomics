---
name: hybrid-genome-assembly-execution
description: Use when you have error-prone PacBio/ONT long reads plus Illumina short
  reads from the same sample and need a fast, low-misassembly draft assembly with
  a genome-size estimate available.
when_to_use_negative:
- Only short reads are available — HASLR is a hybrid tool requiring third-generation
  long reads; use a short-read-only assembler instead.
- A polished, low-indel-rate finished assembly is required without a post-hoc polishing
  step — HASLR employs no polishing and typically shows a higher indel rate (fixable
  with Racon/Pilon).
- A phased assembly of a diploid genome is needed — the current HASLR version does
  not generate phased assemblies, and high-heterozygosity regions may remain unresolved
  because Minia short-read contigs there are fragmented and filtered out.
edam_operation: http://edamontology.org/operation_0525
edam_topics:
- http://edamontology.org/topic_0622
tools:
- name: HASLR
  role: Main hybrid assembler invoked via haslr.py; orchestrates SR contig assembly,
    LR subsampling, alignment, backbone graph construction, and consensus
  repo: https://github.com/vpc-ccg/haslr
- name: Minia
  role: Assembles Illumina short reads into contigs (k=49, abundance 3) that form
    the unique-contig backbone
  repo: https://github.com/GATB/minia
- name: minimap2
  role: Aligns short-read contigs onto long reads to derive graph connections (--aln-block
    500, --aln-sim 0.85)
  repo: https://github.com/lh3/minimap2
- name: SPOA
  role: Consensus calling of long reads between anchor contigs
  repo: https://github.com/rvaser/spoa
- name: fastutils
  role: FASTA/Q manipulation including subsampling long reads to the longest 25x coverage
  repo: https://github.com/haghshenas/fastutils
- name: Bandage
  role: Visualization of backbone GFA graphs for topology inspection against the reference
  repo: https://github.com/rrwick/Bandage
provenance:
  source_task_ids:
  - task_001
  - task_006
  - task_008
  - task_007
  source_papers:
  - doi: 10.1016/j.isci.2020.101389
    title: 'HASLR: Fast Hybrid Assembly of Long Reads'
  source_content_hash: sha256:ca632b091b445bb7972faef054c37eb6ad30a36228342ebd908d07feb8251aa5
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
  - ART
  - Bandage
  - HASLR
  - Minia
  - PBSIM
  - Pilon
  - QUAST
  - Racon
  - SPOA
  - fastutils
  - minimap2
  edam_operation: http://edamontology.org/operation_0525
  edam_topics:
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5
n_claims: 12
---

# hybrid-genome-assembly-execution

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Executes hybrid de novo genome assembly with HASLR, which combines error-prone third-generation long reads (PacBio/ONT) with high-quality Illumina short reads to rapidly produce accurate assemblies, with the lowest misassembly counts and fastest runtime among compared hybrid assemblers.

## When to use

Use when you have error-prone PacBio/ONT long reads plus Illumina short reads from the same sample and need a fast, low-misassembly draft assembly with a genome-size estimate available.

## When NOT to use

- Only short reads are available — HASLR is a hybrid tool requiring third-generation long reads; use a short-read-only assembler instead.
- A polished, low-indel-rate finished assembly is required without a post-hoc polishing step — HASLR employs no polishing and typically shows a higher indel rate (fixable with Racon/Pilon).
- A phased assembly of a diploid genome is needed — the current HASLR version does not generate phased assemblies, and high-heterozygosity regions may remain unresolved because Minia short-read contigs there are fragmented and filtered out.

## Inputs

- Long-read FASTA/FASTQ file from PacBio or Oxford Nanopore sequencing (same sample as short reads)
- Paired-end Illumina FASTQ files (e.g., ecoli_miseq.1.fastq.gz, ecoli_miseq.2.fastq.gz)
- Estimated genome size (passed via -g, e.g., 4.6m for E. coli MG1655)
- Long-read technology flag (pacbio or ont)
- Thread count and output directory

## Outputs

- asm.final.fa — final hybrid assembly contigs
- backbone.01.init.gfa … backbone.06.smallbubble.gfa — staged backbone graphs after weak-edge, tip, simple-bubble, super-bubble, and small-bubble removal
- lr25x.fasta — longest 25x coverage of long reads used for assembly
- sr_k49_a3.contigs.nooverlap.250.fa — short-read contigs at least 250 bp without overlaps
- log_consensus.txt and log_coordinate.txt — logs of consensus calling and long-read coordinate calculation between anchors

## How to apply

Install HASLR via bioconda or build from source; it depends on Minia (short-read assembly, k=49, min abundance 3), minimap2 (aligning short-read contigs onto long reads), SPOA (consensus of long reads), and fastutils (FASTA/Q manipulation). Run haslr.py specifying the estimated genome size with -g, the long-read type with -x (pacbio or ont), the long-read file with -l, the paired short reads with -s, and threads with -t. By default HASLR subsamples to the longest 25x coverage of long reads (--cov-lr 25), filters short-read contigs to at least 250 bp without overlaps, retains alignments with --aln-block 500 and --aln-sim 0.85, and requires --edge-sup 3 long reads per backbone-graph edge before simplifying weak edges, tips, simple bubbles, super bubbles, and small bubbles. The final assembly is emitted as asm.final.fa with intermediate backbone graph stages (backbone.01.init.gfa through backbone.06.smallbubble.gfa) available for inspection in Bandage.

## Related tools

- **HASLR** (Main hybrid assembler invoked via haslr.py; orchestrates SR contig assembly, LR subsampling, alignment, backbone graph construction, and consensus) — https://github.com/vpc-ccg/haslr
- **Minia** (Assembles Illumina short reads into contigs (k=49, abundance 3) that form the unique-contig backbone) — https://github.com/GATB/minia
- **minimap2** (Aligns short-read contigs onto long reads to derive graph connections (--aln-block 500, --aln-sim 0.85)) — https://github.com/lh3/minimap2
- **SPOA** (Consensus calling of long reads between anchor contigs) — https://github.com/rvaser/spoa
- **fastutils** (FASTA/Q manipulation including subsampling long reads to the longest 25x coverage) — https://github.com/haghshenas/fastutils
- **Bandage** (Visualization of backbone GFA graphs for topology inspection against the reference) — https://github.com/rrwick/Bandage

## Examples

```
haslr.py -t 8 -o outdir -g 4.6m -l ecoli_pacbio.fastq.gz -x pacbio -s ecoli_miseq.1.fastq.gz ecoli_miseq.2.fastq.gz
```

## Evaluation signals

- asm.final.fa is produced and is valid FASTA with contigs spanning the expected genome size (-g estimate).
- The staged output chain exists: lr25x.fasta contains the longest 25x of long reads, and sr_k49_a3.contigs.nooverlap.250.fa contains short-read contigs ≥250 bp with no overlaps.
- backbone.06.smallbubble.gfa is valid GFA and, when rendered in Bandage, shows connected backbone components consistent with the reference chromosome structure.
- QUAST evaluation of asm.final.fa against the reference shows low misassembly counts and runtime on par with or better than other hybrid assemblers, with NGA50 and mismatch rates comparable to the best tools.
- Log files (log_consensus.txt, log_coordinate.txt) confirm consensus calling and long-read coordinate steps completed for the anchors.

## Limitations

- HASLR has no polishing step, so assemblies usually show a higher indel rate than other tools; one round of targeted Racon/Pilon polishing on SPOA-derived inter-contig regions brings mismatch and indel rates to parity (Table S4).
- Backbone connectivity depends on unique SR contigs; regions lacking them yield multiple connected components, and high-heterozygosity regions can go unresolved because Minia contigs there are fragmented and filtered.
- The current version does not generate phased assemblies for diploid genomes and does not support ultra-long ONT reads; assemblies sometimes have lower genome fraction/NGA50 than Canu, especially on human genomes.
- Assemblies are typically < 50 kb contigs; unused long reads are discarded rather than assembled via an OLC fallback (e.g., miniasm).

## Evidence
