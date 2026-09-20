---
name: reference-based-assembly-alignment
description: Use when a scaffolded assembly (e.g., ntLink output like celegans_flye.fa.k32.w100.z1000.ntLink.scaffolds.fa)
  must be assessed against a reference genome to confirm contiguity gains and absence
  of new misassemblies.
when_to_use_negative:
- No reference genome is available for the organism — QUAST's NG50, genome fraction,
  and misassembly metrics require reference-based alignment; use reference-free statistics
  only
- The assembly has not been scaffolded (or processed) yet — aligning an unchanged
  draft adds no information about the scaffolding step's effect
edam_operation: http://edamontology.org/operation_0525
edam_topics:
- http://edamontology.org/topic_0196
- http://edamontology.org/topic_0622
tools:
- name: QUAST
  role: Aligns the scaffolded assembly to the reference genome to compute NG50, genome
    fraction, and misassembly counts
  repo: https://github.com/ablab/quast
- name: ABySS
  role: abyss-fac computes contiguity statistics (total length, N50, sequence counts)
    for draft and scaffolded assemblies; abyss-scaffold (ABySS v2.3.0+) traverses
    the ntLink scaffold graph
  repo: https://github.com/BirolLab/abyss
- name: ntLink
  role: Produces the scaffolded assembly with overlap detection that is subsequently
    evaluated against the reference
  repo: https://github.com/bcgsc/ntLink
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1002/cpz1.733
    title: 'ntLink: A Toolkit for <i>De Novo</i> Genome Assembly Scaffolding and Mapping
      Using Long Reads'
  source_content_hash: sha256:23281c5ec3630881889d0809f5c07e9f9af70f092ffe6bd78f900cd9031aaa02
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ablab/quast
  tools:
  - ABySS
  - QUAST
  - ntLink
  edam_operation: http://edamontology.org/operation_0525
  edam_topics:
  - http://edamontology.org/topic_0196
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8461538461538461
n_claims: 13
---

# reference-based-assembly-alignment

## Summary

Evaluates a draft or scaffolded genome assembly by aligning it to a reference genome with QUAST, yielding contiguity and correctness metrics (NG50, genome fraction, misassembly counts) that complement reference-free statistics from abyss-fac.

## When to use

Use when a scaffolded assembly (e.g., ntLink output like celegans_flye.fa.k32.w100.z1000.ntLink.scaffolds.fa) must be assessed against a reference genome to confirm contiguity gains and absence of new misassemblies.

## When NOT to use

- No reference genome is available for the organism — QUAST's NG50, genome fraction, and misassembly metrics require reference-based alignment; use reference-free statistics only
- The assembly has not been scaffolded (or processed) yet — aligning an unchanged draft adds no information about the scaffolding step's effect

## Inputs

- Scaffolded assembly FASTA (e.g., celegans_flye.fa.k32.w100.z1000.ntLink.scaffolds.fa)
- Draft assembly FASTA for baseline comparison (e.g., celegans_flye.fa)
- Reference genome (C. elegans GCA_000002985.3) for QUAST alignment
- Long reads used for scaffolding (SRR10028109.fq.gz) for the ntLink step

## Outputs

- abyss-fac summary statistics (total length, N50, number of sequences) for draft and scaffolded assemblies
- QUAST report with NG50, genome fraction, and misassembly counts for the scaffolded assembly vs. reference
- Comparison table of metrics against the paper's Table 3 values

## How to apply

After scaffolding a draft assembly (e.g., a C. elegans Flye draft scaffolded by ntLink with k=32, w=100, z=1000, overlap=True), compute reference-free statistics first with abyss-fac (total length, N50, sequence counts), then run QUAST aligning the ntLink scaffolds against the reference genome GCA_000002985.3. QUAST reports NG50, genome fraction, and misassembly counts, which are tabulated alongside abyss-fac metrics and compared to the draft assembly and to the paper's Table 3 values. Validation is twofold: NG50/N50 should increase and sequence counts decrease relative to the Flye draft, and misassemblies should not substantially increase — confirming that overlap detection correctly trimmed joined sequences rather than introducing structural errors. ABySS v2.3.0+ is required because ntLink delegates scaffold graph traversal to abyss-scaffold.

## Related tools

- **QUAST** (Aligns the scaffolded assembly to the reference genome to compute NG50, genome fraction, and misassembly counts) — https://github.com/ablab/quast
- **ABySS** (abyss-fac computes contiguity statistics (total length, N50, sequence counts) for draft and scaffolded assemblies; abyss-scaffold (ABySS v2.3.0+) traverses the ntLink scaffold graph) — https://github.com/BirolLab/abyss
- **ntLink** (Produces the scaffolded assembly with overlap detection that is subsequently evaluated against the reference) — https://github.com/bcgsc/ntLink

## Examples

```
quast.py celegans_flye.fa.k32.w100.z1000.ntLink.scaffolds.fa -r GCA_000002985.3.fa -o quast_ntlink_out
```

## Evaluation signals

- The scaffold FASTA exists with the *ntLink.scaffolds.fa suffix
- abyss-fac shows increased N50/NG50 and a decreased number of sequences versus the Flye draft
- QUAST NG50 and genome fraction reproduce the Table 3-reported contiguity gains over the draft
- Misassembly counts from QUAST do not show a large increase relative to the draft assembly

## Limitations

- Validation depends on the accuracy and completeness of the reference genome; NG50 and genome fraction are only meaningful relative to a suitable reference
- Overlap detection trims overlapping joined sequences at scaffold joins, so scaffolded sequence length can differ from the draft in ways unrelated to misassembly

## Evidence

- [results] Running ntLink scaffolding with overlap detection on the C. elegans draft assembly improves contiguity: the NG50 length increases after scaffolding and the number of sequences decreases.: [evidence span withheld — anchor status: not_found]
- [methods] Run QUAST on the ntLink scaffolds aligned against the C. elegans reference GCA_000002985.3 to obtain NG50, genome fraction, and misassembly counts.: [evidence span withheld — anchor status: not_found]
- [abstract] ntLink is a flexible and resource-efficient genome scaffolding tool that utilizes long-read sequencing data to improve upon draft genome assemblies: "ntLink is a flexible and resource-efficient genome scaffolding tool that utilizes long-read sequencing data to improve upon draft genome assemblies"
