---
name: deletion-insertion-f1-evaluation
description: Use when you have VCF call sets from PE SV callers plus a GIAB benchmark
  VCF (e.g. NIST_SVs_Integration_v0.6 for HG002) and need Tier 1-stratified deletion/insertion
  precision, recall and F1 to rank callers.
when_to_use_negative:
- Evaluating small-variant (SNV/indel) caller accuracy — use CHM-eval/syndip-style
  distance-based evaluation instead, since CHM-eval is constructed for small variant
  callers
- Benchmarking long-read SV callers with phased/haplotagged inputs (hiphase/whatshap
  HP and PS tags) — the SV_Benchmark_CMRG_GIAB or SV_benchmark_PacBio_HiFi pipelines
  are the appropriate reproducible benchmarks
- The question concerns duplications or inversions rather than deletions/insertions
  — on simulated data lumpy, not dysgu, showed the highest F1 for duplications and
  inversions
edam_operation: http://edamontology.org/operation_3227
edam_topics:
- http://edamontology.org/topic_0625
- http://edamontology.org/topic_0622
tools:
- name: svbench
  role: 'Comparison engine: matches query VCFs against the GIAB reference VCF with
    PASS-only filtering, min-size and interval-overlap options to produce precision/recall/F1'
  repo: https://github.com/kcleal/svbench
- name: dysgu
  role: Caller under test, run in paired-end mode with default arguments; 'dysgu run
    reference.fa temp_dir input.bam > svs.vcf' produces the query VCF
  repo: https://github.com/kcleal/dysgu
- name: manta
  role: PE baseline caller, strongest competitor (n=3186 true del+ins vs dysgu 3913)
    against which the F1 delta is measured
- name: delly
  role: PE baseline caller run with default settings on the same alignments
- name: lumpy
  role: PE baseline caller run with default settings on the same alignments
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/nar/gkac039
    title: 'Dysgu: efficient structural variant calling using short or long reads'
  source_content_hash: sha256:78e4cd73604f020f433f412df40bf52271bcdd3e8e5c1304b14f3c332fa7cd70
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
  edam_operation: http://edamontology.org/operation_3227
  edam_topics:
  - http://edamontology.org/topic_0625
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8235294117647058
n_claims: 17
---

# deletion-insertion-f1-evaluation

## Summary

Benchmark deletion and insertion calls from paired-end SV callers against a GIAB truth set using svbench, computing per-caller precision, recall and F1 restricted to high-confidence Tier 1 regions. It exists to quantify whether a caller (e.g. dysgu) outperforms baselines such as manta, delly and lumpy.

## When to use

Use when you have VCF call sets from PE SV callers plus a GIAB benchmark VCF (e.g. NIST_SVs_Integration_v0.6 for HG002) and need Tier 1-stratified deletion/insertion precision, recall and F1 to rank callers.

## When NOT to use

- Evaluating small-variant (SNV/indel) caller accuracy — use CHM-eval/syndip-style distance-based evaluation instead, since CHM-eval is constructed for small variant callers
- Benchmarking long-read SV callers with phased/haplotagged inputs (hiphase/whatshap HP and PS tags) — the SV_Benchmark_CMRG_GIAB or SV_benchmark_PacBio_HiFi pipelines are the appropriate reproducible benchmarks
- The question concerns duplications or inversions rather than deletions/insertions — on simulated data lumpy, not dysgu, showed the highest F1 for duplications and inversions

## Inputs

- Sorted, indexed BAM/CRAM of Illumina paired-end alignments for GIAB HG002 (AshkenazimTrio NA24385_son), e.g. NIST_HiSeq_HG002_Homogeneity-10953946
- Indexed reference genome FASTA
- SV/indel call VCFs from dysgu, manta, delly, lumpy (and optionally strelka/gatk for indels >=30 bp)
- GIAB benchmark VCF (NIST_SVs_Integration_v0.6) and Tier 1 high-confidence region BED file

## Outputs

- benchmark_table.tsv with precision, recall and F1 for deletions and insertions per caller, stratified by Tier 1 vs all genomic regions
- Optional per-call TP/FP/FN listings from svbench (--tp/--fp/--fn/--dtp flags)

