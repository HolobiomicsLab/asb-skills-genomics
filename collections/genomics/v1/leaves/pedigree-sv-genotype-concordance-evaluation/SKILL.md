---
name: pedigree-sv-genotype-concordance-evaluation
description: Use when you have HiFi WGS read sets for multiple related samples (e.g.
  a pedigree like CEPH-1463) and need to compare SV callers by genotype concordance
  with known inheritance rather than a truth-set benchmark.
when_to_use_negative:
- Input samples are unrelated or lack a known pedigree/inheritance pattern — pedigree
  concordance cannot be computed; use a truth-set benchmark (e.g. GIAB HG002 T2T SVs
  with truvari) instead.
- Data is short-read or non-HiFi — sawfish, pbsv, and this HiFi-tuned concordance
  workflow are designed for mapped HiFi sequencing reads.
- You only have a single sample — sawfish joint-call requires discover results from
  each sample entering the joint step, and pedigree concordance needs multiple related
  samples.
edam_operation: http://edamontology.org/operation_3227
edam_topics:
- http://edamontology.org/topic_3175
- http://edamontology.org/topic_0625
tools:
- name: sawfish
  role: 'Primary caller: per-sample discover, then joint-call to merge duplicate SV
    haplotypes and jointly genotype SVs across the seven pedigree samples'
  repo: https://github.com/PacificBiosciences/sawfish
- name: pbmm2
  role: Maps HiFi CCS reads to human_GRCh38_no_alt_analysis_set with --preset CCS
    and --sort before SV calling
- name: pbsv
  role: Comparator state-of-the-art long-read SV caller run on the same mapped samples
  repo: https://github.com/PacificBiosciences/pbsv
- name: Sniffles2
  role: Comparator state-of-the-art long-read SV caller, filtered at GQ>=10 for concordance
    comparison
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1093/bioinformatics/btaf136
    title: 'Sawfish: improving long-read structural variant discovery and genotyping
      with local haplotype modeling'
  source_content_hash: sha256:fffa9381958947bff8d471259ef8ea3a3ac68f5c0ea0bf0727354c9ee28c1cf2
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/PacificBiosciences/sawfish
  tools:
  - Sniffles2
  - pbmm2
  - pbsv
  - sawfish
  edam_operation: http://edamontology.org/operation_3227
  edam_topics:
  - http://edamontology.org/topic_3175
  - http://edamontology.org/topic_0625
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 6
---

# pedigree-sv-genotype-concordance-evaluation

## Summary

Evaluates long-read SV callers by jointly genotyping all seven CEPH-1463 pedigree samples and classifying each called SV as pedigree-concordant or not against the known CEPH-1463 inheritance pattern, then tabulating per-caller concordance counts and proportions.

## When to use

Use when you have HiFi WGS read sets for multiple related samples (e.g. a pedigree like CEPH-1463) and need to compare SV callers by genotype concordance with known inheritance rather than a truth-set benchmark.

## When NOT to use

- Input samples are unrelated or lack a known pedigree/inheritance pattern — pedigree concordance cannot be computed; use a truth-set benchmark (e.g. GIAB HG002 T2T SVs with truvari) instead.
- Data is short-read or non-HiFi — sawfish, pbsv, and this HiFi-tuned concordance workflow are designed for mapped HiFi sequencing reads.
- You only have a single sample — sawfish joint-call requires discover results from each sample entering the joint step, and pedigree concordance needs multiple related samples.

## Inputs

- Seven CEPH-1463 HiFi WGS read sets (BAM/CCS reads)
- human_GRCh38_no_alt_analysis_set reference FASTA
- Known CEPH-1463 trio/family inheritance pattern from Kronenberg et al. 2024
- Per-sample mapped BAMs from pbmm2

## Outputs

