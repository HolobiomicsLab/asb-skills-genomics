---
name: fastq-quality-trimming-phred-threshold
description: Use when FASTQ reads with deteriorating 3'-end quality will be used for
  base-level correction or polishing (e.g. Pilon with Illumina data) and raw per-base
  qualities must be filtered to a Phred threshold first.
when_to_use_negative:
- Input is nanopore long-read data being corrected with Racon — Racon takes raw nanopore
  alignments, not sickle-trimmed Illumina reads
- Reads are already adapter- and quality-trimmed or come from a technology without
  quality decay toward the 3'-end
- You need assembly of long reads only (no Illumina polishing step is planned, so
  trimming serves no downstream correction)
edam_operation: http://edamontology.org/operation_3237
edam_topics:
- http://edamontology.org/topic_0080
- http://edamontology.org/topic_0196
tools:
- name: sickle
  role: Windowed adaptive FASTQ quality/length trimming of the Illumina reads prior
    to Pilon polishing
  repo: https://github.com/najoshi/sickle
- name: Pilon
  role: Consumes the sickle-trimmed Illumina reads to correct the Racon-corrected
    nanopore assembly
  repo: https://github.com/broadinstitute/pilon/wiki
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41598-017-07650-6
    title: Rapid de novo assembly of the European eel genome from nanopore sequencing
      reads
  source_content_hash: sha256:0f2bab6ccad16aaa39f1fecaa199d60ec6709f71f7ba15ceb055acd374c46dae
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/najoshi/sickle
  tools:
  - Pilon
  - Racon
  - sickle
  edam_operation: http://edamontology.org/operation_3237
  edam_topics:
  - http://edamontology.org/topic_0080
  - http://edamontology.org/topic_0196
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7272727272727273
n_claims: 11
---

# fastq-quality-trimming-phred-threshold

## Summary

Adaptive windowed quality trimming of FASTQ reads with sickle before downstream correction/polishing. In the eel nanopore-assembly pipeline, Illumina reads (SRR5235521–SRR5235523) are trimmed to a minimum Phred quality of 30 so Pilon polishing is fed high-confidence bases.

## When to use

Use when FASTQ reads with deteriorating 3'-end quality will be used for base-level correction or polishing (e.g. Pilon with Illumina data) and raw per-base qualities must be filtered to a Phred threshold first.

## When NOT to use

- Input is nanopore long-read data being corrected with Racon — Racon takes raw nanopore alignments, not sickle-trimmed Illumina reads
- Reads are already adapter- and quality-trimmed or come from a technology without quality decay toward the 3'-end
- You need assembly of long reads only (no Illumina polishing step is planned, so trimming serves no downstream correction)

## Inputs

- Paired-end Illumina FASTQ files (e.g. SRA accessions SRR5235521–SRR5235523 from a European eel specimen)
- Quality encoding specification (sanger/illumina/solexa; sanger for CASAVA >= 1.8)

## Outputs

- Two trimmed paired FASTQ files (forward and reverse)
- A 'singles' FASTQ containing reads where only one mate passed filtering
- Downstream: a polished assembly (via Pilon 1.21) with improved BUSCO scores (77.5% complete vs 10.8% after Racon alone)

## How to apply

Run sickle in paired-end mode (sickle pe) on the raw Illumina FASTQ pairs, specifying the quality encoding (sanger for CASAVA >= 1.8 data) and the minimum Phred threshold — the eel workflow uses q 30. Sickle slides a window of length 0.1x the read length along the quality string, trimming the 5'-end where average quality rises above threshold and the 3'-end where it drops below; reads shorter than the minimum length (-l) are discarded, and orphans are written to a separate 'singles' file. Use the trimmed reads as the Pilon input (Pilon 1.21 on Racon-corrected nanopore scaffolds in the eel pipeline) so polishing corrections rest on high-quality Illumina evidence. The rationale is that trimming low-quality tails improves the accuracy of short-read mappings that drive base corrections.

## Related tools

- **sickle** (Windowed adaptive FASTQ quality/length trimming of the Illumina reads prior to Pilon polishing) — https://github.com/najoshi/sickle
- **Pilon** (Consumes the sickle-trimmed Illumina reads to correct the Racon-corrected nanopore assembly) — https://github.com/broadinstitute/pilon/wiki

## Examples

```
sickle pe -f SRR5235521_1.fastq -r SRR5235521_2.fastq -t sanger -o trimmed_1.fastq -p trimmed_2.fastq -s singles.fastq -q 30 -l 20
```

## Evaluation signals

- Downstream BUSCO validation: the trimmed-read Pilon-corrected assembly must reach ~77.5% complete / 8.4% missing, and the twice-polished assembly ~79.8% complete / 7.3% missing (vertebrata_odb9, 2586 genes)
- Trimmed FASTQ records retain the same read identifiers as the input pairs, with no remaining low-quality 3'-tails
- A separate 'singles' file is produced containing mates that failed filtering in only one direction, preserving pair bookkeeping
- The quality encoding flag (-t sanger) matches the CASAVA version of the source FASTQ (CASAVA >= 1.8 is Sanger-encoded)

## Limitations

- Sickle discards reads entirely when the surviving length falls below the minimum length threshold, so overly aggressive thresholds shrink the dataset
- Trimmed Illumina data come from a different eel specimen than the nanopore sequencing, so the Pilon consensus can in theory mix up to four haplotypes
- In the absence of a high-quality reference it is not straightforward to establish which assembly correction is correct at apparent error sites

## Evidence

- [readme] sickle - A windowed adaptive trimming tool for FASTQ files using quality: [evidence span withheld — anchor status: external_doc]
- [readme] Sickle quality-window behaviour: [evidence span withheld — anchor status: external_doc]
- [discussion] Illumina data used for polishing a different specimen: "the nanopore data are therefore corrected using Illumina data obtained from a different eel specimen than used for nanopore sequencing"
- [readme] Sickle supports three quality encodings: [evidence span withheld — anchor status: external_doc]
