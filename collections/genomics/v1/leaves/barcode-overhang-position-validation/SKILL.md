---
name: barcode-overhang-position-validation
description: Use when screening assembled contigs (e.g. GenBank/NCBI FASTA records)
  for leftover SQK-RBK114-96 rapid barcodes or flanks, or when validating a claim
  that an assembly carries barcode overhangs at specific positions.
when_to_use_negative:
- Input is raw Nanopore reads rather than assembled contigs — use Barbell's annotate/inspect/filter/trim
  read workflow instead
- The barcode kit is not a tested rapid/native DNA kit (e.g. RNA or Twist kits), where
  Barbell presets were not thoroughly validated
- You need high-sensitivity demultiplexing of a sequencing run rather than detecting
  contamination already baked into a published assembly
edam_operation: http://edamontology.org/operation_3197
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_0080
tools:
- name: Barbell
  role: Provides the kit preset (`barbell kit -k SQK-RBK114-96`) that yields the barcode
    and flank query sequences used for the search
  repo: https://github.com/rickbeeloo/barbell
- name: Sassy
  role: SIMD-accelerated edit-distance approximate matcher used to scan assemblies
    for barcode/flank occurrences
  repo: https://github.com/RagnarGrootKoerkamp/sassy
- name: cargo
  role: 'Installs Barbell from source with native SIMD flags: RUSTFLAGS="-C target-cpu=native"
    cargo install barbell'
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/bioinformatics/btag349
    title: Barbell reveals and resolves demultiplexing and trimming issues in Nanopore
      data
  source_content_hash: sha256:26d1871c09e8f2d2f884e8b69e2947cde9d12e43715a5bdf49810e65a1cb2880
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/rickbeeloo/barbell
  tools:
  - Barbell
  - Sassy
  - cargo
  edam_operation: http://edamontology.org/operation_3197
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_0080
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 5
---

# barcode-overhang-position-validation

## Summary

A verification skill that detects residual Nanopore rapid-barcode sequences (barcodes and their flanks) inside public genome assemblies using edit-distance approximate matching, and reports match counts and positions to confirm contamination (e.g. barcode bleeding with overhangs at opposite contig ends).

## When to use

Use when screening assembled contigs (e.g. GenBank/NCBI FASTA records) for leftover SQK-RBK114-96 rapid barcodes or flanks, or when validating a claim that an assembly carries barcode overhangs at specific positions.

## When NOT to use

- Input is raw Nanopore reads rather than assembled contigs — use Barbell's annotate/inspect/filter/trim read workflow instead
- The barcode kit is not a tested rapid/native DNA kit (e.g. RNA or Twist kits), where Barbell presets were not thoroughly validated
- You need high-sensitivity demultiplexing of a sequencing run rather than detecting contamination already baked into a published assembly

## Inputs

- Assembly contig records in FASTA format downloaded from NCBI/GenBank (e.g. CP131573.1, CP165501.1)
- Rapid-barcode kit preset barcode and flank sequences (SQK-RBK114-96) extracted via Barbell

## Outputs

- Per-accession barcode/flank match counts and coordinates (positions and orientations) written to a TSV such as barcode_verification.tsv
- List of matched accessions/records, e.g. BC86 x11 in CP131573.1; BC10 and BC09 overhangs at opposite ends of CP165501.1

## How to apply

Download the suspect assemblies (e.g. CP131573.1 and CP165501.1) from NCBI in FASTA format and extract the barcode plus flank sequences for the relevant kit preset with `barbell kit -k SQK-RBK114-96`. Scan each assembly with Sassy-style edit-distance approximate matching, keeping the default flank error tolerance as implemented in Barbell, and record every barcode and flank hit with its coordinates and orientation. Count matches per accession and check positional logic: 11 BC86 hits scattered across the 1.43 Mb Photobacterium leiognathi SV5.1 contig (CP131573.1) indicate contamination, while BC10 and BC09 overhangs found at opposite termini of the E. coli plasmid contig CP165501.1 indicate barcode bleeding during the original rapid-barcode run. Write per-accession match counts and coordinates to a TSV (e.g. barcode_verification.tsv) so results are reproducible and comparable against the paper's reported findings.

## Related tools

- **Barbell** (Provides the kit preset (`barbell kit -k SQK-RBK114-96`) that yields the barcode and flank query sequences used for the search) — https://github.com/rickbeeloo/barbell
- **Sassy** (SIMD-accelerated edit-distance approximate matcher used to scan assemblies for barcode/flank occurrences) — https://github.com/RagnarGrootKoerkamp/sassy
- **cargo** (Installs Barbell from source with native SIMD flags: RUSTFLAGS="-C target-cpu=native" cargo install barbell)

## Examples

```
barbell kit -k SQK-RBK114-96 -i reads.fastq -o output_folder --maximize
```

## Evaluation signals

- BC86 occurs exactly 11 times in the 1.43 Mb P. leiognathi SV5.1 contig CP131573.1
- CP165501.1 shows BC10 and BC09 overhangs located at opposite contig ends, consistent with barcode bleeding
- Reported counts and coordinates in barcode_verification.tsv reproduce the paper's reported findings
- Flank matches respect the default flank error tolerance; Fflank-type matches at random locations would signal over-relaxed thresholds
- Match coordinates and orientations are recorded per accession so hits can be re-verified independently

## Limitations

- Barbell presets were thoroughly tested for rapid and native kits only; other kits may produce unexpected annotations and should be reported as issues
- Changing --flank-max-errors (e.g. to 5) can introduce random Fflank matches, so thresholds must be inspected after adjustment
- Installing from GitHub (git version with Sassy V2) can give slightly different results (<0.01% of matches) than release versions

## Evidence

- [abstract] Nanopore barcoding contamination is widespread in public databases, detected in hundreds of NCBI entries, some responsible for artificial taxonomic connections: "with Nanopore sequences detected in hundreds of NCBI entries, some of which are responsible for artificial taxonomic connections."
- [readme] Barbell includes built-in kit presets for many Nanopore kits: "Barbell includes built-in kit *presets* for many Nanopore kits (most DNA kits; RNA and Twist kits are an exception)."
- [readme] Sassy approximate string matching scope: "Sassy is a library and tool for searching short strings in texts, a problem that goes by many names:
- approximate string matching,
- pattern matching,
- fuzzy searching."
- [readme] flank-max-errors guidance: "Slightly increase `--flank-max-errors` (the automatically derived cutoff is reported)."
- [methods] install command with target-cpu=native: "RUSTFLAGS="-C target-cpu=native" cargo install barbell"
