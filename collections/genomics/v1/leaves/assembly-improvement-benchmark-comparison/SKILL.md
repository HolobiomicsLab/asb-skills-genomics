---
name: assembly-improvement-benchmark-comparison
description: Use when a long-read (nanopore) assembly has been improved with Racon
  and Pilon and you need to quantify how each polishing stage changes gene-space completeness
  against a benchmarking lineage dataset such as vertebrata_odb9.
when_to_use_negative:
- A high-quality reference genome is available and structural correctness should be
  assessed by reference-based comparison rather than BUSCO completeness alone — the
  paper notes that in the absence of a high-quality reference it is not straightforward
  to establish which assembly is correct at apparent error sites
- The goal is de novo assembly itself rather than quantifying improvement across polishing
  stages of an existing scaffold set
edam_operation: http://edamontology.org/operation_3180
edam_topics:
- http://edamontology.org/topic_0196
- http://edamontology.org/topic_3673
- http://edamontology.org/topic_0622
tools:
- name: Racon
  role: Consensus correction of the nanopore assembly using nanopore read alignments
    (first polishing stage)
  repo: https://github.com/isovic/racon
- name: Pilon
  role: Illumina-read-based correction/polishing of the Racon-corrected scaffolds,
    version 1.21, run twice
  repo: https://github.com/broadinstitute/pilon/wiki
- name: sickle
  role: Windowed adaptive quality trimming of Illumina paired-end reads to minimum
    Phred 30 before Pilon
  repo: https://github.com/najoshi/sickle
- name: BUSCO
  role: Completeness assessment of each assembly stage against the vertebrata_odb9
    lineage (version 3.0.0)
- name: ont-assembly-polish
  role: Reference ONT pipeline for Canu assembly + optional Racon and Pilon polishing
    workflow this benchmark mirrors
  repo: https://github.com/nanoporetech/ont-assembly-polish
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41598-017-07650-6
    title: Rapid de novo assembly of the European eel genome from nanopore sequencing
      reads
  source_content_hash: sha256:886f675c2fb4c2e224a241f07193e9b1e83aa82ca9feaf018ad1c9f3ea87d9d5
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/isovic/racon
  tools:
  - BUSCO
  - Pilon
  - Racon
  - ont-assembly-polish
  - sickle
  edam_operation: http://edamontology.org/operation_3180
  edam_topics:
  - http://edamontology.org/topic_0196
  - http://edamontology.org/topic_3673
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 6
---

# assembly-improvement-benchmark-comparison

## Summary

Benchmark the effect of sequential post-assembly correction (Racon consensus correction followed by Pilon polishing with Illumina reads) on a nanopore-based genome assembly by running BUSCO completeness assessment independently at each correction stage and comparing the resulting complete/fragmented/missing percentages.

## When to use

Use when a long-read (nanopore) assembly has been improved with Racon and Pilon and you need to quantify how each polishing stage changes gene-space completeness against a benchmarking lineage dataset such as vertebrata_odb9.

## When NOT to use

- A high-quality reference genome is available and structural correctness should be assessed by reference-based comparison rather than BUSCO completeness alone — the paper notes that in the absence of a high-quality reference it is not straightforward to establish which assembly is correct at apparent error sites
- The goal is de novo assembly itself rather than quantifying improvement across polishing stages of an existing scaffold set

## Inputs

- Uncorrected nanopore-based TULIP scaffold set (FASTA)
- Nanopore read alignments to the assembly (for Racon correction)
- Illumina paired-end reads (SRR5235521–SRR5235523), trimmed to min Phred Q30 with sickle
- BUSCO 3.0.0 lineage dataset vertebrata_odb9 (2586 genes)

## Outputs

- Racon-corrected assembly (FASTA scaffolds)
- Pilon-corrected and twice-polished assemblies (FASTA scaffolds)
- Per-stage BUSCO reports with complete/fragmented/missing percentages
- busco_polishing_scores.tsv compiling complete and missing percentages across all four assembly versions

## How to apply

