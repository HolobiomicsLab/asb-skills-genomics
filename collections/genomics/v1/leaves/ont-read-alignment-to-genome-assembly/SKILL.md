---
name: ont-read-alignment-to-genome-assembly
description: Use when a draft or gap-filled long-read assembly (e.g., ntLink scaffolds
  incorporating raw ONT sequence) needs polishing, or when read-to-assembly alignments
  are required for downstream consensus correction and QUAST evaluation.
when_to_use_negative:
- The assembly is already polished or derived only from corrected reads (no raw long-read
  sequence was incorporated, e.g., outputs of Basic Protocol 1 overlap detection without
  gap-filling)
- The reads are not ONT data — minimap2's map-ont preset and this Racon workflow are
  specific to Oxford Nanopore reads
- A different scaffolding step (Tigmint-long correction or ntLink mapping per Alternate
  Protocols 1–2) is the actual goal rather than polishing
edam_operation: http://edamontology.org/operation_0525
edam_topics:
- http://edamontology.org/topic_0196
- http://edamontology.org/topic_0622
tools:
- name: minimap2
  role: Aligns ONT long reads to the gap-filled scaffolds using the map-ont preset
    (-a -t 5 -x map-ont) to produce the SAM alignment input for polishing
  repo: https://github.com/lh3/minimap2
- name: Racon
  role: Generates the Racon-polished consensus FASTA (-u -t 5) from the read-to-scaffold
    alignments
  repo: https://github.com/lbcb-sci/racon
- name: sra-tools
  role: Downloads the ONT long-read FASTQ for accession SRR10028109 from SRA
  repo: https://github.com/ncbi/sra-tools
- name: QUAST
  role: Computes contiguity and correctness statistics on the polished assembly for
    comparison with Table 4
  repo: https://github.com/ablab/quast
- name: ntLink
  role: Produced the gap-filled input scaffolds whose raw long-read gap sequences
    necessitate polishing
  repo: https://github.com/bcgsc/ntLink
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1002/cpz1.733
    title: 'ntLink: A Toolkit for <i>De Novo</i> Genome Assembly Scaffolding and Mapping
      Using Long Reads'
  source_content_hash: sha256:3b1921b0e05b0b867b22d9859053872c5d9e0f76d8995368f94f70ff2a05a176
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/lh3/minimap2
  tools:
  - QUAST
  - Racon
  - minimap2
  - ntLink
  - sra-tools
  edam_operation: http://edamontology.org/operation_0525
  edam_topics:
  - http://edamontology.org/topic_0196
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 7
---

# ont-read-alignment-to-genome-assembly

## Summary

Aligns ONT long reads to a genome assembly (e.g., ntLink gap-filled scaffolds) with minimap2's map-ont preset and uses the resulting read-to-scaffold alignments as input for Racon consensus polishing to correct errors introduced by raw long-read gap filling.

## When to use

Use when a draft or gap-filled long-read assembly (e.g., ntLink scaffolds incorporating raw ONT sequence) needs polishing, or when read-to-assembly alignments are required for downstream consensus correction and QUAST evaluation.

## When NOT to use

- The assembly is already polished or derived only from corrected reads (no raw long-read sequence was incorporated, e.g., outputs of Basic Protocol 1 overlap detection without gap-filling)
- The reads are not ONT data — minimap2's map-ont preset and this Racon workflow are specific to Oxford Nanopore reads
- A different scaffolding step (Tigmint-long correction or ntLink mapping per Alternate Protocols 1–2) is the actual goal rather than polishing

## Inputs

- Gap-filled scaffold assembly FASTA (celegans_flye.fa.k32.w100.z1000.ntLink.scaffolds.fa)
- ONT long-read FASTQ (SRA accession SRR10028109, downloaded via sra-tools)

## Outputs

- SAM file of long-read to scaffold alignments from minimap2
- Racon-polished assembly FASTA: celegans_flye.fa.k32.w100.z1000.ntLink.scaffolds.racon-polished.fa
- QUAST statistics report for the polished assembly (contiguity and correctness metrics)

