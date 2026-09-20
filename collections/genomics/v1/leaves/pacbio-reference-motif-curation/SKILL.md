---
name: pacbio-reference-motif-curation
description: Use when validating de novo methylation motifs detected from ONT data
  (e.g. Snapper output final_motifs_[strand]_[contig].fasta) against an established
  PacBio methylome reference for the same strain.
when_to_use_negative:
- The strain has no previously characterized PacBio methylome, so no reference motif
  set exists for exact-sequence matching.
- You need de novo motif discovery itself from raw nanopore signal — use Snapper (with
  Tombo resquiggling and multi-fast5 conversion) instead of this comparison step.
- The motifs of interest require methylation-type assignment or modification-site-level
  mapping rather than motif sequence concordance.
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_3168
- http://edamontology.org/topic_0654
tools:
- name: Snapper
  role: Produces the de novo ONT methylation motifs (final_motifs_[strand]_[contig_name].fasta)
    that are compared against the PacBio reference set
  repo: https://github.com/DNKonanov/Snapper
- name: snapper-ont
  role: Pip distribution of Snapper used to run motif detection on resquiggled multi-fast5
    ONT data
- name: Python
  role: Implementation language for parsing Snapper FASTA outputs and building the
    motif concordance CSV
- name: biopython
  role: Parses FASTA motif files and handles ambiguous-base motif sequences during
    exact matching
- name: PacBio
  role: Source technology of the reference methylome motif set used as ground truth
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/bioinformatics/btad702
    title: 'Snapper: high-sensitive detection of methylation motifs based on Oxford
      Nanopore reads'
  source_content_hash: sha256:135c557a7221e0a696826be19469a68dad500816fc858a72cf11d80b14fbbc36
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/DNKonanov/Snapper
  tools:
  - PacBio
  - Python
  - Snapper
  - biopython
  - snapper-ont
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_3168
  - http://edamontology.org/topic_0654
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 6
---

# pacbio-reference-motif-curation

## Summary

Curates a PacBio-derived reference set of methylation motifs (e.g. H. pylori J99 motifs from Krebes et al. 2014) and matches it against de novo motifs detected by Snapper on Oxford Nanopore reads to build a concordance table of matched and discordant motifs.

## When to use

Use when validating de novo methylation motifs detected from ONT data (e.g. Snapper output final_motifs_[strand]_[contig].fasta) against an established PacBio methylome reference for the same strain.

## When NOT to use

- The strain has no previously characterized PacBio methylome, so no reference motif set exists for exact-sequence matching.
- You need de novo motif discovery itself from raw nanopore signal — use Snapper (with Tombo resquiggling and multi-fast5 conversion) instead of this comparison step.
- The motifs of interest require methylation-type assignment or modification-site-level mapping rather than motif sequence concordance.

## Inputs

- Snapper de novo motif output files: final_motifs_[strand]_[contig_name].fasta from an ONT run
- PacBio-derived reference methylation motif set for the same strain (e.g. H. pylori J99, Krebes et al. 2014)
- Optionally, the Snapper run parameters (e.g. minimum coverage depth 100×) to contextualize the comparison

## Outputs

- Concordance table motif_concordance_J99.csv listing matched motifs, Snapper-only motifs, and PacBio-only motifs
- Record of matches and discordances between ONT/Snapper motifs and PacBio reference motifs

## How to apply

Parse the Snapper output files final_motifs_[strand]_[contig_name].fasta to collect de novo detected methylation motifs per strand and contig, and load the PacBio-derived reference motif set (for H. pylori J99, that of Krebes et al. 2014). Match motifs between the two sets by exact sequence identity, including ambiguous-base notation where applicable, recording matches and discordances separately. Assemble a concordance table listing matched motifs, Snapper-only motifs, and PacBio-only motifs, and write it as motif_concordance_J99.csv. Rationale: Snapper's greedy position-specific motif enrichment on ONT data is expected to reproduce the PacBio reference set when run with sufficient depth — for the J99 run with minimum coverage depth 100×, all motifs were recovered except the TCNNGA motif.

## Related tools

- **Snapper** (Produces the de novo ONT methylation motifs (final_motifs_[strand]_[contig_name].fasta) that are compared against the PacBio reference set) — https://github.com/DNKonanov/Snapper
- **snapper-ont** (Pip distribution of Snapper used to run motif detection on resquiggled multi-fast5 ONT data)
- **Python** (Implementation language for parsing Snapper FASTA outputs and building the motif concordance CSV)
- **biopython** (Parses FASTA motif files and handles ambiguous-base motif sequences during exact matching)
- **PacBio** (Source technology of the reference methylome motif set used as ground truth)

## Examples

```
snapper -sample_fast5dir ../HelicobacterMod/fast5/J99_multi/ -control_fast5dir ../HelicobacterMod/fast5/J99_wga_multi/ -reference ../HelicobacterMod/genome/J99.fasta -coverage 100
```

## Evaluation signals

- motif_concordance_J99.csv contains all three categories: matched motifs, Snapper-only motifs, and PacBio-only motifs.
- For the H. pylori J99 run (coverage depth 100×), the concordance table shows the two motif sets as identical except for the TCNNGA motif, which appears as the single discordant entry.
- Each motif match is based on exact sequence identity, including ambiguous-base notation where applicable — no fuzzy or partial matches silently accepted.
- Snapper-only and PacBio-only motif lists together cover all reference motifs from Krebes et al. 2014 and all de novo Snapper motifs with no unaccounted entries.
- Motif files parsed cover all strands and contigs present in the final_motifs_[strand]_[contig_name].fasta outputs.

## Limitations

- Snapper on J99 ONT data with coverage depth 100× recovered all PacBio reference motifs except TCNNGA — rare motifs may be missed even at high coverage.
- Snapper requires Python 3.7; later versions might be incompatible because of inner biopython dependencies.
- Comparison by exact sequence identity may miss biologically equivalent motifs written in different ambiguous-base notations unless normalized.

## Evidence

- [results] Snapper applied to H.pylori J99 ONT data with minimum coverage depth 100× discovered the same set of motifs as the PacBio reference set, with the single exception of the TCNNGA motif: [evidence span withheld — anchor status: not_found]
- [abstract] Snapper is implemented using Python and is freely available as a pip package named "snapper-ont.": "Snapper is implemented using Python and is freely available as a pip package named "snapper-ont.""
- [other] Parse the Snapper output files final_motifs_[strand]_[contig_name].fasta for the J99 run to collect the de novo detected methylation motifs per strand and contig.: [evidence span withheld — anchor status: not_found]
- [readme] python 3.7 (later versions might be incompatible because of inner biopython dependencies): "python 3.7 (later versions might be incompatible because of inner biopython dependencies)"
