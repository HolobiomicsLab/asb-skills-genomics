---
name: low-coverage-hifi-read-downsampling
description: 'Use when a benchmark requires standardized low-coverage HiFi inputs:
  full-coverage human or plant HiFi FASTQ/FASTA reads must be reduced to ~5x before
  assembly comparison.'
when_to_use_negative:
- The input reads are already at ~5x or lower coverage — further downsampling would
  deepen the completeness gaps the paper itself notes ('many genomic regions remain
  missing' at 5x)
- The data are noisy long reads (ONT, PacBio CLR) rather than high-fidelity reads
  — HiFiCCL and this evaluation design target specifically low-coverage HiFi data
- The goal is assembly at native/high coverage for maximal contiguity rather than
  a controlled low-coverage benchmark comparison
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_0102
tools:
- name: seqtk
  role: Subsamples the HiFi FASTQ/FASTA read sets to ~5x coverage (version 1.3-r106);
    handles gzip-compressed FASTA/FASTQ natively
  repo: https://github.com/lh3/seqtk
- name: HiFiCCL
  role: Downstream framework that consumes the downsampled ~5x reads for reference-guided,
    chromosome-by-chromosome assembly (hificcl.py -f <reads> -r CHM13v2.0.fasta -t
    20)
  repo: https://github.com/zjjbuqi/HiFiCCL
- name: hifiasm
  role: Baseline assembler run on the identical downsampled reads (0.19.5-r592, -t20
    --primary) for the comparison
  repo: https://github.com/chhylp123/hifiasm
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1002/advs.202515308
    title: Reference‐Guided Chromosome‐by‐Chromosome <i>de novo</i> Assembly at Scale
      Using Low‐Coverage High‐Fidelity Long‐Reads with HiFiCCL
  source_content_hash: sha256:5431913b13042b0f0cff862b1c4bcd7f7552665f2cf019dfc2781de79f580b4a
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/lh3/seqtk
  tools:
  - HiFiCCL
  - Hifiasm
  - hifiasm
  - minimap2
  - seqtk
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_0102
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7692307692307693
n_claims: 13
---

# low-coverage-hifi-read-downsampling

## Summary

Subsample full-coverage PacBio HiFi read sets to ~5x coverage with seqtk (1.3-r106) to create standardized low-coverage inputs for benchmarking HiFiCCL-Hifiasm against hifiasm alone. This controls read depth so assembly-quality differences reflect the framework, not coverage variation.

## When to use

Use when a benchmark requires standardized low-coverage HiFi inputs: full-coverage human or plant HiFi FASTQ/FASTA reads must be reduced to ~5x before assembly comparison.

## When NOT to use

- The input reads are already at ~5x or lower coverage — further downsampling would deepen the completeness gaps the paper itself notes ('many genomic regions remain missing' at 5x)
- The data are noisy long reads (ONT, PacBio CLR) rather than high-fidelity reads — HiFiCCL and this evaluation design target specifically low-coverage HiFi data
- The goal is assembly at native/high coverage for maximal contiguity rather than a controlled low-coverage benchmark comparison

## Inputs

- Full-coverage PacBio HiFi read sets in FASTQ or FASTA (plain or gzip-compressed), e.g., SRA accessions SRR10382244 (HG002) and SRR14611231 (NA19240)
- Target coverage level (~5x) and reference genome size (e.g., human ~3.1 Gbp) for computing the retained read fraction

## Outputs

- Downsampled ~5x-coverage HiFi read sets in FASTQ/FASTA, one per input dataset, used as input to HiFiCCL and hifiasm
- Coverage-normalized benchmark entries (MCL, NG50, NGA50, BUSCO, Merqury, runtime, max RSS) written to assembly_evaluation_metrics.csv

## How to apply

Downsample each full-coverage PacBio HiFi FASTQ/FASTA read set to ~5x coverage using seqtk 1.3-r106, retaining roughly 5x genome-equivalent bases (e.g., ~15 Gbp for a 3.1 Gbp human genome); seqtk parses FASTA and FASTQ directly, including gzip-compressed files. Apply the identical downsampling procedure to every dataset (45 ~5x human datasets in the paper, plus rice SRR11606867 and Arabidopsis thaliana CRR573321 accessions such as HG002/SRR10382244 and NA19240/SRR14611231) so HiFiCCL-Hifiasm and hifiasm-alone baselines are compared on identical read inputs. Feed the downsampled reads to HiFiCCL in default mode (hificcl.py -f <reads> -r CHM13v2.0.fasta -t 20, which invokes hifiasm --primary) and to hifiasm 0.19.5-r592 (-t20 --primary) as the baseline. Evaluate both assemblies with QUAST v5.2.0, BUSCO 5.4.7 (vertebrate conserved single-copy ortholog database), and Merqury, recording wall time and maximum resident set size for both pipelines. The rationale is that holding coverage constant at ~5x isolates the misassembled-contig-length (MCL) reduction attributable to HiFiCCL (average 21.19%, up to 38.58%, across the 45 human datasets).

## Related tools

- **seqtk** (Subsamples the HiFi FASTQ/FASTA read sets to ~5x coverage (version 1.3-r106); handles gzip-compressed FASTA/FASTQ natively) — https://github.com/lh3/seqtk
- **HiFiCCL** (Downstream framework that consumes the downsampled ~5x reads for reference-guided, chromosome-by-chromosome assembly (hificcl.py -f <reads> -r CHM13v2.0.fasta -t 20)) — https://github.com/zjjbuqi/HiFiCCL
- **hifiasm** (Baseline assembler run on the identical downsampled reads (0.19.5-r592, -t20 --primary) for the comparison) — https://github.com/chhylp123/hifiasm

## Examples

```
seqtk sample -s100 HG002_SRR10382244.fq.gz 0.1667 > HG002_5x.fq  # ~5x from ~30x HiFi with seqtk 1.3-r106
```

## Evaluation signals

- Total bases in the downsampled set correspond to ~5x coverage of the target genome (e.g., ~15 Gbp for a human-sized genome)
- The same seqtk (1.3-r106) downsampling procedure is applied uniformly to all datasets, so both pipelines receive byte-identical read inputs
- The hifiasm-alone baseline on the downsampled reads reproduces the published MCL values within tolerance (HG002: 331.84 Mb; NA19240: 245.96 Mb)
- The HiFiCCL-Hifiasm run on the same downsampled input yields the expected MCL reduction (17.38% for HG002, 27.16% for NA19240; average 21.19%, up to 38.58% across 45 datasets)
- All metrics (MCL, NG50, NGA50, BUSCO, Merqury, wall time, max RSS) are recorded for both pipelines in assembly_evaluation_metrics.csv

## Limitations

- At ~5x coverage many genomic regions remain missing, so assembly completeness gaps are unresolved regardless of downsampling fidelity
- Downsampling discards reads permanently; the retained fraction depends on the assumed genome size, and the article describes no per-dataset post-hoc coverage verification
- The resulting low-coverage benchmark was validated on 45 human datasets but only two plant samples, so coverage-normalized conclusions may not extend to more diverse genomes

## Evidence

- [abstract] Tested on 45 human datasets (∼5× coverage), HiFiCCL combined with hifiasm reduces the length of misassembled contigs: "Tested on 45 human datasets (∼5× coverage), HiFiCCL combined with hifiasm reduces the length of misassembled contigs"
- [methods] seqtk tool (1.3‐r106): "seqtk tool (1.3‐r106) (https://github.com/lh3/seqtk)"
- [intro] whole‐genome reads are partitioned by chromosome using the high‐quality reference genome and then assembled individually: "whole‐genome reads are partitioned by chromosome using the high‐quality reference genome and then assembled individually"
- [readme] Seqtk is a fast and lightweight tool for processing sequences in the FASTA or FASTQ format. It seamlessly parses both FASTA and FASTQ files which can also be optionally compressed by gzip.: [evidence span withheld — anchor status: external_doc]
