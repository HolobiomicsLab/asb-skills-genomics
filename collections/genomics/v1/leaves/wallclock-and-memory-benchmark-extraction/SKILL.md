---
name: wallclock-and-memory-benchmark-extraction
description: Use when benchmarking a streaming alignment-to-HLA-prediction pipeline
  (minimap2 piped into HLAminer.pl -a stream) and you need wall-clock elapsed time
  and maximum resident set size to compare against the protocol's reported performance
  envelope.
when_to_use_negative:
- You need per-step profiling attribution — GNU time -v measures the whole piped command,
  though the article reports minimap2 dominates runtime and RAM anyway
- Your dataset is not a WGS long-read shotgun input (e.g. it is the MCF-7 PacBio RNA-seq
  demo) — runtime scales with read count, so the ~40 min / 35 GB envelope does not
  transfer
- You intend to save SAM files to disk rather than stream — the benchmark validates
  the streaming mode specifically and disk I/O would change the timing profile
edam_operation: http://edamontology.org/operation_0335
edam_topics:
- http://edamontology.org/topic_0102
- http://edamontology.org/topic_3170
tools:
- name: minimap2
  role: aligns ONT reads (-ax map-ont -t 60 --MD) to the GRCh38-noChr6 reference and
    dominates the pipeline's run time and RAM being measured
  repo: https://github.com/lh3/minimap2
- name: HLAminer v1.4
  role: receives streamed SAM alignments via HLAminer.pl -a stream and produces HLA
    class I/II predictions downstream of the benchmarked command
  repo: https://github.com/bcgsc/HLAminer
- name: GNU time
  role: wraps the streaming command with 'time -v' to capture elapsed wall-clock time
    and maximum resident set size into a .time report file
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1002/cpz1.70124
    title: Streaming Long‐Read Sequence Alignments for HLA Predictions Using HLAminer
  source_content_hash: sha256:3ae7af294c8f5ffcc86a52479ff8ded855c86f197087e8b8108905c5fdd9e8df
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/lh3/minimap2
  tools:
  - GNU time
  - HLAminer v1.4
  - minimap2
  edam_operation: http://edamontology.org/operation_0335
  edam_topics:
  - http://edamontology.org/topic_0102
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6363636363636364
n_claims: 11
---

# wallclock-and-memory-benchmark-extraction

## Summary

Profiles the streamed minimap2-to-HLAminer v1.4 workflow's wall-clock runtime and peak RAM using GNU time, to verify the protocol's reported benchmark (~40 min / up to 35 GB for a ~10x ONT WGS dataset) and confirm that minimap2 dominates resource use.

## When to use

Use when benchmarking a streaming alignment-to-HLA-prediction pipeline (minimap2 piped into HLAminer.pl -a stream) and you need wall-clock elapsed time and maximum resident set size to compare against the protocol's reported performance envelope.

## When NOT to use

- You need per-step profiling attribution — GNU time -v measures the whole piped command, though the article reports minimap2 dominates runtime and RAM anyway
- Your dataset is not a WGS long-read shotgun input (e.g. it is the MCF-7 PacBio RNA-seq demo) — runtime scales with read count, so the ~40 min / 35 GB envelope does not transfer
- You intend to save SAM files to disk rather than stream — the benchmark validates the streaming mode specifically and disk I/O would change the timing profile

## Inputs

- ONT PromethION WGS FASTQ file (e.g. ERR2585115.fastq.gz, NA19240, ENA PRJEB26791)
- GRCh38-noChr6-HLA-I_II_GEN reference FASTA (GCA_000001405.15, chromosome-only, no chr6)
- HLAminer v1.4 installation with HLA-I_II_GEN.fasta and hla_nom_p.txt databases

## Outputs

- GNU time -v report file (e.g. minimap_hlaminerERR2585115-1mod.time) containing 'Elapsed (wall clock) time' and 'Maximum resident set size'
- Benchmark summary: wall-clock runtime (~40 min expected) and peak RAM (within up-to-35-GB envelope)

## How to apply

