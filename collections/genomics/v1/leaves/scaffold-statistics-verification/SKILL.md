---
name: scaffold-statistics-verification
description: Use when a TULIP (tulipseed.perl + tulipbulb.perl) assembly run has finished
  and you must confirm scaffold counts, size range, >1 Mbp length fraction, and per-seed-link
  read support before polishing.
when_to_use_negative:
- The assembly was produced by a different pipeline (e.g. the Canu/racon/pilon ont-assembly-polish
  workflow) and has no TULIP .graph or *_scaffolds_stats outputs to parse
- The goal is judging per-base sequence correctness at error sites — the paper notes
  that without a high-quality reference it is not straightforward to establish which
  assembly is correct, so contiguity statistics cannot substitute for validation
- Seeds are repetitive or of non-uniform length, violating TULIP's strict requirement
  of unique numeric identifiers and exactly equal seed lengths, making graph untangling
  and any comparison to the eel benchmarks unreliable
edam_operation: http://edamontology.org/operation_0525
edam_topics:
- http://edamontology.org/topic_0622
tools:
- name: TULIP
  role: tulipseed.perl builds the seed graph and ordered scaffolds and writes .graph
    and .scaffolds_stats files; tulipbulb.perl adds seed and long-read sequence, producing
    the scaffolds whose statistics are verified
  repo: github.com/Generade-nl/TULIP
- name: BWA MEM
  role: Aligns each nanopore read file to the 285 bp seeds with chemistry-specific
    settings (-k 14 -W 45 for R7.3 up to -k 19 -W 60 for R9 2D, plus -r 10 -A 1 -B
    1 -O 1 -E 1 -L 0); its Evidence-column counts per link feed the support statistics
  repo: http://bio-bwa.sourceforge.net
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41598-017-07650-6
    title: Rapid de novo assembly of the European eel genome from nanopore sequencing
      reads
  source_content_hash: sha256:648bd71c84f63d61a256be300ed1ec1255d901d68cdb6b2b461c5cf8b5b613f0
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - BWA MEM
  - TULIP
  - tulipbulb.perl
  edam_operation: http://edamontology.org/operation_0525
  edam_topics:
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5555555555555556
n_claims: 9
---

# scaffold-statistics-verification

## Summary

Verifies a TULIP seed-graph genome assembly by summarizing scaffold length statistics and per-link long-read evidence, benchmarking against the European eel reference run (2366 scaffolds, 431 bp–8.7 Mbp, N50 1.2 Mbp, ~7.4 alignments per seed link).

## When to use

Use when a TULIP (tulipseed.perl + tulipbulb.perl) assembly run has finished and you must confirm scaffold counts, size range, >1 Mbp length fraction, and per-seed-link read support before polishing.

## When NOT to use

- The assembly was produced by a different pipeline (e.g. the Canu/racon/pilon ont-assembly-polish workflow) and has no TULIP .graph or *_scaffolds_stats outputs to parse
- The goal is judging per-base sequence correctness at error sites — the paper notes that without a high-quality reference it is not straightforward to establish which assembly is correct, so contiguity statistics cannot substitute for validation
- Seeds are repetitive or of non-uniform length, violating TULIP's strict requirement of unique numeric identifiers and exactly equal seed lengths, making graph untangling and any comparison to the eel benchmarks unreliable

## Inputs

- eel_seeds_285.fasta — pre-selected 285 bp seed sequences (873058 fragments) with unique numeric identifiers excluding 0
- Four nanopore read FASTA files: R7.3.fasta, R9_pass_1d.fasta, R9_pass_2d.fasta, R9.4_pass_1d.fasta
- SAM alignment files clipped to the first six fields (e.g. R73_vs_285.shortsam) listed in an alignments.txt config (format [tab] alignment [tab] original fasta)
- tulip/eel.graph (seed graph, text format) and tulip/eel.scaffolds_stats (length statistics per scaffold)

## Outputs

- Scaffold count and length distribution summary (count, min–max span, N50)
- Fraction of total assembly length contributed by scaffolds >1 Mbp
- Mean number of long-read alignments (Evidence column) per seed link from the .graph file
- Comparison table against the reported candidate-assembly figures

