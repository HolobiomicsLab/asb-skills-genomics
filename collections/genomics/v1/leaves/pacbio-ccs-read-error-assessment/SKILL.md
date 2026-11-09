---
name: pacbio-ccs-read-error-assessment
description: Use when you need to estimate nucleotide-level error rates of de novo
  transcript consensus sequences from PacBio Iso-Seq data, using independent Illumina
  short-read validation (≥2 agreeing reads per position).
when_to_use_negative:
- When no matching Illumina library exists for a primer pair-sample combination, since
  the ≥2-agreeing-read criterion cannot be evaluated for those transcripts
- When the goal is assigning transcripts to individual gene copies — the paper states
  IsoCon/transcriptome-only sequencing 'does not allow us to conclusively assign each
  transcript to a gene copy'
- When reads have not been pre-processed with the Iso-Seq pipeline to remove primers,
  barcodes, and chimeras, since the methods assume cleaned FLNC input
edam_topics:
- http://edamontology.org/topic_0102
tools:
- name: IsoCon
  role: Produces the primary candidate transcripts (final_candidates.fa) whose Illumina-supported
    positions are quantified in this validation
  repo: https://github.com/ksahlin/IsoCon
- name: ICE
  role: PacBio's standard de novo error-correction pipeline whose consensus transcripts
    serve as the comparison baseline
- name: proovread
  role: Illumina-based short-read correction approach used to generate the Illumina-corrected
    PacBio transcript set
- name: Illumina
  role: Nextera XT MiSeq short reads providing the independent ≥2-agreeing-read ground-truth
    signal for validation
- name: PacBio
  role: Long Iso-Seq/CCS reads supplying the original uncorrected transcript set being
    benchmarked
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-018-06910-x
    title: Deciphering highly similar multigene family transcripts from Iso-Seq data
      with IsoCon
  source_content_hash: sha256:4aa8bd27f13b76081c0aa375da8a51b45b324b697d263b557e309ad294102eb7
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ksahlin/IsoCon
  tools:
  - ICE
  - Illumina
  - IsoCon
  - PacBio
  - proovread
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0102
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 10
---

# pacbio-ccs-read-error-assessment

## Summary

Quantifies nucleotide-level accuracy of predicted Iso-Seq transcript consensus sequences (IsoCon, ICE, Illumina-corrected CCS, and original CCS reads) by counting positions supported by at least two agreeing Illumina reads. This reproduces the Fig. 3 validation from the IsoCon paper comparing error rates across the four transcript-calling approaches.

## When to use

Use when you need to estimate nucleotide-level error rates of de novo transcript consensus sequences from PacBio Iso-Seq data, using independent Illumina short-read validation (≥2 agreeing reads per position).

## When NOT to use

- When no matching Illumina library exists for a primer pair-sample combination, since the ≥2-agreeing-read criterion cannot be evaluated for those transcripts
- When the goal is assigning transcripts to individual gene copies — the paper states IsoCon/transcriptome-only sequencing 'does not allow us to conclusively assign each transcript to a gene copy'
- When reads have not been pre-processed with the Iso-Seq pipeline to remove primers, barcodes, and chimeras, since the methods assume cleaned FLNC input

## Inputs

- IsoCon predicted transcripts (final_candidates.fa from 'IsoCon pipeline -fl_reads <flnc.fast[a/q]> -outfolder <output>')
- ICE consensus transcript sequences (FASTA)
- Illumina-corrected PacBio transcript sequences (e.g., via proovread; FASTA)
- Original uncorrected PacBio CCS reads (FASTA/FASTQ)
- Illumina Nextera XT MiSeq reads (indexed per primer pair-sample combination; FASTQ)

## Outputs

- support_fractions.tsv with, per approach, the fraction of nucleotide positions supported by ≥2 agreeing Illumina reads and the fraction of fully supported transcripts
- Per-transcript supported-position counts from the Illumina read alignments

## How to apply

Obtain the four predicted transcript sets from the two-testes targeted Iso-Seq analysis: IsoCon output (final_candidates.fa), ICE consensus transcripts, Illumina-corrected PacBio transcripts, and original uncorrected CCS reads. Align the Illumina Nextera XT MiSeq reads — one indexed library per primer pair-sample combination — to each transcript set. For every predicted transcript, count nucleotide positions at which at least 2 independent Illumina reads covering the position agree with the predicted base. Compute, per approach, the fraction of supported positions over all transcripts and the fraction of transcripts fully supported (every position meeting the ≥2-read criterion), and write both fractions per approach to support_fractions.tsv. The rationale is that two independently agreeing Illumina reads provide a low-error orthologous signal against which the consensus accuracy of each long-read method can be benchmarked.

## Related tools

- **IsoCon** (Produces the primary candidate transcripts (final_candidates.fa) whose Illumina-supported positions are quantified in this validation) — https://github.com/ksahlin/IsoCon
- **ICE** (PacBio's standard de novo error-correction pipeline whose consensus transcripts serve as the comparison baseline)
- **proovread** (Illumina-based short-read correction approach used to generate the Illumina-corrected PacBio transcript set)
- **Illumina** (Nextera XT MiSeq short reads providing the independent ≥2-agreeing-read ground-truth signal for validation)
- **PacBio** (Long Iso-Seq/CCS reads supplying the original uncorrected transcript set being benchmarked)

## Examples

```
IsoCon pipeline -fl_reads <flnc.fastq> -outfolder <output>  # then align Illumina MiSeq reads to <output>/final_candidates.fa and score ≥2-agreeing-read positions
```

## Evaluation signals

- Reproduced supported-position fractions match the reported values: 99% (IsoCon), 93% (ICE), 96% (Illumina-corrected), 79% (original CCS)
- Reproduced fully-supported transcript fractions match: 70% (IsoCon), 2% (ICE), 15% (Illumina-corrected), 20% (original CCS)
- support_fractions.tsv contains exactly two fractions (supported positions, fully supported transcripts) for each of the four approaches
- Every position counted as supported is covered by ≥2 independent Illumina reads agreeing with the predicted base at that position
- IsoCon's supported-position fraction exceeds that of ICE, Illumina-corrected CCS, and original CCS reads (ordering: IsoCon > Illumina-corrected > ICE > original CCS)

## Limitations

- Very low-abundance transcripts might not be captured by Iso-Seq, since deep PacBio sequencing can be cost-prohibitive, so validation coverage may be missing for rare isoforms
- RT-PCR error sources cannot be excluded; replicate libraries and further validations would still be needed beyond Illumina agreement
- Original CCS reads score poorly (79% supported positions, only 20% fully supported transcripts), so CCS-only consensus cannot be treated as a reliable nucleotide-level reference

## Evidence

- [discussion] Illumina Nextera XT library construction for each primer pair-sample combination and MiSeq sequencing: "We constructed separate Nextera XT library (with a unique pair of indices) for each primer pair-sample combination."
- [discussion] Iso-Seq pre-processing requirement: "IsoCon assumes the reads have been pre-processed with the Iso-Seq bioinformatic pipeline to remove primers, barcodes, and reads that are chimeras"
- [discussion] Evaluation scripts and workflow availability: "Scripts for all analyses are available at https://github.com/ksahlin/IsoCon_Eval. This repository also include a snakemake63 workflow"
- [readme] IsoCon CLI invocation producing final_candidates.fa: "IsoCon pipeline -fl_reads <reads.fastq> -outfolder </path/to/output>"
- [readme] Use case scope for IsoCon: "IsoCon is a tool for reconstructing highly similar sequences present in a dataset of from long noisy reads."
