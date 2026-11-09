---
name: seed-based-genome-assembly-layout
description: Use when assembling a moderate-size (~860 Mbp) eukaryotic genome from
  uncorrected Oxford Nanopore reads using a curated set of unique 285 bp seed sequences,
  where standard long-read assemblers are computationally prohibitive.
when_to_use_negative:
- 'Your goal is a fully polished, chromosome-scale reference: TULIP produces a candidate
  assembly that likely omits large repetitive and (sub)telomeric regions and needs
  downstream Racon/Pilon correction and optical-mapping validation'
- 'You have no pre-selected unique seed set: strict requirements are unique numeric
  seed identifiers (excluding 0), identical seed lengths, and ''not too repetitive''
  sequences, otherwise graph untangling becomes very slow'
- Your assembly problem is small enough that Canu or similar downstream assemblers
  are affordable — TULIP exists to split large problems into manageable subsets and
  adds fragmentation when seeds do not fortuitously bridge mergeable scaffolds
edam_operation: http://edamontology.org/operation_0525
edam_topics:
- http://edamontology.org/topic_0196
- http://edamontology.org/topic_0622
tools:
- name: TULIP
  role: The Uncorrected Long-read Integration Process; tulipseed.perl builds the seed
    graph and ordered scaffolds from seed-to-read alignments, tulipbulb.perl adds
    seed and long-read sequence
  repo: https://github.com/Generade-nl/TULIP
- name: BWA MEM
  role: Aligns nanopore long reads to the 285 bp seeds with chemistry-specific -k/-W
    seeding parameters; only the first six SAM fields are retained
- name: tulipbulb.perl
  role: Bundling step that injects original seed sequence (uppercase) and long-read
    sequence (lowercase) into each scaffold FASTA
  repo: https://github.com/Generade-nl/TULIP
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41598-017-07650-6
    title: Rapid de novo assembly of the European eel genome from nanopore sequencing
      reads
  source_content_hash: sha256:4067a9f34b3a70c6f6fc12487974550faf1998ecbaa3413d8b5039fbae349b35
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Generade-nl/TULIP
  tools:
  - BWA MEM
  - TULIP
  - tulipbulb.perl
  edam_operation: http://edamontology.org/operation_0525
  edam_topics:
  - http://edamontology.org/topic_0196
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 15
---

# seed-based-genome-assembly-layout

## Summary

Reproduces the TULIP seed-based assembly of the European eel genome by aligning 873058-fragment 285 bp seeds against four nanopore read sets with chemistry-tuned BWA MEM settings, then running tulipseed.perl and tulipbulb.perl to order seeds into scaffolds and bundle seed/read sequence, yielding 2366 scaffolds (431 bp–8.7 Mbp) with ~7.4 read alignments per seed link.

## When to use

Use when assembling a moderate-size (~860 Mbp) eukaryotic genome from uncorrected Oxford Nanopore reads using a curated set of unique 285 bp seed sequences, where standard long-read assemblers are computationally prohibitive.

## When NOT to use

- Your goal is a fully polished, chromosome-scale reference: TULIP produces a candidate assembly that likely omits large repetitive and (sub)telomeric regions and needs downstream Racon/Pilon correction and optical-mapping validation
- You have no pre-selected unique seed set: strict requirements are unique numeric seed identifiers (excluding 0), identical seed lengths, and 'not too repetitive' sequences, otherwise graph untangling becomes very slow
- Your assembly problem is small enough that Canu or similar downstream assemblers are affordable — TULIP exists to split large problems into manageable subsets and adds fragmentation when seeds do not fortuitously bridge mergeable scaffolds

## Inputs

- eel_seeds_285.fasta: 873058 pre-selected 'not too repetitive' seed sequences, all exactly 285 bp, with unique numeric identifiers (excluding 0)
- R7.3.fasta: Oxford Nanopore R7.3 chemistry reads, FASTA, one read per line
- R9_pass_1d.fasta and R9_pass_2d.fasta: R9 pass 1D and 2D nanopore reads, FASTA
- R9.4_pass_1d.fasta: R9.4 pass 1D nanopore reads, FASTA
- alignments.txt: TULIP configuration file with 'format [tab] alignment [tab] original fasta' rows

## Outputs

