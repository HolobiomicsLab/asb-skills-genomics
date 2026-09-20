---
name: cross-tool-runtime-comparison
description: Use when comparing computational cost (runtime and peak RSS) of multiple
  SV-calling tools or pipeline variants on a shared dataset, e.g. Blackbird SLR/hybrid
  vs Sniffles2 and PBSV on HG002.
when_to_use_negative:
- The goal is to compare SV-calling accuracy (F1-score, precision/recall) rather than
  computational cost — use a variant-benchmarking skill instead
- The tools operate on different input datasets that are not comparable (e.g. different
  samples or coverage), making runtime comparison meaningless
- Only a single tool/mode is being run, so there is no cross-tool comparison to make
edam_operation: http://edamontology.org/operation_0335
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3175
tools:
- name: Blackbird
  role: SV caller benchmarked in both SLR mode (6 h / 12 Gb) and hybrid mode with
    PacBio HiFi long reads (11 h / 117 Gb)
  repo: https://github.com/1dayac/Blackbird
- name: SvABA
  role: SLR-based competitor SV caller benchmarked on the same HG002 SLR dataset (65
    min / 11.5 Gb)
- name: Manta
  role: SLR-based competitor SV caller benchmarked on the same HG002 SLR dataset (15
    min / 6.5 Gb)
- name: Sniffles2
  role: Long-read SV caller benchmarked on the PacBio HiFi dataset (3 min / 0.6 Gb)
- name: PBSV
  role: Long-read SV caller benchmarked on the PacBio HiFi dataset (1 h 19 min / 27
    Gb)
- name: BWA
  role: Used to index the GRCh37/hg19 reference genome required by Blackbird (--reference
    must be BWA-indexed)
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/bioadv/vbaf151
    title: 'Blackbird: structural variant detection using synthetic and low-coverage
      long-reads'
  source_content_hash: sha256:e2c7227c1d4230b15e576b9083fdca26cc7894d397ee9b824665b0e17133a64c
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/1dayac/Blackbird
  tools:
  - BWA
  - Blackbird
  - Manta
  - PBSV
  - Sniffles2
  - SvABA
  edam_operation: http://edamontology.org/operation_0335
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3175
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4
n_claims: 10
---

# cross-tool-runtime-comparison

## Summary

Benchmark wall-clock runtime and peak memory of SV callers (Blackbird SLR and hybrid modes versus SvABA, Manta, Sniffles2, PBSV) on the same HG002 dataset by wrapping each run with /usr/bin/time -v on equivalent hardware, so compute cost can be reported alongside call quality.

## When to use

Use when comparing computational cost (runtime and peak RSS) of multiple SV-calling tools or pipeline variants on a shared dataset, e.g. Blackbird SLR/hybrid vs Sniffles2 and PBSV on HG002.

## When NOT to use

- The goal is to compare SV-calling accuracy (F1-score, precision/recall) rather than computational cost — use a variant-benchmarking skill instead
- The tools operate on different input datasets that are not comparable (e.g. different samples or coverage), making runtime comparison meaningless
- Only a single tool/mode is being run, so there is no cross-tool comparison to make

## Inputs

- Position-sorted, indexed HG002 (NA24385) SLR/stLFR bam with BX barcode tags (GRCh37/hg19)
- BWA-indexed GRCh37/hg19 reference genome FASTA
- Position-sorted, indexed PacBio HiFi long-read bam plus matching FASTQ (34x coverage)
- Installed and compiled binaries of Blackbird, SvABA, Manta, Sniffles2, and PBSV

## Outputs

- Per-tool wall-clock runtime and peak memory (peak RSS) measurements from /usr/bin/time -v
- runtime_memory_benchmark.csv recording measured and reported values for each tool and mode
- Caller output files confirming successful completion (e.g. Blackbird out_50.vcf)

## How to apply

