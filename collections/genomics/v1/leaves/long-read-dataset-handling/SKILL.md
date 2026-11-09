---
name: long-read-dataset-handling
description: Use when the input is a position-sorted, BX-tagged SLR bam (e.g. 10X
  HG002 GRCh37) optionally paired with PacBio HiFi bam/FASTQ, and the goal is indel/SV
  (>50 bp) calling or runtime/memory benchmarking against long-read SV callers.
when_to_use_negative:
- The input is short-read-only data without BX barcode tags — Blackbird requires barcode
  information encoded in SLR reads
- The task needs whole-genome assembly — Blackbird deliberately avoids whole-genome
  assembly in favor of barcode-aware sliding-window local assembly
- Only long-read data are available and no SLR bam exists — Sniffles2 or PBSV alone
  are the appropriate long-read callers in that setting
edam_operation: http://edamontology.org/operation_3228
edam_topics:
- http://edamontology.org/topic_0622
tools:
- name: Blackbird
  role: Primary SV caller run in SLR mode and hybrid mode on SLR + PacBio HiFi datasets
  repo: https://github.com/1dayac/Blackbird
- name: SPAdes
  role: Codebase and local-assembly engine; spades.py from spades_for_blackbird is
    supplied via --spades and compiled with spades_compile.sh
  repo: https://github.com/1dayac/spades_for_blackbird
- name: BWA
  role: Used to index the reference genome required by Blackbird (--reference expects
    a BWA-indexed genome)
- name: Sniffles2
  role: Long-read SV competitor benchmarked for runtime (3 min) and memory (0.6 Gb)
    on the same PacBio HiFi HG002 data
- name: PBSV
  role: Long-read SV competitor benchmarked for runtime (1 h 19 min) and memory (27
    Gb) on the same PacBio HiFi HG002 data
- name: SvABA
  role: SLR-mode competitor benchmarked (65 min / 11.5 Gb) on the same HG002 SLR dataset
- name: Manta
  role: SLR-mode competitor benchmarked (15 min / 6.5 Gb) on the same HG002 SLR dataset
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/bioadv/vbaf151
    title: 'Blackbird: structural variant detection using synthetic and low-coverage
      long-reads'
  source_content_hash: sha256:971fb4fc287672e0c863e81719e250204f8df130bdd85683f51a0dcaf12797c4
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/1dayac/Blackbird
  tools:
  - BWA
  - Blackbird
  - Manta
  - PBSV
  - SPAdes
  - Sniffles2
  - SvABA
  edam_operation: http://edamontology.org/operation_3228
  edam_topics:
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8125
n_claims: 16
---

# long-read-dataset-handling

## Summary

Run Blackbird on synthetic long-read (SLR) datasets and hybrid SLR+long-read datasets for structural variant (50–10,000 bp) detection, and benchmark its runtime and memory against long-read callers such as Sniffles2 and PBSV on HG002.

## When to use

Use when the input is a position-sorted, BX-tagged SLR bam (e.g. 10X HG002 GRCh37) optionally paired with PacBio HiFi bam/FASTQ, and the goal is indel/SV (>50 bp) calling or runtime/memory benchmarking against long-read SV callers.

## When NOT to use

- The input is short-read-only data without BX barcode tags — Blackbird requires barcode information encoded in SLR reads
- The task needs whole-genome assembly — Blackbird deliberately avoids whole-genome assembly in favor of barcode-aware sliding-window local assembly
- Only long-read data are available and no SLR bam exists — Sniffles2 or PBSV alone are the appropriate long-read callers in that setting

## Inputs

- Position-sorted and indexed SLR bam-file with BX tags (e.g. NA24385.GRCh37.phased_possorted_bam.bam)
- BWA-indexed reference genome FASTA (GRCh37/hg19)
- Position-sorted and indexed long-read bam (e.g. PacBio HiFi 34x HG002) — optional, paired with --long-read
- FASTQ file with the same long reads — optional, paired with --long-bam
- Optional regions file with 'chr start end' rows

## Outputs

- out_50.vcf: position-sorted VCF of insertion and deletion calls longer than 50 bp
- out.vcf: all other (less reliable) calls
- Runtime and peak-memory measurements per tool/mode (e.g. runtime_memory_benchmark.csv)

## How to apply

