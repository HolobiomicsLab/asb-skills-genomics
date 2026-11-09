---
name: flank-barcode-edit-distance-annotation
description: Use when Nanopore barcoding reads (rapid or native kits) must be scanned
  for barcode plus flank matches to detect contamination, especially when reproducing
  NCBI core-nt contamination counts from filtered match tables.
when_to_use_negative:
- Input is raw sequence data not generated with a Nanopore barcoding kit — kit presets
  and flank/barcode query patterns will not apply
- You only need barcode identification without contamination screening of public NCBI
  assemblies — a standard demultiplexer call suffices and the NCBI match-table reproduction
  step is unnecessary
- Many matches appear as Fflank at random read locations — the flank error threshold
  is too relaxed and counts would be unreliable until --flank-max-errors is lowered
edam_operation: http://edamontology.org/operation_3198
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3168
tools:
- name: Barbell
  role: Annotates reads with barcode and flank matches (annotate/filter/inspect/trim)
    and provides kit presets with per-match edit-distance costs
  repo: https://github.com/rickbeeloo/barbell
- name: Sassy
  role: SIMD-accelerated approximate string matching engine underlying Barbell's search
    speed
  repo: https://github.com/RagnarGrootKoerkamp/sassy
- name: Dorado
  role: Baseline demultiplexer whose trimming failures motivate the pattern-aware
    annotation approach
  repo: https://github.com/nanoporetech/dorado
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/bioinformatics/btag349
    title: Barbell reveals and resolves demultiplexing and trimming issues in Nanopore
      data
  source_content_hash: sha256:35feae7fe4337c29818a26206aa25ab148481b0700d9471f3e548e0254bcc52e
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/rickbeeloo/barbell
  tools:
  - Barbell
  - Dorado
  - Sassy
  edam_operation: http://edamontology.org/operation_3198
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3168
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 10
---

# Flank-barcode edit-distance annotation

## Summary

Annotates Nanopore reads (or deposited match tables) with flank and barcode matches, recording flank_cost and barcode_cost edit-distance columns per read, so that Ftag-type contamination (barcode flanked by adapter sequences) can be counted and quantified against public NCBI nucleotide assemblies.

## When to use

Use when Nanopore barcoding reads (rapid or native kits) must be scanned for barcode plus flank matches to detect contamination, especially when reproducing NCBI core-nt contamination counts from filtered match tables.

## When NOT to use

- Input is raw sequence data not generated with a Nanopore barcoding kit — kit presets and flank/barcode query patterns will not apply
- You only need barcode identification without contamination screening of public NCBI assemblies — a standard demultiplexer call suffices and the NCBI match-table reproduction step is unnecessary
- Many matches appear as Fflank at random read locations — the flank error threshold is too relaxed and counts would be unreliable until --flank-max-errors is lowered

## Inputs

- Nanopore FASTQ reads from a barcoding kit (e.g. SQK-RBK114-96 rapid or SQK-NBD114-96 native)
- Filtered match TSV tables from Zenodo record 17396505 (rapid-kit and native-kit core-nt searches) with flank_cost and barcode_cost columns
- Kit preset identifier supported by barbell (see data/supported_kits.txt)

## Outputs

- Annotation TSV (anno.tsv) with per-read flank/barcode coordinates, match_type (Ftag/Fflank), flank_cost, barcode_cost, and label columns
- summary_statistics.csv with six figures: total matches, exact 0-edit matches, and assembly counts per kit
- Pattern summaries from `barbell inspect`

## How to apply

Run `barbell annotate` with the kit preset (e.g. SQK-RBK114-96 or SQK-NBD114-96) to produce an annotation TSV containing per-read columns such as read_start_flank, read_end_flank, match_type, flank_cost, and barcode_cost. Classify matches by match_type: Ftag rows indicate a barcode detected within matched flanks (both flank and barcode elements present), while Fflank rows indicate flank-only (often random) matches. To reproduce the NCBI contamination summary, download the deposited filtered match tables from Zenodo record 17396505, load each TSV keeping the flank_cost and barcode_cost columns, and for each kit count (1) rows with both flank and barcode present (Ftag), (2) rows with barcode_cost = 0 (exact 0-edit matches), and (3) distinct assemblies the exact matches map to. The rapid kit should yield 103 total / 68 exact / 67 assemblies; the native kit 462 total / 270 exact / 284 assemblies. Use `barbell inspect` to summarize patterns across the annotation file before filtering, and adjust --flank-max-errors only if many reads are missed, inspecting that matches still occur near read ends rather than at random locations.

## Related tools

- **Barbell** (Annotates reads with barcode and flank matches (annotate/filter/inspect/trim) and provides kit presets with per-match edit-distance costs) — https://github.com/rickbeeloo/barbell
- **Sassy** (SIMD-accelerated approximate string matching engine underlying Barbell's search speed) — https://github.com/RagnarGrootKoerkamp/sassy
- **Dorado** (Baseline demultiplexer whose trimming failures motivate the pattern-aware annotation approach) — https://github.com/nanoporetech/dorado

## Examples

```
barbell annotate --kit SQK-RBK114-96 -i pass_sample.fastq -t 10 -o anno.tsv
```

## Evaluation signals

- Reproduced rapid-kit counts equal the reported 103 total matches / 68 exact (barcode_cost = 0) matches / 67 distinct assemblies
- Reproduced native-kit counts equal the reported 462 total matches / 270 exact matches / 284 distinct assemblies
- Every counted row has both flank and barcode elements present (Ftag match_type) rather than Fflank flank-only matches
- Annotation TSV contains non-empty flank_cost and barcode_cost columns for each matched read
- `barbell inspect` pattern groups show matches near read ends (small rel_dist_to_end), not random positions

## Limitations

- Random Fflank matches appear if --flank-max-errors is set too high; matches should occur near read ends, and lowering the threshold is required if they do not
- Only the rapid (SQK-RBK114-96) and native (SQK-NBD114-96) kits were thoroughly tested; other kits may need issue reporting if annotations are unexpected
- Release-version binaries are slower and results can slightly differ (<0.01% or so) between the git version (using Sassy V2) and released versions

## Evidence

- [methods] Run `annotate` to find matches in reads and output an annotation table (TSV): "Run `annotate` to find matches in reads and output an annotation table (TSV)"
- [readme] match_type, flank_cost, barcode_cost columns: "match_type  flank_cost      barcode_cost    label   strand  cuts"
- [readme] Ftag example row: "dbca7fb9-d6c8-4417-8ae7-bc32ebce9b27    2972    29      48      70      29      111     0       23      Ftag     13      7       BC29    Fwd"
- [readme] flank error threshold guidance: "Slightly increase `--flank-max-errors` (the automatically derived cutoff is reported)."
- [readme] Random Fflank matches caveat: "Random locations: Lower `--flank-max-errors`."