Run every tool on identical inputs under /usr/bin/time -v to capture wall time and peak RSS. For Blackbird SLR mode use the blackbird binary with --bam (stLFR bam with BX tags), BWA-indexed --reference, --output, and --threads 32; for hybrid mode additionally pass the paired --long-bam and --long-read options with the PacBio HiFi 34x data. Run SvABA and Manta on the same HG002 SLR bam, and Sniffles2 and PBSV on the same PacBio HiFi data, all on equivalent hardware. Record measured time and memory per tool (reported: Blackbird SLR 6 h / 12 Gb, Blackbird hybrid 11 h / 117 Gb, SvABA 65 min / 11.5 Gb, Manta 15 min / 6.5 Gb, Sniffles2 3 min / 0.6 Gb, PBSV 1 h 19 min / 27 Gb) into a benchmark CSV and compare against reported values within hardware-dependent tolerance.

## Related tools

- **Blackbird** (SV caller benchmarked in both SLR mode (6 h / 12 Gb) and hybrid mode with PacBio HiFi long reads (11 h / 117 Gb)) — https://github.com/1dayac/Blackbird
- **SvABA** (SLR-based competitor SV caller benchmarked on the same HG002 SLR dataset (65 min / 11.5 Gb))
- **Manta** (SLR-based competitor SV caller benchmarked on the same HG002 SLR dataset (15 min / 6.5 Gb))
- **Sniffles2** (Long-read SV caller benchmarked on the PacBio HiFi dataset (3 min / 0.6 Gb))
- **PBSV** (Long-read SV caller benchmarked on the PacBio HiFi dataset (1 h 19 min / 27 Gb))
- **BWA** (Used to index the GRCh37/hg19 reference genome required by Blackbird (--reference must be BWA-indexed))

## Examples

```
/usr/bin/time -v blackbird -b NA24385.GRCh37.phased_possorted_bam.bam -r genome.fa -o blackbird_hybrid -l pacbio_hifi.bam -m pacbio_hifi.fq -t 32 -s spades_for_blackbird/assembler/spades.py
```

## Evaluation signals

- Every tool and mode completes successfully and produces its expected output (e.g. position-sorted out_50.vcf for Blackbird runs)
- Measured wall time and peak RSS for each entry match reported values (Blackbird SLR 6 h / 12 Gb; Blackbird hybrid 11 h / 117 Gb; SvABA 65 min / 11.5 Gb; Manta 15 min / 6.5 Gb; Sniffles2 3 min / 0.6 Gb; PBSV 1 h 19 min / 27 Gb) within reasonable hardware-dependent tolerance
- All inputs are position-sorted and indexed (SLR bam with BX tags, long-read bam, BWA-indexed reference) before timing runs
- Each tool is timed on equivalent hardware so runtime comparisons are fair
- The benchmark CSV contains one row per tool/mode with both measured and reported time and memory

## Limitations

- Blackbird hybrid mode is substantially more expensive than the dedicated long-read callers (11 h / 117 Gb vs 3 min / 0.6 Gb for Sniffles2), which matters for clinical-scale deployment
- Measured runtimes and memory are hardware-dependent; comparisons are only valid when all tools run on equivalent hardware
- Blackbird was forked from SPAdes and requires separately installing/compiling SPAdes for blackbird (spades.py path may need to be supplied via --spades)

## Evidence

- [readme] Blackbird SLR runtime: "Blackbird is a novel integrated alignment- and local-assembly-based algorithm hat employs the barcode information encoded in SLR reads"
- [readme] hybrid mode inputs: "`--long-bam` or `-l` - Position-sorted and indexed long read file (optional, should be paired with long-read option)"
- [readme] reference requirement: "`--rerefence` or `-r` [required] - BWA-indexed reference genome"
- [readme] successful completion output: "`out_50.vcf` is a position sorted VCF file, that contains insertion and deletion calls longer than 50 bp."
- [abstract] HG002 benchmark context: "On the HG002 Genome in a Bottle (GIAB) benchmark, Blackbird in hybrid mode demonstrated results comparable to state-of-the-art long-read tools"
- [abstract] long-read competitors: "similar to PBSV and Sniffles2 using 10× PacBio Hi-Fi long-read coverage"