Index the GRCh37/hg19 reference with BWA, and confirm both the SLR bam (with BX tags) and the optional long-read bam/FASTQ pair are position-sorted and indexed. Run the Blackbird binary with --bam (SLR bam), --reference, --output, and --threads 32, supplying --long-bam and --long-read together to enable hybrid mode; SPAdes for blackbird (spades.py) must be installed and compiled via spades_compile.sh since Blackbird was forked from the SPAdes codebase. Wrap each run with /usr/bin/time -v to capture wall time and peak RSS (reported: Blackbird SLR 6 h / 12 Gb; Blackbird hybrid 11 h / 117 Gb on HG002), and run competitors SvABA, Manta, Sniffles2, and PBSV under the same conditions for comparison (Sniffles2 3 min / 0.6 Gb; PBSV 1 h 19 min / 27 Gb). Collect out_50.vcf as the primary result and record time/memory measurements alongside reported values for validation.

## Related tools

- **Blackbird** (Primary SV caller run in SLR mode and hybrid mode on SLR + PacBio HiFi datasets) — https://github.com/1dayac/Blackbird
- **SPAdes** (Codebase and local-assembly engine; spades.py from spades_for_blackbird is supplied via --spades and compiled with spades_compile.sh) — https://github.com/1dayac/spades_for_blackbird
- **BWA** (Used to index the reference genome required by Blackbird (--reference expects a BWA-indexed genome))
- **Sniffles2** (Long-read SV competitor benchmarked for runtime (3 min) and memory (0.6 Gb) on the same PacBio HiFi HG002 data)
- **PBSV** (Long-read SV competitor benchmarked for runtime (1 h 19 min) and memory (27 Gb) on the same PacBio HiFi HG002 data)
- **SvABA** (SLR-mode competitor benchmarked (65 min / 11.5 Gb) on the same HG002 SLR dataset)
- **Manta** (SLR-mode competitor benchmarked (15 min / 6.5 Gb) on the same HG002 SLR dataset)

## Examples

```
/usr/bin/time -v blackbird -b NA24385.GRCh37.phased_possorted_bam.bam -r genome.fa -o blackbird_10X_chr1 -t 32 -s spades.py -l long_reads.bam -m long_reads.fq
```

## Evaluation signals

- Blackbird run completes and produces out_50.vcf, which is position-sorted and contains only calls longer than 50 bp
- Hybrid mode was enabled only when --long-bam and --long-read were supplied as a paired pair, matching the README constraint
- Measured wall time and peak RSS agree with reported values (SLR 6 h / 12 Gb; hybrid 11 h / 117 Gb; Sniffles2 3 min / 0.6 Gb; PBSV 1 h 19 min / 27 Gb) within hardware-dependent tolerance
- On HG002 GIAB, hybrid-mode Blackbird call quality (e.g. F1 for deletions/insertions) is comparable to PBSV and Sniffles2 run at 10x HiFi coverage
- SPAdes for blackbird was compiled via spades_compile.sh and spades.py is resolvable at the path given to --spades

## Limitations

- Hybrid mode is memory-hungry: 117 Gb peak RSS and 11 h runtime on HG002, far exceeding Sniffles2 (0.6 Gb, 3 min) and PBSV (27 Gb, 1 h 19 min)
- Calls in out.vcf (other than out_50.vcf) are explicitly described as not reliable
- The --reference option is misspelled as '--rerefence' in the README options list, and regions file reuse of the '-r' shorthand can conflict
- Blackbird 0.1 requires both --long-bam and --long-read together; one cannot be supplied alone

## Evidence

- [readme] Blackbird is able to work with SLR read datasets and a combination of SLR read and long-read datasets.: "Blackbird is able to work with SLR read datasets and a combination of SLR read and long-read datasets."
- [readme] Hybrid inputs must be paired: "`--long-bam` or `-l` - Position-sorted and indexed long read file (optional, should be paired with long-read option)"
- [readme] SLR bam input requirements: "`--bam` or `-b` [required] - position-sorted and indexed SLR Read bam-file with BX tags"
- [abstract] Hybrid-mode accuracy finding on HG002: "On the HG002 Genome in a Bottle (GIAB) benchmark, Blackbird in hybrid mode demonstrated results comparable to state-of-the-art long-read tools"
- [readme] Compilation step from SPAdes codebase: "Initially Blackbird was forked from the SPAdes reposotory and have a common codebase.
To compile Blackbird in assembler foder execute:"
- [abstract] Method rationale for local assembly: "Instead of relying on whole-genome assembly, Blackbird uses a sliding window approach and synthetic long-read barcode information to assemble local segments"
