---
name: graph-simplification-reimplementation
description: Use when reimplementing or validating a seed-graph assembly layout step
  that converts short SAM alignments of long reads to seeds into a simplified, scaffolded
  seed graph with per-link evidence and gap statistics.
when_to_use_negative:
- Your goal is the downstream bundling of seed/read sequence into scaffold sequences
  — that is tulipbulb.perl territory, not the seed graph step
- Input alignments are not SAM format or lack a CIGAR string in field 6 (e.g. DALIGNER
  output), since this version of tulipseed.perl only parses the first 6 SAM fields
- Seeds do not have unique numeric identifiers and identical lengths, violating tulipseed's
  strict input requirements
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_0196
- http://edamontology.org/topic_0622
tools:
- name: TULIP
  role: Prototype pipeline whose tulipseed.perl script builds and simplifies the seed
    graph and outputs scaffolds
  repo: github.com/Generade-nl/TULIP
- name: BWA MEM
  role: Aligns long nanopore reads to the 285-bp seeds with tuned parameters (e.g.
    -k 14 -W 45 -r 10 -A 1 -B 1 -O 1 -E 1 -L 0), producing the SAM input to the graph
    builder
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41598-017-07650-6
    title: Rapid de novo assembly of the European eel genome from nanopore sequencing
      reads
  source_content_hash: sha256:391ac0201f60c7f28896b2e5ac51aafecb39708bf4f1dcb06086c777ddb66ef7
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - BWA MEM
  - TULIP
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_0196
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5454545454545454
n_claims: 11
---

# graph-simplification-reimplementation

## Summary

Reconstructs the TULIP seed graph construction and simplification module (tulipseed.perl), which builds a graph of co-aligned 285-bp seeds from BWA MEM SAM alignments of nanopore reads and simplifies it into linear seed scaffolds.

## When to use

Use when reimplementing or validating a seed-graph assembly layout step that converts short SAM alignments of long reads to seeds into a simplified, scaffolded seed graph with per-link evidence and gap statistics.

## When NOT to use

- Your goal is the downstream bundling of seed/read sequence into scaffold sequences — that is tulipbulb.perl territory, not the seed graph step
- Input alignments are not SAM format or lack a CIGAR string in field 6 (e.g. DALIGNER output), since this version of tulipseed.perl only parses the first 6 SAM fields
- Seeds do not have unique numeric identifiers and identical lengths, violating tulipseed's strict input requirements

## Inputs

- Four BWA MEM SAM alignment files of nanopore reads vs 285-bp seeds (e.g. R73_vs_285.shortsam), truncated to the first 6 SAM fields including CIGAR
- Config file alignments.txt with rows of format: format [tab] alignment [tab] original fasta
- Seed FASTA (eel_seeds_285.fasta) with unique numeric identifiers (excluding 0) and uniform 285-bp seed length

## Outputs

- tulip/eel.graph — text-format seed graph listing connected seed links with orientation, evidence, hypothetical flag, gap min/mean/max/StDev, and scaffold ID
- tulip/eel.graph_tmp — binary-format seed graph for tulipbulb.perl
- tulip/eel.scaffolds and tulip/eel.scaffolds_tmp — ordered seed scaffolds
- tulip/eel.seeds_tmp, tulip/eel.layout_log, tulip/eel.scaffolds_stats — seed usage info, log, and per-scaffold length statistics

## How to apply

Parse the four truncated SAM alignment files (R73_vs_285.shortsam, R91D_vs_285.shortsam, R92D_vs_285.shortsam, R941D_vs_285.shortsam) listed in the alignments.txt config, reading only the first 6 SAM fields up to and including the CIGAR string. Build a graph whose nodes are uniquely numerically identified seeds of uniform 285-bp length and whose edges connect seed pairs co-aligned on the same long read, recording per-link evidence (number of connecting alignments) and gap statistics (minimum, mean, maximum, standard deviation). Simplify locally: remove triangular motifs to recover reads whose seed alignments were missed, then sever links of still-overconnected seeds that are putatively repetitive. Order links into linear seed scaffolds and emit tulip/eel.graph in text format with the column schema Seed 1, orientation (ii/io/oi/oo), Seed 2, evidence, hypothetical flag, minimum, mean, maximum, StDev, scaffold ID. The diploid flag is used for the eel (out: ./tulipseed.perl --seedlength 285 --config alignments.txt --diploid --out tulip/eel).

## Related tools

- **TULIP** (Prototype pipeline whose tulipseed.perl script builds and simplifies the seed graph and outputs scaffolds) — github.com/Generade-nl/TULIP
- **BWA MEM** (Aligns long nanopore reads to the 285-bp seeds with tuned parameters (e.g. -k 14 -W 45 -r 10 -A 1 -B 1 -O 1 -E 1 -L 0), producing the SAM input to the graph builder)

## Examples

```
./tulipseed.perl --seedlength 285 --config alignments.txt --diploid --out tulip/eel
```

## Evaluation signals

- Each scaffold in tulip/eel.graph forms a linear chain of seed links with the correct 10-column schema (Seed 1, orientation ii/io/oi/oo, Seed 2, evidence, hypothetical, minimum, mean, maximum, StDev, scaffold ID)
- Per-link evidence and gap fields (minimum, mean, maximum, StDev) are populated for every emitted link
- Removed (simplified) links are predominantly low-evidence, while retained links carry an average of ~7.4 read alignments per link
- Hypothetical flag distinguishes actual links (1) from inferred links (0)
- Orientation values are restricted to ii, io, oi, oo and seeds match the uniquely numbered 285-bp seed identifiers

## Limitations

- The prototype bug in tulipbulb.perl can add NNN gaps in rare cases (affects downstream bundling of graph output)
- Large repetitive regions and (sub)telomeric repeats may be absent from the assembly because they lack frequent 285-bp islands of unique seed sequence
- Scaffolds that could be merged based on long reads remain separate if those reads do not share a fortuitous seed alignment in the correct position
- TULIP scripts are explicitly prototypes; additional components/implementations were expected to follow

## Evidence

- [readme] Only the first 6 fields of the SAM alignment information is used (up to and including the CIGAR string), so you might want to clip off the rest.: "Only the first 6 fields of the SAM alignment information is used (up to and including the CIGAR string), so you might want to clip off the rest."
- [readme] The .graph file simply lists which seeds are connected in the final simplified graph: "The _.graph_ file simply lists which seeds are connected in the final simplified graph"
- [readme] Strict requirements are that seed sequence identifiers are unique numbers (excluding 0), and that all seeds are of exactly the same length.: "Strict requirements are that seed sequence identifiers are unique numbers (excluding 0), and that all seeds are of exactly the same length."
- [methods] ./tulipseed.perl --seedlength 285 --config alignments.txt --diploid --out tulip/eel: "./tulipseed.perl --seedlength 285 --config alignments.txt --diploid --out tulip/eel"
- [methods] We used BWA MEM to align reads to seeds, but other aligners should work.: "We used BWA MEM to align reads to seeds, but other aligners should work."
