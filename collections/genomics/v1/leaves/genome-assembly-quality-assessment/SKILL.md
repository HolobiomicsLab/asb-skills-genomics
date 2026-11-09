---
name: genome-assembly-quality-assessment
description: Use when you have a draft long-read assembly and its polished/finished
  counterpart (e.g. ntEdit+Sealer output) plus a reference genome, and need to quantify
  mismatches and indels per 100 kbp to confirm polishing improved accuracy.
when_to_use_negative:
- No reference genome is available for the organism, since the mismatch/indel accuracy
  metrics here are reference-based
- The input is raw sequencing reads rather than assembled FASTA contigs/scaffolds
  — polishing and finishing (ntEdit+Sealer) should run first
- You need to evaluate features beyond base accuracy/indels that require the full
  (non --fast) QUAST mode, e.g. detailed structural variant analysis
edam_operation: http://edamontology.org/operation_3180
edam_topics:
- http://edamontology.org/topic_0196
- http://edamontology.org/topic_3697
tools:
- name: QUAST
  role: Reference-based quality assessment tool; computes mismatches and indels per
    100 kbp for draft and polished assemblies in a single comparison table
  repo: https://github.com/ablab/quast
- name: ntEdit+Sealer
  role: Protocol that produced the polished/finished assembly being evaluated; ntEdit
    corrects base errors and small indels, Sealer fills gaps and flagged regions
  repo: https://github.com/BirolLab/ntedit_sealer_protocol
- name: ntEdit
  role: Alignment-free, k-mer/Bloom filter-based polishing tool that corrects base
    mismatches and frame-shift errors in the draft assembly prior to assessment
  repo: https://github.com/BirolLab/ntEdit
- name: Sealer
  role: ABySS component that fills assembly gaps and problematic regions flagged by
    ntEdit in the finished assembly
  repo: https://github.com/BirolLab/abyss
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1002/cpz1.442
    title: 'ntEdit+Sealer: Efficient Targeted Error Resolution and Automated Finishing
      of Long‐Read Genome Assemblies'
  source_content_hash: sha256:55e582cee402834a3e3b3b183c0fbbf9fecb439f687f24e261320c5c58e54f33
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ablab/quast
  tools:
  - QUAST
  - Sealer
  - ntEdit
  - ntEdit+Sealer
  edam_operation: http://edamontology.org/operation_3180
  edam_topics:
  - http://edamontology.org/topic_0196
  - http://edamontology.org/topic_3697
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8235294117647058
n_claims: 17
---

# genome-assembly-quality-assessment

## Summary

Assess the base accuracy and indel error rates of draft versus ntEdit+Sealer-polished genome assemblies using QUAST reference-based evaluation against the E. coli K-12 MG1655 (GCF_000005845.2) reference.

## When to use

Use when you have a draft long-read assembly and its polished/finished counterpart (e.g. ntEdit+Sealer output) plus a reference genome, and need to quantify mismatches and indels per 100 kbp to confirm polishing improved accuracy.

## When NOT to use

- No reference genome is available for the organism, since the mismatch/indel accuracy metrics here are reference-based
- The input is raw sequencing reads rather than assembled FASTA contigs/scaffolds — polishing and finishing (ntEdit+Sealer) should run first
- You need to evaluate features beyond base accuracy/indels that require the full (non --fast) QUAST mode, e.g. detailed structural variant analysis

## Inputs

- Draft long-read assembly FASTA (e.g. Shasta assembly)
- Polished/finished assembly FASTA (.ntedit_edited.prepd.sealer_scaffold.fa)
- Reference genome FASTA (E. coli K-12 MG1655, NCBI GCF_000005845.2)

## Outputs

- QUAST report.tsv summary table with per-assembly accuracy metrics, including '# mismatches per 100 kbp' and '# indels per 100 kbp'
- Comparative assessment of draft vs polished error rates

## How to apply

Collect the draft Shasta assembly FASTA and the polished assembly (suffixed .ntedit_edited.prepd.sealer_scaffold.fa) into one working directory alongside the reference genome FASTA (E. coli K-12 MG1655, GCF_000005845.2). Run QUAST on both assemblies in a single invocation with the --fast option and the reference genome specified, so all assemblies are reported in one comparison table. Parse the generated report.tsv and confirm the '# mismatches per 100 kbp' and '# indels per 100 kbp' columns are present for both assemblies. Compare draft versus polished values of these two error metrics: both are expected to decrease after ntEdit+Sealer polishing, consistent with the paper's claim of highly accurate, error-corrected assemblies.

## Related tools

- **QUAST** (Reference-based quality assessment tool; computes mismatches and indels per 100 kbp for draft and polished assemblies in a single comparison table) — https://github.com/ablab/quast
- **ntEdit+Sealer** (Protocol that produced the polished/finished assembly being evaluated; ntEdit corrects base errors and small indels, Sealer fills gaps and flagged regions) — https://github.com/BirolLab/ntedit_sealer_protocol
- **ntEdit** (Alignment-free, k-mer/Bloom filter-based polishing tool that corrects base mismatches and frame-shift errors in the draft assembly prior to assessment) — https://github.com/BirolLab/ntEdit
- **Sealer** (ABySS component that fills assembly gaps and problematic regions flagged by ntEdit in the finished assembly) — https://github.com/BirolLab/abyss

## Examples

```
quast.py --fast -r GCF_000005845.2.fasta draft_shasta.fa polished.ntedit_edited.prepd.sealer_scaffold.fa -o quast_out
```

## Evaluation signals

- report.tsv contains both '# mismatches per 100 kbp' and '# indels per 100 kbp' columns for every submitted assembly
- Both assemblies appear together in a single QUAST comparison table (one invocation with all FASTAs)
- '# mismatches per 100 kbp' decreases in the polished assembly relative to the draft
- '# indels per 100 kbp' decreases in the polished assembly relative to the draft
- The final summary table is saved as report.tsv

## Limitations

- Accuracy metrics are reference-dependent; conclusions are only valid relative to the chosen reference (E. coli K-12 MG1655, GCF_000005845.2 in this workflow)
- Running QUAST with the --fast option trades runtime for reduced analysis depth compared to a full QUAST run
- ntEdit+Sealer polishing depends on short-read k-mer Bloom filters built by ntHits and ABySS-Bloom; k-mer length (k) and Bloom filter size (b) must be selected appropriately for the dataset

## Evidence

- [abstract] ntEdit+Sealer produces highly accurate, error-corrected genome assemblies: "ntEdit+Sealer produces highly accurate, error-corrected genome assemblies"
- [abstract] The protocol includes ntEdit for correcting base errors and small indels, and for marking potentially problematic regions: "The protocol includes ntEdit for correcting base errors and small indels, and for marking potentially problematic regions"
- [methods] Run the protocol as a Makefile pipeline: "ntedit-sealer finish seqs=draft-assembly.fa reads='reads_1.fq.gz reads_2.fq.gz' k='80 65 50' b=5G"
- [results] Task workflow: run QUAST on both assemblies in one invocation with --fast and the reference; parse report.tsv and compare mismatches/indels per 100 kbp: [evidence span withheld — anchor status: not_found]
- [results] Task workflow: confirm error metric columns in report.tsv decrease after polishing: [evidence span withheld — anchor status: not_found]
- [readme] README: ntEdit polishes the draft assembly and flags erroneous regions, then Sealer fills assembly gaps and erroneous sequence regions flagged by ntEdit: "An automated protocol for finishing long-read genome assemblies using short reads. [ntEdit](https://github.com/BirolLab/ntEdit) polishes the draft assembly and flags erroneous regions, then"
