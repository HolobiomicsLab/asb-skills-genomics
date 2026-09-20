---
name: minisatellite-motif-size-parameter-tuning
description: Use when the analysis goal is de novo identification and genotyping of
  tandem repeats with motif length ≥7 bp (mini-satellites) from haplotype-resolved
  ONT or PacBio BAM alignments, requiring overriding TRiCoLOR's default motif-length
  parameter.
when_to_use_negative:
- The target regions are unassembled parts of the genome (e.g., human centromeres
  and telomeres), where TRiCoLOR cannot identify repeats
- The sample is polyploid or contains haploid chromosomes (e.g., the human Y chromosome),
  which TRiCoLOR does not support
- Default micro-satellite profiling (motifs ≤6 bp) suffices — in that case run TRiCoLOR
  with its default motif-length setting instead of tuning '--size'
edam_topics:
- http://edamontology.org/topic_0102
tools:
- name: TRiCoLOR SENSoR
  role: Scans reads for low-Shannon-entropy windows (20 bp, entropy ≤1.23, ≥5 reads)
    to delimit candidate repetitive regions supplied to REFER
  repo: https://github.com/davidebolo1993/TRiCoLOR/tree/master/TRiCoLOR
- name: TRiCoLOR REFER
  role: Motif-resolution module whose '--size' parameter is tuned from the default
    ≤6 bp to ≥7 bp to detect mini-satellite motifs
  repo: https://github.com/davidebolo1993/TRiCoLOR/tree/master/TRiCoLOR
- name: TRiCoLOR
  role: Overall de novo tandem repeat profiling and genotyping pipeline run on haplotype-resolved
    long-read alignments
  repo: https://github.com/davidebolo1993/TRiCoLOR
- name: minimap2
  role: Aligns SPOA-derived haplotype consensus sequences to the GRCh38 reference;
    chosen for favorable speed and mapping accuracy on synthetic data
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1093/gigascience/giaa101
    title: 'TRiCoLOR: tandem repeat profiling using whole-genome long-read sequencing
      data'
  source_content_hash: sha256:1fc0d6ccd29a69869d1534813b025174c4d8099acbdbfaf89061a7c76d20f456
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - TRiCoLOR
  - TRiCoLOR REFER
  - TRiCoLOR SENSoR
  - minimap2
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0102
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.875
n_claims: 16
---

# minisatellite-motif-size-parameter-tuning

## Summary

Parameterizes TRiCoLOR (via the '--size' option of the REFER module) so its RegEx-based matcher detects motifs longer than the default ≤6 bp micro-satellite setting, enabling de novo mini-satellite profiling from haplotype-resolved long-read alignments.

## When to use

Use when the analysis goal is de novo identification and genotyping of tandem repeats with motif length ≥7 bp (mini-satellites) from haplotype-resolved ONT or PacBio BAM alignments, requiring overriding TRiCoLOR's default motif-length parameter.

## When NOT to use

- The target regions are unassembled parts of the genome (e.g., human centromeres and telomeres), where TRiCoLOR cannot identify repeats
- The sample is polyploid or contains haploid chromosomes (e.g., the human Y chromosome), which TRiCoLOR does not support
- Default micro-satellite profiling (motifs ≤6 bp) suffices — in that case run TRiCoLOR with its default motif-length setting instead of tuning '--size'

## Inputs

- Haplotype-resolved long-read BAM alignment files (ONT or PacBio) of a diploid human whole-genome sample
- GRCh38 reference genome
- SENSoR candidate repetitive regions in BED format
- Optional trio BAM files for Mendelian consistency tracing

## Outputs

- Mini-satellite call set in BCF-compliant format
- BED file describing the identified tandem repeats
- Haplotype-specific BAM files with aligned consensus sequences

## How to apply

Run TRiCoLOR SENSoR on the haplotype-resolved BAM files to scan 20 bp non-overlapping sliding windows for Shannon entropy drops ≤1.23 supported by ≥5 reads, merging nearby windows within 100 bp and emitting candidate repetitive regions in BED format. Feed the SENSoR BED to TRiCoLOR REFER, overriding the default motif length (≤6) with the '--size' parameter so the RegEx matcher identifies perfectly repeated motifs of length ≥7 with a minimum of 5 perfect repeats. SPOA computes haplotype-specific low-error consensus sequences (approximate error reduction of ~77% and ~88% for ONT and PB), which are aligned to GRCh38 with minimap2; approximate string matching then tolerates imperfect repeats up to edit distance ≤1, overlapping predictions are resolved with the N-gram model, and only TRs ≥50 bp varying between haplotypes or the reference are written to BCF-compliant output. Because the paper reports no ground truth for mini-satellite profiling, validation is limited to confirming the pipeline completes without errors and outputs are non-empty.

## Related tools

- **TRiCoLOR SENSoR** (Scans reads for low-Shannon-entropy windows (20 bp, entropy ≤1.23, ≥5 reads) to delimit candidate repetitive regions supplied to REFER) — https://github.com/davidebolo1993/TRiCoLOR/tree/master/TRiCoLOR
- **TRiCoLOR REFER** (Motif-resolution module whose '--size' parameter is tuned from the default ≤6 bp to ≥7 bp to detect mini-satellite motifs) — https://github.com/davidebolo1993/TRiCoLOR/tree/master/TRiCoLOR
- **TRiCoLOR** (Overall de novo tandem repeat profiling and genotyping pipeline run on haplotype-resolved long-read alignments) — https://github.com/davidebolo1993/TRiCoLOR
- **minimap2** (Aligns SPOA-derived haplotype consensus sequences to the GRCh38 reference; chosen for favorable speed and mapping accuracy on synthetic data)

## Examples

```
tricolor REFER --bams hg00733.bam --vcf minisatellites.bcf --size 7
```

## Evaluation signals

- Pipeline completes without errors on real ONT/PB HGSVC inputs (HG00733, HG00514, NA19240, HG00731, HG00732)
- Output BCF file exists and is non-empty, containing calls in BCF-compliant format
- Output BED file of identified TRs exists and is non-empty
- Reported calls satisfy the ≥50 bp minimum TR length filter and vary between haplotypes or the reference
- Predicted repeat motifs have length ≥7 bp (i.e., the '--size' override took effect over the ≤6 bp default)

## Limitations

- Parameter values for profiling mini-satellites are not validated; the tool has been extensively applied only to micro-satellites, so there is no ground truth for mini-satellite profiling
- Exact entropy threshold and window size for de novo identification were only empirically estimated; the discussion notes ranges rather than validated values
- No support for polyploid species or haploid chromosomes (e.g., the human Y chromosome)
- Repeats in unassembled regions (centromeres, telomeres) cannot be detected

## Evidence

- [methods] identifying motifs (motifs of length ≤6, by default) that are perfectly repeated a minimum number of times (5, by default): "identifying motifs (motifs of length ≤6, by default) that are perfectly repeated a minimum number of times (5, by default)"
- [methods] Imperfect repeats and overlap resolution: "looking for approximate repetitions of the identified motifs to account for remaining consensus errors, i.e., imperfect repeated motifs up to a user-defined edit distance (≤1, by default)"
- [discussion] Scope limitation of validation: "TRiCoLOR has been extensively applied so far only to micro-satellites"
- [readme] Readme points to usage documentation: "Please have a look at [TRiCoLOR documentation](https://davidebolo1993.github.io/tricolordoc/) for any installation or usage questions."
