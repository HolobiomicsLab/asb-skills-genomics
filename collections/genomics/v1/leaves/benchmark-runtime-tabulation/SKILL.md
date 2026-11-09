---
name: benchmark-runtime-tabulation
description: Use when comparing computational cost of SV callers on the same sequencing
  data — e.g. verifying claims like 'dysgu was the fastest tool at 57 min using 5.6GB
  memory' against baselines such as delly or manta.
when_to_use_negative:
- Your goal is SV calling accuracy (precision/recall/F1) rather than computational
  cost — use the svbench-based accuracy benchmarking workflow instead of runtime tabulation.
- Callers cannot be run on identical hardware/inputs — runtime comparisons would be
  confounded and the ranking unverifiable.
- The caller needs parallel execution to meet its design intent (e.g. manta's parallel
  mode) and you intend to report production throughputs — the article's manta parallel
  runtime was 'data not shown', so single-core timings won't generalize.
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_3168
tools:
- name: dysgu
  role: Timed SV caller under test for both PE and LR read types; installed via pip
    or bioconda
  repo: https://github.com/kcleal/dysgu
- name: manta
  role: Baseline PE SV caller timed for comparison
- name: delly
  role: Baseline PE SV caller timed for comparison; next quickest after dysgu
- name: lumpy
  role: Baseline PE SV caller timed for comparison
- name: nanovar
  role: Baseline LR SV caller timed for comparison
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/nar/gkac039
    title: 'Dysgu: efficient structural variant calling using short or long reads'
  source_content_hash: sha256:a6a6730b16a11df5b2d22b6bd64ceb16b9381aa7a206d9ab87ca38b6fed3d255
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/kcleal/dysgu
  tools:
  - delly
  - dysgu
  - lumpy
  - manta
  - nanovar
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_3168
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7272727272727273
n_claims: 11
---

# benchmark-runtime-tabulation

## Summary

Times wall-clock execution and memory of structural variant callers (dysgu, manta, delly, lumpy for PE; dysgu, nanovar for LR) on identical BAM inputs and hardware, then ranks them per read type and writes a runtime table.

## When to use

Use when comparing computational cost of SV callers on the same sequencing data — e.g. verifying claims like 'dysgu was the fastest tool at 57 min using 5.6GB memory' against baselines such as delly or manta.

## When NOT to use

- Your goal is SV calling accuracy (precision/recall/F1) rather than computational cost — use the svbench-based accuracy benchmarking workflow instead of runtime tabulation.
- Callers cannot be run on identical hardware/inputs — runtime comparisons would be confounded and the ranking unverifiable.
- The caller needs parallel execution to meet its design intent (e.g. manta's parallel mode) and you intend to report production throughputs — the article's manta parallel runtime was 'data not shown', so single-core timings won't generalize.

## Inputs

- Sorted, indexed paired-end Illumina BAM (GIAB HG002, NIST_HiSeq_HG002_Homogeneity-10953946)
- Sorted, indexed PacBio long-read BAM (SRA SRR10188368 and SRR10188369)
- Indexed reference genome FASTA
- Installed SV caller binaries: dysgu, manta, delly, lumpy (PE); dysgu, nanovar (LR)
- Wall-clock/memory timer (e.g. /usr/bin/time)

## Outputs

- Per-run elapsed wall-clock seconds and peak memory per caller per read type
- Ranked runtime comparison table
- runtime_table.csv with columns caller name, read type, seconds

## How to apply

Fix the inputs to identical BAM files per read type — the GIAB HG002 Illumina PE BAM (NIST_HiSeq_HG002_Homogeneity-10953946) and the PacBio LR BAMs (SRA SRR10188368/SRR10188369) with their reference genome — and confirm all callers are installed. Run each PE caller (dysgu, manta, delly, lumpy) and each LR caller (dysgu, nanovar) with default parameters on the same core count (the article used single-core), wrapping each invocation in a wall-clock timer such as /usr/bin/time to capture elapsed seconds and peak memory (e.g. via --verbose for max RSS). Repeat each timed run to guard against disk/IO variance — the README notes dysgu's ~1h runtime 'depending on hard-drive speed' — and tabulate caller, read type, and seconds. Rank callers within each read type and check the expected ordering (dysgu fastest on PE, more than twice as fast as the next quickest caller delly; dysgu among the two fastest on LR) before writing runtime_table.csv.

## Related tools

- **dysgu** (Timed SV caller under test for both PE and LR read types; installed via pip or bioconda) — https://github.com/kcleal/dysgu
- **manta** (Baseline PE SV caller timed for comparison)
- **delly** (Baseline PE SV caller timed for comparison; next quickest after dysgu)
- **lumpy** (Baseline PE SV caller timed for comparison)
- **nanovar** (Baseline LR SV caller timed for comparison)

## Examples

```
/usr/bin/time -v dysgu run -p1 reference.fa temp_dir input.bam > svs.vcf
```

## Evaluation signals

- runtime_table.csv contains one row per caller per read type (dysgu/manta/delly/lumpy on PE; dysgu/nanovar on LR) with numeric elapsed seconds
- Ranking within PE shows dysgu as fastest and delly as the next quickest, consistent with the article's Table 8 (dysgu 57 min, >2x faster than delly)
- Memory usage for dysgu falls in the reported range (< 6 GB; 5.6 GB recorded in the article)
- Repeated timed runs of the same caller/inputs produce reasonably stable elapsed times (no outlier runs from disk contention, per the README's note on hard-drive speed)
- Default parameters and identical BAM/reference inputs are used for every caller, so the comparison is like-for-like

## Limitations

- Manta's parallel runtime was reported as 'data not shown' in the article, so parallel-mode throughput cannot be reproduced from the paper.
- No software versions are stated for any tool, so exact version pinning must be chosen and documented by the reproducer.
- Runtime depends on hardware and hard-drive speed; the README notes dysgu 'usually takes ~1h to analyse a 30X coverage genome' and that fetch generates a large temp file (>5 GB), so disk I/O can confound timings.
- Single-core timing may not reflect each caller's intended parallel deployment.

## Evidence

- [results] On 40x coverage Illumina data run single-core, dysgu was the fastest tool at 57 min using 5.6GB memory, more than twice as fast as the next quickest caller, delly (Table 8).: [evidence span withheld — anchor status: not_found]
- [readme] Using a single core and depending on hard-drive speed, dysgu usually takes ~1h to analyse a 30X coverage genome of 150 bp paired-end reads and uses < 6 GB memory.: [evidence span withheld — anchor status: external_doc]
- [results] Manta parallel runtime reported as 'data not shown': "can also be run in parallel efficiently (data not shown)"
- [results] Performance was compared to the popular SV callers manta (9), delly (42), and lumpy (43): "Performance was compared to the popular SV callers manta (9), delly (42), and lumpy (43)"
