---
name: allelic-dmr-detection
description: Use when nanopore reads carry haplotype tags (maternal/paternal) plus
  per-read CpG methylation calls, and the goal is to identify allele-specific methylation
  or allelic DMRs between homologous chromosomes.
when_to_use_negative:
- Reads are not haplotype-phased — without parental haplotype tags there is no basis
  for allelic aggregation and DMR calling between homologs.
- Data come from older R9.4 pores where the DeepBAM 9-mer R10.4-trained model is not
  applicable; use the R9.4-based allele-specific DMR sets as reference instead.
- The goal is single-molecule methylation calling accuracy benchmarking rather than
  detecting regions of allelic methylation difference.
edam_operation: http://edamontology.org/operation_3207
edam_topics:
- http://edamontology.org/topic_3674
- http://edamontology.org/topic_3295
tools:
- name: DeepBAM
  role: Calls per-read per-site CpG methylation_rate values on phased pod5 reads via
    extract_and_call_mods
  repo: https://github.com/huicongyao/DeepBAM
- name: DSS
  role: Beta-binomial framework for detecting allelic DMRs between maternal and paternal
    haplotype methylation tracks
- name: Dorado
  role: Baseline methylation caller whose heterogeneous cross-dataset performance
    motivates using DeepBAM for allelic quantification
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/bib/bbae413
    title: 'DeepBAM: a high-accuracy single-molecule CpG methylation detection tool
      for Oxford nanopore sequencing'
  source_content_hash: sha256:1ab611263e176c3eae4f92f5433f23cf1115d62458d23207a7c36e6acddfb398
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-4.0
    url: https://github.com/xiaochuanle/DeepBAM
  repo_url: https://github.com/huicongyao/DeepBAM
  tools:
  - DSS
  - DeepBAM
  - Dorado
  - libtorch
  edam_operation: http://edamontology.org/operation_3207
  edam_topics:
  - http://edamontology.org/topic_3674
  - http://edamontology.org/topic_3295
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 12
---

# allelic-dmr-detection

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Detect allele-specific differentially methylated regions (DMRs) by aggregating DeepBAM per-read CpG methylation calls on haplotype-phased nanopore reads into maternal/paternal methylation frequency tracks and testing each region with the DSS beta-binomial framework.

## When to use

Use when nanopore reads carry haplotype tags (maternal/paternal) plus per-read CpG methylation calls, and the goal is to identify allele-specific methylation or allelic DMRs between homologous chromosomes.

## When NOT to use

- Reads are not haplotype-phased — without parental haplotype tags there is no basis for allelic aggregation and DMR calling between homologs.
- Data come from older R9.4 pores where the DeepBAM 9-mer R10.4-trained model is not applicable; use the R9.4-based allele-specific DMR sets as reference instead.
- The goal is single-molecule methylation calling accuracy benchmarking rather than detecting regions of allelic methylation difference.

## Inputs

- haplotype-tagged HG002 nanopore pod5 reads
- phased BAM with parental haplotype tags (HP tag)
- reference genome FASTA
- DeepBAM 9-mer traced_script_module model (kmer_size 51, motif CG)
- benchmark methylation frequency tracks from PacBio CCS and nanopore R9.4 (for validation)

## Outputs

- per-read per-site tsv of methylation_rate CpG calls
- maternal and paternal CpG methylation frequency tracks
- allelic DMR list (~47,282 regions with >= 20% methylation difference, DSS P < 0.001)
- correlation statistics vs CCS benchmark (maternal/paternal)
- phased methylation plots at PEG3, GNAS, L1HS, AluYa5

## How to apply

Run DeepBAM `extract_and_call_mods` on haplotype-tagged pod5 reads with the phased BAM and reference genome (9-mer traced_script_module, kmer_size 51, motif CG, batch_size 1024, num_workers 10, sub_thread_per_worker 4) to emit a per-read per-site tsv of methylation_rate calls. Aggregate per-read calls by parental haplotype tag to build maternal and paternal CpG methylation frequency tracks. Detect allelic DMRs between the two haplotypes with the DSS beta-binomial framework at P < 0.001 and methylation difference > 0.2, retaining DMRs with >= 20% allelic methylation difference (~47,282 expected on HG002). Validate by correlating haplotype methylation frequencies against a PacBio CCS benchmark (expected ~0.94 maternal / 0.92 paternal) and checking > 80% overlap with CCS- and R9.4-based DMR sets, then plot phased patterns at imprinted loci (PEG3, GNAS) and repeats (L1HS, AluYa5).

## Related tools

- **DeepBAM** (Calls per-read per-site CpG methylation_rate values on phased pod5 reads via extract_and_call_mods) — https://github.com/huicongyao/DeepBAM
- **DSS** (Beta-binomial framework for detecting allelic DMRs between maternal and paternal haplotype methylation tracks)
- **Dorado** (Baseline methylation caller whose heterogeneous cross-dataset performance motivates using DeepBAM for allelic quantification)

## Examples

```
DeepBAM extract_and_call_mods --pod5 hg002_phased.pod5 --bam hg002_phased.bam --ref GRCh38.fa --traced_script_module DeepBAM_9mer.pt --kmer_size 51 --motif CG --batch_size 1024 --num_workers 10 --sub_thread_per_worker 4
```

## Evaluation signals

- Allelic DMR count is approximately 47,282 at the >= 20% allelic methylation difference threshold (DSS P < 0.001, difference > 0.2).
- Maternal and paternal haplotype methylation frequency correlations with the PacBio CCS benchmark reach ~0.94 and ~0.92.
- Allelic DMR set overlaps > 80% with both CCS- and R9.4-based allele-specific DMR reference sets.
- Whole-genome methylation frequency tracks correlate > 0.95 with BS-seq on most datasets.
- Phased methylation plots at PEG3 and GNAS show the expected imprinted allele-specific patterns; L1HS and AluYa5 show transposable-element allelic methylation.

## Limitations

- DeepBAM performance differs significantly across heterogeneous human and plant datasets, so thresholds calibrated on HG002 may not transfer directly to other samples.
- The publication reference in the repository is a placeholder and no software version or release identifier for DeepBAM is stated.
- No accession IDs or hosting locations are given for the plant datasets and human cell line datasets used to train/test the model.
- Quantitative speed/memory comparisons are deferred to a supplementary table not present in the text, and ongoing optimization of DeepBAM is acknowledged as necessary.

## Evidence

- [other] DeepBAM-based allelic methylation quantification on phased HG002 reads achieved high correlations with CCS and nanopore R9.4 profiles: [evidence span withheld — anchor status: not_found]
- [abstract] DeepBAM enables allele-specific methylation detection: "It enables unraveling allele-specific methylation patterns, including regions of transposable elements"
- [methods] DeepBAM extract_and_call_mods workflow step: "### DeepBAM: Extract and Call Modifications"
- [readme] DeepBAM repo update notes on faster calling: "We've enhanced the feature extraction mechanism, resulting in significantly faster processing times for modification calling"
- [abstract] Dorado cross-dataset variability motivating DeepBAM: "comprehensive evaluations, which showed that Dorado performed significantly different across datasets"
