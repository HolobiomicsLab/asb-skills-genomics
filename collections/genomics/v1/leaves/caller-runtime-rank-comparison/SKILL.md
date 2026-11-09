---
name: caller-runtime-rank-comparison
description: Use when comparing compute cost (wall-clock seconds, memory GB) of SV-calling
  tools on the same sequencing data, e.g. verifying that dysgu is fastest on paired-end
  Illumina input or competitive on PacBio/ONT long reads.
when_to_use_negative:
- The goal is variant-calling accuracy (precision/recall/F1) rather than computational
  efficiency — use an accuracy benchmarking skill (e.g. svbench/CHM-eval based) instead
- Callers must run on different datasets or hardware — runtime ranks are only valid
  on the identical BAM under fixed threading and single-core conditions
- Comparing indel/SNV callers or other tool classes — this skill is scoped to structural
  variant callers (dysgu, manta, delly, lumpy, nanovar)
tools:
- name: dysgu
  role: Benchmarked caller under test on both PE and LR data; invoked with 'dysgu
    run' (PE) or 'dysgu call --mode pacbio-revio' (LR) and timed
  repo: https://github.com/kcleal/dysgu
- name: manta
  role: Baseline PE SV caller timed for runtime comparison
- name: delly
  role: Baseline PE SV caller; next-quickest after dysgu at 40x single-core
- name: lumpy
  role: Baseline PE SV caller timed for runtime comparison
- name: nanovar
  role: Baseline LR SV caller timed on the PacBio BAM
- name: svbench
  role: Repository holding analysis scripts used to reproduce the paper's benchmark
    results
  repo: https://github.com/kcleal/svbench
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/nar/gkac039
    title: 'Dysgu: efficient structural variant calling using short or long reads'
  source_content_hash: sha256:6867da6607fc09ad3daac5bdeeb9b6c48e33b93790159f0e32566b4fcffad5a9
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
  - svbench
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 12
---

# caller-runtime-rank-comparison

## Summary

Benchmarks the wall-clock runtime and memory footprint of structural variant callers (dysgu, manta, delly, lumpy for paired-end; dysgu, nanovar for long-read) on identical BAMs under fixed hardware/threading conditions, then ranks callers per read type. Used to substantiate efficiency claims such as dysgu's speed advantage over delly and manta.

## When to use

Use when comparing compute cost (wall-clock seconds, memory GB) of SV-calling tools on the same sequencing data, e.g. verifying that dysgu is fastest on paired-end Illumina input or competitive on PacBio/ONT long reads.

## When NOT to use

- The goal is variant-calling accuracy (precision/recall/F1) rather than computational efficiency — use an accuracy benchmarking skill (e.g. svbench/CHM-eval based) instead
- Callers must run on different datasets or hardware — runtime ranks are only valid on the identical BAM under fixed threading and single-core conditions
- Comparing indel/SNV callers or other tool classes — this skill is scoped to structural variant callers (dysgu, manta, delly, lumpy, nanovar)

## Inputs

- Sorted/indexed GIAB HG002 Illumina paired-end BAM (NIST_HiSeq_HG002_Homogeneity-10953946), 40x coverage
- Sorted/indexed PacBio long-read BAM(s) from SRA accessions SRR10188368 and SRR10188369
- Indexed reference genome FASTA
- Installed SV callers: dysgu, manta, delly, lumpy (PE) and nanovar (LR)

## Outputs

- runtime_table.csv with columns: caller name, read type (PE vs LR), elapsed seconds (and memory usage)
- Per-caller runtime ranking within each read type

## How to apply

Fix the inputs first: the GIAB HG002 Illumina PE BAM (NIST_HiSeq_HG002_Homogeneity-10953946) and the PacBio LR BAMs (SRA SRR10188368/SRR10188369) with an indexed reference FASTA, confirming all callers (dysgu, manta, delly, lumpy, nanovar) are installed and callable. Run each PE caller (dysgu, manta, delly, lumpy) and each LR caller (dysgu, nanovar) on the identical BAM with default parameters, wrapping each invocation in a wall-clock timer (e.g. /usr/bin/time -v) and recording elapsed seconds plus peak memory. Hold threading constant (e.g. single-core, dysgu run -p1) so tools are comparable — the paper's 40x-coverage single-core comparison found dysgu fastest at 57 min / 5.6 GB, more than twice as fast as delly. Repeat each timed run to gauge variance, then tabulate elapsed seconds per caller per read type (PE vs LR) into runtime_table.csv and rank within each read type. For dysgu, note the disk caveat that 'fetch'/'run' generates a >5 GB temp file of SV-associated reads.

## Related tools

- **dysgu** (Benchmarked caller under test on both PE and LR data; invoked with 'dysgu run' (PE) or 'dysgu call --mode pacbio-revio' (LR) and timed) — https://github.com/kcleal/dysgu
- **manta** (Baseline PE SV caller timed for runtime comparison)
- **delly** (Baseline PE SV caller; next-quickest after dysgu at 40x single-core)
- **lumpy** (Baseline PE SV caller timed for runtime comparison)
- **nanovar** (Baseline LR SV caller timed on the PacBio BAM)
- **svbench** (Repository holding analysis scripts used to reproduce the paper's benchmark results) — https://github.com/kcleal/svbench

## Examples

```
/usr/bin/time -v dysgu run -p1 reference.fa temp_dir input.bam > svs.vcf 2> dysgu_time.log
```

## Evaluation signals

- Each caller was run on the identical BAM file with default parameters — verify input paths match across callers
- Threading held fixed (e.g. single-core); re-run timings at least twice and check run-to-run variance before ranking
-  dysgu's measured memory should fall in the <6 GB range for a 30x/40x PE genome, consistent with the README's resource note
- Ranking expectation: dysgu fastest among PE callers and among the two fastest LR callers; delly second-fastest on PE in the 40x single-core comparison
- runtime_table.csv schema check: every row has caller name, read type (PE/LR), and elapsed seconds, and all timed callers appear per read type

## Limitations

- Manta's parallel (multi-core) runtime was reported as 'data not shown', so parallel-mode comparisons cannot be grounded in the paper
- No software versions were stated for any tool in the comparison section, so exact reproducibility requires pinning versions yourself
- When dysgu's fetch/run stage is used, a large temporary file of SV-associated reads (>5 GB) is generated, affecting disk-speed-dependent timings
- Runtime depends on hard-drive speed and core count; single-core timing is the comparable condition reported in the paper

## Evidence

- [results] Performance was compared to the popular SV callers manta (9), delly (42), and lumpy (43): "Performance was compared to the popular SV callers manta (9), delly (42), and lumpy (43)"
- [abstract] Dysgu is among the fastest tools: "Dysgu outperforms existing state-of-the-art tools using paired-end or long-reads, offering high sensitivity and precision whilst being among the fastest tools to run."
- [readme] README resource requirements for dysgu: [evidence span withheld — anchor status: external_doc]
- [readme] README multiprocessing option: "To make use of multiprocessing, set the "-p" parameter::

    dysgu run -p4 reference.fa temp_dir input.bam > svs.vcf"