Obtain the ONT PromethION WGS FASTQ (e.g. ERR2585115.fastq.gz from ENA project PRJEB26791, NA19240) and the GRCh38-noChr6-HLA-I_II_GEN reference FASTA. Wrap the single streaming command — minimap2 -t 60 -ax map-ont --MD piped directly into ./HLAminer.pl -h ../database/HLA-I_II_GEN.fasta -s 500 -q 1 -i 1 -p ../database/hla_nom_p.txt -a stream — in '/usr/bin/time -v -o <name>.time', so SAM alignments never touch disk and resource accounting covers the whole pipeline. Parse the resulting .time file for 'Elapsed (wall clock) time' and 'Maximum resident set size'. Validate by checking that runtime is approximately 40 minutes for the ~10x coverage dataset, peak RAM stays within the up-to-35-GB envelope, and minimap2 dominates both runtime and RAM as the protocol claims runtime scales with read count.

## Related tools

- **minimap2** (aligns ONT reads (-ax map-ont -t 60 --MD) to the GRCh38-noChr6 reference and dominates the pipeline's run time and RAM being measured) — https://github.com/lh3/minimap2
- **HLAminer v1.4** (receives streamed SAM alignments via HLAminer.pl -a stream and produces HLA class I/II predictions downstream of the benchmarked command) — https://github.com/bcgsc/HLAminer
- **GNU time** (wraps the streaming command with 'time -v' to capture elapsed wall-clock time and maximum resident set size into a .time report file)

## Examples

```
/usr/bin/time -v -o minimap_hlaminerERR2585115-1mod.time minimap2 -t 60 -ax map-ont --MD GRCh38-noChr6-HLA-I_II_GEN.fa ERR2585115.fastq.gz | ./HLAminer.pl -h ../database/HLA-I_II_GEN.fasta -s 500 -q 1 -i 1 -p ../database/hla_nom_p.txt -a stream
```

## Evaluation signals

- Wall-clock 'Elapsed (wall clock) time' extracted from the .time file is approximately 40 minutes for the ~10x coverage ERR2585115 dataset, matching the protocol's Table 2 benchmark
- 'Maximum resident set size' stays within the reported up-to-35-GB envelope
- No intermediate SAM file is written to disk — the pipeline runs as a single streamed command, consistent with the protocol's design
- Runtime scales with read count and minimap2 accounts for the dominant share of run time and RAM, as the protocol claims
- HLAminer still emits HLA class I and II predictions (score threshold -s 500, expect -q 1, identity -i 1 in the benchmark invocation), confirming the profiled run completed

## Limitations

- The ~40 min / up-to-35-GB benchmark was measured on a ~10x coverage older-chemistry (2018) PromethION dataset with 60 threads; other coverages, chemistries, or thread counts will scale differently
- GNU time -v profiles the combined piped command, so per-component attribution relies on the protocol's observation that minimap2 dominates, not on direct measurement
- Older, less accurate nanopore chemistry datasets are used in the benchmark, so absolute timings may differ from modern high-accuracy runs

## Evidence

- [abstract] This method is as simple as running minimap2, scales efficiently with the number of sequences: "This method is as simple as running minimap2, scales efficiently with the number of sequences"
- [abstract] works with any read aligner compatible with the SAM file format—eliminating the need to store bulky alignment files on disk: "works with any read aligner compatible with the SAM file format—eliminating the need to store bulky alignment files on disk"
- [methods] Usage: ../bin/HLAminer.pl [v1.4]: "Usage: ../bin/HLAminer.pl [v1.4]"
- [methods] We tested the software on the NA19240 WGS promethion dataset (2018, older chemistry): "We tested the software on the NA19240 WGS promethion dataset (2018, older chemistry)."
- [abstract] demonstrate the robustness of predictions even with older, less accurate WGS nanopore datasets and relatively low (10×) sequencing coverage: "demonstrate the robustness of predictions even with older, less accurate WGS nanopore datasets and relatively low (10×) sequencing coverage"
- [readme] Ability to stream the (.sam) output of modern read aligners, directly into HLAminer.: "Ability to stream the (.sam) output of modern read  aligners, directly into HLAminer."
