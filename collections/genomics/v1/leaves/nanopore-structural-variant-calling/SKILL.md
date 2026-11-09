---
name: nanopore-structural-variant-calling
description: Use when long-read PromethION/nanopore WGS reads must be turned into
  a merged SV call set (DEL/DUP/INS/INV/TRA) and when large CNVs >50 kb or coverage-supported
  polishing of caller output is the goal.
when_to_use_negative:
- Coverage-based polishing is only valid for deletions and duplications, not for insertions,
  inversions, or translocations
- 'disCoverage is not a variant caller: it needs putative SV positions as input, so
  it cannot be used to discover SVs de novo'
- Do not trust very large SVs (>10 Mb) encompassing other SVs or centromeres; these
  are artifacts to discard, not calls to keep
edam_operation: http://edamontology.org/operation_3227
edam_topics:
- http://edamontology.org/topic_3678
- http://edamontology.org/topic_3053
tools:
- name: minimap2
  role: aligner alternative producing one BAM per run for SV calling
- name: NGMLR
  role: aligner alternative; NGMLR + Sniffles gave the largest aCGH/nanopore consensus
    (N = 31)
- name: lra
  role: aligner alternative in the aligner-by-caller comparison
- name: Sniffles
  role: nanopore SV variant caller applied to each aligner BAM
- name: CuteSV
  role: nanopore SV variant caller applied to each aligner BAM
- name: SVIM
  role: nanopore SV variant caller applied to each aligner BAM
- name: NanoVar
  role: nanopore SV variant caller, used only where compatible with the aligner output
- name: disCoverage
  role: coverage-based polishing/filtering of called deletions and duplications using
    p-value < 0.05 plus coverage difference
  repo: https://github.com/javiercguard/disCoverage
- name: mosdepth
  role: BAM/CRAM depth calculation used for the coverage analysis
  repo: https://github.com/brentp/mosdepth
- name: nanopore_pipeline_21
  role: SLURM pipeline scripts for alignment and variant calling
  repo: https://github.com/javiercguard/nanopore_pipeline_21
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1016/j.jare.2022.10.012
    title: Improvement of large copy number variant detection by whole genome nanopore
      sequencing
  source_content_hash: sha256:c18192e8bb9a53c881c4e75a4008256288b37d408e82c05b4561ff52a7d4a0d3
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/javiercguard/disCoverage
  repo_url: https://github.com/javiercguard/disCoverage
  tools:
  - CuteSV
  - NGMLR
  - NanoVar
  - SVIM
  - Sniffles
  - disCoverage
  - lra
  - minimap2
  - mosdepth
  - nanopore_pipeline_21
  edam_operation: http://edamontology.org/operation_3227
  edam_topics:
  - http://edamontology.org/topic_3678
  - http://edamontology.org/topic_3053
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6153846153846154
n_claims: 13
---

# nanopore-structural-variant-calling

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

End-to-end nanopore whole-genome pipeline for calling large (>50 kb) structural variants, especially deletions and duplications, by varying aligner (lra, minimap2, NGMLR) and variant caller (Sniffles, CuteSV, SVIM, NanoVar) combinations against hg38/hg19 and polishing calls with coverage-based filtering via disCoverage.

## When to use

Use when long-read PromethION/nanopore WGS reads must be turned into a merged SV call set (DEL/DUP/INS/INV/TRA) and when large CNVs >50 kb or coverage-supported polishing of caller output is the goal.

## When NOT to use

- Coverage-based polishing is only valid for deletions and duplications, not for insertions, inversions, or translocations
- disCoverage is not a variant caller: it needs putative SV positions as input, so it cannot be used to discover SVs de novo
- Do not trust very large SVs (>10 Mb) encompassing other SVs or centromeres; these are artifacts to discard, not calls to keep

## Inputs

- PromethION nanopore whole-genome sequencing reads (e.g. NA19240 control dataset) as FASTQ
- Reference human genome in FASTA (hg38 preferred over hg19)
- Per-aligner BAM files (lra, minimap2, NGMLR alignments)
- Optional reference variant/truth set (e.g. Chaisson et al. truth set, aCGH call sets) for benchmarking

## Outputs

- One merged SV VCF per aligner (lra, minimap2, NGMLR) after SURVIVOR merging across callers
- Per-caller SV call sets filtered by read support and disCoverage coverage support
- Summary counts table of total SVs and deletions/duplications/insertions percentage distribution per aligner

## How to apply

Align the nanopore reads to the chosen reference (hg38 performed better than hg19) using each of lra, minimap2, and NGMLR to produce one BAM per aligner, then call SVs on each BAM with CuteSV, Sniffles, SVIM, and NanoVar (NanoVar only where compatible with the aligner output). Filter calls by read-support threshold, discard very large SVs (>10 Mb) that encompass other SVs or centromeres as artifacts, and for deletions/duplications apply disCoverage coverage support with a p-value threshold < 0.05 combined with the coverage difference criterion (coverage can be computed with mosdepth). Merge the filtered per-aligner call sets across callers with SURVIVOR to yield one merged VCF per aligner, then tabulate total SV counts and the deletions/duplications/insertions distribution (e.g. NA19240 control: 68.8/66.0/64.0% deletions for lra/minimap2/NGMLR). Rationale: single callers report incomplete and erroneous sets, and coverage analysis polishes results, improving specificity by up to 62% and sensitivity by 15%.

## Related tools

- **minimap2** (aligner alternative producing one BAM per run for SV calling)
- **NGMLR** (aligner alternative; NGMLR + Sniffles gave the largest aCGH/nanopore consensus (N = 31))
- **lra** (aligner alternative in the aligner-by-caller comparison)
- **Sniffles** (nanopore SV variant caller applied to each aligner BAM)
- **CuteSV** (nanopore SV variant caller applied to each aligner BAM)
- **SVIM** (nanopore SV variant caller applied to each aligner BAM)
- **NanoVar** (nanopore SV variant caller, used only where compatible with the aligner output)
- **disCoverage** (coverage-based polishing/filtering of called deletions and duplications using p-value < 0.05 plus coverage difference) — https://github.com/javiercguard/disCoverage
- **mosdepth** (BAM/CRAM depth calculation used for the coverage analysis) — https://github.com/brentp/mosdepth
- **nanopore_pipeline_21** (SLURM pipeline scripts for alignment and variant calling) — https://github.com/javiercguard/nanopore_pipeline_21

## Examples

```
mamba create --use-local -n discoverage discoverage; disCoverage -h
```

## Evaluation signals

- A merged VCF exists for each of the three aligners (file_exists, format .vcf) after SURVIVOR merging
- The summary counts and DEL/DUP/INS type distribution match the reported Table 3 values (e.g. 68.8/66.0/64.0% deletions for lra/minimap2/NGMLR on NA19240)
- No calls with SV size >10 Mb that encompass other SVs or centromeres remain in the filtered set
- Coverage agreement: called deletions/duplications are supported by disCoverage at p < 0.05 with the coverage difference criterion
- Agreement with truth/reference (e.g. aCGH-detected SVs recovered, Chaisson et al. truth set support) improves after disCoverage polishing

## Limitations

- No software versions are reported for any of the aligners or variant callers, so exact reproduction requires testing versions
- NGMLR had known bugs at the time of writing and is slower than alternatives
- SVs whose coordinates could not be translated to hg38 (8 in the study) were impossible to find when using hg38
- Coverage is not uniform: local peaks and valleys occur without necessarily indicating an SV, and shift boundaries may be unclear
- Coverage-based filtering applies only to deletions and duplications; the p-value threshold 0.05 was not confirmed as optimal

## Evidence
