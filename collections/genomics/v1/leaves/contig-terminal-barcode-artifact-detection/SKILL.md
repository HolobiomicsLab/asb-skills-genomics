---
name: contig-terminal-barcode-artifact-detection
description: Use when screening assembled genomes or plasmid contigs (FASTA) from
  NCBI for residual native-barcode barcode/flank sequences, especially matches near
  contig ends that indicate assembler artefacts or cross-sample contamination.
when_to_use_negative:
- Input is raw read data rather than an assembled FASTA — use Barbell's 'kit' demultiplexing
  workflow ('barbell kit -k <kit-name>') instead
- Barcodes belong to rapid barcoding kits or non-native kits whose sequences are not
  in the query FASTA; native kit presets/flanks will not match
- You expect fully intact barcodes rather than error-laden remnants — edit-distance
  matching against native barcode+flank queries is specifically for partial/remnant
  artefacts
edam_operation: http://edamontology.org/operation_3192
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_0102
tools:
- name: Barbell
  role: Provides the 'annotate' command with pattern-aware, Sassy-based edit-distance
    search used to locate barcode/flank matches in genome FASTA records, plus 'inspect'
    for summarizing match patterns
  repo: https://github.com/rickbeeloo/barbell
- name: Sassy
  role: SIMD-accelerated approximate string matching backend that powers Barbell's
    edit-distance barcode search (Sassy V2 makes the git version ~5x faster)
  repo: https://github.com/RagnarGrootKoerkamp/sassy
- name: cargo
  role: 'Build/install path for Barbell from source with SIMD enabled: RUSTFLAGS="-C
    target-cpu=native" cargo install barbell'
  repo: https://github.com/rickbeeloo/barbell
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/bioinformatics/btag349
    title: Barbell reveals and resolves demultiplexing and trimming issues in Nanopore
      data
  source_content_hash: sha256:181fd37ba055a80fdf13fcaa69ffd14557e406459b2a369423e305bbc374d3b5
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/rickbeeloo/barbell
  tools:
  - Barbell
  - Sassy
  - cargo
  edam_operation: http://edamontology.org/operation_3192
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_0102
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# Contig-terminal barcode artifact detection

## Summary

Detects remnant Nanopore native-barcoding barcode sequences (e.g., NB02, NB13 with their shared flanking sequences) left inside public genome assemblies by running Barbell's Sassy-backed edit-distance annotate step on downloaded NCBI FASTA records, revealing contamination footprints such as the NB13 remnant at the end of the 8423 bp plasmid CP142556.1.

## When to use

Use when screening assembled genomes or plasmid contigs (FASTA) from NCBI for residual native-barcode barcode/flank sequences, especially matches near contig ends that indicate assembler artefacts or cross-sample contamination.

## When NOT to use

- Input is raw read data rather than an assembled FASTA — use Barbell's 'kit' demultiplexing workflow ('barbell kit -k <kit-name>') instead
- Barcodes belong to rapid barcoding kits or non-native kits whose sequences are not in the query FASTA; native kit presets/flanks will not match
- You expect fully intact barcodes rather than error-laden remnants — edit-distance matching against native barcode+flank queries is specifically for partial/remnant artefacts

## Inputs

- NCBI genome records in FASTA format (e.g., CP097264.1, CP142556.1)
- Custom query FASTA of native barcoding kit barcode sequences with shared flanking prefix/suffix (NB02, NB13)

## Outputs

- Annotation TSV (anno.tsv) with columns including match_type, label, rel_dist_to_end, bar_start/bar_end, flank_cost, barcode_cost, strand
- Inspect summary of pattern groups per contig
- Optional per-contig output files showing match coordinates relative to contig ends

## How to apply

Download the suspect NCBI records in FASTA (e.g., CP097264.1 from Mycolicibacterium novocastrense and the 8423 bp plasmid CP142556.1). Build a custom query FASTA of native barcoding kit barcodes (NB02, NB13) following Barbell's custom-experiment convention: a shared prefix/suffix flank with a unique barcode region per entry. Run 'barbell annotate' with the '-b Ftag' flag and Barbell's Sassy-based approximate (edit-distance) search so partial/remnant matches are found even with sequencing errors, then summarize with 'barbell inspect -i anno.tsv' and optionally write per-contig patterns via -o to inspect match positions relative to contig ends. Validate by checking that match_type, label, and coordinate columns are populated and that terminal-region matches (e.g., an NB13 overhang consistent with an incorrectly circularized plasmid) are present.

## Related tools

- **Barbell** (Provides the 'annotate' command with pattern-aware, Sassy-based edit-distance search used to locate barcode/flank matches in genome FASTA records, plus 'inspect' for summarizing match patterns) — https://github.com/rickbeeloo/barbell
- **Sassy** (SIMD-accelerated approximate string matching backend that powers Barbell's edit-distance barcode search (Sassy V2 makes the git version ~5x faster)) — https://github.com/RagnarGrootKoerkamp/sassy
- **cargo** (Build/install path for Barbell from source with SIMD enabled: RUSTFLAGS="-C target-cpu=native" cargo install barbell) — https://github.com/rickbeeloo/barbell

## Examples

```
barbell annotate -q native_barcodes.fasta -b Ftag -i CP142556.1.fasta -o anno.tsv && barbell inspect -i anno.tsv
```

## Evaluation signals

- anno.tsv reports exactly 50 matches assigned to label NB02 in CP097264.1
- anno.tsv reports at least one NB13 match at the terminal region of the 8423 bp CP142556.1 contig
- match_type, label, and coordinate columns are populated (non-empty) for all reported matches
- Self-alignment of the annotated-circular CP142556.1 sequence reproduces the overlap that leaves the NB13 barcode as an overhang
- 'barbell inspect -i anno.tsv' pattern groups are consistent with terminal-localized matches rather than random internal Fflank hits

## Limitations

- Remnant matches are partial and error-prone, so a conservative flank edit-distance cutoff may miss true artefacts; relaxing --flank-max-errors risks random Fflank matches and results must always be inspected afterwards
- The current git version of Barbell (using Sassy V2) can differ slightly from releases (<0.01% of results)
- x64 builds without AVX2/NEON fall back to slower scalar/128-bit SIMD paths unless compiled with RUSTFLAGS="-C target-cpu=native"

## Evidence

- [results] an 8423 bp plasmid (CP142556.1) carries a remnant NB13 barcode at the contig end, and self-alignment of the annotated-circular sequence shows an overlap leaving the barcode as an overhang: [evidence span withheld — anchor status: not_found]
- [methods] Run `annotate` to find matches in reads and output an annotation table (TSV): "Run `annotate` to find matches in reads and output an annotation table (TSV)"
- [methods] Summarize patterns across the annotation file:

bash
barbell inspect -i anno.tsv: "Summarize patterns across the annotation file:

bash
barbell inspect -i anno.tsv"
- [abstract] with Nanopore sequences detected in hundreds of NCBI entries, some of which are responsible for artificial taxonomic connections: "with Nanopore sequences detected in hundreds of NCBI entries, some of which are responsible for artificial taxonomic connections."
- [readme] Current git version is around 5x faster than the releases by using [Sassy V2](https://github.com/RagnarGrootKoerkamp/sassy): "Current git version is around 5x faster than the releases by using [Sassy V2](https://github.com/RagnarGrootKoerkamp/sassy)"
