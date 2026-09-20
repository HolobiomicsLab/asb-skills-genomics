---
name: short-read-variant-based-assembly-polishing
description: Use when a long-read (e.g. Oxford Nanopore) assembly has high raw error
  rates and paired Illumina reads are available to raise base-level accuracy, and
  when the improvement must be quantified per polishing stage.
when_to_use_negative:
- The assembly was built from short reads only and is already at base-level accuracy,
  so nanopore-specific Racon consensus correction is unnecessary
- No independent Illumina data are available, in which case Pilon variant-based polishing
  cannot be performed and Racon alone cannot resolve errors the article notes 'cannot
  be resolved by taking a consensus of nanopore data only'
- The goal is structural/long-range validation (e.g. misjoins or scaffold merging)
  rather than base-level error correction — the article points to optical mapping
  for that purpose
edam_operation: http://edamontology.org/operation_3181
edam_topics:
- http://edamontology.org/topic_0196
- http://edamontology.org/topic_3693
tools:
- name: Racon
  role: Consensus correction of the TULIP nanopore scaffolds using nanopore read alignments,
    applied before Illumina polishing
- name: Pilon
  role: Variant-based polishing of the Racon-corrected scaffolds with trimmed Illumina
    reads (version 1.21, two rounds)
  repo: https://github.com/broadinstitute/pilon/wiki
- name: sickle
  role: Windowed adaptive trimming of the Illumina paired-end reads to a minimum Phred
    quality of 30 before Pilon
  repo: https://github.com/najoshi/sickle
- name: BUSCO
  role: Completeness assessment (3.0.0, vertebrata_odb9, 2586 genes) of each of the
    four assembly versions to quantify polishing effect
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41598-017-07650-6
    title: Rapid de novo assembly of the European eel genome from nanopore sequencing
      reads
  source_content_hash: sha256:2582451e679e7bf4c26f2801c0ef57eeeafa98fdd477719c2ddaaf1b0e333715
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - BUSCO
  - Pilon
  - Racon
  - sickle
  edam_operation: http://edamontology.org/operation_3181
  edam_topics:
  - http://edamontology.org/topic_0196
  - http://edamontology.org/topic_3693
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 7
---

# short-read-variant-based-assembly-polishing

## Summary

Improves a nanopore-derived de novo assembly by rounds of Racon consensus correction followed by Illumina short-read variant-based polishing with Pilon, then quantifies the effect with BUSCO completeness assessments against the vertebrata_odb9 lineage.

## When to use

Use when a long-read (e.g. Oxford Nanopore) assembly has high raw error rates and paired Illumina reads are available to raise base-level accuracy, and when the improvement must be quantified per polishing stage.

## When NOT to use

- The assembly was built from short reads only and is already at base-level accuracy, so nanopore-specific Racon consensus correction is unnecessary
- No independent Illumina data are available, in which case Pilon variant-based polishing cannot be performed and Racon alone cannot resolve errors the article notes 'cannot be resolved by taking a consensus of nanopore data only'
- The goal is structural/long-range validation (e.g. misjoins or scaffold merging) rather than base-level error correction — the article points to optical mapping for that purpose

## Inputs

- Uncorrected TULIP scaffold set of the European eel (Anguilla anguilla) genome
- Nanopore reads (R7.3.fasta, R9_pass_1d.fasta, R9_pass_2d.fasta, R9.4_pass_1d.fasta) for Racon alignment-based consensus
- Illumina paired-end reads SRR5235521–SRR5235523 from a different eel specimen
- BUSCO 3.0.0 vertebrata_odb9 lineage dataset (2586 genes)

## Outputs

- Racon-corrected assembly (FASTA scaffolds)
- Pilon-corrected assembly (FASTA scaffolds)
- Twice-polished assembly (FASTA scaffolds)
- busco_polishing_scores.tsv with complete/fragmented/missing BUSCO percentages for all four assembly versions

