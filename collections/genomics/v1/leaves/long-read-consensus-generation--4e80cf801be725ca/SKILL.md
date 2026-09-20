---
name: long-read-consensus-generation
description: Use when error-prone ONT/PacBio reads spanning candidate repetitive regions
  must be collapsed into haplotype-specific low-error consensus sequences, e.g. as
  input to TR profiling or genotyping from haplotype-resolved BAMs.
when_to_use_negative:
- The target region lies in an unassembled part of the genome (e.g., human centromeres
  or telomeres), where TRiCoLOR cannot identify repeats
- Input alignments are not haplotype-resolved, so haplotype-specific consensus sequences
  cannot be separated
- No reads entirely span the candidate region, leaving an insufficient read set for
  partial order alignment
edam_operation: http://edamontology.org/operation_0292
edam_topics:
- http://edamontology.org/topic_0622
tools:
- name: TRiCoLOR REFER
  role: Module that fetches region-spanning reads from haplotype-resolved BAMs, trims
    them to the region, and orchestrates consensus generation
  repo: https://github.com/davidebolo1993/TRiCoLOR/tree/master/TRiCoLOR
- name: SPOA
  role: SIMD-accelerated partial order alignment framework run on trimmed haplotype-specific
    read sets to compute the consensus sequence
- name: TRiCoLOR
  role: Overall tandem repeat profiling tool whose REFER stage performs this consensus
    generation on error-prone long reads
  repo: https://github.com/davidebolo1993/TRiCoLOR
provenance:
  source_task_ids:
  - task_002
  - task_007
  source_papers:
  - doi: 10.1093/gigascience/giaa101
    title: 'TRiCoLOR: tandem repeat profiling using whole-genome long-read sequencing
      data'
  source_content_hash: sha256:029ae3cfa177904132c4615ec01f569b06f27c2d275c60de4d87365e3ac927a8
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - SPOA
  - TRiCoLOR
  - TRiCoLOR REFER
  - TRiCoLOR SAGE
  - minimap2
  edam_operation: http://edamontology.org/operation_0292
  edam_topics:
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 12
---

# long-read-consensus-generation

## Summary

Generates haplotype-specific, low-error consensus sequences from error-prone third-generation long reads (ONT/PacBio) by trimming reads that span a candidate region and applying SIMD-accelerated partial order alignment (POA) via SPOA, as done in TRiCoLOR's REFER module.

## When to use

Use when error-prone ONT/PacBio reads spanning candidate repetitive regions must be collapsed into haplotype-specific low-error consensus sequences, e.g. as input to TR profiling or genotyping from haplotype-resolved BAMs.

## When NOT to use

- The target region lies in an unassembled part of the genome (e.g., human centromeres or telomeres), where TRiCoLOR cannot identify repeats
- Input alignments are not haplotype-resolved, so haplotype-specific consensus sequences cannot be separated
- No reads entirely span the candidate region, leaving an insufficient read set for partial order alignment

## Inputs

- Haplotype-resolved long-read BAM alignment files (ONT or PacBio)
- BED file of candidate repetitive regions R = [S, E] (e.g., produced by TRiCoLOR SENSoR/REFER)

## Outputs

- consensus_sequences.fasta containing one haplotype-specific low-error consensus sequence per candidate region per haplotype

## How to apply

For each candidate region R = [S, E] listed in the input BED file, fetch from the haplotype-specific BAM alignments only those sequencing reads that entirely span the region. Trim each fetched read to retain just the sequence between coordinates S and E, which reduces the runtime of the downstream POA step. Run SPOA, the SIMD-accelerated partial order alignment framework, on the trimmed haplotype-specific read sets to compute a highly accurate consensus per region and haplotype — yielding an approximate error reduction of ~77% for ONT and ~88% for PB reads. Write one consensus per candidate region per haplotype to a FASTA file (consensus_sequences.fasta) for downstream motif resolution or genotyping.

## Related tools

- **TRiCoLOR REFER** (Module that fetches region-spanning reads from haplotype-resolved BAMs, trims them to the region, and orchestrates consensus generation) — https://github.com/davidebolo1993/TRiCoLOR/tree/master/TRiCoLOR
- **SPOA** (SIMD-accelerated partial order alignment framework run on trimmed haplotype-specific read sets to compute the consensus sequence)
- **TRiCoLOR** (Overall tandem repeat profiling tool whose REFER stage performs this consensus generation on error-prone long reads) — https://github.com/davidebolo1993/TRiCoLOR

## Examples

```
docker pull davidebolo1993/tricolor
```

## Evaluation signals

- Consensus error reduction is consistent with the reported ~77% (ONT) and ~88% (PB) improvement over raw error-prone long reads
- Output FASTA contains exactly one consensus per candidate region per haplotype
- Only reads that entirely span the region [S, E] contribute to each consensus, with reads trimmed to the S–E interval before POA
- Consensus sequences align cleanly to GRCh38 (e.g., with minimap2) for downstream genotyping as reference-like or child-TR-like by edit distance

## Limitations

- Cannot generate consensus for repeats in unassembled regions of the genome such as human centromeres and telomeres
- Consensus accuracy depends on sequencing error profile; reported error reduction differs substantially between ONT (~77%) and PB (~88%) data
- Requires haplotype-resolved BAM alignments as input; TRiCoLOR does not yet support polyploid species or haploid chromosomes such as the human Y chromosome

## Evidence

- [intro] TRiCoLOR exploits partial order alignment (POA) [14] to compute haplotype-specific low-error consensus sequences: "TRiCoLOR exploits partial order alignment (POA) [14] to compute haplotype-specific low-error consensus sequences"
- [methods] SPOA error reduction for ONT and PB: "to compute highly accurate consensus sequences with an approximate error reduction of ~77% and ~88% for ONT and PB, respectively"
- [readme] TRiCoLOR availability via Docker: "TRiCoLOR is available to be installed from source or as a [Docker container](https://hub.docker.com/r/davidebolo1993/tricolor)."
