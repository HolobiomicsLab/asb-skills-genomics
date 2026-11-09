---
name: ultra-long-read-dataset-resource-management
description: Use when a long-read correction run targets a genome-scale ONT dataset
  (e.g., 113 Gbp human data, ultra-long reads up to 1.5 Mbp) and runtime, RAM, core
  count, and disk for overlaps must be planned before launching.
when_to_use_negative:
- The dataset is small (e.g., a few hundred Mbp of simulated or bacterial reads) where
  default tmpdir on the working directory and modest RAM suffice, as in the README's
  10x example dataset.
- The goal is assembly polishing rather than read self-correction, which uses CONSENT-polish
  and does not require the ~5 TB overlaps staging described here.
- Corrections require reference-based alignment of short reads (e.g., CoLoRMAP, HECiL
  style), not self-correction.
edam_operation: http://edamontology.org/operation_3184
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3673
tools:
- name: CONSENT
  role: Self-correction tool whose runtime, --nproc, and --tmpdir settings govern
    the resource plan; processes ultra-long ONT reads via overlaps, windowed MSAs,
    and local de Bruijn graph polishing
  repo: https://github.com/morispi/CONSENT
- name: minimap2
  role: Internal dependency used by CONSENT to compute overlaps between long reads,
    producing the large temporary overlaps file that drives the disk requirement
- name: ONT
  role: Sequencing technology of the input ultra-long reads (up to 1.5 Mbp) that dictates
    the --type ONT setting and the extreme resource profile
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41598-020-80757-5
    title: Scalable long read self-correction and assembly polishing with multiple
      sequence alignment
  source_content_hash: sha256:123e90c2cf4cfb2b4af6602e9b17e4f980c10cbd69978755a61718d050ae9bca
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/morispi/CONSENT
  tools:
  - CONSENT
  - ONT
  - minimap2
  edam_operation: http://edamontology.org/operation_3184
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3673
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4166666666666667
n_claims: 12
---

# ultra-long-read-dataset-resource-management

## Summary

Provisioning and executing long-read self-correction (e.g., CONSENT) on ultra-large ONT datasets with reads up to 1.5 Mbp by pre-planning multi-day runtimes, ~100 GB RAM, multi-core parallelism, and ~5 TB of disk for the temporary overlaps file.

## When to use

Use when a long-read correction run targets a genome-scale ONT dataset (e.g., 113 Gbp human data, ultra-long reads up to 1.5 Mbp) and runtime, RAM, core count, and disk for overlaps must be planned before launching.

## When NOT to use

- The dataset is small (e.g., a few hundred Mbp of simulated or bacterial reads) where default tmpdir on the working directory and modest RAM suffice, as in the README's 10x example dataset.
- The goal is assembly polishing rather than read self-correction, which uses CONSENT-polish and does not require the ~5 TB overlaps staging described here.
- Corrections require reference-based alignment of short reads (e.g., CoLoRMAP, HECiL style), not self-correction.

## Inputs

- FASTA or FASTQ file of long ONT reads (e.g., 113 Gbp human dataset, reads up to 1.5 Mbp, PRJEB23027)
- External storage volume with ~5 TB free space for the overlaps file
- Multi-core machine with up to ~100 GB RAM and ~10 days of compute time

## Outputs

- Corrected reads in FASTA format (e.g., correctedReads.fasta)
- Temporary overlaps file (Alignments_dateTimeStamp.paf) in the --tmpdir location
- Runtime, peak RAM, and disk usage measurements for comparison against published benchmarks

## How to apply

Estimate resources from dataset size: CONSENT processed the full 113 Gbp ONT human dataset (PRJEB23027) in about 10 days using less than 100 GB of RAM with multi-core execution. Before launching ./CONSENT-correct with --type ONT, set --tmpdir to external storage because the overlaps file (Alignments_dateTimeStamp.paf) for a full human dataset alone requires ~5 TB of disk. Use --nproc to control the number of parallel processes (defaults to the number of cores). Leave windowing/pile parameters at defaults (windowSize 500, minSupport 4, maxSupport 150, anchorSupport 8, minAnchors 2) unless quality/runtime tuning is intended. After the run, verify outputs (corrected reads FASTA, corrected-read N50 ~12,880 bp) and compare measured runtime and peak memory against the paper's Table 5 values (identity ~93.86%, aligned fraction ~98.4%).

## Related tools

- **CONSENT** (Self-correction tool whose runtime, --nproc, and --tmpdir settings govern the resource plan; processes ultra-long ONT reads via overlaps, windowed MSAs, and local de Bruijn graph polishing) — https://github.com/morispi/CONSENT
- **minimap2** (Internal dependency used by CONSENT to compute overlaps between long reads, producing the large temporary overlaps file that drives the disk requirement)
- **ONT** (Sequencing technology of the input ultra-long reads (up to 1.5 Mbp) that dictates the --type ONT setting and the extreme resource profile)

## Examples

```
./CONSENT-correct --in longReads.fasta --out correctedReads.fasta --type ONT --tmpdir /mnt/external --nproc 32
```

## Evaluation signals

- Total runtime is on the order of ~10 days for a 113 Gbp human ONT dataset, matching the paper's Table 5 report
- Peak RAM stays below ~100 GB during the run
- The overlaps file is written to the --tmpdir external storage without exhausting the primary disk (~5 TB expected)
- Corrected reads align to the reference at ~93.86% average identity with ~98.4% of reads aligned
- Corrected-read N50 is ~12,880 bp, consistent with published results

## Limitations

- The ~5 TB disk requirement comes from the overlaps file, so insufficient storage or a small default tmpdir will cause the run to fail on genome-scale datasets.
- CONSENT's correction step runtime tends to be higher than other state-of-the-art methods, even though it uniquely scales to ultra-long reads.
- The strategy of selecting only the N highest identity overlaps may prove inefficient when repeated regions grow longer, and larger or more complex organisms have not yet been corrected.

## Evidence

- [abstract] CONSENT is the only method that efficiently scales to ultra-long reads, and allows to process a full human dataset, containing reads reaching up to 1.5 Mbp, in 10 days.: "CONSENT is the only method that efficiently scales to ultra-long reads, and allows to process a full human dataset, containing reads reaching up to 1.5 Mbp, in 10 days."
- [discussion] The 5 TB disk space requirement comes from the overlaps file, which can easily be stored on a external hard drive.: "The 5 TB disk space requirement comes from the overlaps file, which can easily be stored on a external hard drive."
- [readme] --tmpdir STRING, -t STRING:    Path where to store the temporary overlaps file (default: working directory, as Alignments_dateTimeStamp.paf).: "--tmpdir STRING, -t STRING:    Path where to store the temporary overlaps file (default: working directory, as Alignments_dateTimeStamp.paf)."
- [readme] --nproc INT, -j INT:           Number of processes to run in parallel (default: number of cores).: "--nproc INT, -j INT:           Number of processes to run in parallel (default: number of cores)."