## How to apply

Run dysgu in paired-end mode ('dysgu --mode pe') with its default argument set on the GIAB HG002 (NA24385_son) Illumina alignments downloaded from the GIAB AshkenazimTrio FTP site, and run manta, delly and lumpy with default settings on the same BAM; include strelka and gatk for indels down to 30 bp. Evaluate every call set with svbench (https://github.com/kcleal/svbench) against the NIST_SVs_Integration_v0.6 GIAB benchmark, keeping only PASS-filtered calls, using minimum event sizes of 30/50 bp, a 1000 bp interval-overlap criterion and 15% size similarity. Stratify results per caller and restrict to high-confidence Tier 1 regions as well as all genomic regions. Tabulate precision, recall and F1 separately for deletions and insertions into a benchmark table; note svbench's interval slop (--slop, default 250) is the knob for the interval-overlap criterion and --min-size-ref/--min-size-query control the size floor. A correct application reproduces the ranking where dysgu leads with deletion precision 95.7%, deletion recall 62.2% and insertion recall 23.7% at 20x on HG002 Tier 1.

## Related tools

- **svbench** (Comparison engine: matches query VCFs against the GIAB reference VCF with PASS-only filtering, min-size and interval-overlap options to produce precision/recall/F1) — https://github.com/kcleal/svbench
- **dysgu** (Caller under test, run in paired-end mode with default arguments; 'dysgu run reference.fa temp_dir input.bam > svs.vcf' produces the query VCF) — https://github.com/kcleal/dysgu
- **manta** (PE baseline caller, strongest competitor (n=3186 true del+ins vs dysgu 3913) against which the F1 delta is measured)
- **delly** (PE baseline caller run with default settings on the same alignments)
- **lumpy** (PE baseline caller run with default settings on the same alignments)

## Examples

```
svbench --pass-only --min-size-ref 50 NIST_SVs_Integration_v0.6.vcf dysgu_calls.vcf manta_calls.vcf delly_calls.vcf lumpy_calls.vcf
```

## Evaluation signals

- dysgu attains the highest Tier 1 F1 among all PE callers, consistent with an F1 4.7% higher than manta for deletions and 13.1% higher for insertions at 20x on HG002
- dysgu's true deletion+insertion count (n = 3913) exceeds the next best caller manta (n = 3186) by 727 calls
- dysgu deletion precision reaches 95.7%, with deletion recall 62.2% and insertion recall 23.7% at 20x on HG002 Tier 1
- All evaluated call sets are PASS-filtered and use minimum event sizes of 30/50 bp with a 1000 bp interval-overlap criterion and 15% size similarity — check svbench options --pass-only, --min-size-ref, --min-size-query reflect these thresholds
- Results are stratified into both high-confidence Tier 1 regions and all genomic regions; a run missing one stratum is incomplete

## Limitations

- No software versions are stated for any caller in the comparison, complicating exact reproduction
- dysgu's calculated insertion sizes are expected to be less accurate than assembly-based callers such as manta and gatk ('data not shown'), so insertion F1 partially reflects size-match sensitivity
- dysgu has no de novo assembly stage, so multi-allelic sites and full-length insertion sequences in PE data remain unresolved, which can cap insertion recall

## Evidence

- [results] Dysgu had the highest precision for deletion calls (95.7%), as well as the highest recall for deletions (62.2%) and insertions (23.7%): "Dysgu had the highest precision for deletion calls (95.7%), as well as the highest recall for deletions (62.2%) and insertions (23.7%)"
- [results] Dysgu called the largest number of true deletions and insertions (n = 3913): "Dysgu called the largest number of true deletions and insertions (n = 3913), with 727 more variants called than the next best caller manta (n = 3186)"
- [supplementary] Analysis scripts used to reproduce results found in this paper: "Analysis scripts used to reproduce results found in this paper can be found under https://github.com/kcleal/svbench."
- [readme] SVBench is a python tool/library for benchmarking variant calls: [evidence span withheld — anchor status: external_doc]
