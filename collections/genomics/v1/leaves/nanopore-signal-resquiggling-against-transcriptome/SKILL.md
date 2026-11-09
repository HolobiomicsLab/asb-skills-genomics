---
name: nanopore-signal-resquiggling-against-transcriptome
description: Use when raw Nanopore direct RNA sequencing FAST5 files must be basecalled,
  re-squiggled, and aligned to a reference transcriptome to produce signal-mapped
  reads for downstream per-site feature extraction (e.g. RedNano m6A detection).
when_to_use_negative:
- Input is already basecalled, resquiggled FAST5s with a RawGenomeCorrected_001 group
  — re-running resquiggle would be redundant (though --overwrite would be required)
- Reference available is only a whole-genome FASTA — Tombo resquiggle in this pipeline
  requires the reference transcript/gene FASTA, not a genome file
- Downstream task needs only modified-probability prediction from an existing aggregated
  feature table (e.g. test.features.sort.aggre.c20.tsv) — preprocessing is already
  done
edam_operation: http://edamontology.org/operation_0292
edam_topics:
- http://edamontology.org/topic_3168
tools:
- name: ont-fast5-api
  role: Splits multi-read FAST5 files into single-read FAST5 files (multi_to_single_fast5)
    as the first preprocessing step
  repo: https://github.com/Derryxu/RedNano
- name: Guppy
  role: Basecalls single-read FAST5 files with the rna_r9.4.1_70bps_hac.cfg config,
    emitting FASTQ and FAST5 outputs used for resquiggling
- name: ont-tombo
  role: Re-squiggle module maps raw current signals to contiguous reference transcript
    bases into the RawGenomeCorrected_001 group
- name: minimap2
  role: Aligns merged basecalled reads to the reference transcriptome (-t 30 -ax map-ont)
- name: samtools
  role: Views, sorts (-@ 30) and indexes (samtools index) the minimap2 alignment to
    produce test.bam
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/bioinformatics/btae375
    title: RNA m6A detection using raw current signals and basecalling errors from
      Nanopore direct RNA sequencing reads
  source_content_hash: sha256:4ac32a230beceb64ea378a981ae2761b3110c27f68c1871c6c22cfe2369ff863
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Derryxu/RedNano
  tools:
  - Guppy
  - RedNano
  - minimap2
  - ont-fast5-api
  - ont-tombo
  - samtools
  edam_operation: http://edamontology.org/operation_0292
  edam_topics:
  - http://edamontology.org/topic_3168
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.625
n_claims: 8
---

# Nanopore signal re-squiggling against a transcriptome reference

## Summary

A RedNano preprocessing skill that converts raw Nanopore DRS FAST5 reads into signal-to-base aligned data: split multi-read FAST5s, basecall with Guppy (rna_r9.4.1_70bps_hac.cfg), re-squiggle raw current signals onto transcript reference bases with Tombo, and align basecalled reads with minimap2/samtools.

## When to use

Use when raw Nanopore direct RNA sequencing FAST5 files must be basecalled, re-squiggled, and aligned to a reference transcriptome to produce signal-mapped reads for downstream per-site feature extraction (e.g. RedNano m6A detection).

## When NOT to use

- Input is already basecalled, resquiggled FAST5s with a RawGenomeCorrected_001 group — re-running resquiggle would be redundant (though --overwrite would be required)
- Reference available is only a whole-genome FASTA — Tombo resquiggle in this pipeline requires the reference transcript/gene FASTA, not a genome file
- Downstream task needs only modified-probability prediction from an existing aggregated feature table (e.g. test.features.sort.aggre.c20.tsv) — preprocessing is already done

## Inputs

- Raw Nanopore DRS multi-read FAST5 files (optionally already single-read)
- Reference transcript/gene FASTA (e.g. referance.transcript.fa), not a genome FASTA
- Guppy rna_r9.4.1_70bps_hac.cfg basecalling configuration
- Tombo v1.5.1, minimap2 v2.17, samtools v1.7, ont-fast5-api v4.0.2 environment

## Outputs

- Single-read FAST5 files with basecalls in group Basecall_1D_001
- Merged basecalled FASTQ (test.fastq)
- Resquiggled FAST5 workspace with signals mapped to reference bases in corrected group RawGenomeCorrected_001
- Sorted, indexed BAM (test.bam plus test.bam.bai) of transcriptome-aligned reads

