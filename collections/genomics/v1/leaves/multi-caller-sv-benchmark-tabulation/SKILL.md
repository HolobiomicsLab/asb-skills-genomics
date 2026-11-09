---
name: multi-caller-sv-benchmark-tabulation
description: Use when HiFi WGS read sets from related/pedigree samples are available
  and you need to compare SV callers by pedigree-concordance, e.g. joint-genotyping
  seven CEPH-1463 samples with sawfish vs Sniffles2 and pbsv.
when_to_use_negative:
- Input is a single sample with no pedigree structure — pedigree concordance cannot
  be assessed and GIAB truth-set benchmarks (e.g. truvari bench against the HG002
  T2T or CMRG SV truth sets) are the appropriate evaluation route.
- Reads are short-read or non-CCS data — sawfish, pbsv and this concordance workflow
  are built for mapped HiFi sequencing reads.
edam_operation: http://edamontology.org/operation_3227
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3175
tools:
- name: sawfish
  role: 'Primary SV caller: per-sample ''discover'' then ''joint-call'' across seven
    samples to merge duplicate SV haplotypes and jointly genotype SVs'
  repo: https://github.com/PacificBiosciences/sawfish
- name: pbmm2
  role: Maps HiFi reads to human_GRCh38_no_alt_analysis_set with --preset CCS and
    --sort before SV calling
- name: pbsv
  role: Comparator state-of-the-art SV caller run on the same mapped samples (unfiltered
    concordance baseline)
- name: Sniffles2
  role: Comparator state-of-the-art SV caller; concordance recomputed at GQ>=10 (8348
    SVs retained)
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1093/bioinformatics/btaf136
    title: 'Sawfish: improving long-read structural variant discovery and genotyping
      with local haplotype modeling'
  source_content_hash: sha256:e54833e07a695120be8460a2e53d2d124732fec91ebb4e3a251ee478027521e0
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
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3175
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8181818181818182
n_claims: 11
---

# multi-caller-sv-benchmark-tabulation

## Summary

Reproduces a multi-caller structural variant (SV) genotyping benchmark by calling SVs with sawfish, Sniffles2, and pbsv on the same mapped samples, classifying calls as pedigree-concordant against the CEPH-1463 inheritance pattern, and tabulating per-caller totals, concordant counts, and concordance proportions.

## When to use

Use when HiFi WGS read sets from related/pedigree samples are available and you need to compare SV callers by pedigree-concordance, e.g. joint-genotyping seven CEPH-1463 samples with sawfish vs Sniffles2 and pbsv.

## When NOT to use

- Input is a single sample with no pedigree structure — pedigree concordance cannot be assessed and GIAB truth-set benchmarks (e.g. truvari bench against the HG002 T2T or CMRG SV truth sets) are the appropriate evaluation route.
- Reads are short-read or non-CCS data — sawfish, pbsv and this concordance workflow are built for mapped HiFi sequencing reads.

## Inputs

- Seven CEPH-1463 HiFi WGS read sets (PacBio CCS/HiFi BAMs)
- human_GRCh38_no_alt_analysis_set reference FASTA
- Known CEPH-1463 inheritance pattern (trio/family pedigree genotypes from Kronenberg et al. 2024)

## Outputs

- Per-caller VCFs of joint-genotyped SVs (sawfish, pbsv, Sniffles2)
- Summary table ceph1463_sv_genotype_concordance.tsv with per-caller total calls, concordant-call counts, and concordance proportions

## How to apply