## How to apply

Download the ONT long-read FASTQ for accession SRR10028109 with sra-tools. Align the reads to the target assembly — in Basic Protocol 2 the gap-filled scaffolds celegans_flye.fa.k32.w100.z1000.ntLink.scaffolds.fa — using minimap2 with the ONT-appropriate preset and flags '-a -t 5 -x map-ont', producing a SAM file. Feed that alignment file to Racon with '-u -t 5' together with the reads and the assembly to generate the consensus-polished FASTA celegans_flye.fa.k32.w100.z1000.ntLink.scaffolds.racon-polished.fa. The rationale is that ntLink gap-filling inserts raw, uncorrected ONT sequence into scaffold gaps, so Racon consensus from the same reads is needed to correct insertion errors before final quality assessment.

## Related tools

- **minimap2** (Aligns ONT long reads to the gap-filled scaffolds using the map-ont preset (-a -t 5 -x map-ont) to produce the SAM alignment input for polishing) — https://github.com/lh3/minimap2
- **Racon** (Generates the Racon-polished consensus FASTA (-u -t 5) from the read-to-scaffold alignments) — https://github.com/lbcb-sci/racon
- **sra-tools** (Downloads the ONT long-read FASTQ for accession SRR10028109 from SRA) — https://github.com/ncbi/sra-tools
- **QUAST** (Computes contiguity and correctness statistics on the polished assembly for comparison with Table 4) — https://github.com/ablab/quast
- **ntLink** (Produced the gap-filled input scaffolds whose raw long-read gap sequences necessitate polishing) — https://github.com/bcgsc/ntLink

## Examples

```
minimap2 -a -t 5 -x map-ont celegans_flye.fa.k32.w100.z1000.ntLink.scaffolds.fa ont_reads.fastq > aln.sam; racon -u -t 5 ont_reads.fastq aln.sam celegans_flye.fa.k32.w100.z1000.ntLink.scaffolds.fa > celegans_flye.fa.k32.w100.z1000.ntLink.scaffolds.racon-polished.fa
```

## Evaluation signals

- The polished FASTA celegans_flye.fa.k32.w100.z1000.ntLink.scaffolds.racon-polished.fa is generated as expected
- minimap2 runs with '-a -t 5 -x map-ont' and yields a valid SAM file of reads aligned to the gap-filled scaffolds
- Racon is invoked with '-u -t 5' and completes without error on the read-to-scaffold alignments
- QUAST produces a statistics report whose contiguity and correctness metrics are consistent with the values reported in Table 4 of the paper

## Limitations

- Polishing only corrects errors in raw long-read sequence incorporated during gap-filling; it does not address larger misassemblies — Tigmint-long correction (Alternate Protocol 2) is the article's mechanism for misassembly detection
- Parameters shown (-t 5 threads) reflect the C. elegans / Flye protocol scale; larger genomes may require adjusted thread counts and resources
- Validation relies on comparison with Table 4 reference statistics; deviations require investigation before trusting the polished assembly

## Evidence

- [methods] The protocol polishes the gap-filled scaffolds by first aligning the long reads with minimap2 (-a -t 5 -x map-ont) to produce a SAM file, then running Racon with -u -t 5: [evidence span withheld — anchor status: not_found]
- [abstract] ntLink is a flexible and resource-efficient genome scaffolding tool that utilizes long-read sequencing data to improve upon draft genome assemblies: "ntLink is a flexible and resource-efficient genome scaffolding tool that utilizes long-read sequencing data to improve upon draft genome assemblies"
- [readme] Consensus module for raw de novo DNA assembly of long uncorrected reads.: [evidence span withheld — anchor status: external_doc]
- [abstract] Recent improvements to ntLink have added important features such as overlap detection, gap-filling, and in-code scaffolding iterations: "Recent improvements to ntLink have added important features such as overlap detection, gap-filling, and in-code scaffolding iterations"
