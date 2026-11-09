---
name: native-barcoding-kit-adapter-detection
description: Use when screening assembled FASTA genomes or contigs (especially NCBI
  records of Nanopore-derived assemblies) for leftover native barcoding kit barcode/adapters
  that signal contamination in the deposited sequence.
when_to_use_negative:
- Input is raw FASTQ reads from a kit experiment — use Barbell's 'kit' demultiplexing
  mode instead of custom-experiment annotation against assemblies.
- Searching for rapid-kit (SQK-RBK114-96) adapters where a built-in kit preset exists
  — prefer the preset rather than hand-built query FASTAs.
- Biological sequence features (repeats, integrons) that resemble barcodes only by
  chance — Fflank matches at random, non-terminal locations indicate spurious hits,
  not contamination footprints.
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_0080
tools:
- name: Barbell
  role: Pattern-aware annotate/inspect engine used to locate native barcode+flank
    matches in assembled FASTA records via the custom-experiment workflow.
  repo: https://github.com/rickbeeloo/barbell
- name: Sassy
  role: SIMD-accelerated approximate string matching backend powering Barbell's edit-distance
    barcode search.
  repo: https://github.com/RagnarGrootKoerkamp/sassy
- name: cargo
  role: Rust build tool used to install Barbell from source with RUSTFLAGS="-C target-cpu=native".
  repo: https://github.com/rickbeeloo/barbell
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/bioinformatics/btag349
    title: Barbell reveals and resolves demultiplexing and trimming issues in Nanopore
      data
  source_content_hash: sha256:9fe8469c1a3ae210cc845a5157c4cb8117aa8dda7d727e4836e9c1623b106981
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/rickbeeloo/barbell
  tools:
  - Barbell
  - Sassy
  - cargo
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_0080
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 12
---

# native-barcoding-kit-adapter-detection

## Summary

Detects remnant Nanopore native barcoding kit barcode sequences (e.g., NB02, NB13) left as contamination footprints in assembled genome records by approximate edit-distance search with Barbell's annotate/inspect commands. This lets a practitioner verify whether public NCBI entries (e.g., CP097264.1, CP142556.1) carry barcode artifacts rather than genuine biological sequence.

## When to use

Use when screening assembled FASTA genomes or contigs (especially NCBI records of Nanopore-derived assemblies) for leftover native barcoding kit barcode/adapters that signal contamination in the deposited sequence.

## When NOT to use

- Input is raw FASTQ reads from a kit experiment — use Barbell's 'kit' demultiplexing mode instead of custom-experiment annotation against assemblies.
- Searching for rapid-kit (SQK-RBK114-96) adapters where a built-in kit preset exists — prefer the preset rather than hand-built query FASTAs.
- Biological sequence features (repeats, integrons) that resemble barcodes only by chance — Fflank matches at random, non-terminal locations indicate spurious hits, not contamination footprints.

## Inputs

- FASTA genome records from NCBI (e.g., CP097264.1, CP142556.1)
- Custom query FASTA of native barcoding kit barcode sequences (NB02, NB13) with shared prefix/suffix flank sequences

## Outputs

- Annotation TSV (anno.tsv) with per-contig match_type, label, barcode/flank coordinates, costs, strand, and cuts columns
- Summarized pattern report from 'barbell inspect' showing match counts per barcode label and their positions relative to contig ends

## How to apply

Download the suspect NCBI records (e.g., CP097264.1 for Mycolicibacterium novocastrense and the 8423 bp plasmid CP142556.1) in FASTA format. Build a query FASTA of the native barcoding kit barcodes (including NB02 and NB13) each with their shared flanking sequences, following Barbell's custom-experiment convention of a shared prefix/suffix flank plus a unique barcode region per entry. Run 'barbell annotate' with the Ftag pattern and Sassy-based edit-distance approximate search against each genome to produce an annotation TSV (anno.tsv) with match_type, label, coordinate, and cost columns. Summarize with 'barbell inspect -i anno.tsv' and inspect match positions relative to contig ends; for CP097264.1 the expected result is exactly 50 NB02 matches, and CP142556.1 should show a remnant NB13 barcode at the contig end, consistent with a circular plasmid self-alignment leaving the barcode as an overhang. If matches are missed, relax --flank-max-errors (e.g., to 5), but re-inspect afterward since random matches appear as Fflank hits at non-terminal positions.

## Related tools

- **Barbell** (Pattern-aware annotate/inspect engine used to locate native barcode+flank matches in assembled FASTA records via the custom-experiment workflow.) — https://github.com/rickbeeloo/barbell
- **Sassy** (SIMD-accelerated approximate string matching backend powering Barbell's edit-distance barcode search.) — https://github.com/RagnarGrootKoerkamp/sassy
- **cargo** (Rust build tool used to install Barbell from source with RUSTFLAGS="-C target-cpu=native".) — https://github.com/rickbeeloo/barbell

## Examples

```
barbell annotate -q native_barcodes.fasta -b Ftag -i CP142556.1.fasta -o anno.tsv && barbell inspect -i anno.tsv
```

## Evaluation signals

- The annotation table (anno.tsv) reports exactly 50 matches assigned to label NB02 in CP097264.1.
- At least one NB13 match appears at the terminal region of the 8423 bp CP142556.1 contig.
- match_type, label, and coordinate columns are populated (non-empty) in anno.tsv rows.
- Match positions sit at contig ends (low rel_dist_to_end), consistent with a circular-plasmid self-alignment leaving the barcode as an overhang.
- No excess Fflank matches at random internal positions after threshold tuning.

## Limitations

- Relaxing --flank-max-errors (e.g., to 5) can introduce random matches that show up as Fflank hits at unexpected locations; results must always be re-inspected after threshold changes.
- Barbell's git (Sassy V2) version is around 5x faster than releases but results can slightly differ (<0.01%).
- Detection depends on the remnant barcode being intact enough for the edit-distance cutoff; heavily degraded barcode remnants may be missed.

## Evidence

- [results] an 8423 bp plasmid (CP142556.1) carries a remnant NB13 barcode at the contig end, and self-alignment of the annotated-circular sequence shows an overlap leaving the barcode as an overhang: [evidence span withheld — anchor status: not_found]
- [abstract] Nanopore barcoding contamination is widespread in public databases, detected in hundreds of NCBI entries, some responsible for artificial taxonomic connections: "with Nanopore sequences detected in hundreds of NCBI entries, some of which are responsible for artificial taxonomic connections."
- [methods] Run `annotate` to find matches in reads and output an annotation table (TSV): "Run `annotate` to find matches in reads and output an annotation table (TSV)"
- [abstract] Barbell reduced contaminated reads from >400 000 (Dorado/Flexiplex) to 166 (99.96% reduction): "Barbell reduced contaminated reads from >400 000 (Dorado/Flexiplex) to 166 (99.96% reduction)"
- [readme] Too many `Fflank` matches: Check `annotation.tsv` to see if matches occur at unexpected locations (not near read ends): "Check `annotation.tsv` to see if matches occur at unexpected locations (not near read ends)."
