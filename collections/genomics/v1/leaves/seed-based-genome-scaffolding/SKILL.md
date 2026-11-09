---
name: seed-based-genome-scaffolding
description: 'Use when assembling a eukaryotic genome from uncorrected nanopore reads
  without heavy compute: long-read FASTAs (e.g. R7.3, R9 1D/2D, R9.'
when_to_use_negative:
- 'You need a finished, error-corrected reference-grade assembly: TULIP output is
  a prototype that needs Racon/Pilon polishing, and large repetitive or (sub)telomeric
  regions lacking frequent 285 bp unique-seed islands may be absent'
- Your input is already assembled contigs/scaffolds or paired short reads only — the
  skill requires uncorrected long reads plus a curated uniform-length seed set
- Seeds are variable length or have non-numeric/duplicate identifiers, which violates
  the strict TULIP input requirements and will break graph construction
edam_operation: http://edamontology.org/operation_0564
edam_topics:
- http://edamontology.org/topic_0196
- http://edamontology.org/topic_0622
tools:
- name: TULIP (tulipseed.perl)
  role: Builds and simplifies the seed graph from long-read-to-seed alignments and
    orders links into linear seed scaffolds
  repo: https://github.com/Generade-nl/TULIP
- name: TULIP (tulipbulb.perl)
  role: Bundles original seed and long-read sequence into the final per-scaffold FASTA
    outputs
  repo: https://github.com/Generade-nl/TULIP
- name: BWA MEM
  role: Aligns nanopore reads to the sparse seed set, emitting clipped SAM (first
    6 fields incl. CIGAR) for graph construction
provenance:
  source_task_ids:
  - task_008
  - task_004
  source_papers:
  - doi: 10.1038/s41598-017-07650-6
    title: Rapid de novo assembly of the European eel genome from nanopore sequencing
      reads
  source_content_hash: sha256:976e9e484a7d46e373f0d7a1c6e42aff1b1aa0c27df5f08fff19ba721e4489da
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Generade-nl/TULIP
  tools:
  - BWA MEM
  - TULIP
  - TULIP (tulipbulb.perl)
  - TULIP (tulipseed.perl / tulipbulb.perl)
  - TULIP (tulipseed.perl)
  edam_operation: http://edamontology.org/operation_0564
  edam_topics:
  - http://edamontology.org/topic_0196
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# seed-based-genome-scaffolding

## Summary

Reconstructs genome scaffolds by aligning uncorrected nanopore long reads to sparse short 'seed' sequences (e.g. 285 bp, non-repetitive) with BWA MEM, building and simplifying a seed graph with tulipseed.perl, and bundling long-read sequence into scaffolds with tulipbulb.perl (TULIP, European eel 860 Mbp genome).

## When to use

Use when assembling a eukaryotic genome from uncorrected nanopore reads without heavy compute: long-read FASTAs (e.g. R7.3, R9 1D/2D, R9.4) plus a pre-selected, uniform-length, non-repetitive seed FASTA are available and a lightweight, near-linear alternative to Canu-style whole-genome assembly is needed.

## When NOT to use

- You need a finished, error-corrected reference-grade assembly: TULIP output is a prototype that needs Racon/Pilon polishing, and large repetitive or (sub)telomeric regions lacking frequent 285 bp unique-seed islands may be absent
- Your input is already assembled contigs/scaffolds or paired short reads only — the skill requires uncorrected long reads plus a curated uniform-length seed set
- Seeds are variable length or have non-numeric/duplicate identifiers, which violates the strict TULIP input requirements and will break graph construction

## Inputs

- eel_seeds_285.fasta — pre-selected, 'not too repetitive' seed sequences, unique nonzero numeric identifiers, all exactly 285 bp
- Nanopore long-read FASTA files: R7.3.fasta, R9_pass_1d.fasta, R9_pass_2d.fasta, R9.4_pass_1d.fasta (unique read IDs per file)
- ShortSAM alignment files (first 6 SAM fields incl. CIGAR): R73_vs_285.shortsam, R91D_vs_285.shortsam, R92D_vs_285.shortsam, R941D_vs_285.shortsam
- alignments.txt configuration file (format: type[tab]alignment[tab]original_fasta)

## Outputs

- tulip/eel.graph — seed graph in text format: Seed 1, orientation (ii/io/oi/oo), Seed 2, evidence, hypothetical flag, minimum/mean/maximum gap, StDev, scaffold ID
- tulip/eel.scaffolds and tulip/eel.scaffolds_stats — ordered seed scaffolds and per-scaffold length statistics
- tulip/eel.*_tmp — binary graph/scaffolds and seed usage files consumed by tulipbulb.perl
- bulb/eel_scaffold_*.fasta and bulb/eel_readbundle_*.fasta — scaffold sequences (seeds uppercase, reads lowercase) and per-scaffold read bundles

## How to apply

First align each nanopore read FASTA to the indexed seed set with BWA MEM using chemistry-specific parameters (R7.3: -k 14 -W 45; R9 1D: -k 16 -W 50; R9 2D: -k 19 -W 60; R9.4: -k 16 -W 60; all with -r 10 -A 1 -B 1 -O 1 -E 1 -L 0) and retain only the first six SAM fields through the CIGAR string (cut -f 1,2,3,4,5,6). List all alignment files in a configuration file with 'type[tab]alignment[tab]original_fasta' lines (e.g. 'sam R73_vs_285.shortsam R7.3.fasta'). Seeds must have unique nonzero numeric identifiers and be exactly the same length (285 bp for eel_seeds_285.fasta); run tulipseed.perl with --seedlength 285 --config alignments.txt --diploid --out tulip/eel to build the seed graph — nodes are seeds, edges connect seeds co-aligned on individual long reads with evidence counts and gap statistics — then simplify it by removing triangular motifs and severing overconnected (putatively repetitive) seed links. Order links into linear scaffolds and bundle long-read sequence with tulipbulb.perl (./tulipbulb --seeds eel_seeds_285.fasta --config alignments.txt --input tulip/eel --out bulb/eel), producing bulb/eel_scaffold_*.fasta where seed-derived sequence is uppercase and read-derived sequence lowercase.

## Related tools

- **TULIP (tulipseed.perl)** (Builds and simplifies the seed graph from long-read-to-seed alignments and orders links into linear seed scaffolds) — https://github.com/Generade-nl/TULIP
- **TULIP (tulipbulb.perl)** (Bundles original seed and long-read sequence into the final per-scaffold FASTA outputs) — https://github.com/Generade-nl/TULIP
- **BWA MEM** (Aligns nanopore reads to the sparse seed set, emitting clipped SAM (first 6 fields incl. CIGAR) for graph construction)

## Examples

```
./tulipseed.perl --seedlength 285 --config alignments.txt --diploid --out tulip/eel
```

## Evaluation signals

- Each scaffold in tulip/eel.graph forms a linear chain of seed links with the 10-column schema (Seed 1, orientation, Seed 2, evidence, hypothetical, min, mean, max, StDev, scaffold ID) populated
- Per-link evidence (number of connecting read alignments) and gap statistics (min/mean/max/StDev) are present for every retained link; removed links are predominantly low-evidence
- Retained links carry a reported average evidence of ~7.4 read alignments per link
- tulipbulb.perl output contains two files per scaffold (eel_scaffold_NNN.fasta and eel_readbundle_NNN.fasta) with seed-derived sequence in uppercase and read-derived sequence in lowercase
- Assembly statistics from tulip/eel.scaffolds_stats reproduce the reported pattern: 270 bp and 285 bp seeds give comparable statistics at equal density, with an optimum near the full seed set (~873058 fragments) and more small-scaffold fragmentation at the highest densities

## Limitations

- tulipbulb.perl contains a bug that inserts NNN gaps in rare cases where sequence cannot be added
- Scaffolds that could be merged based on long reads remain separate if the reads do not share a fortuitous seed alignment in the correct position
- Large repetitive regions and (sub)telomeric repeats may lack frequent 285 bp islands of unique sequence and be absent from the assembly
- The eel assembly is a hybrid of two technologies, three nanopore chemistries and two animals; consensus can in theory mix up to four haplotypes

## Evidence

- [readme] Only the first 6 fields of the SAM alignment information is used (up to and including the CIGAR string), so you might want to clip off the rest.: "Only the first 6 fields of the SAM alignment information is used (up to and including the CIGAR string), so you might want to clip off the rest."
- [readme] Strict requirements are that seed sequence identifiers are unique numbers (excluding 0), and that all seeds are of exactly the same length.: "Strict requirements are that seed sequence identifiers are unique numbers (excluding 0), and that all seeds are of exactly the same length."
- [readme] The .graph file simply lists which seeds are connected in the final simplified graph: "The _.graph_ file simply lists which seeds are connected in the final simplified graph"
- [methods] bwa mem -t 4 -k 14 -W 45 -r 10 -A 1 -B 1 -O 1 -E 1 -L 0 285_seeds R7.3.fasta | cut -f 1,2,3,4,5,6 > R73_vs_285.shortsam: "bwa mem -t 4 -k 14 -W 45 -r 10 -A 1 -B 1 -O 1 -E 1 -L 0 285_seeds R7.3.fasta         | cut -f 1,2,3,4,5,6 > R73_vs_285.shortsam"
- [intro] Named TULIP (for The Uncorrected Long-read Integration Process), its primary purpose is to split up such large assembly problems into manageable subsets of long reads: "Named TULIP (for The Uncorrected Long-read Integration Process), its primary purpose is to split up such large assembly problems into manageable subsets of long reads"
