---
name: nanopore-barcode-annotation
description: Use when Nanopore FASTQ reads need demultiplexing or barcode attachment
  auditing (rapid or native kits like SQK-RBK114-96/SQK-NBD114-96), or when detecting
  complex/fused barcode configurations that simpler demultiplexers mis-handle.
when_to_use_negative:
- Input is not Nanopore barcode data (e.g. already-demultiplexed FASTQ or a feature/count
  table) — there are no barcodes to annotate
- A simple kit demultiplex alone suffices and runtime matters greatly — --use-extended
  is ~3x slower and only needed for fusion-point/artefact searches or consensus-grade
  quality
- Screening contamination at loci requiring exact-match semantics rather than approximate
  edit-distance matching, where Sassy-based fuzzy search could yield random Fflank
  false positives
edam_operation: http://edamontology.org/operation_3223
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3168
tools:
- name: Barbell
  role: Performs pattern-aware annotate/inspect/kit demultiplexing; its Sassy-based
    edit-distance search locates barcode and flank matches in FASTQ reads or genome
    FASTA records
  repo: https://github.com/rickbeeloo/barbell
- name: Sassy
  role: SIMD-accelerated approximate string matching engine underlying Barbell's barcode/flank
    matching; current git version (Sassy V2) makes Barbell ~5x faster than releases
  repo: https://github.com/RagnarGrootKoerkamp/sassy
- name: Dorado
  role: Baseline demultiplexer whose failures (partial trimming, adapter contamination,
    missed complex configurations) motivate pattern-aware annotation
  repo: https://github.com/nanoporetech/dorado
provenance:
  source_task_ids:
  - task_002
  - task_006
  source_papers:
  - doi: 10.1093/bioinformatics/btag349
    title: Barbell reveals and resolves demultiplexing and trimming issues in Nanopore
      data
  source_content_hash: sha256:8f05deb4fdc733105d467a37060095dd41e1cc36be94cafc5a6b7ade2c3872fa
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/rickbeeloo/barbell
  tools:
  - Barbell
  - Dorado
  - Sassy
  - cargo
  edam_operation: http://edamontology.org/operation_3223
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3168
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 10
---

# nanopore-barcode-annotation

## Summary

Pattern-aware annotation of Nanopore barcode/flank attachments using Barbell's Sassy-based edit-distance search, producing a per-read TSV (anno.tsv) of barcode labels, match types, flank/barcode costs, strands, and cut coordinates for downstream inspect/filter/trim steps.

## When to use

Use when Nanopore FASTQ reads need demultiplexing or barcode attachment auditing (rapid or native kits like SQK-RBK114-96/SQK-NBD114-96), or when detecting complex/fused barcode configurations that simpler demultiplexers mis-handle.

## When NOT to use

- Input is not Nanopore barcode data (e.g. already-demultiplexed FASTQ or a feature/count table) — there are no barcodes to annotate
- A simple kit demultiplex alone suffices and runtime matters greatly — --use-extended is ~3x slower and only needed for fusion-point/artefact searches or consensus-grade quality
- Screening contamination at loci requiring exact-match semantics rather than approximate edit-distance matching, where Sassy-based fuzzy search could yield random Fflank false positives

## Inputs

- Nanopore FASTQ reads (e.g. pass_sample.fastq), optionally gzipped
- Barcoding kit preset name (e.g. SQK-RBK114-96, SQK-NBD114-96) or custom query FASTA of barcode sequences with shared flanks
- Optional assembled genome/contig FASTA (e.g. NCBI records CP097264.1, CP142556.1) for contamination screening
- Thread count and threshold parameters (--flank-max-errors, --min-score-diff, --use-extended)

## Outputs

- anno.tsv annotation table with per-read match_type, label, flank_cost, barcode_cost, coordinates, strand, and cuts columns
- inspect summary of top pattern groups (Ftag, Fflank, double left-end fusion patterns like Ftag[fw,*,@left(0..250)]__Ftag[fw,*,@prev_left(0..250)])
- Optional per-read/per-contig pattern output via -o for examining match positions relative to read/contig ends

