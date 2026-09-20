---
name: wall-clock-runtime-benchmarking
description: Use when comparing computational cost of SV callers across the same paired-end
  Illumina or long-read BAM, or verifying claims that a caller (e.g. dysgu) is fastest
  on matched hardware.
when_to_use_negative:
- The goal is variant-calling accuracy (precision/recall/F1), not speed — use a benchmarking
  framework like svbench against a GIAB truth set instead.
- Callers cannot be run on identical inputs and hardware (different read types, references,
  or thread counts), making runtime comparisons confounded.
- Manta is intended to be run in parallel — single-core timing under-represents its
  throughput ('data not shown' in the paper).
edam_operation: http://edamontology.org/operation_0335
edam_topics:
- http://edamontology.org/topic_0622
tools:
- name: dysgu
  role: SV caller being timed; runs on both PE and LR data via dysgu run/call with
    default parameters
  repo: https://github.com/kcleal/dysgu
- name: Manta
  role: PE baseline SV caller timed for runtime comparison
- name: delly
  role: PE baseline SV caller timed for runtime comparison (second fastest on PE)
- name: lumpy
  role: PE baseline SV caller timed for runtime comparison
- name: nanovar
  role: LR baseline SV caller timed for runtime comparison
- name: svbench
  role: Benchmarking framework from the same authors whose analysis scripts reproduce
    paper results; useful for the surrounding benchmark setup
  repo: https://github.com/kcleal/svbench
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/nar/gkac039
    title: 'Dysgu: efficient structural variant calling using short or long reads'
  source_content_hash: sha256:2deaa8763d4e2ac931d5f208790ebbbecc04523c1e5b6c897bf320df5a319960
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/kcleal/dysgu
  tools:
  - Manta
  - delly
  - dysgu
  - lumpy
  - nanovar
  - svbench
  edam_operation: http://edamontology.org/operation_0335
  edam_topics:
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9230769230769231
n_claims: 13
---

# wall-clock-runtime-benchmarking

## Summary

Benchmark the wall-clock runtime and memory footprint of structural variant callers (dysgu, manta, delly, lumpy, nanovar) by running each on identical BAM inputs with default parameters under a wall-clock timer, then ranking callers per read type (PE vs LR).

## When to use

Use when comparing computational cost of SV callers across the same paired-end Illumina or long-read BAM, or verifying claims that a caller (e.g. dysgu) is fastest on matched hardware.

## When NOT to use

- The goal is variant-calling accuracy (precision/recall/F1), not speed — use a benchmarking framework like svbench against a GIAB truth set instead.
- Callers cannot be run on identical inputs and hardware (different read types, references, or thread counts), making runtime comparisons confounded.
- Manta is intended to be run in parallel — single-core timing under-represents its throughput ('data not shown' in the paper).

## Inputs

- Sorted, indexed paired-end Illumina BAM (GIAB HG002 NIST_HiSeq_HG002_Homogeneity-10953946) at matched coverage (e.g. 40x)
- Sorted, indexed PacBio long-read BAM (SRA accessions SRR10188368 and SRR10188369)
- Indexed reference genome FASTA shared by all callers
- Installed and callable SV callers: dysgu, manta, delly, lumpy (PE) and dysgu, nanovar (LR)

## Outputs

- Runtime table (e.g. runtime_table.csv) with columns: caller name, read type (PE vs LR), elapsed seconds
- Per-caller peak memory usage (e.g. GB, from /usr/bin/time)
- Within-read-type runtime ranking used to verify the speed claim

## How to apply

Fix the inputs first: the GIAB HG002 Illumina PE BAM and PacBio long-read BAMs (SRR10188368/SRR10188369) with the same indexed reference genome, and confirm all callers (dysgu, manta, delly, lumpy, nanovar) are installed. Run each PE caller (dysgu, manta, delly, lumpy) on the identical PE BAM with default parameters, wrapping the invocation in a wall-clock timer such as /usr/bin/time to record elapsed seconds; repeat for LR callers (dysgu, nanovar) on the identical LR BAM. Repeat timed runs and tabulate elapsed seconds per caller per read type into a single table (e.g. runtime_table.csv with caller name, read type, seconds), then rank callers within each read type and check the claim that dysgu is fastest on PE and among the two fastest LR tools — the paper reported dysgu fastest on 40x Illumina single-core at 57 min using 5.6 GB memory, more than twice as fast as delly. Keep hardware, thread count (single core), and coverage fixed across all runs so differences are attributable to the caller, and note that dysgu's fetch step generates a large temporary file (>5 GB) of SV-associated reads that must fit on disk.

## Related tools

- **dysgu** (SV caller being timed; runs on both PE and LR data via dysgu run/call with default parameters) — https://github.com/kcleal/dysgu
- **Manta** (PE baseline SV caller timed for runtime comparison)
- **delly** (PE baseline SV caller timed for runtime comparison (second fastest on PE))
- **lumpy** (PE baseline SV caller timed for runtime comparison)
- **nanovar** (LR baseline SV caller timed for runtime comparison)
- **svbench** (Benchmarking framework from the same authors whose analysis scripts reproduce paper results; useful for the surrounding benchmark setup) — https://github.com/kcleal/svbench

## Examples

```
/usr/bin/time -v dysgu run -p1 reference.fa temp_dir HG002.pe.bam > svs.vcf
```

## Evaluation signals

- Every caller ran on the byte-identical BAM and reference with default parameters; only the timer wrapper differed.
- Runtime table contains one elapsed-seconds entry per caller per read type, with repeated runs tabulated (not single measurements).
- PE ranking reproduces the paper's ordering: dysgu fastest, delly next — dysgu ~57 min / 5.6 GB on 40x single-core Illumina.
- Memory and disk usage are plausible: dysgu expected <6 GB RAM and a >5 GB temp BAM from the fetch step on a 30x genome.
- Hardware, coverage, and thread count are recorded and identical across runs.

## Limitations

- Software versions of the callers are not stated in the paper, so exact runtime reproducibility may vary with versions.
- Manta's parallel runtime was not reported ('data not shown'), so single-core timings do not capture its multi-core performance.
-  dysgu's fetch/run stage creates a large temporary file of SV-associated reads (>5 GB) requiring adequate disk space and I/O speed; runtime depends on hard-drive speed.
- Runtime results are hardware- and coverage-dependent (40x single-core Illumina in the paper).

## Evidence

- [results] dysgu was the fastest tool at 57 min using 5.6GB memory: [evidence span withheld — anchor status: not_found]
- [results] Performance was compared to the popular SV callers manta (9), delly (42), and lumpy (43): "Performance was compared to the popular SV callers manta (9), delly (42), and lumpy (43)"
- [readme] Using a single core and depending on hard-drive speed, dysgu usually takes ~1h: [evidence span withheld — anchor status: external_doc]
- [readme] a large temp file is generated consisting of SV-associated reads >5 Gb in size: [evidence span withheld — anchor status: external_doc]