- tulip/eel.graph: seed graph listing connected seeds with orientation (ii/io/oi/oo), evidence count, gap min/mean/max/standard deviation, and final scaffold ID
- tulip/eel.scaffolds and tulip/eel.scaffolds_stats: ordered seed scaffolds with length statistics per scaffold
- bulb/eel_scaffold_999.fasta: scaffold sequence with seed-derived sequence in uppercase and long-read-derived sequence in lowercase
- bulb/eel_readbundle_999.fasta: reads used to construct each scaffold

## How to apply

Index eel_seeds_285.fasta with bwa index, then align each of the four nanopore read files (R7.3, R9_pass_1d, R9_pass_2d, R9.4_pass_1d) with BWA MEM using chemistry-specific seeding (-k 14 -W 45 for R7.3; -k 16 -W 50 for R9 1D; -k 19 -W 60 for R9 2D; -k 16 -W 60 for R9.4 1D, all with -t 4 -r 10 -A 1 -B 1 -O 1 -E 1 -L 0), piping through cut to keep only the first six SAM fields. Build alignments.txt listing each file as 'format [tab] alignment [tab] original fasta'. Run ./tulipseed.perl --seedlength 285 --config alignments.txt --diploid --out tulip/eel to build the seed graph and ordered scaffolds, then ./tulipbulb.perl --seeds eel_seeds_285.fasta --config alignments.txt --input tulip/eel --out bulb/eel to add seed and long-read sequence. The --diploid flag and the 'not too repetitive' seed selection rationale matter: repetitive seeds slow graph untangling; unique numeric seed identifiers (excluding 0) and uniform seed length are strict requirements.

## Related tools

- **TULIP** (The Uncorrected Long-read Integration Process; tulipseed.perl builds the seed graph and ordered scaffolds from seed-to-read alignments, tulipbulb.perl adds seed and long-read sequence) — https://github.com/Generade-nl/TULIP
- **BWA MEM** (Aligns nanopore long reads to the 285 bp seeds with chemistry-specific -k/-W seeding parameters; only the first six SAM fields are retained)
- **tulipbulb.perl** (Bundling step that injects original seed sequence (uppercase) and long-read sequence (lowercase) into each scaffold FASTA) — https://github.com/Generade-nl/TULIP

## Examples

```
bwa mem -t 4 -k 16 -W 60 -r 10 -A 1 -B 1 -O 1 -E 1 -L 0 285_seeds R9.4_pass_1d.fasta | cut -f 1,2,3,4,5,6 > R941D_vs_285.shortsam; ./tulipseed.perl --seedlength 285 --config alignments.txt --diploid --out tulip/eel
```

## Evaluation signals

- Scaffold count and size distribution match the reported candidate assembly: 2366 scaffolds ranging 431 bp to 8.7 Mbp
- 232 scaffolds larger than 1 Mbp comprise 56% of assembly length
- Mean number of long-read alignments per seed link (Evidence column of the .graph file) is ~7.4
- N50 of the assembly reaches the reported ~1.2 Mbp contiguity improvement over the Illumina-only draft
- Entire pipeline completes within the reported two-day runtime on a single desktop CPU

## Limitations

- The tulipbulb.perl prototype contains a bug that inserts NNN gaps in rare cases where sequence cannot be added
- Scaffolds that could be merged based on long reads remain separate unless the reads share a fortuitous seed alignment in the correct position
- Large repetitive regions and (sub)telomeric repeats may be absent because they lack frequent 285 bp islands of unique sequence
- The assembly is a hybrid of two sequencing technologies, three nanopore chemistry generations, and two different animals, so the consensus can in theory mix up to four haplotypes; optical-mapping validation was only planned, not performed

## Evidence

- [readme] Command lines for BWA MEM alignment with chemistry-specific parameters: "bwa mem -t 4 -k 14 -W 45 -r 10 -A 1 -B 1 -O 1 -E 1 -L 0 285_seeds R7.3.fasta         | cut -f 1,2,3,4,5,6 > R73_vs_285.shortsam"
- [readme] tulipseed invocation: "./tulipseed.perl --seedlength 285 --config alignments.txt --diploid --out tulip/eel"
- [readme] Seed selection criteria: "These are pre-selected seed sequenced, with selection criteria 'not too repetitive'."
- [readme] Only first six SAM fields used: "Only the first 6 fields of the SAM alignment information is used (up to and including the CIGAR string)"
- [abstract] Two-day single-CPU runtime and genome size: "For this 860 Mbp genome, the entire computational process takes two days on a single CPU."
- [intro] TULIP purpose: split large problems into manageable subsets: "its primary purpose is to split up such large assembly problems into manageable subsets of long reads."