## How to apply

Run 'barbell annotate' with a kit preset (--kit SQK-RBK114-96 or SQK-NBD114-96) or a custom query FASTA (shared prefix/suffix flanks plus unique barcode regions) over the reads with multiple threads (e.g. -t 10) to emit anno.tsv with columns read_id, read_len, rel_dist_to_end, match_type, flank_cost, barcode_cost, label, strand, cuts. Summarize attachment patterns with 'barbell inspect -i anno.tsv' (default top-10 groups, raise with -n) to see which reads follow the expected single-barcode configuration versus complex attachments. For custom experiments use edit-distance matching (Sassy-based approximate search) and tune --flank-max-errors if many reads are missed, lowering it if Fflank matches appear at random locations; adjust --min-score-diff for non-random flank matches. Use --use-extended (about 3x slower) when fused double-barcode artefacts must be found, and --maximize with 'barbell kit' unless an ultra-strict configuration is required. Annotate can also locate barcode remnants in assembled genome FASTA records, as shown by NB02/NB13 contamination detection in NCBI entries CP097264.1 (50 NB02 matches) and CP142556.1 (NB13 at contig end).

## Related tools

- **Barbell** (Performs pattern-aware annotate/inspect/kit demultiplexing; its Sassy-based edit-distance search locates barcode and flank matches in FASTQ reads or genome FASTA records) — https://github.com/rickbeeloo/barbell
- **Sassy** (SIMD-accelerated approximate string matching engine underlying Barbell's barcode/flank matching; current git version (Sassy V2) makes Barbell ~5x faster than releases) — https://github.com/RagnarGrootKoerkamp/sassy
- **Dorado** (Baseline demultiplexer whose failures (partial trimming, adapter contamination, missed complex configurations) motivate pattern-aware annotation) — https://github.com/nanoporetech/dorado

## Examples

```
barbell annotate --kit SQK-RBK114-96 -i pass_sample.fastq -t 10 -o anno.tsv
```

## Evaluation signals

- anno.tsv rows carry populated match_type, label, and coordinate columns, with flank_cost/barcode_cost reflecting edit distances (intact barcode+flank matches ~3 vs fused second barcode ~7 in the RBK114-96 fusion test)
- Matches occur near read ends (rel_dist_to_end small); matches at unexpected random locations indicate an Fflank threshold problem and require lowering --flank-max-errors
- On known contamination benchmarks, CP097264.1 yields exactly 50 NB02 matches and CP142556.1 shows at least one NB13 match at the terminal region of the 8423 bp contig
- Every synthetic double-barcode read is assigned both barcode labels matching simulation ground truth, and extended mode detects fusion constructs the standard configuration misses
- inspect reports the expected dominant single-barcode pattern group, with complex attachments (e.g. 17% of rapid barcoding reads in the paper) separated into distinct groups

## Limitations

- Results can slightly differ (<0.01%) between the faster git version (Sassy V2) and released versions
- --use-extended fusion-point search is ~3x slower than the standard configuration
- Random Fflank matches can appear if --flank-max-errors is too permissive; thresholds must be re-inspected after adjustment
- Rapid and native kits were thoroughly tested, but other kits (RNA, Twist are exceptions to presets) may show unexpected annotations and require issue reports

## Evidence

- [methods] Run `annotate` to find matches in reads and output an annotation table (TSV): "Run `annotate` to find matches in reads and output an annotation table (TSV)"
- [readme] README kits note: "there is an option `--use-extended` which enables searches for fusion points, and other artefacts. This is around 3 times slower"
- [readme] README Fflank guidance: "Slightly increase `--flank-max-errors` (the automatically derived cutoff is reported)"
- [abstract] Abstract contamination finding: "with Nanopore sequences detected in hundreds of NCBI entries, some of which are responsible for artificial taxonomic connections"
- [methods] Sassy speedup: "Current git version is around 5x faster than the releases by using [Sassy V2](https://github.com/RagnarGrootKoerkamp/sassy)"
