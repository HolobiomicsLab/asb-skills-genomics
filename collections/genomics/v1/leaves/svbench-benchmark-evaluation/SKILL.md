---
name: svbench-benchmark-evaluation
description: Use when multiple SV caller VCFs (e.g. dysgu PacBio/ONT, SVIM, NanoVar,
  Sniffles) must be compared on the same HG002 long-read alignments, and per-caller
  precision/recall/F1 for deletions and insertions is the deliverable.
when_to_use_negative:
- Short-read (paired-end) caller comparisons against Manta/DELLY/Lumpy — use the paper's
  PE benchmarking workflow (HG002 Tier 1 at 20x, syndip benchmark) instead
- Genotype concordance or allele-accuracy evaluation — svbench reports precision/recall/F1;
  a different evaluator (e.g. rtg-based) is needed
- The goal is calling SVs rather than benchmarking existing VCFs — run dysgu directly
  (dysgu run/dysgu call) instead
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3168
tools:
- name: svbench
  role: Benchmarks each caller's VCF against the GIAB HG002 SV reference VCF to produce
    precision, recall and F1 restricted to deletion and insertion classes
  repo: https://github.com/kcleal/svbench
- name: dysgu
  role: Primary caller under evaluation; run with --mode pacbio / --mode nanopore
    and filtered at maximum model probability >=0.5 (PacBio) or >=0.35 (ONT)
  repo: https://github.com/kcleal/dysgu
- name: SVIM
  role: Long-read comparator caller run on the same HG002 PacBio HiFi and ONT BAMs
- name: nanovar
  role: Long-read comparator caller run on the same HG002 alignment files
- name: Sniffles
  role: Long-read comparator caller run on the same HG002 alignment files
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/nar/gkac039
    title: 'Dysgu: efficient structural variant calling using short or long reads'
  source_content_hash: sha256:040f2a2662d8df9bf1dfa9ead98741e94c6d74343896b9f48168f32bf5fc0111
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/kcleal/svbench
  tools:
  - SVIM
  - Sniffles
  - dysgu
  - nanovar
  - svbench
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3168
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9333333333333333
n_claims: 15
---

# svbench-benchmark-evaluation

## Summary

Benchmark structural variant callers (dysgu, SVIM, NanoVar, Sniffles) on HG002 long-read data by evaluating their VCFs against the GIAB v0.6 HG002 SV truth set with svbench, computing precision, recall and F1 restricted to deletion and insertion event classes (paper Table 5).

## When to use

Use when multiple SV caller VCFs (e.g. dysgu PacBio/ONT, SVIM, NanoVar, Sniffles) must be compared on the same HG002 long-read alignments, and per-caller precision/recall/F1 for deletions and insertions is the deliverable.

## When NOT to use

- Short-read (paired-end) caller comparisons against Manta/DELLY/Lumpy — use the paper's PE benchmarking workflow (HG002 Tier 1 at 20x, syndip benchmark) instead
- Genotype concordance or allele-accuracy evaluation — svbench reports precision/recall/F1; a different evaluator (e.g. rtg-based) is needed
- The goal is calling SVs rather than benchmarking existing VCFs — run dysgu directly (dysgu run/dysgu call) instead

## Inputs

