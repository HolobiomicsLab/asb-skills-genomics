---
name: giab-benchmark-region-stratification
description: Use when comparing SV/indel callers on GIAB-benchmarked samples (e.g.
  HG002) and you need region-stratified precision, recall and F1 per event class rather
  than a single genome-wide number.
when_to_use_negative:
- Evaluating long-read-only callers against GIAB v1.1 or CMRG truth sets — use the
  kcleal/SV_Benchmark_CMRG_GIAB or SV_benchmark_PacBio_HiFi pipelines instead
- Benchmarking small-variant (SNV/indel) callers where a distance-based evaluator
  like lh3/CHM-eval (syndip) is the appropriate tool
- When the input VCFs are the reference truth set itself and no caller comparison
  is needed
edam_operation: http://edamontology.org/operation_3197
edam_topics:
- http://edamontology.org/topic_0622
tools:
- name: svbench
  role: Evaluates caller VCFs against the GIAB reference VCF with PASS filtering,
    region --include BED, --slop interval padding, and min-size thresholds
  repo: https://github.com/kcleal/svbench
- name: dysgu
  role: Primary paired-end SV/indel caller under test ('--mode pe' default argument
    set)
  repo: https://github.com/kcleal/dysgu
- name: manta
  role: PE baseline SV caller run with default settings on the same HG002 alignments
- name: delly
  role: PE baseline SV caller run with default settings
- name: lumpy
  role: PE baseline SV caller run with default settings
- name: strelka
  role: Baseline indel caller for events down to 30 bp
- name: gatk
  role: Baseline indel caller for events down to 30 bp
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/nar/gkac039
    title: 'Dysgu: efficient structural variant calling using short or long reads'
  source_content_hash: sha256:d48ff924b84aea8d141fec6dd2225bd40dd35fd8942922f9f1a42c76dbe89390
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/kcleal/svbench
  tools:
  - delly
  - dysgu
  - gatk
  - lumpy
  - manta
  - strelka
  - svbench
  edam_operation: http://edamontology.org/operation_3197
  edam_topics:
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8181818181818182
n_claims: 11
---

# giab-benchmark-region-stratification

## Summary

Benchmark structural-variant caller output against the GIAB HG002 (Ashkenazim son) truth set with svbench, PASS-filtering calls and stratifying precision/recall/F1 per SV type and per confidence region (Tier 1 vs all genomic regions) to compare callers on deletions and insertions.

## When to use

Use when comparing SV/indel callers on GIAB-benchmarked samples (e.g. HG002) and you need region-stratified precision, recall and F1 per event class rather than a single genome-wide number.

## When NOT to use

- Evaluating long-read-only callers against GIAB v1.1 or CMRG truth sets — use the kcleal/SV_Benchmark_CMRG_GIAB or SV_benchmark_PacBio_HiFi pipelines instead
- Benchmarking small-variant (SNV/indel) callers where a distance-based evaluator like lh3/CHM-eval (syndip) is the appropriate tool
- When the input VCFs are the reference truth set itself and no caller comparison is needed

## Inputs

- Sorted/indexed paired-end BAM/CRAM alignments for GIAB HG002 (NA24385_son) from ftp://ftp-trace.ncbi.nlm.nih.gov/giab/ftp/data/AshkenazimTrio/
- Indexed reference genome in FASTA format
- NIST_SVs_Integration_v0.6 GIAB benchmark VCF (reference truth set)
- High-confidence Tier 1 region BED file for --include stratification
- Caller output VCFs (dysgu, manta, delly, lumpy, strelka, gatk)

## Outputs

- Per-caller, per-SV-type precision/recall/F1 table stratified by Tier 1 vs all regions (e.g. benchmark_table.tsv)
- Optionally svbench true-positive/false-positive/false-negative call lists (--tp/--fp/--fn)

## How to apply

