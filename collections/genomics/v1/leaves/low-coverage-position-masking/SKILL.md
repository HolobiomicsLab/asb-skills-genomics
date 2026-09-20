---
name: low-coverage-position-masking
description: Use when building a consensus genome from nanopore reads mapped to a
  reference and Medaka cannot fit a matching basecaller model, so that positions below
  a minimum depth of 20 or with less than 70% nucleotide proportion are masked as
  N.
when_to_use_negative:
- A matching Medaka basecaller model is available for the data — then polish with
  Medaka instead of the Samtools fallback consensus
- The input is de novo contigs rather than reads mapped to a reference — ViMOP's de
  novo assembly step does not use consensus masking
- The sample is a large DNA virus with extensive repetitive regions (e.g. mpox) —
  the README warns such assemblies may be inaccurate draft genomes
edam_operation: http://edamontology.org/operation_3227
edam_topics:
- http://edamontology.org/topic_3673
- http://edamontology.org/topic_0196
tools:
- name: Samtools
  role: Generates the depth file and performs the fallback consensus call, calling
    the most abundant base and masking positions below depth 20 or 70% base proportion
    with N
- name: Medaka
  role: Preferred polisher of the indel-corrected draft; when Medaka cannot fit a
    model for the data, the pipeline falls back to Samtools consensus masking
  repo: nanoporetech/medaka
- name: Sniffles2
  role: Detects large insertions/deletions from the BAM that are applied to the reference
    before masking
- name: BCFtools
  role: Applies Sniffles2 indel calls to the reference to build the indel-corrected
    draft consensus; <INV> records must be pre-filtered to avoid a known BCFtools
    consensus error
- name: ViMOP
  role: Pipeline that orchestrates the reference-guided assembly, masking and consensus
    output steps
  repo: https://github.com/opr-group-bnitm/vimop
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1093/bioinformatics/btaf687
    title: 'ViMOP: a user-friendly and field-applicable pipeline for untargeted viral
      genome nanopore sequencing'
  source_content_hash: sha256:403df671f1fc6482c542ff32724233aea8e29434740c884b388610df9bf53de0
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/nanoporetech/medaka
  tools:
  - BCFtools
  - Medaka
  - Samtools
  - Sniffles2
  - ViMOP
  edam_operation: http://edamontology.org/operation_3227
  edam_topics:
  - http://edamontology.org/topic_3673
  - http://edamontology.org/topic_0196
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# Low-coverage position masking

## Summary

Masks positions in a Nanopore-derived consensus genome as Ns when coverage or base-support thresholds are not met, ensuring only well-supported bases are reported. This skill reproduces the Samtools-consensus fallback path in the ViMOP pipeline for reference-guided viral genome reconstruction.

## When to use

Use when building a consensus genome from nanopore reads mapped to a reference and Medaka cannot fit a matching basecaller model, so that positions below a minimum depth of 20 or with less than 70% nucleotide proportion are masked as N.

## When NOT to use

- A matching Medaka basecaller model is available for the data — then polish with Medaka instead of the Samtools fallback consensus
- The input is de novo contigs rather than reads mapped to a reference — ViMOP's de novo assembly step does not use consensus masking
- The sample is a large DNA virus with extensive repetitive regions (e.g. mpox) — the README warns such assemblies may be inaccurate draft genomes

## Inputs

- Indel-corrected draft reference FASTA (produced by BCFtools consensus from the reference plus Sniffles2 indel VCF)
- Sorted/indexed BAM of trimmed nanopore reads mapped against the selected reference genome
- Per-position depth file from Samtools
- Sniffles2 variant VCF with <INV> records pre-filtered

## Outputs

- Consensus FASTA with N-masked positions (e.g. AB627954.consensus.fasta)
- Alignment BAM (e.g. AB627954.reads.bam)
- Per-position depth file
- Variant VCF from Sniffles2

## How to apply

After mapping trimmed nanopore reads against the BLAST-selected reference and correcting large indels (Sniffles2 VCF applied via BCFtools consensus, pre-filtering <INV> records), check whether Medaka has a matching model via 'auto' or medaka_consensus_model. If Medaka cannot fit a model, fall back to Samtools consensus calling: require a minimum depth of 20 and a minimum base proportion of 70%, call the most abundant base at each position, and replace any position below either threshold with N. Write the masked consensus FASTA (e.g. AB627954.consensus.fasta) alongside the alignment BAM, depth file and variant VCF into the sample's consensus output directory. This ensures ambiguous or weakly supported positions are explicitly flagged rather than guessed.

## Related tools

- **Samtools** (Generates the depth file and performs the fallback consensus call, calling the most abundant base and masking positions below depth 20 or 70% base proportion with N)
- **Medaka** (Preferred polisher of the indel-corrected draft; when Medaka cannot fit a model for the data, the pipeline falls back to Samtools consensus masking) — nanoporetech/medaka
- **Sniffles2** (Detects large insertions/deletions from the BAM that are applied to the reference before masking)
- **BCFtools** (Applies Sniffles2 indel calls to the reference to build the indel-corrected draft consensus; <INV> records must be pre-filtered to avoid a known BCFtools consensus error)
- **ViMOP** (Pipeline that orchestrates the reference-guided assembly, masking and consensus output steps) — https://github.com/opr-group-bnitm/vimop

## Examples

```
bcftools consensus -f ref.fasta -m mask.bed --no-version draft.vcf.gz > draft.fasta; samtools consensus -d 20 -p 0.7 -a N AB627954.reads.bam -o AB627954.consensus.fasta
```

## Evaluation signals

- The consensus FASTA and alignment BAM exist in the sample's consensus output directory
- The consensus contains no ambiguous bases other than N-masked positions
- Every position with coverage below the depth-20 threshold is represented as N in the consensus
- Every position with base proportion below 70% is represented as N in the consensus
- <INV> records were removed from the Sniffles2 VCF before BCFtools consensus was run, avoiding the known BCFtools consensus error

## Limitations

- BCFtools consensus errors when inversions denoted with <INV> are present in the VCF; these must be pre-filtered
- For large DNA viruses with extensive repetitive regions (e.g. mpox), the consensus should be regarded as a draft genome with possible inaccuracies
- Masking only guarantees support at depth ≥ 20 and ≥ 70% base proportion; positions passing these thresholds may still carry residual Nanopore error

## Evidence

- [methods] Samtools consensus thresholds: [evidence span withheld — anchor status: not_found]
- [methods] Medaka vs Samtools decision: [evidence span withheld — anchor status: not_found]
- [discussion] INV pre-filtering: "error when inversions denoted with <INV> are in the vcf-file before consensus calling with bcftools"
- [methods] Validation criteria: [evidence span withheld — anchor status: not_found]
- [readme] Mpox caveat: "for large DNA viruses with extensive repetitive regions (e.g., mpox), assemblies may contain inaccuracies and should be regarded as draft genomes."
