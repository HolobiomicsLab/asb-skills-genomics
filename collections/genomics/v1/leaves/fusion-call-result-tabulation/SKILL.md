---
name: fusion-call-result-tabulation
description: Use when NanoFG/NanoSV/Sniffles candidate fusions must be converted into
  a per-sample table of gene pairs with supporting fusion-spanning read counts, filtered
  to calls with >=2 reads.
when_to_use_negative:
- Input is FFPE or otherwise fragmented DNA, since FUDGE requires non-fragmented DNA
  and will likely fail to comprehensively identify fusion genes from short degraded
  reads
- The goal is to count a single-read fusion call as reliable without validation —
  the paper explicitly requires manual breakpoint PCR for fusions detected with only
  one fusion-spanning read
- Only the WGA amplified material with unknown fusion partner is available, since
  WGA use is only useful for breakpoint localization when both fusion gene partners
  are already known
edam_operation: http://edamontology.org/operation_3227
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_0102
- http://edamontology.org/topic_3168
tools:
- name: NanoFG
  role: End-to-end pipeline that integrates mapping, SV calling, annotation and fusion
    calling to produce the per-sample fusion call records
  repo: https://github.com/SdeBlank/NanoFG
- name: Minimap2
  role: Maps the enriched nanopore FASTQ reads to the GRCh37 human reference genome
    (v2.6)
  repo: https://github.com/lh3/minimap2
- name: NanoSV
  role: Structural variant caller (v1.2.4, min_mapq=12, depth_support=False, mapq_flag=48)
    whose output VCF feeds the candidate-fusion tabulation
  repo: https://github.com/mroosmalen/nanosv
- name: Sniffles
  role: Alternative SV caller (v1.0.9, -s 2 -n -1 --genotype) producing the SV VCF
    used for fusion calling
- name: Samtools
  role: BAM/SAM handling within the NanoFG pipeline (v1.7)
  repo: http://samtools.sourceforge.net/
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-020-16641-7
    title: Partner independent fusion gene detection by multiplexed CRISPR-Cas9 enrichment
      and long read nanopore sequencing
  source_content_hash: sha256:dc7012a14f5e034626bcd95c3bd72ce9fb78f9a94814e21c08833551bc8e35e5
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/SdeBlank/NanoFG
  tools:
  - FUDGE
  - Minimap2
  - NanoFG
  - NanoSV
  - Samtools
  - Sniffles
  edam_operation: http://edamontology.org/operation_3227
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_0102
  - http://edamontology.org/topic_3168
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 15
---

# fusion-call-result-tabulation

## Summary

Tabulate fusion gene calls (sample, gene pair, fusion-spanning read count) from NanoFG analysis of FUDGE-enriched nanopore sequencing data, applying the paper's reliability cut-off of at least two fusion-spanning reads per call.

## When to use

Use when NanoFG/NanoSV/Sniffles candidate fusions must be converted into a per-sample table of gene pairs with supporting fusion-spanning read counts, filtered to calls with >=2 reads.

## When NOT to use

- Input is FFPE or otherwise fragmented DNA, since FUDGE requires non-fragmented DNA and will likely fail to comprehensively identify fusion genes from short degraded reads
- The goal is to count a single-read fusion call as reliable without validation — the paper explicitly requires manual breakpoint PCR for fusions detected with only one fusion-spanning read
- Only the WGA amplified material with unknown fusion partner is available, since WGA use is only useful for breakpoint localization when both fusion gene partners are already known

## Inputs

- Nanopore FASTQ reads from FUDGE CRISPR-Cas9 enriched samples (cell lines, tumor, blinded, WGA, multiplexed libraries)
- GRCh37-aligned BAM (minimap2 v2.6)
- Structural variant VCF from NanoSV v1.2.4 or Sniffles v1.0.9
- ENSEMBL gene annotations for SV-end gene assignment

## Outputs

- fusion_calls.tsv with columns: sample name, fusion gene pair, fusion-spanning read count
- List of retained fusion calls meeting the >=2 fusion-spanning-read cut-off

