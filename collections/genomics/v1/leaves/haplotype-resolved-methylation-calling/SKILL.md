---
name: haplotype-resolved-methylation-calling
description: Use when you have haplotype-phased ONT nanopore reads (e.g. haplotype-tagged
  HG002 pod5/BAM) and need allele-specific CpG methylation frequencies and allelic
  DMR detection.
when_to_use_negative:
- Reads are not haplotype-phased, so per-read calls cannot be split into parental
  methylation tracks
- Data are not ONT R10.4 nanopore reads or the target motif is not CpG/CG (DeepBAM
  was developed for CpG methylation states on mainstream R10.4 data)
- A benchmark comparison to Dorado or DeepSignal-style callers on non-human/plant
  R10.4 datasets is the goal rather than allelic analysis
tools:
- name: DeepBAM
  role: Single-molecule CpG methylation caller run via extract_and_call_mods on haplotype-tagged
    pod5 reads with a 9-mer traced_script_module
  repo: github.com/xiaochuanle/DeepBAM
- name: DSS
  role: Beta-binomial framework for detecting allelic DMRs between maternal and paternal
    methylation tracks (P < 0.001, difference > 0.2)
- name: Dorado
  role: ONT's official methylation-calling baseline/alternative whose R10.4 CpG model
    the article found performs inconsistently across datasets
- name: Dorado
  role: Used for basecalling of nanopore reads prior to methylation calling
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/bib/bbae413
    title: 'DeepBAM: a high-accuracy single-molecule CpG methylation detection tool
      for Oxford nanopore sequencing'
  source_content_hash: sha256:529d622ea362aab48a9a878d528f0eb8873a308213121e595b310ce583647d35
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
  - Dorado
  - libtorch
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.625
n_claims: 8
---

# haplotype-resolved-methylation-calling

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Calling CpG methylation per-read with DeepBAM on haplotype-phased Oxford Nanopore (R10.4) reads and aggregating calls by parental haplotype to detect allele-specific methylation and DMRs, including at imprinted loci and transposable elements.

## When to use

Use when you have haplotype-phased ONT nanopore reads (e.g. haplotype-tagged HG002 pod5/BAM) and need allele-specific CpG methylation frequencies and allelic DMR detection.

## When NOT to use

- Reads are not haplotype-phased, so per-read calls cannot be split into parental methylation tracks
- Data are not ONT R10.4 nanopore reads or the target motif is not CpG/CG (DeepBAM was developed for CpG methylation states on mainstream R10.4 data)
- A benchmark comparison to Dorado or DeepSignal-style callers on non-human/plant R10.4 datasets is the goal rather than allelic analysis

## Inputs

- Haplotype-tagged ONT pod5 reads (e.g. HG002)
- Haplotype-phased BAM aligned to a reference genome
- Reference genome FASTA
- 9-mer DeepBAM traced_script_module (CpG/CG motif)
- Benchmark methylation frequency sets (e.g. PacBio CCS, nanopore R9.4) for validation

## Outputs

- Per-read per-site TSV of methylation_rate CpG calls
- Maternal and paternal haplotype CpG methylation frequency tracks
- Allelic DMR set (>= 20% methylation difference, DSS P < 0.001; ~47,282 DMRs for HG002)
- Phased methylation plots at loci such as PEG3, GNAS, L1HS and AluYa5

## How to apply

Run DeepBAM's `extract_and_call_mods` on haplotype-tagged pod5 reads with the phased BAM and reference genome, using the 9-mer traced_script_module (motif CG, batch_size 1024, num_workers 10, sub_thread_per_worker 4) to emit a per-read per-site tsv of methylation_rate calls. Aggregate per-read calls by parental haplotype tag to build maternal and paternal CpG methylation frequency tracks. Detect allelic DMRs between the two haplotypes with the DSS beta-binomial framework using P < 0.001 and methylation difference > 0.2, retaining DMRs with >= 20% allelic methylation difference (expected ~47,282 DMRs for HG002). Validate by correlating maternal/paternal methylation frequencies against PacBio CCS benchmarks (expected 0.94/0.92) and checking > 80% DMR overlap with CCS- and R9.4-based reference sets. Rationale: DeepBAM provides more stable single-molecule CpG calling on R10.4 data than Dorado, whose performance varies significantly across human and plant datasets.

## Related tools

- **DeepBAM** (Single-molecule CpG methylation caller run via extract_and_call_mods on haplotype-tagged pod5 reads with a 9-mer traced_script_module) — github.com/xiaochuanle/DeepBAM
- **DSS** (Beta-binomial framework for detecting allelic DMRs between maternal and paternal methylation tracks (P < 0.001, difference > 0.2))
- **Dorado** (ONT's official methylation-calling baseline/alternative whose R10.4 CpG model the article found performs inconsistently across datasets)
- **Dorado** (Used for basecalling of nanopore reads prior to methylation calling)

## Examples

```
deepbam extract_and_call_mods --input hg002_haplotagged.pod5 --bam hg002_phased.bam --ref GRCh38.fa --model traced_script_module --kmer_size 9 --motif CG --batch_size 1024 --num_workers 10 --sub_thread_per_worker 4
```

## Evaluation signals

- DMR count is approximately 47,282 with >= 20% allelic methylation difference for HG002
- Maternal/paternal haplotype methylation correlations with PacBio CCS benchmarks reach 0.94/0.92
- Allelic DMR set overlaps > 80% with CCS- and R9.4-based allele-specific DMR sets
- Per-read per-site tsv of methylation_rate calls is emitted with all expected columns/sites
- Whole-genome methylation frequencies correlate > 0.95 with BS-seq on most datasets

## Limitations

- Dorado's methylation-calling model in officially supplied releases remains poorly tested and displays significantly different performance across heterogeneous human and plant datasets
- No software version or release identifier for DeepBAM is stated, and quantitative speed/memory comparisons are deferred to supplementary material not present in the text
- Ongoing optimization of DeepBAM is acknowledged as necessary, implying incomplete current performance; DeepBAM requires phased reads to resolve allelic patterns

## Evidence
