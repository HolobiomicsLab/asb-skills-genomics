---
name: slr-barcode-aware-local-assembly
description: Use when calling structural variants (50–10,000 bp indels) from 10X-style
  BX-tagged SLR BAMs, optionally boosted by low-coverage (~5×) PacBio HiFi long reads,
  especially in clinical/low-cost settings where full WGA or deep long-read coverage
  is impractical.
when_to_use_negative:
- Whole-genome de novo assembly output is required — Blackbird deliberately assembles
  only small sliding-window segments, not the full genome.
- Long-read coverage is already deep (e.g., ≥10× PacBio HiFi) — state-of-the-art long-read-only
  callers like Sniffles2 or PBSV are designed for that setting and Blackbird's advantage
  is in low-coverage hybrid data.
- Calls shorter than 50 bp are the target — out_50.vcf only contains indels >50 bp,
  and other calls in out.vcf are explicitly unreliable.
edam_operation: http://edamontology.org/operation_3227
edam_topics:
- http://edamontology.org/topic_0622
tools:
- name: Blackbird
  role: 'Primary SV caller: barcode-aware sliding-window local assembly of SLR (+
    optional low-coverage long reads) to call 50–10,000 bp indels'
  repo: https://github.com/1dayac/Blackbird
- name: SPAdes
  role: Codebase Blackbird was forked from; spades_for_blackbird's spades.py performs
    the local assembly stage via --spades
  repo: https://github.com/1dayac/spades_for_blackbird
- name: BWA
  role: Indexing the reference genome required by the -r/--reference input
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/bioadv/vbaf151
    title: 'Blackbird: structural variant detection using synthetic and low-coverage
      long-reads'
  source_content_hash: sha256:2636d48976476fa1835ee6ab1a489c29bb4b36a2410afe0d8fee3294c3bd7c98
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/1dayac/Blackbird
  tools:
  - BWA
  - Blackbird
  - SPAdes
  edam_operation: http://edamontology.org/operation_3227
  edam_topics:
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# slr-barcode-aware-local-assembly

## Summary

Run Blackbird, an integrated alignment- and local-assembly-based SV caller that exploits BX barcode information in synthetic long reads (SLR) and optional low-coverage long reads, to sensitively detect medium-size insertions and deletions (50–10,000 bp) without whole-genome assembly.

## When to use

Use when calling structural variants (50–10,000 bp indels) from 10X-style BX-tagged SLR BAMs, optionally boosted by low-coverage (~5×) PacBio HiFi long reads, especially in clinical/low-cost settings where full WGA or deep long-read coverage is impractical.

## When NOT to use

- Whole-genome de novo assembly output is required — Blackbird deliberately assembles only small sliding-window segments, not the full genome.
- Long-read coverage is already deep (e.g., ≥10× PacBio HiFi) — state-of-the-art long-read-only callers like Sniffles2 or PBSV are designed for that setting and Blackbird's advantage is in low-coverage hybrid data.
- Calls shorter than 50 bp are the target — out_50.vcf only contains indels >50 bp, and other calls in out.vcf are explicitly unreliable.

## Inputs

- Position-sorted and indexed SLR read BAM with BX tags (e.g., NA24385.GRCh37.phased_possorted_bam.bam)
- BWA-indexed reference genome FASTA (e.g., GRCh37/hg19 genome.fa)
- Optional hybrid inputs: position-sorted indexed long-read BAM plus matching long-read FASTQ (~5× PacBio HiFi)
- Optional regions file with 'chr start end' rows (e.g., ~200 target autosomal genes from Wagner et al. 2022)

## Outputs

- out_50.vcf — position-sorted VCF of insertion and deletion calls longer than 50 bp
- out.vcf — all other (less reliable) calls
- Truvari confusion counts (TP/FP/FN) and precision/recall/F1 metrics (e.g., hybrid_benchmark_results.csv)

## How to apply

