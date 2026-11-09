---
name: sv-call-accuracy-evaluation-against-truth-set
description: Use when you need to quantify how accurately different long-read mappers
  detect implanted or known inversions by calling SVs from each mapper's alignments
  and comparing them to a truth VCF.
when_to_use_negative:
- No truth set exists (e.g. purely exploratory real-data SV discovery, as in the NA12878
  candidate-variant-calling analysis rather than accuracy benchmarking)
- The question is short-read variant calling or SNV/indel accuracy — this skill is
  specific to long-read inversion structural variant detection
- Evaluating a single mapper with no baselines — the skill's value is the cross-mapper
  accuracy comparison on identical inputs
edam_operation: http://edamontology.org/operation_3227
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3175
tools:
- name: invMap
  role: Long-read mapper under evaluation; specialized to detect inversions in noisy
    reads via chaining score plus anchor re-checking
  repo: https://github.com/zhang134/invMap.git
- name: SURVIVOR
  role: Implants inversions into the reference genome to create the altered reference
    and truth VCF
- name: minimap2
  role: Baseline long-read mapper compared against invMap
- name: NGMLR
  role: Baseline long-read mapper compared against invMap
- name: Winnowmap2
  role: Baseline long-read mapper compared against invMap
- name: lra
  role: Baseline long-read mapper compared against invMap
- name: SVision
  role: Calls structural variants from each mapper's BAM alignments for truth-set
    comparison
- name: samtools
  role: Converts and sorts each mapper's SAM output to BAM
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/bioinformatics/btad726
    title: 'invMap: a sensitive mapping tool for long noisy reads with inversion structural
      variants'
  source_content_hash: sha256:3a888897108eb77d82d014cfde95bdec40a796d9b080f5a7855f86558e0ae462
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/zhang134/invMap.git
  tools:
  - NGMLR
  - SURVIVOR
  - SVision
  - Winnowmap2
  - invMap
  - lra
  - minimap2
  - samtools
  edam_operation: http://edamontology.org/operation_3227
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3175
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# SV call accuracy evaluation against truth set

## Summary

Benchmark long-read mapper accuracy for inversion detection by comparing mapper-specific SV calls against a SURVIVOR-implanted inversion truth set, as done for invMap vs. minimap2, NGMLR, Winnowmap2, and lra on the GIAB HG002 PacBio real-read benchmark.

## When to use

Use when you need to quantify how accurately different long-read mappers detect implanted or known inversions by calling SVs from each mapper's alignments and comparing them to a truth VCF.

## When NOT to use

- No truth set exists (e.g. purely exploratory real-data SV discovery, as in the NA12878 candidate-variant-calling analysis rather than accuracy benchmarking)
- The question is short-read variant calling or SNV/indel accuracy — this skill is specific to long-read inversion structural variant detection
- Evaluating a single mapper with no baselines — the skill's value is the cross-mapper accuracy comparison on identical inputs

## Inputs

- Reference genome in FASTA format (e.g. GRCh37)
- SURVIVOR-implanted inversion truth VCF (e.g. 1000 inversions)
- Real long-read sequencing dataset in FASTA/FASTQ format (e.g. GIAB HG002 PacBio reads)
- Per-mapper SAM alignments (invMap, minimap2, NGMLR, Winnowmap2, lra)

## Outputs

- Sorted BAM files per mapper
- SV calls per mapper from SVision
- Per-mapper inversion detection accuracy table (mapper_accuracy_benchmark.csv)

## How to apply

Implant a known number of inversions (e.g. 1000) into a reference genome using SURVIVOR, generating an altered reference plus a truth VCF. Build indexes for the altered reference with each mapper under comparison, then map the same long-read dataset (e.g. HG002 GIAB PacBio reads) with each mapper, producing one SAM per mapper (e.g. './invMap genome.fa reads.fa > invMap_aligned.sam'). Convert and sort each SAM to BAM with samtools, call structural variants from each alignment set with SVision, then compare each mapper's SV calls against the truth set to compute per-mapper accuracy. In the reference benchmark, invMap reached 76.55%, ahead of minimap2 (75.15%), NGMLR (73.4%), Winnowmap2 (67.10%), and lra (63.25%).

## Related tools

- **invMap** (Long-read mapper under evaluation; specialized to detect inversions in noisy reads via chaining score plus anchor re-checking) — https://github.com/zhang134/invMap.git
- **SURVIVOR** (Implants inversions into the reference genome to create the altered reference and truth VCF)
- **minimap2** (Baseline long-read mapper compared against invMap)
- **NGMLR** (Baseline long-read mapper compared against invMap)
- **Winnowmap2** (Baseline long-read mapper compared against invMap)
- **lra** (Baseline long-read mapper compared against invMap)
- **SVision** (Calls structural variants from each mapper's BAM alignments for truth-set comparison)
- **samtools** (Converts and sorts each mapper's SAM output to BAM)

## Examples

```
./invMap genome.fa reads.fa > invMap_aligned.sam; samtools view -b invMap_aligned.sam | samtools sort -o invMap_aligned.bam
```

## Evaluation signals

- Each mapper produced a valid SAM that converts and sorts to BAM without errors (samtools exit code 0)
- SVision returns a non-empty SV call set for every mapper's BAM on the altered reference
- Per-mapper accuracy is computed against exactly the 1000-implanted-inversion truth set, not against an independent discovery callset
- Result table reproduces the expected ordering: invMap (76.55%) > minimap2 (75.15%) > NGMLR (73.4%) > Winnowmap2 (67.10%) > lra (63.25%) on the HG002 PacBio benchmark
- All mappers were run on the identical altered reference and identical read set, so accuracy differences are attributable to the mapper

## Limitations

- Accuracy numbers depend on the specific implanted-inversion truth set (1000 SURVIVOR inversions) and the SV caller (SVision); different truth sets or callers may shift results
- invMap requires a minimum of 16G of RAM for mapping to large reference genomes and approximately 800 M of disk space
- The benchmark pipeline is Linux-oriented; the README lists Windows 7/10 support but Linux is recommended, and compilation via 'make' is required before use
- On real data without a truth set, invMap is evaluated by number of candidate inversion variant calls rather than accuracy, so this accuracy metric does not transfer

## Evidence

- [other] invMap achieves 76.55% accuracy, higher than minimap2 (75.15%), NGMLR (73.4%), and Winnowmap2 (67.10%): [evidence span withheld — anchor status: not_found]
- [other] SURVIVOR implant workflow: [evidence span withheld — anchor status: not_found]
- [readme] invMap SAM output example: "./invMap genome.fa reads.fa > invMap_aligned.sam"
- [abstract] invMap algorithm description: "invMap first locates the aligned region with a specifically designed scoring method for chaining, then checks the remaining anchors in the aligned region to discover potential inversions"
- [intro] Memory requirement: "invMap requires a minimum of 16G of RAM for mapping sequence datasets to large reference genomes."