## How to apply

Run the NanoFG pipeline (github.com/SdeBlank/NanoFG) on the FUDGE-enriched nanopore FASTQ mapped to GRCh37 with minimap2 v2.6; SVs are called with NanoSV v1.2.4 (min_mapq=12, depth_support=False, mapq_flag=48) or Sniffles v1.0.9 (-s 2 -n -1 --genotype). Remove all SVs failing the built-in NanoSV/Sniffles filters plus all insertions from the VCF, then annotate both ends of each SV with ENSEMBL genes and flag SVs whose two ends fall in different genes as possible fusions. Retain only fusions able to produce a continuous transcript on the same strand, and count the fusion-spanning reads supporting each gene pair. Apply the paper's cut-off of at least two fusion-spanning reads to reliably detect a fusion without further validation; calls with only one supporting read need manual breakpoint PCR validation and should be flagged. Write sample name, fusion gene pair, and fusion-spanning read count for every retained call to a tabular file (e.g. fusion_calls.tsv).

## Related tools

- **NanoFG** (End-to-end pipeline that integrates mapping, SV calling, annotation and fusion calling to produce the per-sample fusion call records) — https://github.com/SdeBlank/NanoFG
- **Minimap2** (Maps the enriched nanopore FASTQ reads to the GRCh37 human reference genome (v2.6)) — https://github.com/lh3/minimap2
- **NanoSV** (Structural variant caller (v1.2.4, min_mapq=12, depth_support=False, mapq_flag=48) whose output VCF feeds the candidate-fusion tabulation) — https://github.com/mroosmalen/nanosv
- **Sniffles** (Alternative SV caller (v1.0.9, -s 2 -n -1 --genotype) producing the SV VCF used for fusion calling)
- **Samtools** (BAM/SAM handling within the NanoFG pipeline (v1.7)) — http://samtools.sourceforge.net/

## Examples

```
bash NanoFG.sh -f /path/to/fastqdir -n B1 -s SELECTION
```

## Evaluation signals

- Every row written to fusion_calls.tsv has a fusion-spanning read count of at least 2, matching the paper's reliability cut-off
- Each reported fusion has both SV ends annotated in different ENSEMBL genes and is compatible with a continuous same-strand transcript
- All insertions and SVs failing built-in NanoSV/Sniffles filters are absent from the output table
- Reproduced calls match reported results (e.g. KMT2A-MLLT6 in blinded sample B1 with 29 reads; BCR-ABL1 in blinded sample B2 with 27 reads)
- Output table contains exactly the three fields: sample name, fusion gene pair, and fusion-spanning read count, one row per retained call

## Limitations

- Fusions detected with only one fusion-spanning read fall below the reliability cut-off and required manual validation by breakpoint PCR
- WGA from low-input DNA is only useful for breakpoint localization when both fusion gene partners are known in advance
- On-target coverage decreases for low-throughput sequencing runs and/or more distal breakpoint events
- The assay has so far been demonstrated on ten recurrent fusion genes; expansion to arbitrary genes is possible but not demonstrated

## Evidence

- [discussion] We set a cut-off of at least two fusion-spanning reads to reliably detect a fusion gene without further validation.: "We set a cut-off of at least two fusion-spanning reads to reliably detect a fusion gene without further validation."
- [discussion] Candidate fusion flagging rule: "If both ends of the SV are positioned in different genes it was flagged as a possible fusion."
- [discussion] Same-strand transcript retention rule: "Only fusions that have the ability to produce a continuous transcript on the same strand were retained"
- [discussion] SV and insertion filter rule: "By default, all SVs that do not pass the built-in NanoSV or Sniffles filters are removed. In addition, all insertions are also removed from the VCF."
- [readme] Repository and run instructions: "bash NanoFG.sh -f </path/to/fastqdir>  [-n SAMPLE_NAME ] [-s SELECTION] [-cf] [-cc] [-df] [-dc]"
