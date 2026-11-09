---
name: structural-variant-anchor-evidence-assembly
description: Use when you have invMap SAM/inversion-call outputs plus competing mapper
  call sets and need to isolate and tabulate novel inversion calls missed by all baselines,
  e.g. invMap-only calls on NA12878.
when_to_use_negative:
- The goal is general SV discovery from scratch rather than assembling evidence for
  candidate calls already produced by invMap and baselines.
- Input reads are short reads rather than long noisy PacBio or Oxford Nanopore reads
  that can span inversion breakpoints.
- No competing-mapper call sets are available, so invMap-only calls cannot be isolated
  by interval overlap.
edam_operation: http://edamontology.org/operation_3227
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_0199
tools:
- name: invMap
  role: Primary mapper whose chaining-based inversion calls supply the candidate inversions
    and supporting anchors to be assembled into the evidence table
  repo: https://github.com/zhang134/invMap.git
- name: Winnowmap2
  role: Competing long-read mapper used as a baseline call set to determine which
    inversions are invMap-only
- name: minimap2
  role: Competing long-read mapper baseline; fails to discover the novel 231 bp inversion
    that invMap detects
- name: NGMLR
  role: Competing long-read mapper baseline for interval-overlap comparison
- name: lra
  role: Competing long-read mapper baseline for interval-overlap comparison
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1093/bioinformatics/btad726
    title: 'invMap: a sensitive mapping tool for long noisy reads with inversion structural
      variants'
  source_content_hash: sha256:e75f5b70ca07ab81718f64255315a4417efbcc280a26aece49bd97b94ed4886f
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/zhang134/invMap.git
  tools:
  - NGMLR
  - Winnowmap2
  - invMap
  - lra
  - minimap2
  edam_operation: http://edamontology.org/operation_3227
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_0199
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6153846153846154
n_claims: 13
---

# structural-variant-anchor-evidence-assembly

## Summary

Assembles evidence tables for inversion structural-variant calls by comparing invMap calls against competing long-read mappers (Winnowmap2, minimap2, NGMLR, lra) via interval overlap on reference coordinates, isolating invMap-only discoveries such as the novel 231 bp inversion on the NA12878 real human genome dataset.

## When to use

Use when you have invMap SAM/inversion-call outputs plus competing mapper call sets and need to isolate and tabulate novel inversion calls missed by all baselines, e.g. invMap-only calls on NA12878.

## When NOT to use

- The goal is general SV discovery from scratch rather than assembling evidence for candidate calls already produced by invMap and baselines.
- Input reads are short reads rather than long noisy PacBio or Oxford Nanopore reads that can span inversion breakpoints.
- No competing-mapper call sets are available, so invMap-only calls cannot be isolated by interval overlap.

## Inputs

- invMap inversion-call output (invMap_aligned.sam in SAM format) for the NA12878 real human genome sequencing dataset
- Competing long-read mapper call sets (Winnowmap2, minimap2, NGMLR, lra)
- Simulated ground-truth inversion labels and Supplementary Table S5
- invMap chaining output with supporting anchors

## Outputs

- novel_inversions_evidence.csv with columns for identity, inversion length, anchors, and ground-truth status
- InvMap-only call list isolated by reference-coordinate interval overlap
- Mean alignment identity of non-ground-truth invMap-only calls (expected 81.95%)

## How to apply

Load the invMap inversion-call outputs for the NA12878 real human genome sequencing dataset together with the corresponding Winnowmap2, minimap2, NGMLR, and lra call sets. Compare invMap calls against the other mappers via interval overlap on reference coordinates to isolate inversions predicted only by invMap. For each unique invMap call, record alignment identity, inversion length (e.g. the 231 bp novel inversion), and supporting anchors from the invMap chaining output. Cross-reference extracted calls with Supplementary Table S5 and Figures S3-S4 to label ground-truth-matching versus non-ground-truth calls and compute the mean identity of the latter (81.95%). Write the assembled records to a CSV with columns for identity, length, anchors, and truth status.

## Related tools

- **invMap** (Primary mapper whose chaining-based inversion calls supply the candidate inversions and supporting anchors to be assembled into the evidence table) — https://github.com/zhang134/invMap.git
- **Winnowmap2** (Competing long-read mapper used as a baseline call set to determine which inversions are invMap-only)
- **minimap2** (Competing long-read mapper baseline; fails to discover the novel 231 bp inversion that invMap detects)
- **NGMLR** (Competing long-read mapper baseline for interval-overlap comparison)
- **lra** (Competing long-read mapper baseline for interval-overlap comparison)

## Examples

```
./invMap genome.fa reads.fa > invMap_aligned.sam
```

## Evaluation signals

- The evidence table reproduces the novel 231 bp inversion entry that is not part of the simulated ground truth.
- The mean alignment identity of non-ground-truth invMap-only calls equals the reported 81.95%.
- Winnowmap2, minimap2, NGMLR, and lra show zero interval overlap with each invMap-only call on reference coordinates.
- Each record carries supporting anchors traceable to the invMap chaining output.
- Truth-status labels are consistent with Supplementary Table S5 and Figures S3-S4.

## Limitations

- invMap requires a minimum of 16G of RAM for mapping sequence datasets to large reference genomes.
- Novel invMap-only calls do not belong to the simulated ground truth, so their correctness cannot be verified against simulation labels.
- Evidence in the article is demonstrated on the NA12878 real human genome sequencing dataset; extension to other datasets is not established here.

## Evidence

- [abstract] Longer reads produced by PacBio or Oxford Nanopore sequencers could more frequently span the breakpoints of structural variations (SVs) than shorter reads: "Longer reads produced by PacBio or Oxford Nanopore sequencers could more frequently span the breakpoints of structural variations (SVs) than shorter reads"
- [results] invMap-only detection finding: [evidence span withheld — anchor status: not_found]
- [abstract] The real human genome sequencing dataset of NA12878 illustrates that invMap can effectively find more candidate variant calls for inversions than the competing methods: "The real human genome sequencing dataset of NA12878 illustrates that invMap can effectively find more candidate variant calls for inversions than the competing methods"
- [abstract] chaining workflow: "invMap first locates the aligned region with a specifically designed scoring method for chaining, then checks the remaining anchors in the aligned region to discover potential inversions"
- [readme] README invocation: "Step 1: compile codes using make command"
