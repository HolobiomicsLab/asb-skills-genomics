---
name: nanopore-genome-assembly
description: Use when assembling a large (hundreds of Mbp to multi-Gbp) eukaryotic
  genome from uncorrected nanopore FASTQ/FASTA reads on modest hardware, and existing
  de novo assemblers such as Canu are computationally prohibitive (tens of thousands
  of CPU hours).
when_to_use_negative:
- The task is polishing or correcting an existing long-read assembly with Illumina
  data — use the separate Racon/Pilon polishing step instead (TULIP deliberately skips
  per-read correction).
- You require a single monolithic de novo assembler run on small genomes where Canu-style
  assembly is affordable — TULIP's value is in chunking very large genomes.
- Seeds contain highly repetitive sequence or violate the strict requirements (nonzero
  unique numeric identifiers, exactly equal lengths), which slows graph untangling
  and can invalidate the layout.
edam_operation: http://edamontology.org/operation_0525
edam_topics:
- http://edamontology.org/topic_0196
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_0797
tools:
- name: TULIP
  role: Core two-script pipeline (tulipseed.perl for seed-graph layout, tulipbulb.perl
    for bundling long reads into scaffolds)
  repo: https://github.com/Generade-nl/TULIP
- name: BWA MEM
  role: Aligns nanopore long reads to the sparse seed set; only the first six SAM
    fields (through CIGAR) are retained for TULIP input
- name: Racon
  role: Optional downstream consensus correction of nanopore scaffolds (discussed,
    not part of the TULIP assembly itself)
  repo: https://github.com/isovic/racon
- name: Pilon
  role: Polishes the nanopore assembly with Illumina data from a different eel specimen
  repo: https://github.com/broadinstitute/pilon/wiki
- name: Canu
  role: 'Contrast/comparator: state-of-the-art long-read assembler cited as computationally
    prohibitive for large genomes'
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41598-017-07650-6
    title: Rapid de novo assembly of the European eel genome from nanopore sequencing
      reads
  source_content_hash: sha256:a8f101d0d62620f16b6aee3ba9b188e8cf86e8d165e3b5e7fa63a48cf386ed1d
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Generade-nl/TULIP
  tools:
  - BWA MEM
  - Canu
  - Pilon
  - Racon
  - TULIP
  - TULIP (tulipseed.perl / tulipbulb.perl)
  edam_operation: http://edamontology.org/operation_0525
  edam_topics:
  - http://edamontology.org/topic_0196
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_0797
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9375
n_claims: 16
---

# nanopore-genome-assembly

## Summary

De novo assembly of large eukaryotic genomes from uncorrected Oxford Nanopore long reads using TULIP (The Uncorrected Long-read Integration Process), demonstrated by assembling the 860 Mbp European eel (Anguilla anguilla) genome in two days on a single CPU. TULIP splits large assembly problems into manageable subsets of long reads anchored on sparse short 'seed' alignments, replacing quadratic scaling with nearly linear behaviour.

## When to use

Use when assembling a large (hundreds of Mbp to multi-Gbp) eukaryotic genome from uncorrected nanopore FASTQ/FASTA reads on modest hardware, and existing de novo assemblers such as Canu are computationally prohibitive (tens of thousands of CPU hours).

## When NOT to use

- The task is polishing or correcting an existing long-read assembly with Illumina data — use the separate Racon/Pilon polishing step instead (TULIP deliberately skips per-read correction).
- You require a single monolithic de novo assembler run on small genomes where Canu-style assembly is affordable — TULIP's value is in chunking very large genomes.
- Seeds contain highly repetitive sequence or violate the strict requirements (nonzero unique numeric identifiers, exactly equal lengths), which slows graph untangling and can invalidate the layout.

## Inputs

- Long-read FASTA files from Oxford Nanopore MinION sequencing (e.g. R7.3.fasta, R9_pass_1d.fasta, R9_pass_2d.fasta, R9.4_pass_1d.fasta; reads one per line, unique identifiers per file)
- Seed FASTA set (eel_seeds_285.fasta: 285 bp pre-selected non-repetitive seeds with unique nonzero numeric identifiers, all identical length)
- SAM alignment files of reads vs seeds (first 6 SAM fields only, produced by BWA MEM piped through 'cut -f 1,2,3,4,5,6')

## Outputs

- tulip/eel.graph, tulip/eel.graph_tmp (seed graph, text and binary formats)
- tulip/eel.scaffolds, tulip/eel.scaffolds_tmp (ordered seed scaffolds)
- tulip/eel.scaffolds_stats (length statistics per scaffold)
- bulb/eel_scaffold_*.fasta (scaffold sequences; seeds uppercase, long reads lowercase)
- bulb/eel_readbundle_*.fasta (reads used to construct each scaffold)

## How to apply

