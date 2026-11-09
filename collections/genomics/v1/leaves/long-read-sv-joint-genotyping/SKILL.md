---
name: long-read-sv-joint-genotyping
description: Use when multiple mapped HiFi (CCS) sample BAMs are available and the
  goal is jointly genotyped SV calls whose inheritance can be checked against a known
  pedigree, or when comparing SV caller concordance.
when_to_use_negative:
- Only a single sample is available — sawfish's joint-call step requires per-sample
  discover outputs and joint genotyping loses its advantage; single-sample calling
  is more appropriate
- Input reads are short-read or non-HiFi data — sawfish is designed for mapped HiFi
  sequencing reads (pbmm2 --preset CCS)
- The goal is somatic SV calling — sawfish discovers germline structural variants
  only
edam_operation: http://edamontology.org/operation_3227
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3175
tools:
- name: sawfish
  role: 'Primary SV/CNV caller: ''discover'' per sample then ''joint-call'' to merge
    SV haplotypes and jointly genotype across samples'
  repo: https://github.com/PacificBiosciences/sawfish
- name: pbmm2
  role: Maps HiFi CCS reads to the GRCh38 no-alt analysis-set reference prior to SV
    discovery
- name: pbsv
  role: Comparator state-of-the-art long-read SV caller for pedigree-concordance benchmarking
  repo: https://github.com/PacificBiosciences/pbsv
- name: Sniffles2
  role: Comparator state-of-the-art long-read SV caller, filtered at GQ>=10 in concordance
    assessment
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1093/bioinformatics/btaf136
    title: 'Sawfish: improving long-read structural variant discovery and genotyping
      with local haplotype modeling'
  source_content_hash: sha256:b7889238fa5d378f39b37ca830226f8eb7e73cd16e71766a8619a76a71693ac9
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
yes_rate: 0.6666666666666666
n_claims: 6
---

# long-read-sv-joint-genotyping

## Summary

Jointly genotype structural variants across multiple HiFi-sequenced samples using sawfish's two-step discover/joint-call workflow with local SV haplotype modeling, and benchmark pedigree concordance against Sniffles2 and pbsv on the CEPH-1463 pedigree.

## When to use

Use when multiple mapped HiFi (CCS) sample BAMs are available and the goal is jointly genotyped SV calls whose inheritance can be checked against a known pedigree, or when comparing SV caller concordance.

## When NOT to use

- Only a single sample is available — sawfish's joint-call step requires per-sample discover outputs and joint genotyping loses its advantage; single-sample calling is more appropriate
- Input reads are short-read or non-HiFi data — sawfish is designed for mapped HiFi sequencing reads (pbmm2 --preset CCS)
- The goal is somatic SV calling — sawfish discovers germline structural variants only

## Inputs

- Per-sample HiFi WGS read sets (e.g. seven CEPH-1463 samples, CCS BAM/FASTQ)
- human_GRCh38_no_alt_analysis_set reference FASTA
- Known pedigree/trio inheritance pattern for the samples (CEPH-1463, Kronenberg et al. 2024)
- sawfish discover output directories (one per sample)

## Outputs

- Jointly genotyped multi-sample SV VCF from sawfish joint-call
- Comparator SV VCFs from pbsv and Sniffles2
- Concordance summary table (e.g. ceph1463_sv_genotype_concordance.tsv) with per-caller total calls, concordant-call counts, and concordance proportions

## How to apply

First map each sample's HiFi WGS reads to human_GRCh38_no_alt_analysis_set with pbmm2 (pbmm2 align --preset CCS --sort). Run 'sawfish discover' once per mapped sample BAM to scan for SV/CNV evidence via local sequence assembly, then run 'sawfish joint-call' with all seven discover output directories to merge duplicate SV haplotypes and jointly genotype SVs across samples while unifying SV/CNV calls. As comparators, call SVs on the same BAMs with pbsv and Sniffles2. Classify each called SV as pedigree-concordant or not by checking trio/family genotypes against the known CEPH-1463 inheritance pattern from Kronenberg et al. 2024. Optionally apply quality-based filtering — sawfish VCF GQ>=40 and Sniffles2 GQ>=10, leaving pbsv unfiltered — to recompute high-quality concordance. Tabulate per-caller total calls, concordant counts, and concordance proportions (e.g. to ceph1463_sv_genotype_concordance.tsv).

## Related tools

- **sawfish** (Primary SV/CNV caller: 'discover' per sample then 'joint-call' to merge SV haplotypes and jointly genotype across samples) — https://github.com/PacificBiosciences/sawfish
- **pbmm2** (Maps HiFi CCS reads to the GRCh38 no-alt analysis-set reference prior to SV discovery)
- **pbsv** (Comparator state-of-the-art long-read SV caller for pedigree-concordance benchmarking) — https://github.com/PacificBiosciences/pbsv
- **Sniffles2** (Comparator state-of-the-art long-read SV caller, filtered at GQ>=10 in concordance assessment)

## Examples

```
pbmm2 align --preset CCS --sort sample.hifi_reads.bam human_GRCh38_no_alt_analysis_set.fasta sample.sorted.bam && sawfish discover sample.sorted.bam sample_discover/ && sawfish joint-call --output joint.vcf.gz sample1_discover/ ... sample7_discover/
```

## Evaluation signals

- sawfish reproduces ~81% (81.2%) pedigree-concordant SV proportion across the seven CEPH-1463 samples, with >9000 (9160) more concordant SVs than Sniffles2 or pbsv
- Sawfish concordance at GQ>=40 reaches 87.8% (88%), consistent with the paper's quality-model selection claim
- pbsv concordance is ~77.7% unfiltered and Sniffles2 ~85.2% after GQ>=10 filtering (8348 SVs retained), matching Supplementary Table S5
- Per-caller totals and concordance proportions in the summary TSV are internally consistent (proportion = concordant/total per caller)
- Each sample has exactly one successful 'sawfish discover' run before joint-call, and joint-call consumes all seven discover directories

## Limitations

- sawfish version used in the published analysis was 0.12.9; behavior of other versions may differ
- Quality-model filtering trades call count for concordance: GQ>=40 selection retains fewer calls while calling nearly 5000 more pedigree-concordant SVs than other callers
- Pedigree concordance is a proxy metric, not a truth-set validation; genome-level accuracy elsewhere was assessed against GIAB T2T HG002 and CMRG benchmarks with truvari 4.2.2
- VACmap is still unsupported in the companion tooling; problematic alignments from any source may lack clear error messages

## Evidence

- [abstract] joint-genotyping seven CEPH-1463 samples: sawfish has over 9000 more pedigree-concordant calls than other state-of-the-art SV callers, with the highest proportion of concordant SVs (81%): "sawfish has over 9000 more pedigree-concordant calls than other state-of-the-art SV callers, with the highest proportion of concordant SVs (81%)."
- [methods] two-step workflow: "`discover` - The discover step needs to be run once on each sample prior to entering that sample in the the joint-call step."
- [methods] joint-call definition: "`joint-call` - Given the 'discover' command results from each sample, the joint call step merges and genotypes SVs, calls CNVs and unifies SV/CNV calls"
- [methods] read mapping step: "pbmm2 --log-level INFO align -j 16 --preset CCS --sample HG002 --sort --unmapped"
- [readme] sawfish scope: "It discovers germline structural variants from local sequence assembly and jointly genotypes these variants across multiple samples."
