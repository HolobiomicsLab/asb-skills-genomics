---
name: ultralong-ont-read-handling
description: Use when self-correcting ONT long reads that include ultra-long molecules
  (up to 1.5 Mbp, e.g. full human datasets), or when deciding whether error correction
  improves Miniasm/Flye assembly quality versus raw reads.
when_to_use_negative:
- Reads are short reads or require hybrid correction with short-read alignment (e.g.
  CoLoRMAP/HECiL-style workflows); CONSENT is a self-correction method using only
  long-read overlaps
- The goal is polishing an existing raw assembly rather than read correction — use
  CONSENT-polish instead
- 'Turnaround must be fast: CONSENT''s correction step runtime tends to be higher
  than other state-of-the-art methods, and on human chr 1 it is almost four times
  faster than Canu but still takes days-scale compute'
edam_operation: http://edamontology.org/operation_3187
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3489
tools:
- name: CONSENT
  role: Self-corrects ultra-long ONT reads via overlapping-read MSA consensus plus
    local de Bruijn graph polishing
  repo: https://github.com/morispi/CONSENT
- name: Minimap2
  role: Computes the long-read overlaps CONSENT requires, and maps reads in the Miniasm
    overlap-layout-consensus assembly pipeline
- name: Miniasm
  role: Assembles raw and corrected reads via the overlap-layout-consensus pipeline
    for the comparison benchmark
- name: Flye
  role: Assembles raw and CONSENT-corrected H. sapiens chromosome 1 reads to test
    whether correction improves assembly quality
- name: QUAST-LG
  role: Evaluates each assembly against its reference genome (NGA50, NGA75, errors
    per 100 kbp, misassemblies)
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41598-020-80757-5
    title: Scalable long read self-correction and assembly polishing with multiple
      sequence alignment
  source_content_hash: sha256:0d0ea826c9c9a47d83326262af630cd101c5afbc725586fd938e57d2acd576c2
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/morispi/CONSENT
  tools:
  - CONSENT
  - Flye
  - Miniasm
  - Minimap2
  - QUAST-LG
  edam_operation: http://edamontology.org/operation_3187
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3489
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 13
---

# ultralong-ont-read-handling

## Summary

Correct and assess ultra-long Oxford Nanopore long reads (up to 1.5 Mbp) with CONSENT, the only self-correction method that efficiently scales to ultra-long ONT data, and evaluate the impact of correction on downstream Miniasm/Flye assemblies with QUAST-LG.

## When to use

Use when self-correcting ONT long reads that include ultra-long molecules (up to 1.5 Mbp, e.g. full human datasets), or when deciding whether error correction improves Miniasm/Flye assembly quality versus raw reads.

## When NOT to use

- Reads are short reads or require hybrid correction with short-read alignment (e.g. CoLoRMAP/HECiL-style workflows); CONSENT is a self-correction method using only long-read overlaps
- The goal is polishing an existing raw assembly rather than read correction — use CONSENT-polish instead
- Turnaround must be fast: CONSENT's correction step runtime tends to be higher than other state-of-the-art methods, and on human chr 1 it is almost four times faster than Canu but still takes days-scale compute

## Inputs

- FASTA or FASTQ file of raw ONT long reads (including ultra-long reads up to 1.5 Mbp)
- Reference genome for QUAST-LG assembly evaluation (e.g. H. sapiens chromosome 1)

## Outputs

- FASTA of CONSENT-corrected long reads
- Miniasm and Flye assemblies from raw and corrected reads
- QUAST-LG metrics table (contig counts, NGA50, NGA75, genome coverage, errors per 100 kbp, misassemblies)

## How to apply

