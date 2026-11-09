---
name: tandem-repeat-genotyping-from-long-reads
description: Use when profiling tandem repeats de novo from haplotype-resolved long-read
  alignments (ONT or PacBio BAMs) with no annotated motifs or positions, and per-haplotype
  genotypes are needed.
when_to_use_negative:
- Target repeats lie in unassembled regions such as human centromeres or telomeres,
  where TRiCoLOR cannot identify repeats.
- The genome is polyploid or contains haploid chromosomes (e.g., the human Y chromosome),
  which TRiCoLOR does not support.
- The profiled repeats are mini-satellites rather than micro-satellites, since TRiCoLOR
  has been extensively applied so far only to micro-satellites.
edam_operation: http://edamontology.org/operation_3227
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3308
tools:
- name: TRiCoLOR
  role: end-to-end tandem repeat profiler combining SENSoR and REFER modules for de
    novo TR identification and genotyping
  repo: https://github.com/davidebolo1993/TRiCoLOR
- name: TRiCoLOR SENSoR
  role: detects candidate repetitive regions via Shannon entropy in 20 bp sliding
    windows with entropy drop <= 1.23
  repo: https://github.com/davidebolo1993/TRiCoLOR
- name: TRiCoLOR REFER
  role: builds haplotype-specific consensus, realigns with minimap2, and resolves
    motif and multiplicity via RegEx approximate string matching
  repo: https://github.com/davidebolo1993/TRiCoLOR
- name: SPOA
  role: partial order alignment to compute low-error haplotype-specific consensus
    sequences (~77% ONT / ~88% PB error reduction)
- name: minimap2
  role: aligns consensus sequences and long reads to the GRCh38 reference genome
provenance:
  source_task_ids:
  - task_005
  - task_006
  source_papers:
  - doi: 10.1093/gigascience/giaa101
    title: 'TRiCoLOR: tandem repeat profiling using whole-genome long-read sequencing
      data'
  source_content_hash: sha256:cc93e8cdb650f631235071284494058520e5098381c9da58cdc31803db3c8901
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/davidebolo1993/TRiCoLOR
  tools:
  - Docker
  - NCRF
  - SPOA
  - TRiCoLOR
  - TRiCoLOR REFER
  - TRiCoLOR SENSoR
  - minimap2
  edam_operation: http://edamontology.org/operation_3227
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9333333333333333
n_claims: 15
---

# tandem-repeat-genotyping-from-long-reads

## Summary

De novo identification and genotyping of tandem repeats from haplotype-resolved ONT/PacBio long-read BAMs using TRiCoLOR's two modules, SENSoR (Shannon-entropy detection of repetitive regions) and REFER (POA consensus plus RegEx-based motif/multiplicity resolution), producing BCF-compliant TR calls without prior knowledge of motifs or locations.

## When to use

Use when profiling tandem repeats de novo from haplotype-resolved long-read alignments (ONT or PacBio BAMs) with no annotated motifs or positions, and per-haplotype genotypes are needed.

## When NOT to use

- Target repeats lie in unassembled regions such as human centromeres or telomeres, where TRiCoLOR cannot identify repeats.
- The genome is polyploid or contains haploid chromosomes (e.g., the human Y chromosome), which TRiCoLOR does not support.
- The profiled repeats are mini-satellites rather than micro-satellites, since TRiCoLOR has been extensively applied so far only to micro-satellites.

## Inputs

- haplotype-resolved long-read alignment BAM files (ONT or PacBio, e.g. minimap2-aligned to GRCh38)
- long-read FASTQ files (HGSVC ONT FASTQ aligned with minimap2)
- optional Illumina FM indexes of the same samples for validation

## Outputs

- candidate repetitive regions in BED format (SENSoR)
- tandem repeat calls with motif, multiplicity, and haplotype-specific genotypes in BCF-compliant format (REFER, TRs >= 50 bp)
- interactive visualizations of repeats within haplotype-specific sequence context
- Mendelian-consistency traces of TR genotypes in pedigrees

## How to apply

Run TRiCoLOR SENSoR on each haplotype-specific BAM using default Shannon-entropy settings — non-overlapping 20 bp sliding windows, entropy drop threshold <= 1.23, >= 5 supporting reads, and merging of nearby windows within 100 bp — to emit candidate repetitive regions in BED format. Optionally filter candidate regions by mean coverage (>8x for ONT, >10x for PB) as done on HGSVC data. Run TRiCoLOR REFER on the BED file: fetch and trim spanning reads, build haplotype-specific consensus sequences with SPOA (approximate error reduction ~77% for ONT, ~88% for PB), realign them to GRCh38 with minimap2, and detect motifs of length <= 6 bp perfectly repeated >= 5 times with approximate repetitions up to edit distance <= 1, resolving competing predictions with an N-gram model favoring the most frequent perfect repeat motif. Retain TRs >= 50 bp and store them in BCF-compliant format; downstream, visualize repeats interactively and trace Mendelian consistency in trios.