Obtain the Illumina paired-end alignments for GIAB AshkenazimTrio HG002 (NA24385_son) from the GIAB FTP site, run each caller (dysgu 'dysgu --mode pe' with default args; baselines manta, delly, lumpy with defaults; strelka and gatk for indels down to 30 bp) on the same BAM. Evaluate every call set with svbench against the NIST_SVs_Integration_v0.6 benchmark, restricting with --pass-only and an --include BED of high-confidence Tier 1 regions, using minimum event sizes of 30/50 bp, a 1000 bp interval-overlap criterion and 15% size similarity. Run the evaluation twice — once restricted to Tier 1 and once over all genomic regions — and tabulate precision, recall and F1 separately for deletions and insertions per caller into a benchmark table. Validation anchor from the paper: dysgu should attain the highest Tier 1 F1, with deletion precision ~95.7% and deletion/insertion recall ~62.2%/23.7% at 20x.

## Related tools

- **svbench** (Evaluates caller VCFs against the GIAB reference VCF with PASS filtering, region --include BED, --slop interval padding, and min-size thresholds) — https://github.com/kcleal/svbench
- **dysgu** (Primary paired-end SV/indel caller under test ('--mode pe' default argument set)) — https://github.com/kcleal/dysgu
- **manta** (PE baseline SV caller run with default settings on the same HG002 alignments)
- **delly** (PE baseline SV caller run with default settings)
- **lumpy** (PE baseline SV caller run with default settings)
- **strelka** (Baseline indel caller for events down to 30 bp)
- **gatk** (Baseline indel caller for events down to 30 bp)

## Examples

```
svbench --include tier1.bed --pass-only --min-size-ref 50 --min-size-query 30 NIST_SVs_Integration_v0.6.vcf dysgu.vcf manta.vcf delly.vcf lumpy.vcf
```

## Evaluation signals

- dysgu attains the highest Tier 1 F1 among compared callers, with F1 4.7% higher than manta for deletions and 13.1% higher for insertions at 20x
- Reported anchor values reproduce: deletion precision 95.7%, deletion recall 62.2%, insertion recall 23.7% on HG002 Tier 1 at 20x
- svbench invocation succeeds with reference VCF first: 'svbench [OPTIONS] REFERENCE_VCF QUERY_VCFS...' and PASS-only/min-size flags are active (--min-size-query default 30)
- Per-SV-type tables separate deletions and insertions and report both Tier 1 and all-region stratifications
- True positive counts are internally consistent: dysgu called the largest number of true deletions and insertions (n = 3913), 727 more than manta (n = 3186)

## Limitations

- svbench applies a default --slop of 250 bp around breakpoints; the paper's 1000 bp interval-overlap criterion and 15% size similarity must be set explicitly to reproduce its numbers
- No software versions for manta/delly/lumpy/strelka/gatk/dysgu are stated in the results section, complicating exact reproduction
- Tier 1 stratification is only high-confidence regions; Tier 1 excludes hard-to-map sequence, so genome-wide performance will be lower and is not interchangeable with Tier 1 results
- The paper notes dysgu's calculated insertion sizes are expected to be less accurate compared to de novo assembly-based callers such as manta and gatk (data not shown)

## Evidence

- [discussion] Primarily we assessed the HG002 benchmark, analysing in detail high-confidence Tier 1 regions, as well as all genomic regions.: "Primarily we assessed the HG002 benchmark, analysing in detail high-confidence Tier 1 regions, as well as all genomic regions."
- [results] Dysgu had the highest precision for deletion calls (95.7%), as well as the highest recall for deletions (62.2%) and insertions (23.7%): "Dysgu had the highest precision for deletion calls (95.7%), as well as the highest recall for deletions (62.2%) and insertions (23.7%)"
- [supplementary] Illumina sequencing data for Ashkenazim HG002 (34) sample was downloaded from GIAB (ftp://ftp-trace.ncbi.nlm.nih.gov/giab/ftp/data/AshkenazimTrio/: "Illumina sequencing data for Ashkenazim HG002 (34) sample was downloaded from GIAB (ftp://ftp-trace.ncbi.nlm.nih.gov/giab/ftp/data/AshkenazimTrio/"
- [readme] Usage: svbench [OPTIONS] REFERENCE_VCF QUERY_VCFS...: [evidence span withheld — anchor status: external_doc]
- [supplementary] Analysis scripts used to reproduce results found in this paper can be found under https://github.com/kcleal/svbench.: "Analysis scripts used to reproduce results found in this paper can be found under https://github.com/kcleal/svbench."