Map each HiFi WGS read set to human_GRCh38_no_alt_analysis_set with pbmm2 (align, --preset CCS, --sort). Run 'sawfish discover' per mapped sample, then 'sawfish joint-call' across all seven discover output directories to merge duplicate SV haplotypes and jointly genotype SVs; run pbsv and Sniffles2 on the same mapped BAMs as state-of-the-art comparators. Classify each called SV as pedigree-concordant by checking trio/family genotypes against the known CEPH-1463 inheritance pattern from Kronenberg et al. 2024. Recompute sawfish high-quality concordance using the VCF GQ field at GQ>=40 and Sniffles2 concordance at GQ>=10, leaving pbsv unfiltered. Tabulate per-caller total calls, concordant-call counts, and concordance proportions into a TSV (ceph1463_sv_genotype_concordance.tsv) for direct comparison against Supplementary Table S5.

## Related tools

- **sawfish** (Primary SV caller: per-sample 'discover' then 'joint-call' across seven samples to merge duplicate SV haplotypes and jointly genotype SVs) — https://github.com/PacificBiosciences/sawfish
- **pbmm2** (Maps HiFi reads to human_GRCh38_no_alt_analysis_set with --preset CCS and --sort before SV calling)
- **pbsv** (Comparator state-of-the-art SV caller run on the same mapped samples (unfiltered concordance baseline))
- **Sniffles2** (Comparator state-of-the-art SV caller; concordance recomputed at GQ>=10 (8348 SVs retained))

## Examples

```
pbmm2 align --preset CCS --sort sample.hifi_reads.bam human_GRCh38_no_alt_analysis_set.fasta sample.bam && sawfish discover sample.bam && sawfish joint-call -o sawfish_joint/ sample1.discover/ ... sample7.discover/
```

## Evaluation signals

- sawfish concordance proportion reproduces the reported 81.2% (rounds to the abstract's 81%)
- sawfish yields 9160 more pedigree-concordant SVs than either Sniffles2 or pbsv (i.e. the 'over 9000 more' abstract claim)
- sawfish at GQ>=40 reaches 87.8% concordance (~88% abstract claim) while still calling nearly 5000 more concordant SVs than comparators
- pbsv unfiltered concordance equals 77.7% and Sniffles2 at GQ>=10 equals 85.2% with 8348 SVs retained
- Summary table values match Supplementary Table S5 row-for-row

## Limitations

- Quality-filtered comparisons use different GQ thresholds per caller (sawfish GQ>=40, Sniffles2 GQ>=10, pbsv unfiltered), so filtered concordance figures are not strictly like-for-like.
- Assessment depends on the accuracy of the known CEPH-1463 inheritance pattern from Kronenberg et al. 2024; pedigree misassignments propagate into concordance counts.
- Truvari-based assessments required consolidation (bench/refine with MAFFT) for the T2T SV benchmark, and bcftools filtering to SVTYPE records plus autosome restriction were needed for confident regions — naive comparisons without these steps are not reproducible.

## Evidence

- [abstract] joint-genotyping seven CEPH-1463 samples: sawfish has over 9000 more pedigree-concordant calls than other state-of-the-art SV callers, with the highest proportion of concordant SVs (81%): "sawfish has over 9000 more pedigree-concordant calls than other state-of-the-art SV callers, with the highest proportion of concordant SVs (81%)."
- [abstract] Quality-model selection yields higher concordance: "Sawfish's quality model enables selection for an even higher proportion of concordant SVs (88%), while still calling nearly 5000 more pedigree-concordant SVs than other callers."
- [methods] Two-step sawfish workflow: discover then joint-call: "`discover` - The discover step needs to be run once on each sample prior to entering that sample in the the joint-call step."
- [methods] Joint-call merges and genotypes SVs across samples: "`joint-call` - Given the 'discover' command results from each sample, the joint call step merges and genotypes SVs, calls CNVs and unifies SV/CNV calls"
- [methods] pbmm2 alignment command used for mapping: "pbmm2 --log-level INFO align -j 16 --preset CCS --sample HG002 --sort --unmapped"
- [readme] Sawfish definition as joint SV/CNV caller for HiFi reads: "Sawfish is a joint structural variant (SV) and copy number variant (CNV) caller for mapped HiFi sequencing reads."
