---
name: multi-technology-alignment-preprocessing
description: Use when comparing variant-calling accuracy across Illumina, PacBio,
  and ONT reads and the input BAM/FASTQ alignments need to be harmonized (coverage
  downsampled, reads re-aligned to a decoy reference) before calling.
when_to_use_negative:
- The input is already a set of consistently aligned BAMs at matched coverage — re-alignment
  would only add variance
- The target is somatic/trio variant calling or multi-allelic variants, which the
  Clairvoyante pipeline does not support
- The sequencing technology is novel and lacks a high-quality GIAB-style training
  dataset on standard samples
edam_operation: http://edamontology.org/operation_3182
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3168
tools:
- name: NGMLR
  role: Long-read aligner (v0.2.3) used to re-align PacBio and ONT reads to GRCh37
    decoy version 5
- name: Clairvoyante
  role: Downstream five-layer CNN variant caller that consumes the harmonized BAMs
    for cross-technology F1 evaluation
  repo: https://github.com/aquaskyline/Clairvoyante
- name: Oxford Nanopore
  role: Source of the 'rel3' MinION 1D R9.4 whole-genome HG001 reads used as the ONT
    input
  repo: https://github.com/nanopore-wgs-consortium/NA12878
- name: PacBio
  role: Source of the PacBio reads re-aligned with NGMLR to the decoy reference
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-019-09025-z
    title: A multi-task convolutional deep neural network for variant calling in single
      molecule sequencing
  source_content_hash: sha256:c7935b462f3b5d0fe98c837eda1657f350d9dfea82c67484ca90b4362628305b
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/aquaskyline/Clairvoyante
  tools:
  - Clairvoyante
  - Illumina
  - NGMLR
  - Oxford Nanopore
  - PacBio
  edam_operation: http://edamontology.org/operation_3182
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3168
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# multi-technology-alignment-preprocessing

## Summary

Prepare BAM alignments from Illumina, PacBio, and Oxford Nanopore whole-genome data so that downstream variant callers (e.g., Clairvoyante) can be evaluated comparably across sequencing technologies, following the Clairvoyante HG001 workflow.

## When to use

Use when comparing variant-calling accuracy across Illumina, PacBio, and ONT reads and the input BAM/FASTQ alignments need to be harmonized (coverage downsampled, reads re-aligned to a decoy reference) before calling.

## When NOT to use

- The input is already a set of consistently aligned BAMs at matched coverage — re-alignment would only add variance
- The target is somatic/trio variant calling or multi-allelic variants, which the Clairvoyante pipeline does not support
- The sequencing technology is novel and lacks a high-quality GIAB-style training dataset on standard samples

## Inputs

- HG001 (NA12878) Illumina BAM alignments (downsampled to 50×)
- PacBio BAM/reads re-aligned to GRCh37 decoy version 5 with NGMLR v0.2.3
- Oxford Nanopore 'rel3' MinION 1D R9.4 whole-genome reads (nanopore-wgs-consortium release)
- GRCh37 decoy version 5 reference genome

## Outputs

- Coverage-matched, consistently aligned BAM files per technology suitable for genome-wide variant calling
- Three technology-specific callsets (VCF) producible downstream with Clairvoyante at a 0.2 alternative allele frequency cutoff

## How to apply

Start with three HG001 BAM sets: Illumina reads, PacBio reads, and Oxford Nanopore MinION 1D R9.4 'rel3' reads. Downsample the Illumina dataset to 50× coverage to match the available coverage of the SMS technologies. Re-extract PacBio and ONT reads from their downloaded alignments and re-align them to GRCh37 decoy version 5 using NGMLR version 0.2.3, which handles the indel-enriched error profile of single-molecule reads better than generic aligners. Do not remove alignments by mapping quality, as the Clairvoyante authors explicitly advise against it. The resulting BAMs (50× Illumina, NGMLR-realigned PacBio and ONT to GRCh37 decoy v5) are then fed genome-wide to a caller such as Clairvoyante for cross-technology comparison against the GIAB v3.3.2 HG001 truth set with RTG vcfeval.

## Related tools

- **NGMLR** (Long-read aligner (v0.2.3) used to re-align PacBio and ONT reads to GRCh37 decoy version 5)
- **Clairvoyante** (Downstream five-layer CNN variant caller that consumes the harmonized BAMs for cross-technology F1 evaluation) — https://github.com/aquaskyline/Clairvoyante
- **Oxford Nanopore** (Source of the 'rel3' MinION 1D R9.4 whole-genome HG001 reads used as the ONT input) — https://github.com/nanopore-wgs-consortium/NA12878
- **PacBio** (Source of the PacBio reads re-aligned with NGMLR to the decoy reference)

## Examples

```
ngmlr -t 16 -r GRCh37_decoy_v5.fa -q rel3-nanopore-wgs.fastq.gz -o ont_rel3_grch37_decoy_v5.sam
```

## Evaluation signals

- Illumina BAM coverage should approximate 50× after downsampling, matching the SMS datasets
- PacBio and ONT BAM headers must indicate alignment to GRCh37 decoy version 5 with NGMLR 0.2.3
- Genome-wide F1-scores from the subsequent Clairvoyante calls should reproduce 98.65% (Illumina), 92.57% (PacBio), and 87.26% (ONT) against GIAB HG001 v3.3.2 via RTG vcfeval
- No alignments should have been removed by mapping-quality filtering per the authors' recommendation

## Limitations

- Illumina downsampling to 50× trades away depth to match SMS coverage, so absolute Illumina sensitivity may be understated
- ONT data were only available for HG001, limiting cross-sample evaluation and combined-sample training on ONT
- Downstream calling performance is asymmetric: Indel F1-score remains around 50% for ONT data
- The current Clairvoyante implementation does not consider base quality of the sequencing reads, so alignment preprocessing cannot compensate for per-base quality signal

## Evidence

- [discussion] Illumina downsampling: "we further downsampled the two datasets to 50× to match the available data coverage of the other two SMS technologies"
- [discussion] Re-alignment strategy: "Reads were extracted from the downloaded alignments and aligned again to GRCh37 decoy version 5 using NGMLR27 version 0.2.3"
- [discussion] ONT input dataset: "we used the ‘rel3’ release sequenced on the Oxford Nanopore MinION using 1D ligation kits (450 bp/s) and R9.4 chemistry"
- [discussion] No MAPQ filtering: "We do not suggest removing any alignment by their mapping quality"
- [discussion] Truth dataset for evaluation: "we used as our truth dataset the latest dataset version 3.3.2 for HG001"
