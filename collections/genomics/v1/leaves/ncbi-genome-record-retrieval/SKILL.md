---
name: ncbi-genome-record-retrieval
description: Use when screening public NCBI genome/plasmid records for Nanopore barcoding
  contamination, e.g. checking whether assembled entries carry remnant native barcode
  (NB02/NB13) or adapter sequences at contig ends.
when_to_use_negative:
- Input is raw Nanopore reads requiring demultiplexing/trimming with a supported kit
  preset — use 'barbell kit -k <kit-name>' instead of a custom-barcode genome screen.
- The genome records are not in FASTA format or no query FASTA of native barcode/flank
  sequences can be constructed following the shared-flank convention.
- Searching for random low-complexity matches rather than structured barcode+flank
  configurations — too many random Fflank matches indicates thresholds are too lenient.
edam_operation: http://edamontology.org/operation_2422
edam_topics:
- http://edamontology.org/topic_0080
- http://edamontology.org/topic_0622
tools:
- name: Barbell
  role: Pattern-aware annotator that locates native barcode/flank matches (Ftag pattern)
    in the downloaded FASTA records via 'barbell annotate', producing the anno.tsv
    table
  repo: github.com/rickbeeloo/barbell
- name: Sassy
  role: SIMD-accelerated approximate string matching engine underpinning Barbell's
    edit-distance search for barcode/flank sequences in the genome records
  repo: github.com/RagnarGrootKoerkamp/sassy
- name: Rust
  role: Language in which Barbell is written; installation from source uses cargo
    with RUSTFLAGS="-C target-cpu=native"
  repo: https://github.com/rickbeeloo/barbell
- name: cargo
  role: 'Installation channel for Barbell from source: RUSTFLAGS="-C target-cpu=native"
    cargo install barbell'
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/bioinformatics/btag349
    title: Barbell reveals and resolves demultiplexing and trimming issues in Nanopore
      data
  source_content_hash: sha256:76dcbeb4a8bee3e89534470f2c338ce240092cc1bbbf2bad584e0ccabe42e76f
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Barbell
  - Rust
  - Sassy
  - cargo
  edam_operation: http://edamontology.org/operation_2422
  edam_topics:
  - http://edamontology.org/topic_0080
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 10
---

# ncbi-genome-record-retrieval

## Summary

Retrieves specific public NCBI genome records (e.g. CP097264.1 for Mycolicibacterium novocastrense and the 8423 bp plasmid CP142556.1) in FASTA format and annotates them with Nanopore native-kit barcode/flank sequences to expose contamination footprints in public database entries.

## When to use

Use when screening public NCBI genome/plasmid records for Nanopore barcoding contamination, e.g. checking whether assembled entries carry remnant native barcode (NB02/NB13) or adapter sequences at contig ends.

## When NOT to use

- Input is raw Nanopore reads requiring demultiplexing/trimming with a supported kit preset — use 'barbell kit -k <kit-name>' instead of a custom-barcode genome screen.
- The genome records are not in FASTA format or no query FASTA of native barcode/flank sequences can be constructed following the shared-flank convention.
- Searching for random low-complexity matches rather than structured barcode+flank configurations — too many random Fflank matches indicates thresholds are too lenient.

## Inputs

- NCBI genome record CP097264.1 (Mycolicibacterium novocastrense) in FASTA format
- NCBI plasmid contig CP142556.1 (8423 bp) in FASTA format
- Query FASTA of native barcoding kit barcode sequences (NB02, NB13, etc.) with shared flanking sequences (shared prefix/suffix flank plus unique barcode region per entry)

## Outputs

- Annotation table anno.tsv with columns including match_type, label, and coordinate columns populated per match
- Inspection summary of match patterns from 'barbell inspect'
- Validated findings: exactly 50 NB02 matches in CP097264.1; at least one NB13 match at the terminal region of CP142556.1

## How to apply

Download the target NCBI records — CP097264.1 (Mycolicibacterium novocastrense) and CP142556.1 (8423 bp plasmid contig) — in FASTA format. Build a query FASTA of native barcoding kit barcodes (including NB02 and NB13) with their shared prefix/suffix flanking sequences, following Barbell's custom-experiment convention. Run 'barbell annotate -q native_barcodes.fasta -b Ftag -i <genome.fasta> -o anno.tsv', which uses Sassy-based edit-distance approximate matching to locate barcode/flank matches, then summarize with 'barbell inspect -i anno.tsv' and examine match coordinates relative to contig ends. The rationale is that the paper detected Nanopore barcoding contamination in hundreds of NCBI entries, some responsible for artificial taxonomic connections, so barcode overhangs at terminal regions indicate assembly artefacts from contaminated native-barcode libraries.

## Related tools

- **Barbell** (Pattern-aware annotator that locates native barcode/flank matches (Ftag pattern) in the downloaded FASTA records via 'barbell annotate', producing the anno.tsv table) — github.com/rickbeeloo/barbell
- **Sassy** (SIMD-accelerated approximate string matching engine underpinning Barbell's edit-distance search for barcode/flank sequences in the genome records) — github.com/RagnarGrootKoerkamp/sassy
- **Rust** (Language in which Barbell is written; installation from source uses cargo with RUSTFLAGS="-C target-cpu=native") — https://github.com/rickbeeloo/barbell
- **cargo** (Installation channel for Barbell from source: RUSTFLAGS="-C target-cpu=native" cargo install barbell)

## Examples

```
barbell annotate -q native_barcodes.fasta -b Ftag -i CP142556.1.fasta -o anno.tsv; barbell inspect -i anno.tsv
```

## Evaluation signals

- anno.tsv reports exactly 50 matches assigned to label NB02 in CP097264.1.
- CP142556.1 shows at least one NB13 match located at the terminal (contig-end) region of the 8423 bp contig.
- match_type, label, and coordinate columns are populated in anno.tsv for the expected hits.
- Self-alignment of the annotated-circular CP142556.1 sequence shows an overlap leaving the NB13 barcode as an overhang.
- No excess of random Fflank matches at non-terminal, unexpected locations (if present, relax/tighten --flank-max-errors and re-inspect).

## Limitations

- Results can slightly differ (<0.01%) between the git version (Sassy V2) and release installations.
- If many reads/matches are missed, the flank edit-distance cutoff may need relaxing (e.g. --flank-max-errors 5), but this risks random matches appearing as Fflank hits.
- Only rapid and native kits were thoroughly tested; unexpected annotations with other kits should be reported.

## Evidence

- [full_text] Download the NCBI genome records CP097264.1 (Mycolicibacterium novocastrense) and CP142556.1 (8423 bp plasmid contig) in FASTA format.: [evidence span withheld — anchor status: not_found]
- [abstract] Nanopore barcoding contamination is widespread in public databases: "with Nanopore sequences detected in hundreds of NCBI entries, some of which are responsible for artificial taxonomic connections."
- [readme] annotate command: "Run `annotate` to find matches in reads and output an annotation table (TSV)"
- [full_text] custom experiment query design: [evidence span withheld — anchor status: not_found]