- sawfish discover output directories (per-sample SV/CNV evidence)
- Joint-genotyped SV VCFs from sawfish, pbsv, and Sniffles2
- ceph1463_sv_genotype_concordance.tsv with per-caller total calls, concordant-call counts, and concordance proportions

## How to apply

Map each sample's HiFi reads to the human_GRCh38_no_alt_analysis_set reference with pbmm2 (align, --preset CCS, --sort). Run sawfish discover once per mapped BAM, then sawfish joint-call with all seven discover output directories so duplicate SV haplotypes are merged and SVs are jointly genotyped across samples. Run pbsv and Sniffles2 on the same mapped BAMs as comparator state-of-the-art callers. Classify every called SV as pedigree-concordant or not by checking trio/family genotypes against the known CEPH-1463 inheritance pattern from Kronenberg et al. 2024, then apply caller-specific quality re-scoring: sawfish high-quality concordance at GQ>=40, Sniffles2 at GQ>=10, pbsv unfiltered. Tabulate per-caller total calls, concordant counts, and concordance proportions into ceph1463_sv_genotype_concordance.tsv.

## Related tools

- **sawfish** (Primary caller: per-sample discover, then joint-call to merge duplicate SV haplotypes and jointly genotype SVs across the seven pedigree samples) — https://github.com/PacificBiosciences/sawfish
- **pbmm2** (Maps HiFi CCS reads to human_GRCh38_no_alt_analysis_set with --preset CCS and --sort before SV calling)
- **pbsv** (Comparator state-of-the-art long-read SV caller run on the same mapped samples) — https://github.com/PacificBiosciences/pbsv
- **Sniffles2** (Comparator state-of-the-art long-read SV caller, filtered at GQ>=10 for concordance comparison)

## Examples

```
pbmm2 --log-level INFO align -j 16 --preset CCS --sample HG002 --sort --unmapped
```

## Evaluation signals

- sawfish concordance proportion should reach ~81.2% unfiltered, with 9160 more pedigree-concordant SVs than either pbsv or Sniffles2
- sawfish GQ>=40 concordance should reach ~87.8% while still calling nearly 5000 more pedigree-concordant SVs than other callers
- pbsv unfiltered concordance should be ~77.7%; Sniffles2 at GQ>=10 should be ~85.2% with 8348 SVs retained
- sawfish should show the highest proportion of concordant calls among the three callers
- Summary table matches Supplementary Table S5 in per-caller total calls, concordant counts, and proportions

## Limitations

- Pedigree concordance is only defined relative to the known CEPH-1463 inheritance pattern; it is not a substitute for a truth-set benchmark on unrelated samples.
- Caller-specific GQ thresholds (sawfish GQ>=40, Sniffles2 GQ>=10, pbsv unfiltered) are not symmetric, so filtered and unfiltered numbers must be reported side by side to avoid misleading comparisons.
- sawfish joint-call requires running discover once per sample before that sample enters the joint-call step; skipping per-sample discover breaks the workflow.

## Evidence

- [abstract] Joint-genotyping seven CEPH-1463 samples with sawfish yields over 9000 more pedigree-concordant calls than comparators: "sawfish has over 9000 more pedigree-concordant calls than other state-of-the-art SV callers, with the highest proportion of concordant SVs (81%)."
- [abstract] Sawfish quality-based selection raises concordance to ~88%: "Sawfish's quality model enables selection for an even higher proportion of concordant SVs (88%), while still calling nearly 5000 more pedigree-concordant SVs than other callers."
- [methods] discover must run per sample before joint-call: "`discover` - The discover step needs to be run once on each sample prior to entering that sample in the the joint-call step."
- [methods] pbmm2 alignment invocation used in the workflow: "pbmm2 --log-level INFO align -j 16 --preset CCS --sample HG002 --sort --unmapped"
- [methods] joint-call merges and genotypes SVs across samples: "`joint-call` - Given the 'discover' command results from each sample, the joint call step merges and genotypes SVs, calls CNVs and unifies SV/CNV calls"
