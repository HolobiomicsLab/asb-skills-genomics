---
name: shannon-entropy-repetitive-region-detection
description: Use when you need to de novo identify candidate tandem repeat regions
  from haplotype-resolved ONT or PacBio whole-genome long-read BAM alignments (e.g.,
  HGSVC data) with no prior motif or locus annotation.
when_to_use_negative:
- The target repeats lie in unassembled genome regions (e.g., human centromeres and
  telomeres), which TRiCoLOR cannot identify
- The genome is polyploid or haploid (e.g., the human Y chromosome), which TRiCoLOR
  does not yet support
- You already have an annotated TR locus and only want genotyping of a known motif
  rather than de novo region discovery
edam_operation: http://edamontology.org/operation_3192
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_0102
tools:
- name: TRiCoLOR SENSoR
  role: 'Core module: computes Shannon entropy per read in 20 bp non-overlapping sliding
    windows, thresholds at 1.23, requires >=5 supporting reads, merges windows within
    100 bp, and emits candidate repetitive regions as BED'
  repo: https://github.com/davidebolo1993/TRiCoLOR/tree/master/TRiCoLOR
- name: TRiCoLOR
  role: Parent pipeline whose first stage (SENSoR) this skill executes; candidate
    regions are then consumed by the REFER module for motif profiling
  repo: https://github.com/davidebolo1993/TRiCoLOR
provenance:
  source_task_ids:
  - task_005
  - task_008
  - task_006
  source_papers:
  - doi: 10.1093/gigascience/giaa101
    title: 'TRiCoLOR: tandem repeat profiling using whole-genome long-read sequencing
      data'
  source_content_hash: sha256:74c2dd76b5783d30e223e4e6e44caa9be11d75feff851a95ca00d2d4b30d2761
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Docker
  - NCRF
  - SPOA
  - TRiCoLOR
  - TRiCoLOR REFER
  - TRiCoLOR SENSoR
  - minimap2
  edam_operation: http://edamontology.org/operation_3192
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_0102
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.85
n_claims: 20
---

# shannon-entropy-repetitive-region-detection

## Summary

Scan haplotype-resolved long-read (ONT/PacBio) BAM alignments with TRiCoLOR SENSoR, which uses the Shannon entropy of DNA sequences in sliding windows to flag candidate repetitive regions de novo, without prior knowledge of motifs or locations. The output BED anchors the downstream TRiCoLOR REFER genotyping step.

## When to use

Use when you need to de novo identify candidate tandem repeat regions from haplotype-resolved ONT or PacBio whole-genome long-read BAM alignments (e.g., HGSVC data) with no prior motif or locus annotation.

## When NOT to use

- The target repeats lie in unassembled genome regions (e.g., human centromeres and telomeres), which TRiCoLOR cannot identify
- The genome is polyploid or haploid (e.g., the human Y chromosome), which TRiCoLOR does not yet support
- You already have an annotated TR locus and only want genotyping of a known motif rather than de novo region discovery

## Inputs

- Haplotype-resolved long-read BAM alignments (ONT or PacBio) aligned to GRCh38
- Sequencing accuracy ~0.85–0.95 for simulated inputs (VISOR-generated BAMs)
- Optional --size override for longer (mini-satellite) motif lengths in the downstream REFER step

## Outputs

- BED file of candidate low-entropy repetitive regions
- Coverage-filtered BED regions (mean coverage >8x ONT, >10x PB) ready for TRiCoLOR REFER

## How to apply

Run TRiCoLOR SENSoR on each haplotype-specific BAM: it computes, for each sequencing read, its Shannon entropy in non-overlapping sliding windows of 20 bp (default); windows where multiple reads show an entropy drop below the default threshold of 1.23 and are supported by at least 5 reads are marked as candidate repetitive, and nearby windows within 100 bp intervals are merged into candidate regions emitted in BED format. The rationale is that repetitive sequence has reduced Shannon entropy, and the article reports that the 1.23 threshold efficiently discriminates repetitive from non-repetitive DNA on synthetic ONT and PB reads. On real data, filter the resulting low-entropy regions by mean coverage — keep regions with mean coverage >8x for ONT and >10x for PB — before passing the BED to TRiCoLOR REFER for consensus building (SPOA) and motif profiling.

## Related tools

- **TRiCoLOR SENSoR** (Core module: computes Shannon entropy per read in 20 bp non-overlapping sliding windows, thresholds at 1.23, requires >=5 supporting reads, merges windows within 100 bp, and emits candidate repetitive regions as BED) — https://github.com/davidebolo1993/TRiCoLOR/tree/master/TRiCoLOR
- **TRiCoLOR** (Parent pipeline whose first stage (SENSoR) this skill executes; candidate regions are then consumed by the REFER module for motif profiling) — https://github.com/davidebolo1993/TRiCoLOR

## Evaluation signals

- Output BED is non-empty and every reported region contains windows with an entropy drop <=1.23 supported by >=5 reads
- Nearby candidate windows were merged within 100 bp intervals and window size is 20 bp non-overlapping as per defaults
- On real HGSVC data, coverage filtering reduces ONT regions to those with mean coverage >8x and PB regions to those with mean coverage >10x (PB regions reduced to ~150,000 in the paper)
- When chained with REFER, per-class recall on simulated ONT/PB BAMs should match reported benchmarks (TRiCoLOR R = 0.97 contractions, 0.86 expansions, 0.73 long expansions vs NCRF R = 0.87/0.74/0.53)
- Pipeline completes without errors on real ONT/PB BAMs and downstream BCF/BED outputs exist and are non-empty

## Limitations

- The entropy threshold (1.23) and window size (20 bp) were empirically estimated on synthetic ONT and PB reads; exact values for de novo identification of other repeat classes (e.g., mini-satellites) are not reported or validated
- Repeats in unassembled regions of the genome (e.g., human centromeres and telomeres) cannot be identified
- No support for polyploid species or haploid chromosomes such as the human Y chromosome

## Evidence

- [methods] This is achieved using the SENSoR module, which uses the Shannon entropy of DNA sequences to identify candidate repetitive segments in genomic sequences: "This is achieved using the SENSoR module, which uses the Shannon entropy of DNA sequences to identify candidate repetitive segments in genomic sequences"
- [methods] The default entropy treshold of 1.23 efficiently discriminates between repetitive and non-repetitive DNA sequences using syntethic ONT and PB reads: "The default entropy treshold of 1.23 efficiently discriminates between repetitive and non-repetitive DNA sequences using syntethic ONT and PB reads"
- [methods] computes, for each sequencing read, its Shannon entropy content in non-overlapping, sliding windows of a pre-trained size (20 bp, by default).: "computes, for each sequencing read, its Shannon entropy content in non-overlapping, sliding windows of a pre-trained size (20 bp, by default)."
- [results] after filtering for regions with mean coverage >8: "after filtering for regions with mean coverage >8"
- [readme] TRiCoLOR is available to be installed from source or as a [Docker container](https://hub.docker.com/r/davidebolo1993/tricolor).: "TRiCoLOR is available to be installed from source or as a [Docker container](https://hub.docker.com/r/davidebolo1993/tricolor)."