## How to apply

First split multi-read FAST5 files into single-read FAST5 files with ont_fast5_api's multi_to_single_fast5 (-t 30), then basecall with guppy_basecaller using rna_r9.4.1_70bps_hac.cfg (--fast5_out, 2 GPU runners per device, 2500 chunks per runner, CUDA:0) and merge the resulting FASTQs into test.fastq. Re-squiggle raw signals with 'tombo resquiggle' against the reference transcript (gene) FASTA — explicitly not a genome file — using --fit-global-scale --include-event-stdev, basecall-group Basecall_1D_001, corrected-group RawGenomeCorrected_001, and 30 processes. The rationale is that the RawGenomeCorrected_001 group stores raw current signals mapped to contiguous reference bases, which RedNano's feature extraction needs alongside minimap2 (v2.17, -ax map-ont) alignments of the merged reads, sorted and indexed with samtools (v1.7). Validate that the resquiggled workspace and a sorted, indexed BAM were produced before proceeding to feature extraction.

## Related tools

- **ont-fast5-api** (Splits multi-read FAST5 files into single-read FAST5 files (multi_to_single_fast5) as the first preprocessing step) — https://github.com/Derryxu/RedNano
- **Guppy** (Basecalls single-read FAST5 files with the rna_r9.4.1_70bps_hac.cfg config, emitting FASTQ and FAST5 outputs used for resquiggling)
- **ont-tombo** (Re-squiggle module maps raw current signals to contiguous reference transcript bases into the RawGenomeCorrected_001 group)
- **minimap2** (Aligns merged basecalled reads to the reference transcriptome (-t 30 -ax map-ont))
- **samtools** (Views, sorts (-@ 30) and indexes (samtools index) the minimap2 alignment to produce test.bam)

## Examples

```
tombo resquiggle --overwrite /path/to/fast5_guppy/workspace/ --basecall-group Basecall_1D_001 referance.transcript.fa --fit-global-scale --include-event-stdev --corrected-group RawGenomeCorrected_001 --processes 30
```

## Evaluation signals

- test.bam exists, is a valid sorted BAM, and a BAM index (test.bam.bai) was created via samtools index
- The resquiggled FAST5 workspace contains reads with the RawGenomeCorrected_001 corrected group populated
- All per-file FASTQ outputs from guppy were merged into a single non-empty test.fastq
- Alignment was run against the transcript/gene FASTA (not genome), consistent with Tombo resquiggle requirements
- Resquiggle completed with the specified parameters (basecall-group Basecall_1D_001, --fit-global-scale, --include-event-stdev, --processes 30) without read-dropout across the sample set

## Limitations

- Tombo resquiggle requires the reference transcript (gene) FASTA; supplying a genome file is incorrect for this pipeline
- The README documents a mismatch between tool versions (article card says Guppy config workflow; README workflow states guppy v3.1.5) — version consistency should be checked
- FAST5 files must be split to single-read format first; multi-read FAST5s are not directly usable by the resquiggle workflow
- GPU resources are required for Guppy basecalling (CUDA:0, 2 gpu_runners_per_device, 2500 chunks_per_runner)

## Evidence

- [readme] Tombo resquiggle requires transcript/gene FASTA: "The tombo (v1.5.1) resquiggle referance.transcript.fa should not be genome file, it should be the referance gene fasta file."
- [readme] Tombo resquiggle command with parameters: "tombo resquiggle --overwrite /path/to/fast5_guppy/workspace/ --basecall-group Basecall_1D_001 referance.transcript.fa --fit-global-scale --include-event-stdev --corrected-group RawGenomeCorrected_001"
- [readme] Guppy basecalling configuration: "guppy_basecaller -i /path/to/fast5/ -r -s /path/to/fast5_guppy --fast5_out -c rna_r9.4.1_70bps_hac.cfg --gpu_runners_per_device 2 --chunks_per_runner 2500"
- [methods] Alignment and BAM sorting rationale from the article finding: [evidence span withheld — anchor status: not_found]
- [readme] minimap2 alignment piped to samtools: "minimap2 -t 30 -ax map-ont /path/to/referance.transcript.fa /test.fastq | samtools view -hSb | samtools sort -@ 30 -o test.bam"
