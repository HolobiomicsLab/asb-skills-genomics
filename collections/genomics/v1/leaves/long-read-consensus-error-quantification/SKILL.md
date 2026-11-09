---
name: long-read-consensus-error-quantification
description: Use when evaluating consensus-sequence quality of third-generation long
  reads — specifically when raw ONT/PacBio read error rates must be compared against
  SPOA/POA consensus error rates to compute a percentage error reduction.
when_to_use_negative:
- Input reads are already high-accuracy (e.g., Illumina or HiFi reads) — the skill
  targets error-prone third-generation reads where POA consensus yields substantial
  error reduction
- The goal is tandem repeat variant calling itself rather than benchmarking the consensus
  step's error reduction
- Repeats lie in unassembled genomic regions such as centromeres and telomeres, which
  TRiCoLOR cannot identify
edam_operation: http://edamontology.org/operation_3182
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3673
tools:
- name: TRiCoLOR REFER
  role: Computes haplotype-specific low-error consensus sequences via SPOA partial
    order alignment, the object of the error-reduction quantification
  repo: https://github.com/davidebolo1993/TRiCoLOR/tree/master/TRiCoLOR
- name: TRiCoLOR
  role: Pipeline whose REFER module generates the consensus BAMs; installable from
    source or Docker (davidebolo1993/tricolor)
  repo: https://github.com/davidebolo1993/TRiCoLOR
- name: minimap2
  role: Aligns raw simulated reads and consensus sequences to the same reference;
    compared favorably to alternative aligners in speed and mapping accuracy
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/gigascience/giaa101
    title: 'TRiCoLOR: tandem repeat profiling using whole-genome long-read sequencing
      data'
  source_content_hash: sha256:8d58e4ff9e7631c52b387d6f72175a074c4590e5b548d9a6b95ab7a92d54256b
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - TRiCoLOR
  - TRiCoLOR REFER
  - minimap2
  edam_operation: http://edamontology.org/operation_3182
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3673
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 5
---

# long-read-consensus-error-quantification

## Summary

Quantifies how much the POA-based (SPOA) consensus computation in TRiCoLOR's REFER module reduces per-base sequencing error of error-prone long reads, benchmarked against simulated ONT and PacBio data where reductions of ~77% (ONT) and ~88% (PB) are reported.

## When to use

Use when evaluating consensus-sequence quality of third-generation long reads — specifically when raw ONT/PacBio read error rates must be compared against SPOA/POA consensus error rates to compute a percentage error reduction.

## When NOT to use

- Input reads are already high-accuracy (e.g., Illumina or HiFi reads) — the skill targets error-prone third-generation reads where POA consensus yields substantial error reduction
- The goal is tandem repeat variant calling itself rather than benchmarking the consensus step's error reduction
- Repeats lie in unassembled genomic regions such as centromeres and telomeres, which TRiCoLOR cannot identify

## Inputs

- Haplotype-specific consensus BAM files from TRiCoLOR REFER (SPOA POA + minimap2 realignment)
- Raw error-prone long reads from third-generation sequencing (ONT and PacBio FASTQ, e.g., VISOR-simulated ONT reads with mean length 8,000 bp)
- Reference genome for minimap2 alignment (e.g., GRCh38)

## Outputs

- consensus_error_reduction.csv with columns: technology, raw error rate, consensus error rate, reduction percentage, reported reference value
- Per-base error rates (substitutions, insertions, deletions per aligned base) for raw and consensus reads, per platform

## How to apply

Generate haplotype-specific consensus BAM files with TRiCoLOR REFER, which uses SPOA partial order alignment followed by minimap2 realignment. Align raw simulated reads and consensus sequences to the same reference with minimap2 and compute per-base error rates (substitutions, insertions, deletions per aligned base) via edit-distance comparison. For VISOR-simulated reads, note the platform-specific error profiles: ONT ~45:25:30 (substitution:insertion:deletion) with mean read length 8,000 bp, and PacBio ~15:50:35. Compute error reduction as (raw error rate − consensus error rate) / raw error rate × 100, separately for ONT and PB, and compare against the reported ~77% (ONT) and ~88% (PB) from Supplementary Note S3 / Supplementary Fig. S2 with a tolerance of ±5 percentage points. Write results to consensus_error_reduction.csv with columns technology, raw error rate, consensus error rate, reduction percentage, and reported reference value.

## Related tools

- **TRiCoLOR REFER** (Computes haplotype-specific low-error consensus sequences via SPOA partial order alignment, the object of the error-reduction quantification) — https://github.com/davidebolo1993/TRiCoLOR/tree/master/TRiCoLOR
- **TRiCoLOR** (Pipeline whose REFER module generates the consensus BAMs; installable from source or Docker (davidebolo1993/tricolor)) — https://github.com/davidebolo1993/TRiCoLOR
- **minimap2** (Aligns raw simulated reads and consensus sequences to the same reference; compared favorably to alternative aligners in speed and mapping accuracy)

## Examples

```
docker run davidebolo1993/tricolor tricolor REFER --bam sample.bam --fasta ref.fa --threads 8  # then align consensus vs raw reads with minimap2 and compute reduction = (raw_err - cons_err)/raw_err*100
```

## Evaluation signals

- Computed error reduction falls within ±5 percentage points of ~77% for ONT and ~88% for PB
- Raw error rates reflect the simulated error profiles: ONT substitution:insertion:deletion ~45:25:30 and PB ~15:50:35
- consensus_error_reduction.csv exists with the required columns (technology, raw error rate, consensus error rate, reduction percentage, reported reference value)
- Raw and consensus reads are aligned to the identical reference with minimap2 so per-base error rates are directly comparable
- Consensus error rate is strictly lower than raw error rate on both platforms

## Limitations

- Reported values are approximate (~77% ONT, ~88% PB) and come from Supplementary Note S3 / Supplementary Fig. S2, whose full content is not included in the article text
- TRiCoLOR cannot identify repeats in unassembled regions of the genome (e.g., human centromeres and telomeres), bounding where consensus quality can be profiled
- Exact entropy threshold and window size values for de novo identification were empirically estimated and not fully reported in all sections

## Evidence

- [methods] SPOA achieves approximate error reduction of ~77% and ~88% for ONT and PB, respectively: "to compute highly accurate consensus sequences with an approximate error reduction of ~77% and ~88% for ONT and PB, respectively"
- [methods] minimap2 accuracy evidence: "minimap2 [19], which compared favorably to alternative aligners on syntethic data, in terms of both speed and mapping accuracy"
- [intro] POA consensus workflow step: "TRiCoLOR exploits partial order alignment (POA) [14] to compute haplotype-specific low-error consensus sequences"
- [readme] Docker availability from README: "TRiCoLOR is available to be installed from source or as a [Docker container](https://hub.docker.com/r/davidebolo1993/tricolor)."