Compile Blackbird from its forked SPAdes codebase with ./spades_compile.sh, also installing spades_for_blackbird and pointing --spades at its spades.py. Supply the required position-sorted, indexed SLR BAM with BX tags (--bam), a BWA-indexed reference genome (-r), and an output folder (-o); optionally run hybrid mode by pairing a position-sorted indexed long-read BAM (--long-bam) with the matching long-read FASTQ (--long-read). Restrict calling to targets such as the ~200 challenging medically relevant autosomal genes via a space-separated 'chr start end' regions file (--regions) and increase throughput with multiple threads (-t). Blackbird assembles small genome segments using a barcode-aware sliding window with local assembly (SPAdes-derived) and integrates long reads, emitting out_50.vcf with position-sorted insertion/deletion calls longer than 50 bp. Evaluate calls (e.g., against the HG002 GIAB or Wagner et al. 2022 curated callset) with Truvari, computing precision/recall from TP/FP/FN; expect hybrid mode to reach F1 of 0.835 (deletions) and 0.808 (insertions) at only 5× long-read coverage, and ~86% precision / ~71% recall on the challenging Wagner gene subset.

## Related tools

- **Blackbird** (Primary SV caller: barcode-aware sliding-window local assembly of SLR (+ optional low-coverage long reads) to call 50–10,000 bp indels) — https://github.com/1dayac/Blackbird
- **SPAdes** (Codebase Blackbird was forked from; spades_for_blackbird's spades.py performs the local assembly stage via --spades) — https://github.com/1dayac/spades_for_blackbird
- **BWA** (Indexing the reference genome required by the -r/--reference input)

## Examples

```
blackbird -b NA24385.GRCh37.phased_possorted_bam.bam -r genome.fa -o blackbird_10X_chr1 -g genome.txt -t 32 -l long.bam -m long.fastq -s spades_for_blackbird/assembler/spades.py
```

## Evaluation signals

- Output folder contains out_50.vcf as a position-sorted VCF restricted to indels longer than 50 bp.
- Truvari comparison against the Wagner et al. 2022 curated callset reproduces ~86% precision and ~71% recall for hybrid mode within matching tolerance.
- HG002 GIAB benchmark at 5× long-read coverage yields F1-scores near 0.835 (deletions) and 0.808 (insertions), comparable to PBSV/Sniffles2 at 10×.
- Run with the -d hidden flag and verify the retained per-window assembly contigs used for SV calling are inspectable and consistent with called breakpoints.
- Use more than one thread (-t) and confirm output is complete and reproducible relative to single-thread runs.

## Limitations

- Calls reported in out.vcf (i.e., outside out_50.vcf) are explicitly 'not reliable'.
- Performance degrades on relatively complex events — Blackbird underperforms Sniffles2 and PBSV on the Wagner et al. 2022 challenging medically relevant gene subset.
- SLR BAM must be position-sorted, indexed, and carry BX tags; the long-read BAM and FASTQ must be matching, position-sorted, and indexed pairs.
- SPAdes lineage: designed and tested on small genomes/assemblies — Blackbird mitigates this via local sliding-window assembly rather than whole-genome assembly.

## Evidence

- [abstract] Instead of relying on whole-genome assembly, Blackbird uses a sliding window approach and synthetic long-read barcode information to assemble local segments: "Instead of relying on whole-genome assembly, Blackbird uses a sliding window approach and synthetic long-read barcode information to assemble local segments"
- [abstract] Blackbird requires only 5× coverage to achieve F1-scores (0.835 and 0.808 for deletions and insertions) similar to PBSV and Sniffles2 using 10× PacBio Hi-Fi long-read coverage.: "Blackbird requires only 5× coverage to achieve F1-scores (0.835 and 0.808 for deletions and insertions) similar to PBSV and Sniffles2 using 10× PacBio Hi-Fi long-read coverage."
- [readme] `--bam` or `-b` [required] - position-sorted and indexed SLR Read bam-file with BX tags: "`--bam` or `-b` [required] - position-sorted and indexed SLR Read bam-file with BX tags"
- [readme] `out_50.vcf` is a position sorted VCF file, that contains insertion and deletion calls longer than 50 bp.: "`out_50.vcf` is a position sorted VCF file, that contains insertion and deletion calls longer than 50 bp."
- [methods] Blackbird is a novel integrated alignment- and local-assembly-based algorithm hat employs the barcode information encoded in SLR reads: "Blackbird is a novel integrated alignment- and local-assembly-based algorithm hat employs the barcode information encoded in SLR reads"
