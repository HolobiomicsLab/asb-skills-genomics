---
name: barcode-aware-local-assembly-sv-detection
description: Use when input is a position-sorted, BX-tagged SLR bam aligned to a BWA-indexed
  reference and the goal is sensitive insertion/deletion SV calling, optionally boosted
  by paired PacBio HiFi long-read bam/FASTQ.
when_to_use_negative:
- The question requires whole-genome assembly or SVs outside the 50–10,000 bp medium-size
  range that Blackbird targets
- Input lacks BX barcode tags or is not a position-sorted, indexed bam — required
  inputs for Blackbird's barcode-aware sliding window
- Only short-read data is available and no SLR or long-read data exist; SVs in the
  50–10,000 bp range 'cannot be resolved with short-read sequencing' per the article
edam_operation: http://edamontology.org/operation_3228
edam_topics:
- http://edamontology.org/topic_3175
tools:
- name: Blackbird
  role: 'Primary caller: barcode-aware sliding-window local assembly of SLR (and hybrid
    SLR + long-read) data to call insertions and deletions >50 bp'
  repo: https://github.com/1dayac/Blackbird
- name: SPAdes for blackbird
  role: Forked SPAdes assembly engine compiled and invoked by Blackbird for local
    segment assembly; provide its spades.py via -s if not in PATH
  repo: https://github.com/1dayac/spades_for_blackbird
- name: BWA
  role: Indexing the reference genome; Blackbird requires a BWA-indexed reference
    (-r)
- name: PBSV
  role: State-of-the-art long-read SV caller used as benchmark comparator on the same
    long-read BAMs
- name: Sniffles2
  role: State-of-the-art long-read SV caller used as benchmark comparator on the same
    long-read BAMs
- name: Truvari
  role: 'Benchmarking: compare Blackbird VCF callsets against ground truth to compute
    recall and precision per SV type'
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/bioadv/vbaf151
    title: 'Blackbird: structural variant detection using synthetic and low-coverage
      long-reads'
  source_content_hash: sha256:5797da91043d09a3a4ae3b9c6bd640d45c783a7bfb984a04c379aaca6a79f7ba
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/1dayac/Blackbird
  tools:
  - BWA
  - Blackbird
  - PBSV
  - SPAdes for blackbird
  - Sniffles2
  - Truvari
  edam_operation: http://edamontology.org/operation_3228
  edam_topics:
  - http://edamontology.org/topic_3175
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8181818181818182
n_claims: 11
---

# barcode-aware-local-assembly-sv-detection

## Summary

Detect medium-size structural variants (50–10,000 bp) from synthetic long reads (SLR) alone or hybrid SLR + low-coverage long reads using Blackbird, which performs barcode-aware sliding-window local assembly instead of whole-genome assembly.

## When to use

Use when input is a position-sorted, BX-tagged SLR bam aligned to a BWA-indexed reference and the goal is sensitive insertion/deletion SV calling, optionally boosted by paired PacBio HiFi long-read bam/FASTQ.

## When NOT to use

- The question requires whole-genome assembly or SVs outside the 50–10,000 bp medium-size range that Blackbird targets
- Input lacks BX barcode tags or is not a position-sorted, indexed bam — required inputs for Blackbird's barcode-aware sliding window
- Only short-read data is available and no SLR or long-read data exist; SVs in the 50–10,000 bp range 'cannot be resolved with short-read sequencing' per the article

## Inputs

- Position-sorted and indexed SLR read bam-file with BX barcode tags (-b)
- BWA-indexed reference genome FASTA (-r)
- Optional: position-sorted and indexed long-read bam (-l) paired with a FASTQ of the same long reads (-m) for hybrid mode
- Optional: regions file with 'chr start end' rows
- spades.py from the spades_for_blackbird repository (-s)

## Outputs

- out_50.vcf: position-sorted VCF containing insertion and deletion calls longer than 50 bp
- out.vcf: VCF with all other calls (considered unreliable)
- Per-window assembly contigs if run with the hidden -d flag (assembly folders retained)

## How to apply

