---
name: large-indel-structural-variant-detection
description: Use when untargeted nanopore reads have been mapped to a selected viral
  reference (BAM produced with Samtools) and large structural differences such as
  insertions or deletions must be incorporated into a draft consensus genome before
  polishing.
when_to_use_negative:
- Input is already assembled de novo contigs that directly match a reference — ViMOP
  matches contigs to references first; this skill operates on read-to-reference alignments
- The sample is a large DNA virus with extensive repetitive regions (e.g. mpox), where
  the README warns assemblies may contain inaccuracies and should be regarded as draft
  genomes
edam_operation: http://edamontology.org/operation_3227
edam_topics:
- http://edamontology.org/topic_0622
tools:
- name: Sniffles2
  role: Calls large insertions/deletions from the per-reference BAM and records them
    in a VCF
- name: BCFtools
  role: Applies the Sniffles2 indel VCF to the reference sequence to build an indel-corrected
    draft consensus
- name: Samtools
  role: Sorts/indexes the read alignment and provides the fallback consensus path
    (min depth 20, min 70% base proportion)
- name: Medaka
  role: Polishes the indel-corrected draft with a neural-network consensus model ('auto'
    by default)
  repo: https://github.com/nanoporetech/medaka
- name: ViMOP
  role: Nextflow pipeline that orchestrates the mapping, indel detection, and consensus
    steps
  repo: https://github.com/opr-group-bnitm/vimop
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1093/bioinformatics/btaf687
    title: 'ViMOP: a user-friendly and field-applicable pipeline for untargeted viral
      genome nanopore sequencing'
  source_content_hash: sha256:94a380b9d0ac457eaf6157fa37cd8f5a55b879e0d8415f92801500a5e4199bd2
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
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5833333333333334
n_claims: 12
---

# large-indel-structural-variant-detection

## Summary

Detects large insertions and deletions in nanopore read alignments with Sniffles2 and applies them to a BLAST-selected reference genome via BCFtools consensus, forming the indel-corrected draft stage of ViMOP's reference-guided consensus assembly.

## When to use

Use when untargeted nanopore reads have been mapped to a selected viral reference (BAM produced with Samtools) and large structural differences such as insertions or deletions must be incorporated into a draft consensus genome before polishing.

## When NOT to use

- Input is already assembled de novo contigs that directly match a reference — ViMOP matches contigs to references first; this skill operates on read-to-reference alignments
- The sample is a large DNA virus with extensive repetitive regions (e.g. mpox), where the README warns assemblies may contain inaccuracies and should be regarded as draft genomes

## Inputs

- Sorted/indexed BAM of trimmed nanopore reads mapped against the BLAST-selected reference genome (e.g. AB627954.reads.bam)
- BLAST-selected reference genome FASTA
- VCF of large indels called by Sniffles2 (with <INV> records pre-filtered)

## Outputs

- Indel-corrected draft reference sequence (BCFtools consensus FASTA)
- Sniffles2 variant VCF (large insertions/deletions)
- Final polished consensus FASTA (e.g. AB627954.consensus.fasta) with sub-threshold positions masked as N
- Supporting files: alignment BAM, depth file, and variant VCF in the sample's consensus output directory

## How to apply

Map the trimmed nanopore reads against the BLAST-selected reference genome and sort/index the alignment with Samtools to obtain a per-reference BAM. Run Sniffles2 on that BAM to call large insertions and deletions and record them in a VCF. Pre-filter any <INV> records out of the VCF, since BCFtools consensus is known to error on inversion records, then apply the remaining indels to the reference sequence with BCFtools consensus to yield an indel-corrected draft. Polish this draft with Medaka (model 'auto' unless a specific medaka_consensus_model is set) or, if no fitting model is available, fall back to Samtools consensus requiring a minimum depth of 20 and at least 70% base proportion, masking failing positions with N.

## Related tools

- **Sniffles2** (Calls large insertions/deletions from the per-reference BAM and records them in a VCF)
- **BCFtools** (Applies the Sniffles2 indel VCF to the reference sequence to build an indel-corrected draft consensus)
- **Samtools** (Sorts/indexes the read alignment and provides the fallback consensus path (min depth 20, min 70% base proportion))
- **Medaka** (Polishes the indel-corrected draft with a neural-network consensus model ('auto' by default)) — https://github.com/nanoporetech/medaka
- **ViMOP** (Nextflow pipeline that orchestrates the mapping, indel detection, and consensus steps) — https://github.com/opr-group-bnitm/vimop

## Evaluation signals

- The consensus FASTA and alignment BAM both exist in the sample's consensus output directory
- The consensus contains no ambiguous bases other than N-masked positions
- All positions with coverage below the depth-20 threshold or base proportion below 70% are represented as N
- No <INV> records were passed to BCFtools consensus (they must be pre-filtered from the VCF)
- Indels reported by Sniffles2 in the VCF are reflected as applied differences between the reference and the draft consensus

## Limitations

- BCFtools consensus throws an unresolved error when <INV>-denoted inversions are present in the VCF, so inversion records must be removed beforehand
- For large DNA viruses with extensive repetitive regions (e.g. mpox), the resulting genomes may contain inaccuracies and should be regarded as drafts
- The Samtools fallback masks positions with N when depth < 20 or the most abundant base is < 70%, leaving gaps in low-coverage genomes
- Medaka polishing requires a matching basecaller model; when none fits the data, the lower-accuracy Samtools consensus path is used

## Evidence

- [other] Sniffles2 indel calling then BCFtools application: [evidence span withheld — anchor status: not_found]
- [discussion] INV pre-filter rationale: "error when inversions denoted with <INV> are in the vcf-file before consensus calling with bcftools"
- [other] Samtools fallback thresholds: [evidence span withheld — anchor status: not_found]
- [readme] Pipeline context: "The pipeline automatically finds well fitting virus genomes and uses them as references to build reference based consensus genomes."
- [methods] Alignment prerequisite: "Reads are mapped against the reference genome."