## How to apply

First trim the Illumina reads (here SRR5235521–SRR5235523) to a minimum Phred quality of 30 with sickle. Correct the uncorrected TULIP scaffold set with Racon using nanopore read alignments, then run Pilon 1.21 on the Racon-corrected scaffolds with the trimmed Illumina reads, and optionally a second Pilon round on the first Pilon output to produce a twice-polished assembly. Run BUSCO 3.0.0 with the vertebrata_odb9 lineage dataset (2586 genes) independently on all four assembly versions (uncorrected, Racon-corrected, Pilon-corrected, twice-polished) and compile complete/fragmented/missing percentages into a table such as busco_polishing_scores.tsv. The rationale is that Racon consensus from noisy nanopore data alone is insufficient, and short reads from a different specimen are used to fix substitution- and indel-level errors.

## Related tools

- **Racon** (Consensus correction of the TULIP nanopore scaffolds using nanopore read alignments, applied before Illumina polishing)
- **Pilon** (Variant-based polishing of the Racon-corrected scaffolds with trimmed Illumina reads (version 1.21, two rounds)) — https://github.com/broadinstitute/pilon/wiki
- **sickle** (Windowed adaptive trimming of the Illumina paired-end reads to a minimum Phred quality of 30 before Pilon) — https://github.com/najoshi/sickle
- **BUSCO** (Completeness assessment (3.0.0, vertebrata_odb9, 2586 genes) of each of the four assembly versions to quantify polishing effect)

## Examples

```
sickle pe -f SRR5235521_1.fastq -r SRR5235521_2.fastq -t sanger -q 30 -o trim_1.fastq -p trim_2.fastq -s singles.fastq && java -Xmx64G -jar pilon-1.21.jar --genome racon_corrected.fasta --frags trim.bam --output pilon_corrected
```

## Evaluation signals

- BUSCO scores for the four assemblies must match published values: ~3.0% / 10.8% / 77.5% / 79.8% complete for uncorrected, Racon-corrected, Pilon-corrected and twice-polished assemblies
- BUSCO missing percentages must decrease monotonically: 92.9% / 67.6% / 8.4% / 7.3% across the four stages
- Pilon polishing must raise complete BUSCO genes sharply (from 10.8% after Racon to 77.5%), with fragmented genes reduced from 21.6% to 14.1% and missing from 67.6% to 8.4%
- Illumina input to Pilon must be the trimmed reads (min Phred 30 produced by sickle), not raw FASTQ files
- Each of the four scaffold sets must be assessed with BUSCO independently, and results compiled into busco_polishing_scores.tsv

## Limitations

- Illumina correction data come from a different eel specimen than the nanopore data, so in theory the polished consensus can mix up to four different haplotypes
- In the absence of a high-quality reference genome it is not straightforward to establish which base at an apparent error site is correct
- The ONT ont-assembly-polish pipeline repository is unsupported and its authors do not recommend its continued use

## Evidence

- [results] Racon correction of the nanopore assembly raises BUSCO scores to 10.8% complete, 21.6% fragmented and 67.6% missing, and subsequent correction with Pilon further increases this to 77.5% complete, 14.1% fragmented and 8.4% missing.: [evidence span withheld — anchor status: not_found]
- [methods] Run a second Pilon 1.21 round on the first Pilon output to produce the twice-polished assembly.: [evidence span withheld — anchor status: not_found]
- [discussion] cannot be resolved by taking a consensus of nanopore data only (e.g. using Racon): "cannot be resolved by taking a consensus of nanopore data only (e.g. using Racon)"
- [discussion] the nanopore data are therefore corrected using Illumina data obtained from a different eel specimen than used for nanopore sequencing: "the nanopore data are therefore corrected using Illumina data obtained from a different eel specimen than used for nanopore sequencing"
- [readme] Perform correction of contigs using [pilon] and the Illumina dataset.: [evidence span withheld — anchor status: not_found]