## How to apply

After running ./tulipseed.perl --seedlength 285 --config alignments.txt --diploid --out tulip/eel and ./tulipbulb.perl with the 873058-fragment 285 bp seed set, parse tulip/eel.scaffolds_stats for per-scaffold length statistics and tulip/eel.graph for the seed links. The .graph file's Evidence column gives the number of long read alignments connecting each seed pair; average these to check the reported ~7.4 alignments per seed link. Summarize scaffold lengths to confirm the candidate assembly profile: 2366 scaffolds spanning 431 bp–8.7 Mbp, with the 232 scaffolds larger than 1 Mbp comprising 56% of assembly length. Compare contiguity against the prior Illumina-only draft (TULIP N50 1.2 Mbp, two orders of magnitude less fragmented) and confirm the whole pipeline completed within the reported two-day single-CPU runtime. Flag links marked Hypothetical = 0 (inferred rather than actual) separately when computing support statistics.

## Related tools

- **TULIP** (tulipseed.perl builds the seed graph and ordered scaffolds and writes .graph and .scaffolds_stats files; tulipbulb.perl adds seed and long-read sequence, producing the scaffolds whose statistics are verified) — github.com/Generade-nl/TULIP
- **BWA MEM** (Aligns each nanopore read file to the 285 bp seeds with chemistry-specific settings (-k 14 -W 45 for R7.3 up to -k 19 -W 60 for R9 2D, plus -r 10 -A 1 -B 1 -O 1 -E 1 -L 0); its Evidence-column counts per link feed the support statistics) — http://bio-bwa.sourceforge.net

## Examples

```
./tulipseed.perl --seedlength 285 --config alignments.txt --diploid --out tulip/eel && ./tulipbulb.perl --seeds eel_seeds_285.fasta --config alignments.txt --input tulip/eel --out bulb/eel
```

## Evaluation signals

- Scaffold count and size range match the reported candidate assembly: 2366 scaffolds spanning 431 bp–8.7 Mbp
- 232 scaffolds larger than 1 Mbp account for 56% of total assembly length
- Mean Evidence value over actual links (Hypothetical = 1) in the .graph file is ~7.4 nanopore read alignments per seed link
- Assembly N50 reaches 1.2 Mbp, two orders of magnitude less fragmented than the prior Illumina-based draft
- Whole pipeline (seed layout plus bundling) completes in roughly the reported two days on a single CPU

## Limitations

- tulipbulb.perl contains a bug that inserts NNN gaps in rare cases instead of sequence
- Scaffolds that could be merged based on long reads remain separate if the reads do not share a fortuitous seed alignment, so statistics likely undercount true contiguity
- Large repetitive and (sub)telomeric regions may lack frequent 285 bp unique-seed islands and be absent from the assembly, and the consensus can in theory mix up to four haplotypes
- The candidate assembly had not yet been validated with optical mapping at the time of reporting

## Evidence

- [abstract] The resulting genome assembly significantly improves on a previous draft based on short reads only, both in terms of contiguity (N50 1.2 Mbp) and structural quality.: "The resulting genome assembly significantly improves on a previous draft based on short reads only, both in terms of contiguity (N50 1.2 Mbp) and structural quality."
- [readme] tulip/eel.scaffolds_stats   Length statistics per scaffold: "tulip/eel.scaffolds_stats   Length statistics per scaffold"
- [readme] Evidence          The number of long read alignments connecting these seeds: "* Evidence          The number of long read alignments connecting these seeds"
- [discussion] scaffolds that could be merged based on long reads remain separate if these reads do not share a fortuitous seed alignment: "scaffolds that could be merged based on long reads remain separate if these reads do not share a fortuitous seed alignment in the correct position"
- [intro] The entire computational procedure takes two days on a desktop computer, and yields an assembly that is two orders of magnitude less fragmented: "The entire computational procedure takes two days on a desktop computer, and yields an assembly that is two orders of magnitude less fragmented than the previous Illumina-based draft."
