---
name: caller-vcf-comparison-analysis
description: Use when you have SV/indel VCFs from multiple callers (e.g. dysgu, manta,
  delly, lumpy, strelka, gatk) and a reference benchmark VCF, and need per-event-class
  precision/recall/F1 to rank callers.
when_to_use_negative:
- The input is a single caller's VCF and no reference truth set exists — svbench cannot
  compute precision/recall without a reference VCF
- You are evaluating small-variant (SNP/indel <30 bp) calling accuracy on a non-benchmark
  sample; CHM-eval/syndip-style small-variant evaluation is a different problem
- You need long-read caller benchmarks (sniffles, cuteSV, severus, sawfish) rather
  than paired-end caller comparison
edam_operation: http://edamontology.org/operation_3227
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_0625
tools:
- name: dysgu
  role: SV caller under test, run in paired-end mode with default args to produce
    svs.vcf
  repo: https://github.com/kcleal/dysgu
- name: manta
  role: PE SV-caller baseline run with default settings
- name: delly
  role: PE SV-caller baseline run with default settings
- name: lumpy
  role: PE SV-caller baseline run with default settings
- name: strelka
  role: Indel-calling baseline down to 30 bp
- name: gatk
  role: Indel-calling baseline down to 30 bp
- name: svbench
  role: Evaluation tool computing precision/recall/F1 of query VCFs vs the GIAB reference
    VCF
  repo: https://github.com/kcleal/svbench
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/nar/gkac039
    title: 'Dysgu: efficient structural variant calling using short or long reads'
  source_content_hash: sha256:33fb0d7466bf401f1a7d0fade7f11558ae3934952fb60885463e809e02f089ed
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/kcleal/dysgu
  tools:
  - delly
  - dysgu
  - gatk
  - lumpy
  - manta
  - strelka
  - svbench
  edam_operation: http://edamontology.org/operation_3227
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_0625
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9090909090909091
n_claims: 11
---

# caller-vcf-comparison-analysis

## Summary

Benchmark structural-variant caller VCFs against a GIAB truth set using svbench, computing per-caller precision, recall and F1 for deletions and insertions stratified by variant class and confidence region (e.g. HG002 Tier 1).

## When to use

Use when you have SV/indel VCFs from multiple callers (e.g. dysgu, manta, delly, lumpy, strelka, gatk) and a reference benchmark VCF, and need per-event-class precision/recall/F1 to rank callers.

## When NOT to use

- The input is a single caller's VCF and no reference truth set exists — svbench cannot compute precision/recall without a reference VCF
- You are evaluating small-variant (SNP/indel <30 bp) calling accuracy on a non-benchmark sample; CHM-eval/syndip-style small-variant evaluation is a different problem
- You need long-read caller benchmarks (sniffles, cuteSV, severus, sawfish) rather than paired-end caller comparison

## Inputs

- Sorted/indexed BAM/CRAM alignments (e.g. GIAB AshkenazimTrio HG002_NA24385_son Illumina paired-end data)
- Indexed reference genome FASTA
- Reference truth-set VCF (e.g. NIST_SVs_Integration_v0.6 GIAB benchmark)
- Query VCFs from each SV caller (dysgu, manta, delly, lumpy, strelka, gatk)
- High-confidence Tier 1 region BED file for --include

## Outputs

- Per-caller VCF call sets (e.g. svs.vcf from dysgu run)
- Per-caller, per-event-class precision, recall and F1 values compiled into benchmark_table.tsv
- Optionally TP/FP/FN call lists via svbench --tp/--fp/--fn

## How to apply

Call SVs/indels with each tool on the same aligned BAMs (for paired-end data run dysgu in PE mode, e.g. 'dysgu --mode pe', with default args, and run manta, delly and lumpy with default settings; include strelka and gatk for indels down to 30 bp). Evaluate every call set with svbench against the NIST_SVs_Integration_v0.6 GIAB benchmark, keeping only PASS variants, using minimum event sizes of 30/50 bp, a 1000 bp interval-overlap criterion and a 15% size-similarity threshold. Restrict evaluation both to high-confidence Tier 1 regions (via the --include BED) and to all genomic regions, and stratify results per caller and per event type (deletions vs insertions). Tabulate precision, recall and F1 per caller into a table (benchmark_table.tsv) and compare, e.g. dysgu's reported HG002 Tier 1 20x deletion precision of 95.7% and recall of 62.2% (23.7% for insertions).

## Related tools

- **dysgu** (SV caller under test, run in paired-end mode with default args to produce svs.vcf) — https://github.com/kcleal/dysgu
- **manta** (PE SV-caller baseline run with default settings)
- **delly** (PE SV-caller baseline run with default settings)
- **lumpy** (PE SV-caller baseline run with default settings)
- **strelka** (Indel-calling baseline down to 30 bp)
- **gatk** (Indel-calling baseline down to 30 bp)
- **svbench** (Evaluation tool computing precision/recall/F1 of query VCFs vs the GIAB reference VCF) — https://github.com/kcleal/svbench

## Examples

```
svbench --pass-only --include Tier1.bed --min-size-ref 30 --min-size-query 30 NIST_SVs_Integration_v0.6.vcf dysgu.svs.vcf manta.vcf delly.vcf lumpy.vcf
```

## Evaluation signals

- On HG002 Tier 1 SVs at 20x coverage, the top-ranked caller should match the paper's result: dysgu with the highest deletion precision (95.7%) and highest recall for deletions (62.2%) and insertions (23.7%)
- Dysgu should call the largest number of true deletions plus insertions (n = 3913), about 727 more than the next best caller manta (n = 3186)
- All call sets must be PASS-filtered and evaluated with the same svbench settings (min sizes 30/50 bp, 1000 bp overlap, 15% size similarity) or the per-caller comparison is invalid
- Dysgu's F1 should be ~4.7% higher than manta for deletions and ~13.1% higher for insertions on HG002 Tier 1 at 20x
- TP + FN counts for each caller should sum to the same reference-set total within a given region stratum and minimum-size filter

## Limitations

- No software versions are stated for any caller in the paper, so exact version pinning must be chosen by the practitioner
- Insertion size accuracy of dysgu was not shown ('data not shown'); calculated insertion sizes are expected to be less accurate than de novo assembly-based callers such as manta and gatk
- Svbench defaults differ from the paper's settings (default slop is 250 bp; the paper uses a 1000 bp interval-overlap and 15% size-similarity criterion), so parameters must be set explicitly

## Evidence

- [results] Dysgu had the highest precision for deletion calls (95.7%), as well as the highest recall for deletions (62.2%) and insertions (23.7%): "Dysgu had the highest precision for deletion calls (95.7%), as well as the highest recall for deletions (62.2%) and insertions (23.7%)"
- [results] Performance was compared to the popular SV callers manta, delly, and lumpy: "Performance was compared to the popular SV callers manta (9), delly (42), and lumpy (43)"
- [readme] SVBench is a python tool/library for benchmarking variant calls vs a reference dataset.: [evidence span withheld — anchor status: external_doc]
- [supplementary] Analysis scripts used to reproduce results found in this paper can be found under https://github.com/kcleal/svbench.: "Analysis scripts used to reproduce results found in this paper can be found under https://github.com/kcleal/svbench."
