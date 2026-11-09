---
name: sam-alignment-parsing-and-location-accuracy-computation
description: Use when benchmarking long-read mappers on simulated reads (PacBio CCS/CLR
  or ONT) where ground-truth origin coordinates exist and per-mapper location accuracy
  must be quantified from SAM output.
when_to_use_negative:
- Evaluating real sequencing data (e.g. NA12878) where no ground-truth origin region
  exists — variant-calling sensitivity, not location accuracy, is the appropriate
  metric there
- Assessing SV/inversion calling performance rather than mapping-position accuracy
  — this skill measures only whether the reported aligned region matches the true
  origin
edam_operation: http://edamontology.org/operation_0292
edam_topics:
- http://edamontology.org/topic_0102
- http://edamontology.org/topic_0622
tools:
- name: invMap
  role: Long-read mapper whose SAM output is parsed for location-accuracy computation
  repo: https://github.com/zhang134/invMap.git
- name: PBSIM
  role: Simulates PacBio CCS and CLR reads with known true origin regions
- name: PBSIM2
  role: Simulates ONT reads with known true origin regions
- name: minimap2
  role: Competing long-read mapper run with long-read presets; its SAM output is scored
    for comparison
- name: Winnowmap2
  role: Competing long-read mapper run with long-read presets; its SAM output is scored
    for comparison
- name: lra
  role: Competing long-read mapper run with long-read presets; its SAM output is scored
    for comparison
- name: NGMLR
  role: Competing long-read mapper run with long-read presets; its SAM output is scored
    for comparison
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/bioinformatics/btad726
    title: 'invMap: a sensitive mapping tool for long noisy reads with inversion structural
      variants'
  source_content_hash: sha256:3c083b27b9ac33a9c862d21d1a733902a07e466b5bfb60c93cc6dd5fc5c4e7e3
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/zhang134/invMap.git
  tools:
  - NGMLR
  - PBSIM
  - PBSIM2
  - Winnowmap2
  - invMap
  - lra
  - minimap2
  edam_operation: http://edamontology.org/operation_0292
  edam_topics:
  - http://edamontology.org/topic_0102
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 10
---

# sam-alignment-parsing-and-location-accuracy-computation

## Summary

Parse SAM alignments produced by long-read mappers (invMap, minimap2, Winnowmap2, lra, NGMLR) and compute location accuracy — the fraction of simulated reads whose reported aligned region matches the true simulated origin region — to benchmark mapping position accuracy.

## When to use

Use when benchmarking long-read mappers on simulated reads (PacBio CCS/CLR or ONT) where ground-truth origin coordinates exist and per-mapper location accuracy must be quantified from SAM output.

## When NOT to use

- Evaluating real sequencing data (e.g. NA12878) where no ground-truth origin region exists — variant-calling sensitivity, not location accuracy, is the appropriate metric there
- Assessing SV/inversion calling performance rather than mapping-position accuracy — this skill measures only whether the reported aligned region matches the true origin

## Inputs

- Reference genome FASTA files (e.g. Neisseria meningitidis, Abortiporus biennis, GRCh37 chromosome 1)
- Simulated PacBio CCS/CLR reads (PBSIM) and ONT reads (PBSIM2) with known true origin regions
- SAM alignment files from invMap, minimap2, Winnowmap2, lra, and NGMLR

## Outputs

- Per-mapper, per-genome, per-platform location-accuracy table (location_accuracy_s4.csv)
- Fraction of reads whose aligned region matches the true simulated origin region, per mapper

## How to apply

Simulate PacBio CCS and CLR reads with PBSIM and ONT reads with PBSIM2 from each reference genome (e.g. Neisseria meningitidis, Abortiporus biennis, GRCh37 chromosome 1) at the paper's coverages, keeping the true origin region of every read. Map each read set back to its reference with invMap via './invMap genome.fa reads.fa > invMap_aligned.sam' and with minimap2, Winnowmap2, lra, and NGMLR using their long-read presets. Parse each mapper's SAM file and compute location accuracy as the fraction of reads whose reported aligned region matches the true simulated origin region. Aggregate per-mapper, per-genome, per-platform accuracy values into location_accuracy_s4.csv for comparison against Supplementary Table S4 / Fig. 4.

## Related tools

- **invMap** (Long-read mapper whose SAM output is parsed for location-accuracy computation) — https://github.com/zhang134/invMap.git
- **PBSIM** (Simulates PacBio CCS and CLR reads with known true origin regions)
- **PBSIM2** (Simulates ONT reads with known true origin regions)
- **minimap2** (Competing long-read mapper run with long-read presets; its SAM output is scored for comparison)
- **Winnowmap2** (Competing long-read mapper run with long-read presets; its SAM output is scored for comparison)
- **lra** (Competing long-read mapper run with long-read presets; its SAM output is scored for comparison)
- **NGMLR** (Competing long-read mapper run with long-read presets; its SAM output is scored for comparison)

## Examples

```
./invMap genome.fa reads.fa > invMap_aligned.sam
```

## Evaluation signals

- invMap attains >99% location accuracy on PacBio CCS and CLR read sets
- invMap attains >98% location accuracy on ONT read sets
- invMap location accuracy exceeds that of minimap2, Winnowmap2, lra, and NGMLR on the same simulated read sets
- Reproduced per-mapper, per-genome, per-platform values match Supplementary Table S4 / Fig. 4 in location_accuracy_s4.csv
- Every SAM file parses without error and each read is assigned a binary match/no-match against its known simulated origin region

## Limitations

- invMap requires a minimum of 16G of RAM for mapping sequence datasets to large reference genomes
- Location accuracy is only defined for simulated reads with known ground-truth origin regions; it cannot be computed on real datasets
- Accuracy depends on the genome size and coverage settings used in the simulation, as reflected in the per-genome aggregation

## Evidence

- [supplementary] Map each simulated read set back to its reference with invMap using './invMap genome.fa reads.fa > invMap_aligned.sam': [evidence span withheld — anchor status: not_found]
- [supplementary] compute location accuracy: [evidence span withheld — anchor status: not_found]
- [supplementary] validation thresholds: [evidence span withheld — anchor status: not_found]
- [results] invMap accuracy finding: [evidence span withheld — anchor status: not_found]
- [intro] invMap RAM requirement: "invMap requires a minimum of 16G of RAM for mapping sequence datasets to large reference genomes."