Run CONSENT-correct on the raw long reads setting --type ONT for Oxford Nanopore data (or --type PB for PacBio), which computes overlaps with Minimap2, splits alignment piles into windows, derives per-window consensus via multiple sequence alignment, polishes each consensus with a local de Bruijn graph, and locally realigns it to the read. Tune windowing parameters only when necessary: --minSupport 4, --maxSupport 150, --anchorSupport 8, --minAnchors 2, --solid 4 are defaults; k-mer chaining further divides MSA instances for scalability. Assemble raw and corrected reads with the Minimap2+Miniasm overlap-layout-consensus pipeline (and Flye for H. sapiens chromosome 1) and evaluate all assemblies against the reference genome with QUAST-LG. Judge success by comparing NGA50/NGA75, contig counts, genome coverage, errors per 100 kbp, and misassembly counts: only CONSENT-corrected reads yield larger NGA50 and NGA75 than raw reads in Flye assemblies. Expect substantial runtime (CONSENT corrected a full human ultra-long dataset in 10 days) and disk footprint (about 5 TB, driven by the overlaps file, which can be redirected with --tmpdir to an external drive).

## Related tools

- **CONSENT** (Self-corrects ultra-long ONT reads via overlapping-read MSA consensus plus local de Bruijn graph polishing) — https://github.com/morispi/CONSENT
- **Minimap2** (Computes the long-read overlaps CONSENT requires, and maps reads in the Miniasm overlap-layout-consensus assembly pipeline)
- **Miniasm** (Assembles raw and corrected reads via the overlap-layout-consensus pipeline for the comparison benchmark)
- **Flye** (Assembles raw and CONSENT-corrected H. sapiens chromosome 1 reads to test whether correction improves assembly quality)
- **QUAST-LG** (Evaluates each assembly against its reference genome (NGA50, NGA75, errors per 100 kbp, misassemblies))

## Examples

```
./CONSENT-correct --in longReads.fastq --out correctedReads.fasta --type ONT
```

## Evaluation signals

- Corrected reads assemble with Flye to larger NGA50 and NGA75 than the raw-read assembly — CONSENT should be the only correction method achieving this
- QUAST-LG reports contig counts, NGA50/NGA75, genome coverage, errors per 100 kbp, and misassembly counts matching the reported Table 3 values for each dataset
- CONSENT completes correction on ONT ultra-long reads (up to 1.5 Mbp) where other tools fail to scale efficiently
- A valid .paf overlaps file is produced (default Alignments_dateTimeStamp.paf, redirectable via --tmpdir); expect ~5 TB for a full human dataset

## Limitations

- Runtime of CONSENT's correction step tends to be higher than other state-of-the-art methods
- Selecting only the N highest identity overlaps per alignment pile may prove inefficient when repeated regions grow longer
- About 5 TB of disk space is needed for a full human dataset, driven by the overlaps file (storable on an external drive)
- CONSENT assemblies may cover a smaller proportion of the reference sequence; unaligned contigs have not been analyzed in detail
- Correction of larger and more complex organisms has not yet been performed

## Evidence

- [discussion] CONSENT is the only method able to efficiently scale to the correction of ONT ultra-long reads: "CONSENT is the only method able to efficiently scale to the correction of ONT ultra-long reads"
- [results] Only CONSENT-corrected reads improve Flye assembly quality: [evidence span withheld — anchor status: not_found]
- [readme] CONSENT-correct usage with --type flag: "`./CONSENT-correct --in longReads.fast[a|q] --out result.fasta --type readsTechnology`"
- [readme] Algorithm description: overlaps, windows, MSA, de Bruijn graph polishing: "Each read's alignment pile is then further divided into smaller windows, that are corrected idependently."
- [abstract] Correction of a full human ultra-long dataset in 10 days: "CONSENT is the only method that efficiently scales to ultra-long reads, and allows to process a full human dataset, containing reads reaching up to 1.5 Mbp, in 10 days."
- [abstract] On human data, raw assembly + polishing beats correction then assembly: "on a human dataset, assembling the raw data and polishing the assembly is less resource consuming than correcting and then assembling the reads, while providing better results."
- [discussion] 5 TB disk requirement from overlaps file: "The 5 TB disk space requirement comes from the overlaps file, which can easily be stored on a external hard drive."
