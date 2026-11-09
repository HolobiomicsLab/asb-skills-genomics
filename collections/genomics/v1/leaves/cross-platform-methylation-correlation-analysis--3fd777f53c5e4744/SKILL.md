---
name: cross-platform-methylation-correlation-analysis
description: Use when nanopore R10.4 methylation calls must be benchmarked against
  BS-seq, PacBio CCS, or R9.4 reference methylation profiles, or when allele-specific
  methylation patterns need cross-platform validation.
when_to_use_negative:
- Data are not CpG methylation calls from ONT R10.4 nanopore reads — DeepBAM's model
  is trained for CpG motif on R10.4 and only a few tools (DeepMod2, Dorado) support
  R10.4, so earlier pore chemistries or other motifs need different callers
- No orthogonal benchmark (BS-seq, CCS, or R9.4 methylation profiles) exists to correlate
  against — the validation framework has nothing to compare to
- Reads are unphased and the goal is allele-specific methylation — haplotype aggregation
  and allelic DMR detection require haplotype-tagged reads
edam_operation: http://edamontology.org/operation_3204
edam_topics:
- http://edamontology.org/topic_3674
- http://edamontology.org/topic_3295
tools:
- name: DeepBAM
  role: calls single-molecule CpG methylation states from ONT R10.4 pod5 reads via
    `extract_and_call_mods`, producing per-read per-site methylation_rate TSVs
  repo: github.com/huicongyao/DeepBAM
- name: DSS
  role: beta-binomial framework for detecting allelic DMRs between maternal and paternal
    haplotype tracks (P < 0.001, methylation difference > 0.2)
- name: Dorado
  role: ONT's official methylation caller used as the comparison baseline; DeepBAM
    outperformed it (98.47% AUROC average, up to 7.36% improvement)
- name: DeepMod2
  role: alternative R10.4-compatible methylation caller referenced as one of few compatible
    with the newer pore chemistry
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/bib/bbae413
    title: 'DeepBAM: a high-accuracy single-molecule CpG methylation detection tool
      for Oxford nanopore sequencing'
  source_content_hash: sha256:a94feadf466441dc09344c9f1c43cfe3ca2e30932bf4c8d7e575d5867c9f3324
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-4.0
    url: https://github.com/xiaochuanle/DeepBAM
  tools:
  - DSS
  - DeepBAM
  - DeepMod2
  - Dorado
  - libtorch
  edam_operation: http://edamontology.org/operation_3204
  edam_topics:
  - http://edamontology.org/topic_3674
  - http://edamontology.org/topic_3295
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8181818181818182
n_claims: 11
---

# cross-platform-methylation-correlation-analysis

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Validates nanopore CpG methylation calls by correlating whole-genome and haplotype-specific methylation frequencies against orthogonal benchmarks (BS-seq, PacBio CCS, R9.4 nanopore), then quantifying concordance of downstream differential methylation regions. Grounded in DeepBAM, which achieved >0.95 correlations with BS-seq on four of five datasets and 47,282 allelic DMRs on phased HG002 reads.

## When to use

Use when nanopore R10.4 methylation calls must be benchmarked against BS-seq, PacBio CCS, or R9.4 reference methylation profiles, or when allele-specific methylation patterns need cross-platform validation.

## When NOT to use

- Data are not CpG methylation calls from ONT R10.4 nanopore reads — DeepBAM's model is trained for CpG motif on R10.4 and only a few tools (DeepMod2, Dorado) support R10.4, so earlier pore chemistries or other motifs need different callers
- No orthogonal benchmark (BS-seq, CCS, or R9.4 methylation profiles) exists to correlate against — the validation framework has nothing to compare to
- Reads are unphased and the goal is allele-specific methylation — haplotype aggregation and allelic DMR detection require haplotype-tagged reads

## Inputs

- Haplotype-phased BAM of ONT R10.4 nanopore reads plus matching pod5 signal files
- DeepBAM per-read per-site TSV of methylation_rate calls (motif CG)
- Reference whole-genome BS-seq methylation frequency tracks
- Benchmark allele-specific methylation sets from PacBio CCS and nanopore R9.4
- Reference genome for phasing-aware site lookup

## Outputs

- Per-site CpG methylation frequency tracks per platform (maternal and paternal haplotypes separately)
- Correlation coefficients between nanopore and benchmark methylation frequencies (e.g. 0.94/0.92 vs CCS)
- List of allelic DMRs (~47,282 with >= 20% methylation difference at P < 0.001, diff > 0.2 via DSS)
- Overlap statistics of DeepBAM DMR set against CCS- and R9.4-derived DMR sets (> 80%)
- Phased methylation plots at loci of interest (e.g. PEG3, GNAS, L1HS, AluYa5)

## How to apply

Run DeepBAM `extract_and_call_mods` (9-mer traced_script_module, kmer_size 51, motif CG, batch_size 1024) on nanopore pod5 reads to emit per-read per-site methylation_rate calls, then aggregate per-site frequencies. Compute per-site or per-region correlation (Pearson) between the nanopore methylation frequency track and each benchmark platform track — DeepBAM's design target is >0.95 correlation with BS-seq on most datasets, exceeding Dorado in all instances. For allele-specific analysis, aggregate per-read calls by parental haplotype tag from a phased BAM to build maternal/paternal CpG tracks, call allelic DMRs with the DSS beta-binomial framework at P < 0.001 and methylation difference > 0.2, and retain DMRs with >= 20% allelic difference. Judge success by correlation coefficients against the benchmark (expected 0.94 maternal / 0.92 paternal vs CCS) and by DMR set overlap (> 80% with CCS and R9.4 reference DMR sets).

## Related tools

- **DeepBAM** (calls single-molecule CpG methylation states from ONT R10.4 pod5 reads via `extract_and_call_mods`, producing per-read per-site methylation_rate TSVs) — github.com/huicongyao/DeepBAM
- **DSS** (beta-binomial framework for detecting allelic DMRs between maternal and paternal haplotype tracks (P < 0.001, methylation difference > 0.2))
- **Dorado** (ONT's official methylation caller used as the comparison baseline; DeepBAM outperformed it (98.47% AUROC average, up to 7.36% improvement))
- **DeepMod2** (alternative R10.4-compatible methylation caller referenced as one of few compatible with the newer pore chemistry)

## Examples

```
DeepBAM extract_and_call_mods --input phased_hg002.pod5 --bam phased.bam --ref genome.fa --model traced_script_module --kmer_size 51 --motif CG --batch_size 1024
```

## Evaluation signals

- Whole-genome methylation frequencies correlate with BS-seq at coefficients > 0.95 on most datasets, outperforming Dorado
- Maternal/paternal haplotype methylation frequencies correlate with PacBio CCS benchmarks at ~0.94/0.92
- Allelic DMR count is approximately 47,282 under the >= 20% difference threshold
- DMR set overlaps > 80% with CCS- and R9.4-based allele-specific DMR sets
- Single-molecule AUROC stays >= 0.956 (DeepBAM averaged 98.47% with up to 7.36% improvement over Dorado)

## Limitations

- Dorado's methylation calling performance differs significantly across heterogeneous human and plant datasets, so cross-platform concordance is dataset-dependent
- No software version or release identifier for DeepBAM is stated, and no accession IDs are given for the human cell line or plant benchmark datasets
- Quantitative speed/memory comparisons are deferred to supplementary material not present in the text
- Ongoing optimization of DeepBAM is acknowledged as necessary, implying incomplete current performance

## Evidence
