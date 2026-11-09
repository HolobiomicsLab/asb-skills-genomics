---
name: edit-distance-approximate-matching
description: Use when checking whether short (~20–100 nt) Nanopore rapid-barcoding
  or adapter sequences occur in assembled contigs or read sets despite sequencing
  errors, e.g. screening GenBank assemblies for barcode contamination.
when_to_use_negative:
- Reads are already demultiplexed and trimmed and you only need downstream analysis
  — no fuzzy barcode search is needed
- Queries are longer than Sassy's intended range (it works well for patterns up to
  length ~1000, best at 20–100 nt DNA)
- You need exact matching only — edit-distance tolerance would inflate counts with
  spurious near-matches
edam_operation: http://edamontology.org/operation_0346
edam_topics:
- http://edamontology.org/topic_0102
tools:
- name: Sassy
  role: SIMD-accelerated edit-distance approximate string matching engine used by
    Barbell to locate barcode/flank patterns in sequences; used here to scan FASTA
    assemblies for barcode hits
  repo: https://github.com/RagnarGrootKoerkamp/sassy
- name: Barbell
  role: Pattern-aware demultiplexer supplying built-in kit presets (e.g. SQK-RBK114-96)
    that define the barcode and flank query sequences, and annotate/filter/trim workflow
    for read-level scanning
  repo: https://github.com/rickbeeloo/barbell
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/bioinformatics/btag349
    title: Barbell reveals and resolves demultiplexing and trimming issues in Nanopore
      data
  source_content_hash: sha256:6e203f8c78ba0e180889ed4c4dc9cfa5561e23cc8a75e9eaf5e6d42d3d764554
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/RagnarGrootKoerkamp/sassy
  tools:
  - Barbell
  - Sassy
  - cargo
  edam_operation: http://edamontology.org/operation_0346
  edam_topics:
  - http://edamontology.org/topic_0102
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 18
---

# edit-distance-approximate-matching

## Summary

Fuzzy (edit-distance) search of short Nanopore barcode and flank sequences against FASTA assemblies or reads, using Sassy-accelerated approximate matching as implemented in Barbell, to detect barcode contamination in public NCBI records.

## When to use

Use when checking whether short (~20–100 nt) Nanopore rapid-barcoding or adapter sequences occur in assembled contigs or read sets despite sequencing errors, e.g. screening GenBank assemblies for barcode contamination.

## When NOT to use

- Reads are already demultiplexed and trimmed and you only need downstream analysis — no fuzzy barcode search is needed
- Queries are longer than Sassy's intended range (it works well for patterns up to length ~1000, best at 20–100 nt DNA)
- You need exact matching only — edit-distance tolerance would inflate counts with spurious near-matches

## Inputs

- Nanopore kit barcode and flank sequences from a kit preset (e.g. SQK-RBK114-96 via `barbell kit -k SQK-RBK114-96`)
- GenBank/NCBI assembly records in FASTA format (e.g. CP131573.1, CP165501.1)
- Optionally Nanopore FASTQ reads for annotate-mode scanning

## Outputs

- Per-accession match counts, positions, and orientations of barcode/flank hits (e.g. barcode_verification.tsv)
- Barbell annotation TSV with columns such as read_id, bar_start, bar_end, match_type, flank_cost, barcode_cost, label, strand, cuts

## How to apply

Extract the barcode and flank query sequences from the Nanopore kit preset — e.g. run `barbell kit -k SQK-RBK114-96` to obtain the SQK-RBK114-96 barcode and flank sequences — and download the query assemblies (e.g. CP131573.1, CP165501.1) as FASTA. Scan the assembly with Sassy-style edit-distance approximate matching, keeping Barbell's default flank error tolerance so only near-exact barcode/flank configurations are accepted. Count and record per-accession matches and coordinates: for example, barcode BC86 occurs 11 times in the 1.43 Mb Photobacterium leiognathi SV5.1 contig CP131573.1, while BC10 and BC09 overhangs sit at opposite ends of the E. coli plasmid contig CP165501.1, the latter indicating barcode bleeding. Write per-accession match counts and positions to a TSV (e.g. barcode_verification.tsv). If too few hits are found, relax the flank threshold (e.g. --flank-max-errors 5) but re-inspect, since over-loose cutoffs yield random matches that appear as Fflank hits.

## Related tools

- **Sassy** (SIMD-accelerated edit-distance approximate string matching engine used by Barbell to locate barcode/flank patterns in sequences; used here to scan FASTA assemblies for barcode hits) — https://github.com/RagnarGrootKoerkamp/sassy
- **Barbell** (Pattern-aware demultiplexer supplying built-in kit presets (e.g. SQK-RBK114-96) that define the barcode and flank query sequences, and annotate/filter/trim workflow for read-level scanning) — https://github.com/rickbeeloo/barbell

## Examples

```
barbell annotate --kit SQK-RBK114-96 -i pass_sample.fastq -t 10 -o anno.tsv
```

## Evaluation signals

- BC86 matches in CP131573.1 equal 11, matching the paper's reported count for the 1.43 Mb P. leiognathi SV5.1 contig
- BC10 and BC09 matches in CP165501.1 occur at opposite contig termini (overhangs at both ends), confirming barcode bleeding
- Reported flank_cost values stay within the default flank-based edit-distance cutoff; relaxing to --flank-max-errors 5 must not introduce random Fflank matches at non-end positions
- Output TSV follows Barbell's annotate schema (read_id, read_len, rel_dist_to_end, bar_start, bar_end, match_type, flank_cost, barcode_cost, label, strand, cuts)
- Total contaminated-read counts after filtering should agree with the paper's scale (hundreds of NCBI entries carry Nanopore barcode contamination)

## Limitations

- Loosening --flank-max-errors can produce random matches, visible as Fflank annotations at unexpected (non-read-end) locations; thresholds must be re-inspected after adjustment
- The git version of Barbell (using Sassy V2, ~5x faster) can give slightly different results (<0.01%) than cargo/conda releases
- Barbell presets were thoroughly tested only for rapid and native Nanopore kits; other kits may show unexpected annotations and should be reported upstream

## Evidence

- [readme] Sassy is a library and tool for searching short strings in texts, a problem that goes by many names: - approximate string matching,: [evidence span withheld — anchor status: external_doc]
- [readme] The motivating application is searching short (length 20 to 100) DNA sequences in a human genome or e.g. in a set of reads.: [evidence span withheld — anchor status: external_doc]
- [readme] This uses a conservative flank-based edit-distance cutoff. If many reads are missed during `annotate`, you can relax the flank error threshold, for example:: "This uses a conservative flank-based edit-distance cutoff. If many reads are missed during `annotate`, you can relax the flank error threshold, for example:"
- [methods] Current git version is around 5x faster than the releases by using [Sassy V2](https://github.com/RagnarGrootKoerkamp/sassy): "Current git version is around 5x faster than the releases by using [Sassy V2](https://github.com/RagnarGrootKoerkamp/sassy)"
- [abstract] with Nanopore sequences detected in hundreds of NCBI entries, some of which are responsible for artificial taxonomic connections.: "with Nanopore sequences detected in hundreds of NCBI entries, some of which are responsible for artificial taxonomic connections."
