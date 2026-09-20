---
name: pacbio-subread-ccs-generation
description: Use when input data are PacBio subread BAM files from Iso-Seq (single-molecule,
  long-read RNA) sequencing and the goal is to produce circular consensus (HiFi) reads
  as the entry point to the nf-core/isoseq FLNC preprocessing chain.
when_to_use_negative:
- Input is already a CCS BAM or FLNC BAM file — proceed directly to LIMA/isoseq3 refine
  instead of regenerating CCS
- Data are short-read Illumina FASTQ rather than PacBio subread BAMs, since CCS generation
  is specific to circularly sequenced PacBio molecules
edam_topics:
- http://edamontology.org/topic_0196
tools:
- name: CCS
  role: Runs on the input subread BAM files to generate Circular Consensus Sequences
    in the first step of the nf-core/isoseq preprocessing stage
  repo: https://github.com/PacificBiosciences/ccs
- name: LIMA
  role: 'Downstream consumer of the CCS output: selects sequences containing valid
    pairs of primers and removes the primers'
  repo: https://github.com/pacificbiosciences/barcoding/
- name: IsoSeq (isoseq3 refine)
  role: Downstream filter of the retained CCS reads, discarding chimeric sequences
    after primer trimming
  repo: https://github.com/PacificBiosciences/IsoSeq
- name: nf-core/isoseq
  role: The Nextflow pipeline orchestrating the CCS generation stage within end-to-end
    Iso-Seq read processing and genome annotation
  repo: https://github.com/nf-core/isoseq
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/bioinformatics/btad150
    title: 'nf-core/isoseq: simple gene and isoform annotation with PacBio Iso-Seq
      long-read sequencing'
  source_content_hash: sha256:36397dd94dfd6767c8a85aaf9d69cbbc10a149fa0583b7540eae7d4a838eb230
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/PacificBiosciences/ccs
  tools:
  - BAMTools
  - CCS
  - IsoSeq (isoseq3 refine)
  - LIMA
  - TAMA
  - nf-core/isoseq
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0196
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7692307692307693
n_claims: 13
---

# pacbio-subread-ccs-generation

## Summary

Generates Circular Consensus Sequences (CCS) from PacBio subread BAM inputs, the first stage of the nf-core/isoseq read-processing workflow that feeds downstream LIMA primer removal and isoseq3 refine chimera filtering.

## When to use

Use when input data are PacBio subread BAM files from Iso-Seq (single-molecule, long-read RNA) sequencing and the goal is to produce circular consensus (HiFi) reads as the entry point to the nf-core/isoseq FLNC preprocessing chain.

## When NOT to use

- Input is already a CCS BAM or FLNC BAM file — proceed directly to LIMA/isoseq3 refine instead of regenerating CCS
- Data are short-read Illumina FASTQ rather than PacBio subread BAMs, since CCS generation is specific to circularly sequenced PacBio molecules

## Inputs

- PacBio subread BAM files (input subreads from Iso-Seq single-molecule sequencing)

## Outputs

- Circular Consensus Sequence (CCS) reads in BAM format, ready for LIMA primer selection/removal

## How to apply

In the nf-core/isoseq workflow, the PacBio ccs tool is run on the input subread BAM files to generate Circular Consensus Sequences, producing a CCS BAM output. This CCS output is then consumed by LIMA, which selects sequences containing valid pairs of primers and removes the primers, and subsequently by isoseq3 refine, which detects and discards chimeric reads to yield FLNC reads. The rationale is that subreads alone are noisy; collapsing the repeated passes over a single molecule into a CCS gives the high-quality sequence that the primer- and chimera-removal steps require. Correct application is judged by the presence of valid CCS BAM artifacts that LIMA can successfully trim, ultimately yielding chunked polyA-tail-free FLNC FASTA files downstream.

## Related tools

- **CCS** (Runs on the input subread BAM files to generate Circular Consensus Sequences in the first step of the nf-core/isoseq preprocessing stage) — https://github.com/PacificBiosciences/ccs
- **LIMA** (Downstream consumer of the CCS output: selects sequences containing valid pairs of primers and removes the primers) — https://github.com/pacificbiosciences/barcoding/
- **IsoSeq (isoseq3 refine)** (Downstream filter of the retained CCS reads, discarding chimeric sequences after primer trimming) — https://github.com/PacificBiosciences/IsoSeq
- **nf-core/isoseq** (The Nextflow pipeline orchestrating the CCS generation stage within end-to-end Iso-Seq read processing and genome annotation) — https://github.com/nf-core/isoseq

## Evaluation signals

- The final chunked FASTA artifacts downstream exist in FASTA format and contain FLNC reads, confirming the CCS output propagated correctly through the pipeline
- Retained reads after LIMA contain valid primer pairs that were successfully removed, indicating the CCS reads were properly formed
- Chimeras are discarded by isoseq3 refine from the retained reads, a check that only functions if CCS generation produced usable consensus sequences
- The CCS step completes on every sample in the samplesheet, since the pipeline reads all samples from a samplesheet and parallelizes computation per sample
- The CCS BAM is a valid input that LIMA can 'clean' — a LIMA failure or empty output suggests the CCS stage produced malformed or empty consensus reads

## Limitations

- No software version numbers are recorded for the tools/modules in this changelog section, creating a reproducibility gap for pinning the CCS version used
- CCS generation is only the first stage; primer validity, chimera removal, and polyA trimming depend on the downstream LIMA, isoseq3 refine, and TAMA cleanup steps, so a correct CCS alone does not guarantee clean FLNC output

## Evidence

- [intro] [CCS](https://github.com/PacificBiosciences/ccs) generate a Circular Consensus Sequence from subreads.: "[CCS](https://github.com/PacificBiosciences/ccs) generate a Circular Consensus Sequence from subreads."
- [intro] [LIMA](https://github.com/pacificbiosciences/barcoding/) clean generated CCS. It selects sequences containing valid pairs of primers and removed it.: "[LIMA](https://github.com/pacificbiosciences/barcoding/) clean generated CCS. It selects sequences containing valid pairs of primers and removed it."
- [readme] IsoSeq contains the newest tools to identify transcripts in PacBio single-molecule sequencing data.: [evidence span withheld — anchor status: not_found]
- [intro] The pipeline is built using [Nextflow](https://www.nextflow.io/) and processes data using the following steps: "The pipeline is built using [Nextflow](https://www.nextflow.io/) and processes data using the following steps"
