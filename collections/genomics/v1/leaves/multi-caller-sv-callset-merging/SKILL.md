---
name: multi-caller-sv-callset-merging
description: Use when you hold filtered VCF call sets from several nanopore SV callers
  run on the same lra/minimap2/NGMLR alignment and need one consensus SV call set
  per aligner plus a summary type-distribution table.
when_to_use_negative:
- When the goal is coverage-based validation of insertions or inversions — coverage
  analysis such as disCoverage can only be applied to deletions and duplications
- When 'disCoverage' alone is expected to produce the call set — it is not a variant
  caller and needs putative SV positions obtained from another technique or multi-sample
  analysis
- When input is a single caller's already-merged or raw VCF with no cross-caller call
  sets to consensus on — no merging signal exists
edam_operation: http://edamontology.org/operation_3227
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_0625
tools:
- name: Sniffles
  role: Nanopore SV variant caller whose per-caller VCF is one input to the per-aligner
    merge
- name: CuteSV
  role: Nanopore SV variant caller contributing call sets to the merge
- name: SVIM
  role: Nanopore SV variant caller contributing call sets to the merge
- name: NanoVar
  role: Nanopore SV variant caller used only where compatible with the aligner output
- name: SURVIVOR
  role: Merges the filtered per-aligner call sets across callers into one merged VCF
    per aligner
- name: disCoverage
  role: Polishes/filter caller results before merging by checking coverage support
    of deletions and duplications (p < 0.05 plus coverage difference)
  repo: https://github.com/javiercguard/disCoverage
- name: lra
  role: Long-read aligner producing one of the BAMs on which callers are run before
    merging
- name: minimap2
  role: Long-read aligner producing one of the BAMs on which callers are run before
    merging
- name: NGMLR
  role: Long-read aligner producing one of the BAMs on which callers are run before
    merging
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1016/j.jare.2022.10.012
    title: Improvement of large copy number variant detection by whole genome nanopore
      sequencing
  source_content_hash: sha256:9906a55ed7207433bb5c8a546d5d7e8821421052db56930895400563e11761d6
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
  - SURVIVOR
  - SVIM
  - Sniffles
  - disCoverage
  - lra
  - minimap2
  edam_operation: http://edamontology.org/operation_3227
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_0625
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# multi-caller-sv-callset-merging

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Combines structural variant call sets produced by multiple long-read variant callers (Sniffles, CuteSV, SVIM, NanoVar) on the same nanopore BAM into one consensus VCF per aligner, then tabulates SV counts and DEL/DUP/INS distribution. It counters the paper's finding that single callers report incomplete variant sets containing erroneous calls.

## When to use

Use when you hold filtered VCF call sets from several nanopore SV callers run on the same lra/minimap2/NGMLR alignment and need one consensus SV call set per aligner plus a summary type-distribution table.

## When NOT to use

- When the goal is coverage-based validation of insertions or inversions — coverage analysis such as disCoverage can only be applied to deletions and duplications
- When 'disCoverage' alone is expected to produce the call set — it is not a variant caller and needs putative SV positions obtained from another technique or multi-sample analysis
- When input is a single caller's already-merged or raw VCF with no cross-caller call sets to consensus on — no merging signal exists

## Inputs

- Long-read nanopore reads in FASTQ (e.g., public NA19240 PromethION control dataset)
- One BAM per aligner from lra, minimap2, and NGMLR alignments against the hg38 human reference
- Filtered per-caller SV VCFs from CuteSV, Sniffles, SVIM, and NanoVar (where compatible) for each aligner
- Coverage results from disCoverage for deletions and duplications

## Outputs

- One merged VCF per aligner (lra, minimap2, NGMLR) containing consensus SV calls including deletions, duplications, insertions, inversions, and translocations
- Summary counts table of total SV counts and per-aligner DEL/DUP/INS type distribution

## How to apply

First align the nanopore reads (e.g., NA19240 PromethION) to hg38 with each of lra, minimap2, and NGMLR, then run the four nanopore callers CuteSV, Sniffles, SVIM, and NanoVar (NanoVar only where compatible with the aligner output) on each BAM. Filter each caller's VCF by read-support threshold, discarding very large SVs (>10 Mb) that encompass other SVs or centromeres as artifacts; for deletions and duplications, additionally use disCoverage with a p-value threshold < 0.05 combined with coverage difference as the coverage-support criterion. Merge the filtered per-aligner call sets across callers with SURVIVOR, yielding one merged VCF per aligner (lra, minimap2, NGMLR). Finally tabulate total SV counts and the deletions/duplications/insertions distribution per aligner, and check the distribution against reference values such as 68.8/66.0/64.0% deletions for lra/minimap2/NGMLR. The rationale is that consensus across callers, after artifact and coverage filtering, reduces the false positives and incompleteness inherent to individual third-generation-sequencing call sets.

## Related tools

- **Sniffles** (Nanopore SV variant caller whose per-caller VCF is one input to the per-aligner merge)
- **CuteSV** (Nanopore SV variant caller contributing call sets to the merge)
- **SVIM** (Nanopore SV variant caller contributing call sets to the merge)
- **NanoVar** (Nanopore SV variant caller used only where compatible with the aligner output)
- **SURVIVOR** (Merges the filtered per-aligner call sets across callers into one merged VCF per aligner)
- **disCoverage** (Polishes/filter caller results before merging by checking coverage support of deletions and duplications (p < 0.05 plus coverage difference)) — https://github.com/javiercguard/disCoverage
- **lra** (Long-read aligner producing one of the BAMs on which callers are run before merging)
- **minimap2** (Long-read aligner producing one of the BAMs on which callers are run before merging)
- **NGMLR** (Long-read aligner producing one of the BAMs on which callers are run before merging)

## Examples

```
python nanopore_slurm_pipeline.py  # handles alignment and variant calling; edit config/config.sh to fit the input data (see README)
```

## Evaluation signals

- A merged VCF exists for each of the three aligners (lra, minimap2, NGMLR), in .vcf format
- The DEL/DUP/INS type distribution per aligner matches the reported Table 3 values, e.g., 68.8/66.0/64.0% deletions for lra/minimap2/NGMLR
- No SVs larger than 10 Mb encompassing other SVs or centromeres remain in the merged call sets
- Total SV counts per aligner are comparable to the reported summary counts table
- Merged sets retain inversions and translocations alongside deletions, duplications, and insertions

## Limitations

- No software versions were reported for any of the aligners or variant callers (lra, minimap2, NGMLR, Sniffles, SVIM, CuteSV, NanoVar), so exact reproducibility is uncertain
- NGMLR had known bugs at the time of writing and is slower than alternatives
- Coverage-based polishing (disCoverage) is limited by non-uniform coverage, local peaks and valleys, unresolved genome regions, and unclear shift boundaries, and applies only to deletions and duplications
- The p-value threshold of 0.05 for coverage support was not confirmed as optimal; further studies are required
- Variant callers (and hence merged sets) underperform on large (>50 kb) CNVs, and newer tools have not improved this

## Evidence
