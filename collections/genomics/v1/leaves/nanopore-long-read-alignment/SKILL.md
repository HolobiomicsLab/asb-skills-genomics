---
name: nanopore-long-read-alignment
description: Use when long nanopore FASTQ reads from targeted (e.g. FUDGE-enriched)
  sequencing must be aligned to a human reference to support downstream SV/fusion
  calling with NanoSV, Sniffles or NanoFG.
when_to_use_negative:
- Input consists of short Illumina paired-end reads — minimap2's 'sr' preset exists,
  but the NanoFG fusion-detection workflow requires long nanopore reads to span fusion
  breakpoints
- DNA is fragmented (e.g. FFPE material), where the article notes short read lengths
  will fail to comprehensively identify fusion genes
- The analysis goal is de novo genome assembly rather than reference-guided variant/fusion
  calling
edam_operation: http://edamontology.org/operation_0292
edam_topics:
- http://edamontology.org/topic_0622
tools:
- name: minimap2
  role: 'Primary aligner: maps nanopore FASTQ reads to the GRCh37 human reference
    genome (v2.6, ONT presets)'
  repo: https://github.com/lh3/minimap2
- name: Samtools
  role: Sorts and indexes the minimap2 alignment output so it can be consumed by NanoSV
    and NanoFG
- name: NanoFG
  role: Pipeline wrapper that invokes minimap2 on the input FASTQ and manages tool
    paths via paths.ini
  repo: https://github.com/SdeBlank/NanoFG
- name: NanoSV
  role: Downstream consumer of the sorted BAM for structural-variant calling on the
    aligned long reads
  repo: https://github.com/mroosmalen/nanosv
merged_aliases:
- long-read-alignment-nanopore-genome
merged_alias_records:
- alias: long-read-nanopore-read-mapping
  slug: long-read-nanopore-read-mapping
  jaccard_score: 0.6
  method: token-set-jaccard
  decision: needs_review
provenance:
  source_task_ids:
  - task_002
  - task_003
  - task_001
  source_papers:
  - doi: 10.1038/s41467-020-16641-7
    title: Partner independent fusion gene detection by multiplexed CRISPR-Cas9 enrichment
      and long read nanopore sequencing
  source_content_hash: sha256:f34d0fb9dec5f331ed1070b6457c1ad4477e8e5346938ec806b5c9f81e307ead
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/lh3/minimap2
  tools:
  - FUDGE
  - LAST
  - Minimap2
  - NanoFG
  - NanoSV
  - RetroInspector
  - SAMtools
  - SVIM
  - Samtools
  - Sniffles
  - Sniffles2
  - Wtdbg2
  - cuteSV
  - minimap2
  edam_operation: http://edamontology.org/operation_0292
  edam_topics:
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7272727272727273
n_claims: 11
---

# nanopore-long-read-alignment

## Summary

Maps enriched Oxford Nanopore long reads against the GRCh37 human reference genome with minimap2 (v2.6) and indexes/sorts the resulting BAM with Samtools, producing the alignment file that NanoSV/Sniffles screen for structural variants and NanoFG refines into fusion-gene calls.

## When to use

Use when long nanopore FASTQ reads from targeted (e.g. FUDGE-enriched) sequencing must be aligned to a human reference to support downstream SV/fusion calling with NanoSV, Sniffles or NanoFG.

## When NOT to use

- Input consists of short Illumina paired-end reads — minimap2's 'sr' preset exists, but the NanoFG fusion-detection workflow requires long nanopore reads to span fusion breakpoints
- DNA is fragmented (e.g. FFPE material), where the article notes short read lengths will fail to comprehensively identify fusion genes
- The analysis goal is de novo genome assembly rather than reference-guided variant/fusion calling

## Inputs

- FASTQ file (nanopore reads, e.g. FUDGE-enriched or NanoFG test-set FASTQ)
- GRCh37 human reference genome (FASTA or pre-built minimap2 .mmi index)
- Optionally a pre-existing BAM and VCF passed to NanoFG via -b/-v