First prepare the short-read data used for polishing: trim the Illumina reads (e.g. SRR5235521–SRR5235523) to a minimum Phred quality of 30 with sickle. Correct the uncorrected TULIP nanopore scaffold set with Racon using nanopore read alignments, then run Pilon 1.21 with the trimmed Illumina reads, and optionally a second Pilon round, producing four scaffold sets (uncorrected, Racon-corrected, Pilon-corrected, twice-polished). Run BUSCO 3.0.0 independently on each scaffold set with the vertebrata_odb9 lineage (2586 genes). Compile the complete and missing percentages for all four versions into a table (busco_polishing_scores.tsv) and compare against expected published values to judge whether each polishing step improved completeness. In the eel case, Racon alone lifted BUSCO completeness only to ~10.8%, showing that nanopore-only consensus correction cannot resolve the error profile, whereas Illumina-based Pilon polishing raised completeness to 77.5% (and 79.8% after a second round) and reduced missing genes from 67.6% to 8.4%.

## Related tools

- **Racon** (Consensus correction of the nanopore assembly using nanopore read alignments (first polishing stage)) — https://github.com/isovic/racon
- **Pilon** (Illumina-read-based correction/polishing of the Racon-corrected scaffolds, version 1.21, run twice) — https://github.com/broadinstitute/pilon/wiki
- **sickle** (Windowed adaptive quality trimming of Illumina paired-end reads to minimum Phred 30 before Pilon) — https://github.com/najoshi/sickle
- **BUSCO** (Completeness assessment of each assembly stage against the vertebrata_odb9 lineage (version 3.0.0))
- **ont-assembly-polish** (Reference ONT pipeline for Canu assembly + optional Racon and Pilon polishing workflow this benchmark mirrors) — https://github.com/nanoporetech/ont-assembly-polish

## Examples

```
BUSCO3.0.0.py -i pilon_corrected_scaffolds.fasta -o busco_pilon -l vertebrata_odb9 -m genome
```

## Evaluation signals

- BUSCO scores for the four assemblies match the published values: ~3.0% / 10.8% / 77.5% / 79.8% complete for uncorrected, Racon-corrected, Pilon-corrected and twice-polished assemblies respectively
- Missing BUSCO genes match ~92.9% / 67.6% / 8.4% / 7.3% across the four stages
- Completeness should increase monotonically and missing percentage decrease monotonically across the correction stages
- busco_polishing_scores.tsv contains complete and missing percentages for all four assembly versions
- Illumina input to Pilon has been trimmed to minimum Phred quality 30 as specified in the workflow

## Limitations

- Racon-only consensus correction of nanopore data left BUSCO completeness at just 10.8%, indicating nanopore-only consensus cannot fix the dominant error types; Illumina polishing is required for substantial gains
- No high-quality reference genome was available to determine which assembly is correct at apparent error sites, so BUSCO is an indirect quality proxy
- The hybrid assembly mixes two sequencing technologies, three nanopore chemistries and two different animals, so the consensus can theoretically mix up to four haplotypes

## Evidence

- [results] Racon correction of the nanopore assembly raises BUSCO scores to 10.8% complete, 21.6% fragmented and 67.6% missing, and subsequent correction with Pilon further increases this to 77.5% complete, 14.1% fragmented and 8.4% missing.: [evidence span withheld — anchor status: not_found]
- [methods] Run BUSCO 3.0.0 with the vertebrata_odb9 lineage dataset (2586 genes) independently on the uncorrected, Racon-corrected, Pilon-corrected and twice-polished scaffold sets.: [evidence span withheld — anchor status: not_found]
- [discussion] the nanopore data are therefore corrected using Illumina data obtained from a different eel specimen than used for nanopore sequencing: "the nanopore data are therefore corrected using Illumina data obtained from a different eel specimen"
- [discussion] In the absence of a high-quality reference, it is not straightforward to establish which assembly is correct: "In the absence of a high-quality reference, it is not straightforward to establish which assembly is correct"
- [readme] Polish canu contigs using [racon](https://github.com/isovic/racon) (*optional*).: [evidence span withheld — anchor status: external_doc]
