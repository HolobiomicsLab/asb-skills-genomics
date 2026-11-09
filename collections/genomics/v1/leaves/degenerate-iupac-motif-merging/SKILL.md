---
name: degenerate-iupac-motif-merging
description: Use when interpreting or reproducing Nanodisco automated-mode output
  on ONT methylation data where distinct detected motifs (e.g. CATG and GCGC on H.pylori
  J99) are merged into one degenerate IUPAC motif such as GCRYG.
when_to_use_negative:
- When motifs are distinct and should be reported separately — the merge is an automated-mode
  artifact of motif collisions, and PacBio data for J99 confirms CATG and GCGC as
  independent methylated motifs
- When a deterministic tool without manual curation is required — Snapper does not
  require manual control during enrichment and was designed to avoid this failure
  mode
- When the goal is the true J99 methylome composition rather than reproducing Nanodisco's
  automated behavior
edam_operation: http://edamontology.org/operation_0238
edam_topics:
- http://edamontology.org/topic_0157
tools:
- name: Nanodisco
  role: performs the automated (non-iterative) motif detection whose consensus step
    merges detected motifs into degenerate IUPAC motifs such as GCRYG
- name: Snapper
  role: greedy position-specific motif enrichment alternative that does not require
    manual control and correctly recovers distinct motifs; used as the deterministic
    comparison baseline
  repo: github.com/DNKonanov/Snapper
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/bioinformatics/btad702
    title: 'Snapper: high-sensitive detection of methylation motifs based on Oxford
      Nanopore reads'
  source_content_hash: sha256:7e91b846de0b93c5a41ee4ac0f2cc97d962aef8ff09d66242da30af50c4ffb5b
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Nanodisco
  - Snapper
  edam_operation: http://edamontology.org/operation_0238
  edam_topics:
  - http://edamontology.org/topic_0157
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# degenerate-iupac-motif-merging

## Summary

Reproduces and interprets Nanodisco's fully automated (non-iterative) methylation motif calling, in which its consensus step collapses the most-represented H.pylori J99 motifs CATG and GCGC into the single degenerate IUPAC motif GCRYG due to excessive motif collisions — a documented failure mode of automated motif discovery.

## When to use

Use when interpreting or reproducing Nanodisco automated-mode output on ONT methylation data where distinct detected motifs (e.g. CATG and GCGC on H.pylori J99) are merged into one degenerate IUPAC motif such as GCRYG.

## When NOT to use

- When motifs are distinct and should be reported separately — the merge is an automated-mode artifact of motif collisions, and PacBio data for J99 confirms CATG and GCGC as independent methylated motifs
- When a deterministic tool without manual curation is required — Snapper does not require manual control during enrichment and was designed to avoid this failure mode
- When the goal is the true J99 methylome composition rather than reproducing Nanodisco's automated behavior

## Inputs

- H.pylori J99 ONT sample fast5 dataset
- H.pylori J99 WGA-control fast5 dataset
- J99 reference assembly (J99.fasta)
- Nanodisco v1.0.3 automated-mode (non-iterative) configuration

## Outputs

- nanodisco_automated_motifs.csv containing the automated-mode motif record reporting merged motif GCRYG in place of separate CATG and GCGC entries

## How to apply

Prepare the J99 sample and WGA-control fast5 datasets and run Nanodisco v1.0.3 in fully automated mode — the non-iterative configuration requiring no manual curation between enrichment rounds — performing signal-level k-mer comparisons of sample versus WGA control against the J99.fasta reference. Extract the most-represented detected motifs from the automated-mode results, which should be CATG and GCGC. Nanodisco's automated consensus step then merges these into the degenerate IUPAC motif GCRYG (R = A/G, Y = C/T), a merge driven by excessive motif collisions rather than by genuine single-site methylation. Write the automated-mode motif record to nanodisco_automated_motifs.csv and treat the presence of the merged GCRYG motif — instead of separate CATG and GCGC entries — as the expected outcome of the collision. Validate by comparing against the PacBio-characterized J99 methylome, where CATG and GCGC are distinct methylated motifs, so the merge represents a loss of motif specificity, not a biological finding.

## Related tools

- **Nanodisco** (performs the automated (non-iterative) motif detection whose consensus step merges detected motifs into degenerate IUPAC motifs such as GCRYG)
- **Snapper** (greedy position-specific motif enrichment alternative that does not require manual control and correctly recovers distinct motifs; used as the deterministic comparison baseline) — github.com/DNKonanov/Snapper

## Evaluation signals

- nanodisco_automated_motifs.csv exists and its motif record reports the merged GCRYG motif in place of separate CATG and GCGC entries
- the degenerate motif's IUPAC expansion must cover both source motifs at every aligned position (R = A/G, Y = C/T)
- the most-represented detected motifs before the consensus step are CATG and GCGC, matching the PacBio-characterized J99 methylome
- no separate CATG or GCGC entries remain in the automated-mode output after the consensus merge
- the run used the fully automated (non-iterative, no-manual-curation) Nanodisco configuration, distinguishable from manually curated iterative results

## Limitations

- In fully automated mode Nanodisco fails to correctly discover the most-represented motifs on J99, merging CATG and GCGC into GCRYG due to excessive motif collisions
- No existing methylation motif detection algorithms are sensitive enough and return deterministic results; MEME does not extract all H.pylori methylation sites de novo even with Nanodisco's iterative approach
- The merged GCRYG motif masks the identity of the underlying distinct methylation sites, so downstream methylation-site annotation is underdetermined from the automated output alone

## Evidence

- [other] In fully automated mode on H.pylori J99, Nanodisco fails to correctly discover the most-represented motifs CATG and GCGC, merging them into the single motif GCRYG due to excessive motif collisions.: [evidence span withheld — anchor status: not_found]
- [abstract] the MEME suit does not extract all Helicobacter pylori methylation sites de novo even using the iterative approach implemented in the most up-to-date methylation analysis tool Nanodisco: "the MEME suit does not extract all Helicobacter pylori methylation sites de novo even using the iterative approach implemented in the most up-to-date methylation analysis tool Nanodisco"
- [abstract] Snapper does not require manual control during the enrichment process and has enrichment sensitivity higher than MEME coupled with Tombo or Nanodisco instruments: "Snapper does not require manual control during the enrichment process and has enrichment sensitivity higher than MEME coupled with Tombo or Nanodisco instruments"
- [methods] sample and WGA control fast5 directories with the J99 reference genome used for signal-level k-mer methylation comparisons: "snapper -sample_fast5dir ../HelicobacterMod/fast5/J99_multi/ -control_fast5dir ../HelicobacterMod/fast5/J99_wga_multi/ -reference ../HelicobacterMod/genome/J99.fasta"
