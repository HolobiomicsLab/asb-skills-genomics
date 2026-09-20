---
name: repetitive-element-methylation-visualization
description: Use when nanopore R10.4 reads have been methylation-called and you need
  per-read or haplotype-resolved CpG methylation plots over transposable-element/repetitive
  loci (L1HS, AluYa5) or imprinted regions.
when_to_use_negative:
- Input is short-read BS-seq or array methylation data — this skill requires single-molecule
  nanopore signal, not converted reads
- Methylation motifs other than CpG (DeepBAM is a CpG-specific caller with motif CG)
- Allele-specific visualization on unphased reads lacking haplotype tags — haplotype-level
  tracks cannot be built
edam_operation: http://edamontology.org/operation_3208
edam_topics:
- http://edamontology.org/topic_3168
tools:
- name: DeepBAM
  role: Single-molecule CpG methylation calling on nanopore R10.4 pod5 reads via extract_and_call_mods,
    producing per-read methylation rates for plotting
  repo: https://github.com/huicongyao/DeepBAM.git
- name: DSS
  role: Beta-binomial detection of allelic DMRs between maternal and paternal haplotype
    methylation tracks (P < 0.001, difference > 0.2)
- name: Dorado
  role: Reference ONT methylation caller against which DeepBAM per-read performance
    and allele-specific DMR overlap are benchmarked
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/bib/bbae413
    title: 'DeepBAM: a high-accuracy single-molecule CpG methylation detection tool
      for Oxford nanopore sequencing'
  source_content_hash: sha256:c6f9d8e5e772de53e71b57764b4acf1ea13fbbb503e883a2a87fc1ef04dd6539
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-4.0
    url: https://github.com/xiaochuanle/DeepBAM
  repo_url: https://github.com/huicongyao/DeepBAM.git
  tools:
  - DSS
  - DeepBAM
  - Dorado
  - libtorch
  edam_operation: http://edamontology.org/operation_3208
  edam_topics:
  - http://edamontology.org/topic_3168
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 12
---

# repetitive-element-methylation-visualization

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Visualize allele-specific CpG methylation patterns at repetitive elements (e.g., L1HS LINE-1 and AluYa5 SINE) using single-molecule methylation calls from DeepBAM on Oxford Nanopore R10.4 reads, optionally split by parental haplotype.

## When to use

Use when nanopore R10.4 reads have been methylation-called and you need per-read or haplotype-resolved CpG methylation plots over transposable-element/repetitive loci (L1HS, AluYa5) or imprinted regions.

## When NOT to use

- Input is short-read BS-seq or array methylation data — this skill requires single-molecule nanopore signal, not converted reads
- Methylation motifs other than CpG (DeepBAM is a CpG-specific caller with motif CG)
- Allele-specific visualization on unphased reads lacking haplotype tags — haplotype-level tracks cannot be built

## Inputs

- Nanopore R10.4 pod5 read files
- Haplotype-phased BAM aligned to a reference genome (e.g., HG002 phased BAM)
- Reference genome FASTA
- DeepBAM 9-mer traced_script_module model (kmer_size 51, motif CG)

## Outputs

- Per-read per-site tsv of CpG methylation_rate calls
- Maternal and paternal haplotype CpG methylation frequency tracks
- Allelic DMR list (~47,282 DMRs at >= 20% methylation difference)
- Phased methylation plots at PEG3, GNAS, L1HS, and AluYa5 loci

## How to apply

Run DeepBAM's `extract_and_call_mods` on pod5 reads with a phased BAM and reference genome, using the 9-mer traced_script_module (kmer_size 51, motif CG, batch_size 1024, num_workers 10, sub_thread_per_worker 4) to emit a per-read per-site tsv of methylation_rate values. Aggregate calls by haplotype tag to build maternal/paternal CpG methylation frequency tracks, then detect allelic DMRs with the DSS beta-binomial framework (P < 0.001, methylation difference > 0.2, retaining DMRs with >= 20% allelic difference). Finally, plot phased methylation patterns at the imprinted loci PEG3 and GNAS and at the repeats L1HS and AluYa5 to expose allele-specific methylation, including hypermethylation of transposable elements. Rationale: DeepBAM's per-molecule calls on R10.4 data support single-molecule resolution that enables methylation assessment within repeat copies, which ensemble callers blur.

## Related tools

- **DeepBAM** (Single-molecule CpG methylation calling on nanopore R10.4 pod5 reads via extract_and_call_mods, producing per-read methylation rates for plotting) — https://github.com/huicongyao/DeepBAM.git
- **DSS** (Beta-binomial detection of allelic DMRs between maternal and paternal haplotype methylation tracks (P < 0.001, difference > 0.2))
- **Dorado** (Reference ONT methylation caller against which DeepBAM per-read performance and allele-specific DMR overlap are benchmarked)

## Examples

```
DeepBAM extract_and_call_mods --pod5 HG002.pod5 --bam HG002_phased.bam --ref genome.fa --kmer_size 51 --motif CG --batch_size 1024 --num_workers 10 --sub_thread_per_worker 4 --traced_script_module 9mer.traced_script_module.pt
```

## Evaluation signals

- Per-read per-site tsv contains a methylation_rate value for every CpG in the motif CG set (kmer_size 51)
- Whole-genome methylation frequencies from DeepBAM calls correlate > 0.95 with BS-seq on most datasets
- Single-molecule AUROC ~98.47% on average (up to 7.36% over Dorado) when re-validating calls against high-confidence sites
- Allelic DMR count is approximately 47,282 at >= 20% methylation difference, with > 80% overlap against CCS- and R9.4-based DMR sets
- Maternal/paternal methylation frequency correlations with PacBio CCS benchmarks reach 0.94/0.92

## Limitations

- Dorado methylation calling performance differs significantly across heterogeneous human and plant datasets, so DeepBAM's advantage must be checked per dataset rather than assumed
- Only the 9-mer modules have been updated/released so far; other kmer modules were pending ('We will soon release all the other kmer modules')
- Ongoing optimization of DeepBAM is acknowledged as necessary, implying incomplete current performance; no software version identifier is stated
- No accession IDs or hosting locations are given for the plant or human cell line R10.4 training/testing datasets

## Evidence