- HG002 PacBio HiFi BAM (SRA accessions SRR10188368, SRR10188369)
- HG002 ONT BAM from GIAB AshkenazimTrio FTP directory (ftp://ftp-trace.ncbi.nlm.nih.gov/giab/ftp/data/AshkenazimTrio/)
- Indexed reference genome FASTA
- GIAB v0.6 HG002 SV benchmark VCF (truth set)
- Tier 1 high-confidence regions BED file
- Caller VCFs: dysgu PacBio, dysgu ONT, SVIM, NanoVar, Sniffles

## Outputs

- Per-caller per-read-type F1, precision and recall table (lr_sv_comparison_table.csv)
- Optional svbench TP/FP/FN listings (--tp/--fp/--fn) for error inspection

## How to apply

Run dysgu on the HG002 PacBio HiFi BAM (SRR10188368/SRR10188369) with --mode pacbio and on the HG002 ONT BAM from the GIAB AshkenazimTrio FTP directory with --mode nanopore, then filter calls with maximum model probability >=0.5 for PacBio and >=0.35 for ONT (paper Table 7). Run SVIM, NanoVar and Sniffles on the same PacBio HiFi and ONT alignment files so all call sets are derived from identical BAMs. Evaluate each VCF against the GIAB v0.6 HG002 SV benchmark with svbench, passing a Tier 1 high-confidence regions BED via --include, restricting to PASS variants with --pass-only, and using the default 250 bp slop and --min-size-query 30 to match SVs. Assess only deletion and insertion event classes and tabulate F1, precision and recall per caller per read type, writing the result to lr_sv_comparison_table.csv. On HG002 Tier 1 long-read calls the expected ordering is dysgu F1 0.945 (deletions) and 0.934 (insertions), closely followed by svim at 0.935 and 0.928.

## Related tools

- **svbench** (Benchmarks each caller's VCF against the GIAB HG002 SV reference VCF to produce precision, recall and F1 restricted to deletion and insertion classes) — https://github.com/kcleal/svbench
- **dysgu** (Primary caller under evaluation; run with --mode pacbio / --mode nanopore and filtered at maximum model probability >=0.5 (PacBio) or >=0.35 (ONT)) — https://github.com/kcleal/dysgu
- **SVIM** (Long-read comparator caller run on the same HG002 PacBio HiFi and ONT BAMs)
- **nanovar** (Long-read comparator caller run on the same HG002 alignment files)
- **Sniffles** (Long-read comparator caller run on the same HG002 alignment files)

## Examples

```
svbench --pass-only --include HG002_Tier1.bed GIAB_HG002_SV_v0.6.vcf.gz dysgu_pacbio.vcf.gz svim_pacbio.vcf.gz sniffles_pacbio.vcf.gz
```

## Evaluation signals

- lr_sv_comparison_table.csv contains one row per caller per read type (dysgu PacBio, dysgu ONT, SVIM, NanoVar, Sniffles) with F1, precision and recall columns
- Expected result on HG002 Tier 1: dysgu F1 0.945 (deletions) and 0.934 (insertions), svim 0.935 and 0.928 — dysgu ranks highest, svim second
- TP + FN must equal the number of deletion/insertion events in the GIAB v0.6 HG002 truth set restricted to Tier 1 (invariant of the caller under test)
- svbench flag settings check out: --pass-only used, --slop 250 (default), --min-size-query 30, Tier 1 BED supplied via --include
- Only deletion and insertion event classes are scored; other SV classes are excluded from the tabulation

## Limitations

- Software versions for dysgu, SVIM, NanoVar, Sniffles and svbench are not stated in the paper, so exact reproduction requires pinning versions independently
- svbench's default 250 bp slop around breakpoints and duplicate-TP handling (--no-duplicates) affect reported precision/recall and should be kept consistent across callers
- Insertion size accuracy of dysgu is 'data not shown' in the paper; F1 matching does not guarantee accurate insertion sizes
- Manually labelled/curated truth sets induce observer bias, so scores are benchmark-dependent

## Evidence

- [readme] SVBench is a python tool/library for benchmarking variant calls vs a reference dataset.: [evidence span withheld — anchor status: external_doc]
- [readme] svbench slop and include options: "--slop INTEGER            Add intervals +/- slop around breakpoints
                                [default: 250]"
- [results] probability filter thresholds: "filtering calls with a maximum model probability ≥0.5 for PacBio, or ≥0.35 for ONT combinations (Table 7)"
- [discussion] HG002 Tier 1 evaluation scope: "Primarily we assessed the HG002 benchmark, analysing in detail high-confidence Tier 1 regions, as well as all genomic regions."
- [readme] dysgu long-read modes: "Presets are available using the `--mode` option for PacBio `pacbio-sequel2 | pacbio-revio`,
and ONT `nanopore-r9 | nanopore-r10`."
