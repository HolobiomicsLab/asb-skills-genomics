---
name: long-read-consensus-polishing
description: Use when a long-read (nanopore) genome assembly shows consensus-level
  base errors — e.g. low BUSCO completeness (3.0% complete / 92.9% missing in the
  uncorrected eel scaffolds) — and Illumina paired-end data from the same species
  is available for polishing.
when_to_use_negative:
- The assembly already has near-complete BUSCO scores or no Illumina data from the
  species is available, so Racon-against-nanopore-only is the only option (the article
  notes nanopore-only consensus 'cannot be resolved by taking a consensus of nanopore
  data only')
- The input is raw reads rather than an assembled scaffold/contig set — assembly (e.g.
  with TULIP or Canu) must precede polishing
- Only short reads are available and no long-read assembly exists; this skill corrects
  long-read assemblies, not short-read-only drafts
edam_operation: http://edamontology.org/operation_0310
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_0196
tools:
- name: Racon
  role: Consensus correction of the nanopore TULIP scaffolds using nanopore read alignments
    to produce the Racon-corrected assembly
- name: Pilon
  role: Illumina-based polishing (version 1.21, two rounds) of the Racon-corrected
    scaffolds using trimmed paired-end reads
  repo: https://github.com/broadinstitute/pilon/wiki
- name: sickle
  role: Windowed adaptive quality trimming of the Illumina pairs to a minimum Phred
    quality of 30 before Pilon polishing
  repo: https://github.com/najoshi/sickle
- name: BUSCO
  role: Assesses completeness of each of the four assembly versions with the vertebrata_odb9
    lineage to quantify the polishing effect
- name: ont-assembly-polish
  role: Reference Makefile pipeline (Canu + optional Racon + BWA MEM + Pilon) implementing
    the same assembly-polishing workflow
  repo: https://github.com/nanoporetech/ont-assembly-polish
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41598-017-07650-6
    title: Rapid de novo assembly of the European eel genome from nanopore sequencing
      reads
  source_content_hash: sha256:689cb23adba1dd53371f3347dc3d816ada785f310644ab279d50c2d938b9e657
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - BUSCO
  - Pilon
  - Racon
  - ont-assembly-polish
  - sickle
  edam_operation: http://edamontology.org/operation_0310
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_0196
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 6
---

# long-read-consensus-polishing

## Summary

Post-assembly error correction pipeline in which a nanopore-based de novo assembly is first consensus-corrected with Racon against long reads, then polished with Pilon using quality-trimmed Illumina reads, with each stage assessed by BUSCO completeness against the vertebrata_odb9 lineage.

## When to use

Use when a long-read (nanopore) genome assembly shows consensus-level base errors — e.g. low BUSCO completeness (3.0% complete / 92.9% missing in the uncorrected eel scaffolds) — and Illumina paired-end data from the same species is available for polishing.

## When NOT to use

- The assembly already has near-complete BUSCO scores or no Illumina data from the species is available, so Racon-against-nanopore-only is the only option (the article notes nanopore-only consensus 'cannot be resolved by taking a consensus of nanopore data only')
- The input is raw reads rather than an assembled scaffold/contig set — assembly (e.g. with TULIP or Canu) must precede polishing
- Only short reads are available and no long-read assembly exists; this skill corrects long-read assemblies, not short-read-only drafts

## Inputs

- Uncorrected long-read scaffold set (TULIP nanopore scaffolds for the European eel, ~860 Mbp genome)
- Nanopore long reads (FASTA/FASTQ from MinION R7.3, R9 and R9.4 flow cells) for Racon consensus alignment
- Paired-end Illumina FASTQ reads (SRR5235521–SRR5235523) from a different eel specimen, quality-trimmed with sickle
- BUSCO lineage dataset vertebrata_odb9 (2586 genes)

## Outputs

- Racon-corrected scaffold set
- Pilon-corrected scaffold set (first polishing round)
- Twice-polished scaffold set (second Pilon 1.21 round)
- BUSCO complete/fragmented/missing percentages for all four assembly versions, compiled into busco_polishing_scores.tsv

## How to apply

First trim the Illumina reads (e.g. SRR5235521–SRR5235523) with sickle to a minimum Phred quality of 30, since Pilon needs high-quality short reads to call consensus corrections. Align nanopore reads to the uncorrected TULIP scaffold set and run Racon to obtain a consensus-corrected assembly; in the eel case this raises BUSCO to 10.8% complete and 67.6% missing. Then run Pilon 1.21 with the trimmed Illumina pairs on the Racon-corrected scaffolds, and optionally run a second Pilon round on the first output to produce a twice-polished assembly. Judge success by running BUSCO 3.0.0 with the vertebrata_odb9 dataset (2586 genes) independently on all four scaffold sets and comparing to the published progression: ~3.0% / 10.8% / 77.5% / 79.8% complete and 92.9% / 67.6% / 8.4% / 7.3% missing.

## Related tools

- **Racon** (Consensus correction of the nanopore TULIP scaffolds using nanopore read alignments to produce the Racon-corrected assembly)
- **Pilon** (Illumina-based polishing (version 1.21, two rounds) of the Racon-corrected scaffolds using trimmed paired-end reads) — https://github.com/broadinstitute/pilon/wiki
- **sickle** (Windowed adaptive quality trimming of the Illumina pairs to a minimum Phred quality of 30 before Pilon polishing) — https://github.com/najoshi/sickle
- **BUSCO** (Assesses completeness of each of the four assembly versions with the vertebrata_odb9 lineage to quantify the polishing effect)
- **ont-assembly-polish** (Reference Makefile pipeline (Canu + optional Racon + BWA MEM + Pilon) implementing the same assembly-polishing workflow) — https://github.com/nanoporetech/ont-assembly-polish

## Examples

```
sickle pe -f SRR5235521_1.fastq -r SRR5235521_2.fastq -t sanger -q 30 -o pe1_trimmed.fastq -p pe2_trimmed.fastq -s singles.fastq  # then: make all in ont-assembly-polish
```

## Evaluation signals

- Racon-corrected assembly must reach approximately 10.8% complete, 21.6% fragmented and 67.6% missing BUSCO genes
- Pilon-corrected assembly must reach approximately 77.5% complete, 14.1% fragmented and 8.4% missing BUSCO genes
- Twice-polished assembly must reach approximately 79.8% complete and 7.3% missing; each polishing stage should monotonically raise complete BUSCO percentage and lower missing percentage
- BUSCO must be run with vertebrata_odb9 (2586 genes) independently on all four scaffold sets (uncorrected, Racon, Pilon, twice-polished) and results compiled into busco_polishing_scores.tsv
- Pilon must be run on Illumina reads trimmed to Phred quality 30 with sickle, not on raw reads

## Limitations

- In the absence of a high-quality reference genome, it is not straightforward to establish which assembly base is correct at apparent error sites
- The assembly is a hybrid of two sequencing technologies, three nanopore chemistries, and two different animals; the Pilon consensus could in theory mix up to four different haplotypes because Illumina data come from a different eel specimen
- The ont-assembly-polish reference repository is unsupported and its maintainers do not recommend its use
- Racon consensus correction alone cannot fix errors that short reads resolve; polishing is required for the large BUSCO gains

## Evidence

- [discussion] Racon consensus correction of nanopore scaffolds: "cannot be resolved by taking a consensus of nanopore data only (e.g. using Racon)"
- [discussion] Pilon polishing with Illumina data: "the nanopore data are therefore corrected using Illumina data obtained from a different eel specimen than used for nanopore sequencing"
- [readme] Pipeline performs Canu assembly, optional Racon, BWA MEM mapping, Pilon correction: "Polish canu contigs using [racon](https://github.com/isovic/racon) (*optional*).
- Map a paired-end Illumina dataset onto the contigs"
- [readme] Sickle trimming algorithm: [evidence span withheld — anchor status: external_doc]