## Outputs

- SAM/BAM alignment file of nanopore reads against GRCh37
- Coordinate-sorted, Samtools-indexed BAM ready for NanoSV/Sniffles SV calling

## How to apply

Clone and build minimap2 from the lh3/minimap2 repository, then align the enriched nanopore FASTQ to the GRCh37 human reference using the map-ont preset for ONT reads ('minimap2 -ax map-ont ref.fa reads.fq'), which produces a SAM/BAM with base-level alignment. Sort and index the resulting BAM with Samtools (v1.7) so the coordinate-sorted file can serve as input both to NanoSV (v1.2.4; min_mapq=12, depth_support=False, mapq_flag=48) and to NanoFG's read-extraction step. In the NanoFG pipeline, tool paths are configured via paths.ini and the mapping step is invoked automatically through 'bash NanoFG.sh -f <fastqdir> -n <sample>'; alternatively an existing BAM (-b) and VCF (-v) can be supplied. Rationale: long, noisy ONT reads (error rate up to ~15%) are the query class minimap2 is explicitly tuned for, and accurate coordinate-sorted alignments are the prerequisite for NanoSV breakpoint calling and the later LAST remapping pass that refines fusion breakpoints at nucleotide resolution.

## Related tools

- **minimap2** (Primary aligner: maps nanopore FASTQ reads to the GRCh37 human reference genome (v2.6, ONT presets)) — https://github.com/lh3/minimap2
- **Samtools** (Sorts and indexes the minimap2 alignment output so it can be consumed by NanoSV and NanoFG)
- **NanoFG** (Pipeline wrapper that invokes minimap2 on the input FASTQ and manages tool paths via paths.ini) — https://github.com/SdeBlank/NanoFG
- **NanoSV** (Downstream consumer of the sorted BAM for structural-variant calling on the aligned long reads) — https://github.com/mroosmalen/nanosv

## Examples

```
bash NanoFG.sh -f /path/to/fastqdir -n sample  # pipeline maps reads internally with minimap2 -ax map-ont GRCh37.fa reads.fq | samtools sort
```

## Evaluation signals

- The output BAM is coordinate-sorted and indexed (Samtools index succeeds) before any SV caller is run
- Alignment rate is consistent with an enriched nanopore library (the paper reports ~68x average breakpoint-spanning coverage from ~25,000 reads per NanoFG run)
- Feeding the BAM to NanoSV produces a VCF whose entries pass the built-in caller filters (min_mapq=12, mapq_flag=48), with insertions subsequently removed
- On the bundled NanoFG test set (~25,000 reads), the downstream fusion report exists and reports exact breakpoint locations supported by at least two fusion-spanning reads

## Limitations

- Applying the FUDGE/NanoFG protocol to fragmented DNA (e.g. FFPE material) will most likely fail to comprehensively identify fusion genes because read lengths are too short
- Decreased on-target coverage was observed for low-throughput sequencing runs and/or more distal breakpoint events
- Whole genome amplification (Repli-g mini kit) for low-input DNA is only useful with prior knowledge of both fusion gene partners

## Evidence

- [discussion] Reads were mapped to the human reference genome version GRCHh37 by using minimap2 (v. 2.6): "Reads were mapped to the human reference genome version GRCHh37 by using minimap2 (v. 2.6)"
- [methods] Minimap2 (2.6) - https://github.com/lh3/minimap2: "Minimap2 (2.6) - https://github.com/lh3/minimap2"
- [readme] Minimap2 is a versatile sequence alignment program that aligns DNA or mRNA sequences against a large reference database.: "Minimap2 is a versatile sequence alignment program that aligns DNA or mRNA
sequences against a large reference database."
- [discussion] Run time of NanoFG on +−25000 nanopore reads is approximately 20 min using a single thread.: "The run time of NanoFG on +−25000 nanopore reads is approximately 20 min using a single thread."
