---
name: reference-genome-preparation
description: Use when starting an ONT SNV-calling benchmark or analysis and you need
  indexed reference assemblies (e.g. GRCh38, GRCm39, TAIR10, BDGP6) plus a miniSNV
  population-variant index.
when_to_use_negative:
- The analysis target is not SNV calling against a reference (e.g. de novo assembly
  or reference-free variant discovery), since miniSNV's pipeline is reference- and
  population-VCF-dependent
- Only indels or structural variants are of interest — miniSNV and this preparation
  workflow target SNV genotyping
- The input BAM is not aligned to the same assembly/contig naming as the reference
  FASTA (mismatched contig names will break chunked processing)
edam_operation: http://edamontology.org/operation_0362
edam_topics:
- http://edamontology.org/topic_0102
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3673
tools:
- name: miniSNV
  role: SNV caller whose inputs (samtools-indexed FASTA/BAM, optional known-variant
    index, --human flag for human BAMs) this preparation step must satisfy
  repo: https://github.com/CuiMiao-HIT/miniSNV
- name: Clair3
  role: DL-based SNV caller run alongside miniSNV; requires the ONT-trained model
    from the rerio clair3_models directory
  repo: https://github.com/nanoporetech/rerio
- name: Minimap2
  role: Aligner used to produce the sorted BAMs against each prepared reference in
    the simulated benchmark workflow
- name: WhatsHap
  role: Dependency installed for miniSNV's read-based phasing step on the prepared
    alignments
provenance:
  source_task_ids:
  - task_004
  - task_005
  source_papers:
  - doi: 10.1093/bib/bbae473
    title: 'miniSNV: accurate and fast single nucleotide variant calling from nanopore
      sequencing data'
  source_content_hash: sha256:193aa7bb86e1cacd1fbe3535c0b7af9fe9112d56bfd6debcc8adc7f875b04cf0
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-4.0
    url: https://github.com/CuiMiao-HIT/miniSNV
  repo_url: https://github.com/CuiMiao-HIT/miniSNV
  tools:
  - Clair3
  - DeepVariant
  - Longshot
  - Minimap2
  - Nanocaller
  - PEPPER-Margin-DeepVariant
  - Pbsim3
  - WhatsHap
  - miniSNV
  edam_operation: http://edamontology.org/operation_0362
  edam_topics:
  - http://edamontology.org/topic_0102
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3673
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5833333333333334
n_claims: 12
---

# reference-genome-preparation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Prepare species-appropriate reference genome FASTAs and the auxiliary indexes (samtools FASTA index, BAM index, and the miniSNV prebuilt known-variants index from 1000 Genomes population VCFs) required before SNV calling on ONT data.

## When to use

Use when starting an ONT SNV-calling benchmark or analysis and you need indexed reference assemblies (e.g. GRCh38, GRCm39, TAIR10, BDGP6) plus a miniSNV population-variant index.

## When NOT to use

- The analysis target is not SNV calling against a reference (e.g. de novo assembly or reference-free variant discovery), since miniSNV's pipeline is reference- and population-VCF-dependent
- Only indels or structural variants are of interest — miniSNV and this preparation workflow target SNV genotyping
- The input BAM is not aligned to the same assembly/contig naming as the reference FASTA (mismatched contig names will break chunked processing)

## Inputs

- Reference genome FASTA per species/assembly (GRCh38 chr2, GRCm39 chr2, TAIR10, BDGP6), samtools-indexed
- Sorted, samtools-indexed BAM of ONT reads aligned to the reference (e.g. Minimap2 alignments of Pbsim3-simulated reads)
- 1000 Genomes population VCF of known common variants (miniSNV default variation background)
- miniSNV index folder containing five files, if using --useindex

## Outputs

- Indexed reference FASTA (.fai) and indexed BAM (.bai) ready for miniSNV / Clair3 / Longshot
- miniSNV known-variants index folder usable with --useindex
- Prepared Clair3 model directory from nanoporetech/rerio (e.g. ONT r1041 sup models)

## How to apply

Obtain the correct public reference assembly for each species in the study (GRCh38 chr2 for Human, GRCm39 chr2 for Mouse, TAIR10 for Arabidopsis, BDGP6 for Drosophila) and samtools-index both the FASTA and every input BAM, since miniSNV's chr_chunk_task.py requires samtools-indexed inputs. For miniSNV, additionally download the prebuilt index folder (five files) or build the known common-variants background from a population VCF such as the 1000 Genomes default, because miniSNV uses this variation background to classify candidate loci as high- or low-quality (known variant with R_alt > 0.3; unknown with R_alt > 0.7 and no other candidate within 10 bp). For Clair3 runs on ONT data, fetch the matching ONT-trained model from the nanoporetech/rerio clair3_models directory. Match contig names between the reference, BAM, and any BED annotations, and use the --human flag only for human BAMs.

## Related tools

- **miniSNV** (SNV caller whose inputs (samtools-indexed FASTA/BAM, optional known-variant index, --human flag for human BAMs) this preparation step must satisfy) — https://github.com/CuiMiao-HIT/miniSNV
- **Clair3** (DL-based SNV caller run alongside miniSNV; requires the ONT-trained model from the rerio clair3_models directory) — https://github.com/nanoporetech/rerio
- **Minimap2** (Aligner used to produce the sorted BAMs against each prepared reference in the simulated benchmark workflow)
- **WhatsHap** (Dependency installed for miniSNV's read-based phasing step on the prepared alignments)

## Examples

```
python chr_chunk_task.py --fin_ref GRCh38.fa --fin_bam hg002_sup.bam --workDir work/ --useindex -i miniSNV_index/ --human --threads 16
```

## Evaluation signals

- Both the reference FASTA and input BAM carry valid samtools indices (.fai / .bai) before invoking chr_chunk_task.py
- The miniSNV index folder contains exactly five files when --useindex is set
- Reproduced F1 values match the reported benchmark, e.g. miniSNV F1 ~99.90% on Arabidopsis and ~99.58% on Human chr2 at 60x
- On the HG002 sup-dataset, miniSNV F1-score and recall exceed 99.7%, confirming the reference/index setup is correct
- Contig names in the reference, BAM, and population VCF are consistent per species (Human, Mouse, Drosophila, Arabidopsis)

## Limitations

- The prebuilt miniSNV index must be downloaded from an external drive link; without it, a population VCF (1000 Genomes default) must be provided
- Reference preparation is assembly-specific: results on GRCh38 do not transfer to non-human species (the paper observed unexpectedly low Clair3 recalls on non-human species)
- FMAlign and HAlign, alternative MSA tools in the pipeline, do not support producing multiple consensus sequences at heterozygous sites, limiting substitutability

## Evidence
