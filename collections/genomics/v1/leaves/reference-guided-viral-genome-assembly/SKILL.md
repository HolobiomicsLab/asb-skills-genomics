---
name: reference-guided-viral-genome-assembly
description: Use when nanopore reads from an untargeted viral sequencing run have
  been trimmed, filtered, and de novo assembled, and a -scoring reference genome from
  a BLAST search in the virus reference database is available to guide consensus construction.
when_to_use_negative:
- No suitable reference can be identified — de novo contigs do not match any entry
  in the reference database or only partially
- The sample targets large DNA viruses with extensive repetitive regions (e.g. mpox),
  where the pipeline itself warns assemblies 'may contain inaccuracies and should
  be regarded as draft genomes'
- Reads have not been adapter/primer-trimmed or filtered for host/contaminant/non-viral
  content, so mapping against the reference would include off-target reads
edam_operation: http://edamontology.org/operation_0310
edam_topics:
- http://edamontology.org/topic_0196
- http://edamontology.org/topic_3673
- http://edamontology.org/topic_0622
tools:
- name: Sniffles2
  role: Calls large insertions/deletions from the per-reference BAM into a VCF that
    is applied to the reference
- name: BCFtools
  role: Applies the Sniffles2 indel VCF to the reference sequence ('bcftools consensus')
    to yield the indel-corrected draft
- name: Medaka
  role: Polishes the draft with a neural-network consensus model ('auto' unless medaka_consensus_model
    is specified)
  repo: https://github.com/nanoporetech/medaka
- name: Samtools
  role: Maps/sorts/indexes reads against the reference and provides the fallback consensus
    path (min depth 20, min 70% base proportion)
- name: ViMOP
  role: Nextflow-based pipeline orchestrating the reference-guided consensus assembly
    workflow
  repo: https://github.com/opr-group-bnitm/vimop
- name: Nextflow
  role: Executes the ViMOP pipeline with a single command or via EPI2ME Desktop
  repo: https://www.nextflow.io/
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1093/bioinformatics/btaf687
    title: 'ViMOP: a user-friendly and field-applicable pipeline for untargeted viral
      genome nanopore sequencing'
  source_content_hash: sha256:6920e06c20fd22441f3a874e300f84bef55624a459859a0e5f84b45175083fd7
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/nanoporetech/medaka
  tools:
  - BCFtools
  - Medaka
  - Nextflow
  - Samtools
  - Sniffles2
  - ViMOP
  edam_operation: http://edamontology.org/operation_0310
  edam_topics:
  - http://edamontology.org/topic_0196
  - http://edamontology.org/topic_3673
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5714285714285714
n_claims: 7
---

# reference-guided-viral-genome-assembly

## Summary

Reconstructs consensus viral genomes from untargeted nanopore reads by mapping trimmed reads against a BLAST-selected reference, integrating Sniffles2-called indels into a BCFtools consensus draft, and polishing the final sequence with Medaka or Samtools.

## When to use

Use when nanopore reads from an untargeted viral sequencing run have been trimmed, filtered, and de novo assembled, and a best-scoring reference genome from a BLAST search in the virus reference database is available to guide consensus construction.

## When NOT to use

- No suitable reference can be identified — de novo contigs do not match any entry in the reference database or only partially
- The sample targets large DNA viruses with extensive repetitive regions (e.g. mpox), where the pipeline itself warns assemblies 'may contain inaccuracies and should be regarded as draft genomes'
- Reads have not been adapter/primer-trimmed or filtered for host/contaminant/non-viral content, so mapping against the reference would include off-target reads

## Inputs

- Trimmed nanopore FASTQ reads (per barcode/sample)
- BLAST-selected reference genome sequence from the ViMOP virus reference database
- Sorted/indexed per-reference alignment BAM
- Sniffles2 VCF of large indels (<INV> records pre-filtered)
- Optional: medaka_consensus_model parameter

## Outputs

- Consensus genome FASTA (e.g. AB627954.consensus.fasta) with low-confidence positions N-masked
- Per-reference alignment BAM (e.g. AB627954.reads.bam)
- Depth file
- Variant VCF from Sniffles2

## How to apply

First map the trimmed nanopore reads against the BLAST-selected reference genome and sort/index the alignment with Samtools to produce a per-reference BAM. Call large insertions and deletions from this BAM with Sniffles2 into a VCF, pre-filtering any <INV> records before applying the indels to the reference with 'bcftools consensus' to produce an indel-corrected draft (the <INV> filtering avoids a known BCFtools consensus error). Polish the draft with Medaka using model 'auto' unless a specific medaka_consensus_model is given; if no matching basecaller model can be fitted, fall back to the Samtools consensus path, which requires a minimum depth of 20 and a minimum nucleotide proportion of 70%, calling the most abundant base and masking positions below these thresholds with N. Write the final consensus FASTA (e.g. AB627954.consensus.fasta) together with the alignment BAM (AB627954.reads.bam), depth file, and variant VCF into the sample's consensus output directory.

## Related tools

- **Sniffles2** (Calls large insertions/deletions from the per-reference BAM into a VCF that is applied to the reference)
- **BCFtools** (Applies the Sniffles2 indel VCF to the reference sequence ('bcftools consensus') to yield the indel-corrected draft)
- **Medaka** (Polishes the draft with a neural-network consensus model ('auto' unless medaka_consensus_model is specified)) — https://github.com/nanoporetech/medaka
- **Samtools** (Maps/sorts/indexes reads against the reference and provides the fallback consensus path (min depth 20, min 70% base proportion))
- **ViMOP** (Nextflow-based pipeline orchestrating the reference-guided consensus assembly workflow) — https://github.com/opr-group-bnitm/vimop
- **Nextflow** (Executes the ViMOP pipeline with a single command or via EPI2ME Desktop) — https://www.nextflow.io/

## Examples

```
nextflow run opr-group-bnitm/vimop --fastq data/ --targets "EBOV" -profile docker
```

## Evaluation signals

- The consensus FASTA and the alignment BAM both exist in the sample's consensus output directory
- The consensus contains no ambiguous bases other than N-masked positions
- All positions with coverage below the depth-20 threshold or with base proportion below 70% are represented as N in the Samtools-consensus path
- No <INV> records remain in the VCF that is passed to 'bcftools consensus' (pre-filtered to avoid the known error)
- The Sniffles2 VCF records the large indels actually detected from the per-reference BAM

## Limitations

- For large DNA viruses with extensive repetitive regions (e.g. mpox), assemblies may contain inaccuracies and should be regarded as draft genomes
- A known unresolved error occurs when <INV>-denoted inversions are present in the VCF before consensus calling with bcftools, so <INV> records must be pre-filtered
- The Samtools consensus fallback masks positions below depth 20 or below 70% base proportion with N, so low-coverage regions yield ambiguous output
- Carefully review the output (e.g. the alignment .bam files); Centrifuge-based filtering is limited in accuracy and may remove some false positives

## Evidence

- [methods] The reference-guided assembly is a two-stage process: [evidence span withheld — anchor status: not_found]
- [methods] Samtools consensus thresholds: [evidence span withheld — anchor status: not_found]
- [discussion] <INV> pre-filtering rationale: "error when inversions denoted with <INV> are in the vcf-file before consensus calling with bcftools"
- [readme] Pipeline scope and draft-genome caveat: "for large DNA viruses with extensive repetitive regions (e.g., mpox), assemblies may contain inaccuracies and should be regarded as draft genomes"
- [methods] Reference identification via BLAST: "Each contig is used to for a BLAST search in the virus reference database. The highest scoring hit is then used as a reference genome."
