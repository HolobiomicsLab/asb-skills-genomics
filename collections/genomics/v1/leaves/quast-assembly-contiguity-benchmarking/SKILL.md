---
name: quast-assembly-contiguity-benchmarking
description: Use when you have two or more genome assembly FASTA files — such as a
  draft and a corrected version (e.g., celegans-abyss vs. celegans-abyss.cut500.tigmint.fa)
  — and need to judge whether correction/scaffolding improved contiguity and correctness.
when_to_use_negative:
- The input is raw read data (e.g., FASTQ from SRR10028109) or read-to-contig mappings
  rather than assembled FASTA contigs/scaffolds
- You need to perform the scaffolding or correction itself — use ntLink scaffolding
  protocols or Tigmint-long first; this skill only evaluates the resulting assemblies
- You have only a single assembly with no baseline to compare against, so no corrected-vs-baseline
  metric comparison can be made
edam_operation: http://edamontology.org/operation_0310
edam_topics:
- http://edamontology.org/topic_0193
tools:
- name: QUAST
  role: Evaluates contiguity and correctness metrics of the corrected and baseline
    assemblies for comparison (Table 6)
  repo: https://github.com/ablab/quast
- name: Tigmint-long
  role: Produces the corrected FASTA (celegans-abyss.cut500.tigmint.fa) that is benchmarked
    against the baseline
  repo: https://github.com/BirolLab/tigmint
- name: ntLink
  role: Supplies the lightweight minimizer-based long-read mappings used by Tigmint-long
    to cut the draft at putative misassemblies
  repo: https://github.com/BirolLab/ntLink
- name: ABySS
  role: Generates the celegans-abyss draft assembly that serves as the uncorrected
    baseline in the QUAST comparison
  repo: https://github.com/BirolLab/abyss
- name: sra-tools
  role: Downloads the C. elegans long-read dataset SRR10028109 used for the correction
    step upstream of benchmarking
  repo: https://github.com/ncbi/sra-tools
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1002/cpz1.733
    title: 'ntLink: A Toolkit for <i>De Novo</i> Genome Assembly Scaffolding and Mapping
      Using Long Reads'
  source_content_hash: sha256:b9011af3e37e4fa35601d83388017abfa36ebfc180925a529ec494f5cdace0f1
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ablab/quast
  tools:
  - ABySS
  - QUAST
  - Tigmint-long
  - ntLink
  - sra-tools
  edam_operation: http://edamontology.org/operation_0310
  edam_topics:
  - http://edamontology.org/topic_0193
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8235294117647058
n_claims: 17
---

# quast-assembly-contiguity-benchmarking

## Summary

Benchmark genome assemblies (e.g., a Tigmint-long corrected FASTA vs. its uncorrected baseline) with QUAST to quantify contiguity and correctness improvements after scaffolding or misassembly correction, as done in Table 6 of the ntLink protocols.

## When to use

Use when you have two or more genome assembly FASTA files — such as a draft and a corrected version (e.g., celegans-abyss vs. celegans-abyss.cut500.tigmint.fa) — and need to judge whether correction/scaffolding improved contiguity and correctness.

## When NOT to use

- The input is raw read data (e.g., FASTQ from SRR10028109) or read-to-contig mappings rather than assembled FASTA contigs/scaffolds
- You need to perform the scaffolding or correction itself — use ntLink scaffolding protocols or Tigmint-long first; this skill only evaluates the resulting assemblies
- You have only a single assembly with no baseline to compare against, so no corrected-vs-baseline metric comparison can be made

## Inputs

- Corrected assembly FASTA (e.g., celegans-abyss.cut500.tigmint.fa produced by Tigmint-long with ntLink mappings)
- Uncorrected baseline/draft assembly FASTA (e.g., the ABySS celegans-abyss draft)

## Outputs

- QUAST report with contiguity and correctness metrics for each input assembly, enabling corrected-vs-baseline comparison as in Table 6

## How to apply

After an assembly-correction step (here, Tigmint-long run with ntLink minimizer-based mappings via `tigmint-make tigmint-long draft=celegans-abyss reads=SRR10028109 mapping=ntLink t=5 span=2`, which cuts the draft at putative misassemblies and writes celegans-abyss.cut500.tigmint.fa), run QUAST (ablab/quast) on both the corrected assembly and the uncorrected baseline. Compare the resulting QUAST contiguity and correctness metrics between the two FASTA files side by side, following the article's comparison in Table 6. The run is accepted if the corrected assembly shows improved contiguity/correctness consistent with the reported QUAST comparison; this validation step is what closes the Alternate Protocol 2 workflow that starts from the ABySS draft and the C. elegans long-read dataset SRR10028109 (downloaded with ncbi/sra-tools).

## Related tools

- **QUAST** (Evaluates contiguity and correctness metrics of the corrected and baseline assemblies for comparison (Table 6)) — https://github.com/ablab/quast
- **Tigmint-long** (Produces the corrected FASTA (celegans-abyss.cut500.tigmint.fa) that is benchmarked against the baseline) — https://github.com/BirolLab/tigmint
- **ntLink** (Supplies the lightweight minimizer-based long-read mappings used by Tigmint-long to cut the draft at putative misassemblies) — https://github.com/BirolLab/ntLink
- **ABySS** (Generates the celegans-abyss draft assembly that serves as the uncorrected baseline in the QUAST comparison) — https://github.com/BirolLab/abyss
- **sra-tools** (Downloads the C. elegans long-read dataset SRR10028109 used for the correction step upstream of benchmarking) — https://github.com/ncbi/sra-tools

## Examples

```
quast.py celegans-abyss.cut500.tigmint.fa celegans-abyss.fa
```

## Evaluation signals

- QUAST runs to completion on both the corrected FASTA (celegans-abyss.cut500.tigmint.fa) and the uncorrected baseline without errors
- The corrected assembly shows improved contiguity/correctness metrics consistent with the reported Table 6 QUAST comparison
- The corrected FASTA reflects cuts at putative misassemblies (produced with t=5, span=2 Tigmint-long parameters), i.e., a different contig/scaffold count than the baseline
- The same assemblies (ABySS draft celegans-abyss and long reads SRR10028109) reproduce the pipeline's expected corrected output filename

## Limitations

- This skill only benchmarks assemblies; it does not itself detect or fix misassemblies — correction must be performed beforehand with Tigmint-long and ntLink mappings
- Acceptance of the correction is judged by comparison against the article's reported QUAST results (Table 6); other datasets may need their own baseline reference points
- The QUAST README provided is truncated, so tool-specific runtime parameters and reference-based options cannot be grounded in this context

## Evidence

- [abstract] ntLink is a flexible and resource-efficient genome scaffolding tool that utilizes long-read sequencing data to improve upon draft genome assemblies: "ntLink is a flexible and resource-efficient genome scaffolding tool that utilizes long-read sequencing data to improve upon draft genome assemblies"
- [intro] current state-of-the-art de novo long-read assemblers such as Flye (Kolmogorov et al., 2019) are generating highly contiguous assemblies: "current state-of-the-art de novo long-read assemblers such as Flye (Kolmogorov et al., 2019) are generating highly contiguous assemblies"
- [readme] Correct misassemblies in genome assembly drafts using linked or long sequencing: [evidence span withheld — anchor status: not_found]
- [readme] ABySS is a *de novo* sequence assembler intended for short paired-end reads and genomes of all sizes.: [evidence span withheld — anchor status: not_found]