## Related tools

- **TRiCoLOR** (end-to-end tandem repeat profiler combining SENSoR and REFER modules for de novo TR identification and genotyping) — https://github.com/davidebolo1993/TRiCoLOR
- **TRiCoLOR SENSoR** (detects candidate repetitive regions via Shannon entropy in 20 bp sliding windows with entropy drop <= 1.23) — https://github.com/davidebolo1993/TRiCoLOR
- **TRiCoLOR REFER** (builds haplotype-specific consensus, realigns with minimap2, and resolves motif and multiplicity via RegEx approximate string matching) — https://github.com/davidebolo1993/TRiCoLOR
- **SPOA** (partial order alignment to compute low-error haplotype-specific consensus sequences (~77% ONT / ~88% PB error reduction))
- **minimap2** (aligns consensus sequences and long reads to the GRCh38 reference genome)

## Examples

```
docker pull davidebolo1993/tricolor
```

## Evaluation signals

- Benchmark against NCRF on VISOR-simulated haplotype-resolved ONT/PB BAMs: expected recall ~0.97 (contractions), ~0.86 (expansions), ~0.73 (long expansions) for TRiCoLOR vs ~0.87/0.74/0.53 for NCRF.
- At 1-motif discrepancy threshold, per-class F1 should be close to 1; at 2-motif threshold, precision and recall should both approach 1.
- Identified motifs must be correctly recovered in all simulated TRs (motif length <= 6 bp, >= 5 perfect repeats, edit distance <= 1).
- On real HGSVC data, validate calls against Illumina FM indexes (<= 2 bp discrepancies): expected validation ratios ~82% (HG00733), ~85% (HG00514), ~86% (NA19240).
- For HG00733, confirm non-reference TR alleles against the assembly-based truth set at ~42/58 (~75%) with same variant type and size difference <= 50 bp.

## Limitations

- TRiCoLOR cannot identify repeats in unassembled regions of the genome (e.g., human centromeres and telomeres).
- No support for polyploid species or haploid chromosomes such as the human Y chromosome (listed as future work).
- The tool has been extensively applied so far only to micro-satellites; mini-satellite profiling parameters are not validated.
- Haplotype-resolved long-read alignments are required as input; sequencing accuracy and per-haplotype coverage (e.g. 5-20x in simulations; coverage filters >8x ONT / >10x PB) drive performance.

## Evidence

- [abstract] The method can identify repetitive regions in sequencing data without a prior knowledge of their motifs or locations and resolve repeat multiplicity and period size in a haplotype-specific manner.: "The method can identify repetitive regions in sequencing data without a prior knowledge of their motifs or locations and resolve repeat multiplicity and period size in a haplotype-specific manner."
- [methods] computes, for each sequencing read, its Shannon entropy content in non-overlapping, sliding windows of a pre-trained size (20 bp, by default).: "computes, for each sequencing read, its Shannon entropy content in non-overlapping, sliding windows of a pre-trained size (20 bp, by default). Genomic coordinates of windows in which multiple reads"
- [methods] identifying motifs (motifs of length ≤6, by default) that are perfectly repeated a minimum number of times (5, by default): "identifying motifs (motifs of length ≤6, by default) that are perfectly repeated a minimum number of times (5, by default)"
- [methods] to compute highly accurate consensus sequences with an approximate error reduction of ~77% and ~88% for ONT and PB, respectively: "to compute highly accurate consensus sequences with an approximate error reduction of ~77% and ~88% for ONT and PB, respectively"
- [methods] TRs (those ≥50 bp, by default) varying between the haplotypes or the reference are eventually stored in BCF-compliant format: "TRs (those ≥50 bp, by default) varying between the haplotypes or the reference are eventually stored in BCF-compliant format"
- [discussion] TRiCoLOR cannot identify repeats in unassembled regions of the genome (e.g., human centromeres and telomeres): "TRiCoLOR cannot identify repeats in unassembled regions of the genome (e.g., human centromeres and telomeres)"
- [readme] A dockerized version of TRiCoLOR is available at https://hub.docker.com/r/davidebolo1993/tricolor: "A dockerized version of TRiCoLOR is available at https://hub.docker.com/r/davidebolo1993/tricolor"
