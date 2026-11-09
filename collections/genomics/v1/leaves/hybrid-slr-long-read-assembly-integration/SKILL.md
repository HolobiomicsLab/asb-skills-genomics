---
name: hybrid-slr-long-read-assembly-integration
description: Use when calling insertions/deletions (50 bp–10 kbp) from a position-sorted,
  BX-tagged 10X SLR bam plus a low-coverage long-read bam/FASTQ pair, and short-read-only
  or SLR-only callers are insufficient.
when_to_use_negative:
- Input lacks one of the paired hybrid files — the long-read bam without its matching
  long-read FASTQ (or vice versa) is invalid, as --long-bam and --long-read must be
  paired.
- The goal is whole-genome assembly — Blackbird deliberately avoids whole-genome assembly
  and only assembles small sliding-window segments for SV calling.
- Calls in out.vcf are needed as reliable results — the README states these 'other
  calls... are not reliable'; only out_50.vcf (>50 bp) should be trusted.
edam_operation: http://edamontology.org/operation_0525
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3175
tools:
- name: Blackbird
  role: Runs the barcode-aware sliding-window local assembly pipeline and hybrid SV
    calling; binary in bin folder after ./spades_compile.sh
  repo: https://github.com/1dayac/Blackbird
- name: SPAdes (spades_for_blackbird fork)
  role: Barcode- and long-read-aware modified assembler executed per segment via --spades
    path
  repo: https://github.com/1dayac/spades_for_blackbird
- name: BWA
  role: Indexes the reference genome used for alignment and segmentation
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1093/bioadv/vbaf151
    title: 'Blackbird: structural variant detection using synthetic and low-coverage
      long-reads'
  source_content_hash: sha256:2004cd92405c4a2a40feb02b7f14f11e44c5aa4dcdce6951b6b0b0314296a475
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/1dayac/Blackbird
  tools:
  - BWA
  - Blackbird
  - SPAdes (spades_for_blackbird fork)
  edam_operation: http://edamontology.org/operation_0525
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3175
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 10
---

# hybrid-slr-long-read-assembly-integration

## Summary

Integrates low-coverage long reads into Blackbird's barcode-aware sliding-window local assembly of synthetic long-read (SLR) data, improving structural variant detection for medium-size events (50–10,000 bp) at reduced long-read coverage (5× vs 10×).

## When to use

Use when calling insertions/deletions (50 bp–10 kbp) from a position-sorted, BX-tagged 10X SLR bam plus a low-coverage long-read bam/FASTQ pair, and short-read-only or SLR-only callers are insufficient.

## When NOT to use

- Input lacks one of the paired hybrid files — the long-read bam without its matching long-read FASTQ (or vice versa) is invalid, as --long-bam and --long-read must be paired.
- The goal is whole-genome assembly — Blackbird deliberately avoids whole-genome assembly and only assembles small sliding-window segments for SV calling.
- Calls in out.vcf are needed as reliable results — the README states these 'other calls... are not reliable'; only out_50.vcf (>50 bp) should be trusted.

## Inputs

- Position-sorted and indexed SLR read bam-file with BX tags (e.g. 10X phased possorted bam)
- BWA-indexed reference genome FASTA
- Position-sorted and indexed long-read bam (optional, paired with long-read FASTQ)
- FASTQ file with the same long reads (must be paired with the long-read bam)
- Optional regions file with 'chr start end' rows

## Outputs

- out_50.vcf: position-sorted VCF with insertion and deletion calls longer than 50 bp
- out.vcf: VCF of other (less reliable) calls
- Per-segment local assembly contigs (assembly folders, retained with -d)

## How to apply

First align SLR reads with BWA against the indexed reference and supply Blackbird a position-sorted, indexed BX-tagged SLR bam (--bam/-b) plus the BWA-indexed reference (--reference/-r). Enable hybrid mode by providing BOTH the position-sorted, indexed long-read bam (--long-bam/-l) AND a matching FASTQ of the same long reads (--long-read/-m); the options must be paired. Blackbird divides the genome into 50 kbp sliding-window segments with 10 kbp overlap, builds per-segment barcode lists B(s) by keeping SLR barcodes with at least 3 read pairs spanning >5 kbp, collects reads sharing any barcode in M(B(s)) (including reads linked outside the window), and merges the long reads into the per-segment input for the modified SPAdes assembler (spades_for_blackbird fork, passed via --spades/-s). Increase --threads/-t for speed; use the hidden -d flag to retain assembly folders for inspecting the contigs used for SV calling.

## Related tools

- **Blackbird** (Runs the barcode-aware sliding-window local assembly pipeline and hybrid SV calling; binary in bin folder after ./spades_compile.sh) — https://github.com/1dayac/Blackbird
- **SPAdes (spades_for_blackbird fork)** (Barcode- and long-read-aware modified assembler executed per segment via --spades path) — https://github.com/1dayac/spades_for_blackbird
- **BWA** (Indexes the reference genome used for alignment and segmentation)

## Examples

```
blackbird -b NA24385.GRCh37.phased_possorted_bam.bam -r genome.fa -l long_reads.bam -m long_reads.fastq -o blackbird_hybrid_out -t 32 -s spades_for_blackbird/assembler/spades.py -d
```

## Evaluation signals

- out_50.vcf is produced, position-sorted, and contains only insertion/deletion calls longer than 50 bp.
- With the hidden -d flag, assembly folders containing per-segment contigs remain available for inspection.
- Hybrid-mode results on the HG002 GIAB benchmark should be comparable to PBSV and Sniffles2 run at 10× PacBio HiFi, despite only ~5× long-read coverage (F1 ≈ 0.835 deletions, 0.808 insertions).
- Blackbird binary exists in the bin folder after successfully running ./spades_compile.sh, and the spades.py from spades_for_blackbird resolves via -s or PATH.

## Limitations

- Long-read options must be paired: --long-bam/-l requires --long-read/-m and vice versa.
- Only calls longer than 50 bp (out_50.vcf) are reliable; out.vcf calls 'are not reliable'.
- 5× long-read coverage is sufficient for reported performance, but no numeric threshold is reported for per-segment assembly quality itself.
- SPAdes (and hence Blackbird) was designed for small genomes; Blackbird mitigates this via local windows, but mammalian-scale runs are computationally nontrivial.

## Evidence

- [abstract] Hybrid sliding-window local assembly with barcode and long-read integration: "Instead of relying on whole-genome assembly, Blackbird uses a sliding window approach and synthetic long-read barcode information to assemble local segments, integrating long reads to improve"
- [abstract] 5× coverage requirement with F1 comparable to 10× long-read callers: "Blackbird requires only 5× coverage to achieve F1-scores (0.835 and 0.808 for deletions and insertions) similar to PBSV and Sniffles2 using 10× PacBio Hi-Fi long-read coverage."
- [readme] Paired long-read options and hidden -d assembly-retention flag: "`--long-bam` or `-l` - Position-sorted and indexed long read file (optional, should be paired with long-read option)"
- [methods] Barcode-aware sliding window assembly description: "Blackbird uses a barcode-aware
sliding window approach to assemble small segments of the target genome and sensitively call
SVs in these segments."
- [methods] Modified SPAdes assembler installation requirement: "you should install and compile [SPAdes for blackbird](https://github.com/1dayac/spades_for_blackbird)"
- [methods] Barcode selection criteria for segment assembly: [evidence span withheld — anchor status: not_found]
