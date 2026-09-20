---
name: nanopore-barcode-sequence-search
description: Use when validating whether submitted GenBank/NCBI assemblies (e.g. CP131573.1,
  CP165501.1) carry detectable Nanopore rapid-barcode artefacts from kits like SQK-RBK114-96.
when_to_use_negative:
- The input is Nanopore reads requiring demultiplexing/trimming rather than assembled
  reference sequences — use the Barbell annotate/filter/trim workflow instead.
- You need exact string matching with zero errors only; the method relies on edit-distance
  approximate matching with a flank error tolerance and will report imperfect matches.
- The barcode kit is not covered by a Barbell preset or supported_kits.txt (RNA and
  Twist kits are exceptions to the tested DNA kits).
edam_operation: http://edamontology.org/operation_0346
edam_topics:
- http://edamontology.org/topic_3168
- http://edamontology.org/topic_0622
tools:
- name: Barbell
  role: Provides the kit presets (e.g. barbell kit -k SQK-RBK114-96) from which the
    barcode and flank query sequences are extracted for the search.
  repo: github.com/rickbeeloo/barbell
- name: Sassy
  role: SIMD-accelerated edit-distance approximate string matching engine used to
    scan assemblies for barcode and flank sequences with a flank error tolerance.
  repo: github.com/RagnarGrootKoerkamp/sassy
- name: cargo
  role: 'Used to install Barbell from source with native SIMD flags: RUSTFLAGS="-C
    target-cpu=native" cargo install barbell.'
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/bioinformatics/btag349
    title: Barbell reveals and resolves demultiplexing and trimming issues in Nanopore
      data
  source_content_hash: sha256:d84b7e67c4cd0c1e71f7aa34f1b9077e8a1061f6195a0dc86c955c29c6e1c648
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Barbell
  - Sassy
  - cargo
  edam_operation: http://edamontology.org/operation_0346
  edam_topics:
  - http://edamontology.org/topic_3168
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# nanopore-barcode-sequence-search

## Summary

Searches public NCBI/GenBank nucleotide assemblies for residual Nanopore rapid-barcode and flank sequences using edit-distance approximate matching (Sassy) with kit presets extracted via Barbell, to document barcode contamination in submitted genomes.

## When to use

Use when validating whether submitted GenBank/NCBI assemblies (e.g. CP131573.1, CP165501.1) carry detectable Nanopore rapid-barcode artefacts from kits like SQK-RBK114-96.

## When NOT to use

- The input is Nanopore reads requiring demultiplexing/trimming rather than assembled reference sequences — use the Barbell annotate/filter/trim workflow instead.
- You need exact string matching with zero errors only; the method relies on edit-distance approximate matching with a flank error tolerance and will report imperfect matches.
- The barcode kit is not covered by a Barbell preset or supported_kits.txt (RNA and Twist kits are exceptions to the tested DNA kits).

## Inputs

- GenBank/NCBI assembly records in FASTA format (e.g. CP131573.1, CP165501.1)
- Nanopore kit barcode and flank sequences as query FASTA, extracted from a Barbell kit preset (e.g. SQK-RBK114-96)

## Outputs

- Per-accession barcode match counts and coordinates table, e.g. barcode_verification.tsv
- List of hit positions and orientations per barcode (including contig-terminus overhang detections)

## How to apply

Download the GenBank records of interest in FASTA format (e.g. the 1.43 Mb Photobacterium leiognathi SV5.1 contig CP131573.1 and the E. coli plasmid CP165501.1). Extract the SQK-RBK114-96 barcode and flank sequences as query FASTA using the Barbell kit preset (`barbell kit -k SQK-RBK114-96`). Scan each assembly with Sassy-style edit-distance approximate matching, allowing the default flank error tolerance so partial/flank-degraded hits are still recovered. Count matches per barcode (e.g. how many times BC86 occurs in CP131573.1) and record positions and orientations of other barcodes (e.g. BC10 and BC09 in CP165501.1), checking whether they occur at opposite contig termini — the signature of barcode bleeding. Write per-accession match counts and coordinates to a TSV such as barcode_verification.tsv and compare against the published findings (BC86 x11 in CP131573.1; BC10/BC09 overhangs at opposite ends of CP165501.1).

## Related tools

- **Barbell** (Provides the kit presets (e.g. barbell kit -k SQK-RBK114-96) from which the barcode and flank query sequences are extracted for the search.) — github.com/rickbeeloo/barbell
- **Sassy** (SIMD-accelerated edit-distance approximate string matching engine used to scan assemblies for barcode and flank sequences with a flank error tolerance.) — github.com/RagnarGrootKoerkamp/sassy
- **cargo** (Used to install Barbell from source with native SIMD flags: RUSTFLAGS="-C target-cpu=native" cargo install barbell.)

## Examples

```
barbell kit -k SQK-RBK114-96 -i reads.fastq -o output_folder --maximize
```

## Evaluation signals

- BC86 matches in the 1.43 Mb P. leiognathi SV5.1 contig CP131573.1 total exactly 11.
- CP165501.1 shows BC10 and BC09 overhangs at opposite contig ends, confirming barcode bleeding.
- Per-accession match counts and coordinates are written to barcode_verification.tsv with one row per accession/barcode.
- Flank matches with excessive errors appear as Fflank-type random hits; if such random-location matches dominate, the flank error tolerance was set too leniently.
- Query sequences reproduce the SQK-RBK114-96 barcode/flank configuration defined in the Barbell preset.

## Limitations

- Results can differ slightly (<0.01%) between the released Barbell and the current git version built against Sassy V2.
- The automatically derived flank-based edit-distance cutoff may miss reads/matches if too strict, or produce random Fflank matches if relaxed too far; always inspect results after changing --flank-max-errors.
- Barbell presets were thoroughly tested only for rapid and native Nanopore DNA kits, not RNA or Twist kits.

## Evidence

- [abstract] Nanopore barcoding contamination is widespread in public databases, detected in hundreds of NCBI entries, some of which are responsible for artificial taxonomic connections: "with Nanopore sequences detected in hundreds of NCBI entries, some of which are responsible for artificial taxonomic connections."
- [readme] Rapid barcoding kit preset invocation: "barbell kit -k SQK-RBK114-96 -i reads.fastq -o output_folder --maximize"
- [readme] Sassy approximate matching description: [evidence span withheld — anchor status: external_doc]
- [methods] Sassy speed via Sassy V2: "Current git version is around 5x faster than the releases by using [Sassy V2](https://github.com/RagnarGrootKoerkamp/sassy)"