Compile Blackbird with ./spades_compile.sh and install SPAdes for blackbird, since Blackbird was forked from the SPAdes codebase. Run blackbird with the required -b (position-sorted indexed SLR bam with BX tags), -r (BWA-indexed reference genome), and -o (output folder); for hybrid mode supply -l (indexed long-read bam) paired with -m (FASTQ of the same long reads), using -t for threads and optionally -s pointing to spades.py from spades_for_blackbird. Blackbird segments the genome via a sliding window, assembles small segments locally using SLR barcode information, integrates the paired long reads, and calls insertions and deletions per segment. Read the primary result from out_50.vcf, a position-sorted VCF restricted to calls longer than 50 bp; treat out.vcf (all other calls) as unreliable. To validate, compare callsets to a ground truth (e.g. the 1437 SVs from Huddleston & Eichler 2016) with Truvari, expecting hybrid-mode F1 of ~0.835 (deletions) and 0.808 (insertions) at 5× PacBio HiFi coverage — comparable to PBSV/Sniffles2 at 10×.

## Related tools

- **Blackbird** (Primary caller: barcode-aware sliding-window local assembly of SLR (and hybrid SLR + long-read) data to call insertions and deletions >50 bp) — https://github.com/1dayac/Blackbird
- **SPAdes for blackbird** (Forked SPAdes assembly engine compiled and invoked by Blackbird for local segment assembly; provide its spades.py via -s if not in PATH) — https://github.com/1dayac/spades_for_blackbird
- **BWA** (Indexing the reference genome; Blackbird requires a BWA-indexed reference (-r))
- **PBSV** (State-of-the-art long-read SV caller used as benchmark comparator on the same long-read BAMs)
- **Sniffles2** (State-of-the-art long-read SV caller used as benchmark comparator on the same long-read BAMs)
- **Truvari** (Benchmarking: compare Blackbird VCF callsets against ground truth to compute recall and precision per SV type)

## Examples

```
./spades_compile.sh && blackbird -b NA24385.GRCh37.phased_possorted_bam.bam -r genome.fa -o blackbird_10X_chr1 -g genome.txt -t 32 -s /home/dmm2017/spades_for_blackbird/assembler/spades.py
```

## Evaluation signals

- out_50.vcf exists in the output folder and is position-sorted, containing only insertion and deletion calls longer than 50 bp
- Truvari comparison against ground-truth SVs yields recall/precision in expected ranges: 85–87% recall for deletions, 75–77% recall for insertions, 90–93% precision across 5×–30× long-read coverage
- Hybrid-mode F1 at 5× long-read coverage approximates 0.835 (deletions) and 0.808 (insertions), matching PBSV/Sniffles2 run at 10× coverage
- Running with hidden flag -d retains assembly folders, allowing inspection of the contigs actually used for SV calling

## Limitations

- Calls in out.vcf (shorter than 50 bp or otherwise excluded) are explicitly 'not reliable'
- The -r flag is doubly defined in the README (reference genome and regions file), so care is needed when supplying a regions file
- Requires BX barcode tags on SLR reads and paired long-read bam + FASTQ in hybrid mode; the long-read options must be supplied together

## Evidence

- [methods] Blackbird is a novel integrated alignment- and local-assembly-based algorithm hat employs the barcode information encoded in SLR reads: "Blackbird is a novel integrated alignment- and local-assembly-based algorithm hat employs the barcode information encoded in SLR reads"
- [methods] Blackbird uses a barcode-aware sliding window approach to assemble small segments of the target genome and sensitively call SVs in these segments.: "Blackbird uses a barcode-aware
sliding window approach to assemble small segments of the target genome and sensitively call
SVs in these segments."
- [abstract] Blackbird requires only 5× coverage to achieve F1-scores (0.835 and 0.808 for deletions and insertions) similar to PBSV and Sniffles2 using 10× PacBio Hi-Fi long-read coverage.: "Blackbird requires only 5× coverage to achieve F1-scores (0.835 and 0.808 for deletions and insertions) similar to PBSV and Sniffles2 using 10× PacBio Hi-Fi long-read coverage."
- [readme] `--bam` or `-b` [required] - position-sorted and indexed SLR Read bam-file with BX tags: "`--bam` or `-b` [required] - position-sorted and indexed SLR Read bam-file with BX tags"
- [readme] `out_50.vcf` is a position sorted VCF file, that contains insertion and deletion calls longer than 50 bp.: "`out_50.vcf` is a position sorted VCF file, that contains insertion and deletion calls longer than 50 bp."
