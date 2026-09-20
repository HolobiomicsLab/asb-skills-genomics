---
name: barcode-fusion-artifact-detection
description: Use when Nanopore reads (e.g. SQK-RBK114-96 rapid barcoding) show unexpected
  double-barcode patterns at the read start, or when essential-quality output (consensus
  generation) requires detecting fusion-point barcode artefacts that default demux
  misses.
when_to_use_negative:
- Reads follow the expected single-barcode configuration and no fusion artefacts are
  suspected — the --use-extended mode is ~3x slower and unnecessary for routine demux
- Data come from kits other than the thoroughly tested rapid/native DNA kits (RNA
  and Twist kits are exceptions) — unexpected annotations should be reported upstream
  instead of interpreted as fusions
edam_operation: http://edamontology.org/operation_2928
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3168
tools:
- name: Barbell
  role: pattern-aware demultiplexer; annotate/inspect/kit subcommands detect and label
    the fused double left-end barcode pattern when --use-extended enables fusion-point
    search
  repo: https://github.com/rickbeeloo/barbell
- name: Sassy
  role: SIMD-accelerated approximate string matching backend underlying Barbell's
    search speed
  repo: https://github.com/RagnarGrootKoerkamp/sassy
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/bioinformatics/btag349
    title: Barbell reveals and resolves demultiplexing and trimming issues in Nanopore
      data
  source_content_hash: sha256:b725583ba42717b2c129a685bf1769cd8594b5c7f26602c79957667443892c97
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/rickbeeloo/barbell
  tools:
  - Barbell
  - Sassy
  edam_operation: http://edamontology.org/operation_2928
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3168
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.625
n_claims: 8
---

# barcode-fusion-artifact-detection

## Summary

Detects fused double left-end barcode artefacts in Nanopore reads using Barbell's extended fusion-pattern search (--use-extended), where a first barcode's right flank is directly fused to a second barcode lacking its left flank and part of its prefix. This catches artefact configurations that standard kit demultiplexing misses.

## When to use

Use when Nanopore reads (e.g. SQK-RBK114-96 rapid barcoding) show unexpected double-barcode patterns at the read start, or when essential-quality output (consensus generation) requires detecting fusion-point barcode artefacts that default demux misses.

## When NOT to use

- Reads follow the expected single-barcode configuration and no fusion artefacts are suspected — the --use-extended mode is ~3x slower and unnecessary for routine demux
- Data come from kits other than the thoroughly tested rapid/native DNA kits (RNA and Twist kits are exceptions) — unexpected annotations should be reported upstream instead of interpreted as fusions

## Inputs

- Nanopore FASTQ reads potentially carrying fused double left-end barcodes (e.g. synthetic SQK-RBK114-96 fusion constructs or real rapid-barcoding runs)
- Barbell kit preset for the barcoding kit in use (e.g. SQK-RBK114-96) with its barcode and flank sequences

## Outputs

- anno.tsv annotation table with per-read barcode/flank match positions, match_type, flank_cost, barcode_cost, label, strand, and cuts
- barbell inspect pattern summary showing detected pattern groups including the fused double left-end pattern
- fusion_barcode_labels.tsv mapping each read_id to its first (fusion) and second (original sample) barcode labels
- kit demultiplexed output folder from the --use-extended run

## How to apply

Build the pattern query from the kit's barcode and flank sequences, then run 'barbell annotate --kit SQK-RBK114-96 -i reads.fastq -t 10 -o anno.tsv' and 'barbell inspect -i anno.tsv' to confirm the double left-end fusion pattern (e.g. Ftag[fw, *, @left(0..250)]__Ftag[fw, *, @prev_left(0..250)]) appears among detected groups. Next run 'barbell kit -k SQK-RBK114-96 -i reads.fastq -o output_folder --use-extended --maximize' so fusion points are searched; expect ~3x slower runtime, which is justified when quality is essential (e.g. generating consensus sequences). Rationale: in a fusion construct the first barcode's right flank abuts a second barcode missing its left flank and a prefix deletion, so the fused second barcode matches with a higher edit distance (mean ~7 vs ~3 for an intact barcode+flank match) — the annotation table's per-read barcode and flank match positions and costs reveal this. Finally extract per read the outer fusion barcode and inner/original sample barcode labels and write read_id with both labels to fusion_barcode_labels.tsv.

## Related tools

- **Barbell** (pattern-aware demultiplexer; annotate/inspect/kit subcommands detect and label the fused double left-end barcode pattern when --use-extended enables fusion-point search) — https://github.com/rickbeeloo/barbell
- **Sassy** (SIMD-accelerated approximate string matching backend underlying Barbell's search speed) — https://github.com/RagnarGrootKoerkamp/sassy

## Examples

```
barbell kit -k SQK-RBK114-96 -i reads.fastq -o output_folder --use-extended --maximize
```

## Evaluation signals

- barbell inspect lists the double left-end pattern, e.g. Ftag[fw, *, @left(0..250)]__Ftag[fw, *, @prev_left(0..250)], among detected groups
- Every synthetic read with a known fusion ground truth is assigned both barcode labels matching the simulation
- The fused second barcode shows a higher match cost in anno.tsv (mean edit distance ~7) than intact barcode+flank matches (~3)
- Extended mode detects fusion constructs that the standard (non-extended) kit configuration misses
- Match positions in anno.tsv occur near read ends, not at random internal locations (which would indicate Fflank noise from a too-lenient --flank-max-errors)

## Limitations

- --use-extended is around 3x slower than the default kit run, so it is reserved for cases where quality is essential (e.g. generating consensus sequences)
- Results can slightly differ (<0.01%) between release and current git versions using Sassy V2
- Barbell presets were thoroughly tested for rapid and native kits but not others; unexpected annotations on other kits should be reported as issues
- Too-lenient --flank-max-errors can produce random Fflank matches at non-end locations; adjust only after inspecting annotation.tsv

## Evidence

- [readme] Note, there is an option `--use-extended` which enables searches for fusion points, and other artefacts. This is around 3 times slower, but worth it if quality is essential (i.e. generating consensus sequences).: "--use-extended` which enables searches for fusion points, and other artefacts. This is around 3 times slower, but worth it if quality is essential"
- [results] Fused double left-end barcodes form when the right flank of the first barcode is directly fused to the second barcode, with loss of the second barcode's left flank and a partial deletion of its prefix, so the second barcode shows a higher edit distance (mean 7 versus 3) than an intact barcode.: [evidence span withheld — anchor status: not_found]
- [abstract] we found that only 83% of reads followed the expected single-barcode configuration, while 17% showed complex barcode attachments.: "only 83% of reads followed the expected single-barcode configuration, while 17% showed complex barcode attachments"
- [methods] Run `annotate` to find matches in reads and output an annotation table (TSV): "Run `annotate` to find matches in reads and output an annotation table (TSV)"