Build a seed set (e.g. eel_seeds_285.fasta: pre-selected 285 bp 'not too repetitive' fragments with unique nonzero numeric identifiers, all exactly the same length; alternative 270 bp seeds also work). Align each nanopore FASTA read set (R7.3.fasta, R9_pass_1d.fasta, R9_pass_2d.fasta, R9.4_pass_1d.fasta) against the seeds with BWA MEM (e.g. -t 4, per-file -k 14-19, -W 45-60, -r 10 -A 1 -B 1 -O 1 -E 1 -L 0), retaining only the first six SAM fields through the CIGAR string. Write a configuration file with lines of 'type[tab]alignment[tab]original_fasta' (e.g. 'sam  R92D_vs_285.shortsam  R9_pass_2d.fasta'). Run './tulipseed.perl --seedlength 285 --config alignments.txt --diploid --out tulip/eel' to lay out seed scaffolds, then './tulipbulb.perl --seeds eel_seeds_285.fasta --config alignments.txt --input tulip/eel --out bulb/eel' to bundle long-read sequence into scaffold FASTAs. Assess scaffold statistics from tulip/eel.scaffolds_stats; the resulting eel assembly reached N50 1.2 Mbp, two orders of magnitude less fragmented than the previous Illumina draft. Seed density affects assembly size and contiguity, with an optimum around the full seed set (~873058 fragments) and 270 vs 285 bp seeds yielding comparable statistics at equal density.

## Related tools

- **TULIP** (Core two-script pipeline (tulipseed.perl for seed-graph layout, tulipbulb.perl for bundling long reads into scaffolds)) — https://github.com/Generade-nl/TULIP
- **BWA MEM** (Aligns nanopore long reads to the sparse seed set; only the first six SAM fields (through CIGAR) are retained for TULIP input)
- **Racon** (Optional downstream consensus correction of nanopore scaffolds (discussed, not part of the TULIP assembly itself)) — https://github.com/isovic/racon
- **Pilon** (Polishes the nanopore assembly with Illumina data from a different eel specimen) — https://github.com/broadinstitute/pilon/wiki
- **Canu** (Contrast/comparator: state-of-the-art long-read assembler cited as computationally prohibitive for large genomes)

## Examples

```
./tulipseed.perl --seedlength 285 --config alignments.txt --diploid --out tulip/eel
```

## Evaluation signals

- tulipseed.perl produces the expected file set: tulip/eel.graph, .graph_tmp, .scaffolds, .scaffolds_tmp, .seeds_tmp, .layout_log, .scaffolds_stats
- .scaffolds_stats scaffold length statistics show N50 ~1.2 Mbp and total size near the ~860 Mbp genome-size estimate from k-mer analysis, and are two orders of magnitude less fragmented than the Illumina draft (969 Mbp, 77.6 kbp N50)
- The .graph file lists connected seed pairs with orientation (ii/io/oi/oo), evidence counts, gap min/mean/max/SD, and final scaffold IDs in the documented 10-column layout
- Sensitivity check: 270 bp and 285 bp seed sets yield comparable assembly statistics at identical seed density, with an optimum near the full ~873058-seed set (Supplementary Table S2; Fig. 4c,d)
- Every scaffold in bulb/eel_scaffold_*.fasta has reads bundled from bulb/eel_readbundle_*.fasta; rare gaps appear as NNN runs due to the known tulipbulb bug

## Limitations

- Scaffolds that could be merged based on long reads remain separate if the reads do not share a fortuitous seed alignment in the correct position, so the assembly is likely suboptimal
- Large repetitive regions and (sub)telomeric repeats may be absent because they lack frequent ~285 bp islands of unique seed sequence
- In the absence of a high-quality reference, it is not straightforward to establish which assembly is correct at apparent error sites
- The assembly is a hybrid of two technologies, three nanopore chemistries, and two different animals; the consensus could in theory mix up to four haplotypes
- The tulipbulb.perl prototype contains a bug that adds gaps (NNNs) instead of sequence in rare cases
- Optical mapping validation was only planned, not yet performed

## Evidence

- [intro] TULIP splits large problems: "its primary purpose is to split up such large assembly problems into manageable subsets of long reads."
- [abstract] For this 860 Mbp genome, the entire computational process takes two days on a single CPU.: "For this 860 Mbp genome, the entire computational process takes two days on a single CPU."
- [abstract] N50 1.2 Mbp improvement over short-read draft: "The resulting genome assembly significantly improves on a previous draft based on short reads only, both in terms of contiguity (N50 1.2 Mbp) and structural quality."
- [readme] Seed requirements from README: "Strict requirements are that seed sequence identifiers are unique numbers (excluding 0), and that all seeds are of exactly the same length."
- [readme] BWA MEM alignment command from README: "bwa mem -t 4 -k 14 -W 45 -r 10 -A 1 -B 1 -O 1 -E 1 -L 0 285_seeds R7.3.fasta         | cut -f 1,2,3,4,5,6 > R73_vs_285.shortsam"
- [readme] Only six SAM fields used: "Only the first 6 fields of the SAM alignment information is used (up to and including the CIGAR string), so you might want to clip off the rest."
- [methods] tulipseed invocation: "./tulipseed.perl --seedlength 285 --config alignments.txt --diploid --out tulip/eel"
- [discussion] Repetitive region caveat: "large repetitive regions in the genome, as well as (sub) telomeric repeats will not always contain frequent 285 bp islands of unique sequence, and hence could be absent from the assembly"
- [discussion] Suboptimal merge caveat: "scaffolds that could be merged based on long reads remain separate if these reads do not share a fortuitous seed alignment in the correct position"
