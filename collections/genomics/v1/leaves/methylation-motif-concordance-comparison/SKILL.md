---
name: methylation-motif-concordance-comparison
description: Use when a de novo methylation motif set from ONT sequencing data must
  be validated against a published reference methylome motif set (e.g. PacBio-derived
  H. pylori J99 motifs from Krebes et al. 2014) to quantify concordance.
when_to_use_negative:
- Input is raw ONT signal (fast5 files) rather than an already-detected motif set
  — resquiggle with Tombo and run Snapper first
- The reference motif set is not from the same strain or is not sequence-comparable
  (different genome or modified-base context), making exact-sequence identity matching
  meaningless
- Motifs are to be compared at modified-base or methyltransferase-gene level rather
  than as sequence motifs
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_3674
tools:
- name: Snapper
  role: Produces the de novo methylation motif set from ONT data whose motifs are
    parsed from final_motifs_[strand]_[contig_name].fasta for concordance matching
  repo: https://github.com/DNKonanov/Snapper
- name: Python
  role: Implementation language for parsing FASTA motif outputs, matching motifs by
    exact sequence identity, and writing the concordance CSV
- name: biopython
  role: Dependency used by Snapper for FASTA parsing of motif outputs
- name: snapper-ont
  role: pip distribution of Snapper that generates the motif FASTA outputs used as
    comparison input
- name: PacBio
  role: Technology that produced the reference motif set (Krebes et al. 2014) against
    which ONT/Snapper motifs are compared
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/bioinformatics/btad702
    title: 'Snapper: high-sensitive detection of methylation motifs based on Oxford
      Nanopore reads'
  source_content_hash: sha256:14083e5d141517d0e84161858453d9305e99884740d78db680f13d42bff6118a
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
  - http://edamontology.org/topic_3674
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5555555555555556
n_claims: 9
---

# methylation-motif-concordance-comparison

## Summary

Compares a de novo methylation motif set detected from Oxford Nanopore data (e.g. by Snapper) against a reference motif set derived from an orthogonal technology such as PacBio, by exact sequence matching including ambiguous-base notation, and reports matched and discordant motifs in a concordance table.

## When to use

Use when a de novo methylation motif set from ONT sequencing data must be validated against a published reference methylome motif set (e.g. PacBio-derived H. pylori J99 motifs from Krebes et al. 2014) to quantify concordance.

## When NOT to use

- Input is raw ONT signal (fast5 files) rather than an already-detected motif set — resquiggle with Tombo and run Snapper first
- The reference motif set is not from the same strain or is not sequence-comparable (different genome or modified-base context), making exact-sequence identity matching meaningless
- Motifs are to be compared at modified-base or methyltransferase-gene level rather than as sequence motifs

## Inputs

- Snapper de novo motif output files final_motifs_[strand]_[contig_name].fasta from the ONT run
- Reference methylation motif set from prior PacBio characterization (e.g. Krebes et al. 2014 for H. pylori J99)
- Motif sequences with ambiguous-base notation where applicable

## Outputs

- Concordance table motif_concordance_J99.csv listing matched motifs, Snapper-only motifs, and PacBio-only motifs
- List of discordant motif entries (e.g. TCNNGA)

## How to apply

Parse the Snapper output files final_motifs_[strand]_[contig_name].fasta from the ONT run (e.g. H. pylori J99, run at minimum coverage depth 100×) to collect the de novo detected motifs per strand and contig. Load the reference motif set derived by the orthogonal technology (PacBio, Krebes et al. 2014 for J99) and match motifs between the two sets by exact sequence identity, retaining ambiguous-base notation where applicable. Record matches and discordances separately, then assemble a concordance table listing matched motifs, Snapper-only motifs, and PacBio-only motifs, and write it as motif_concordance_J99.csv. Rationale: Snapper's greedy position-specific motif enrichment should reproduce the PacBio reference set; validation is that the table shows the sets as identical except for a single discordant entry, the TCNNGA motif.

## Related tools

- **Snapper** (Produces the de novo methylation motif set from ONT data whose motifs are parsed from final_motifs_[strand]_[contig_name].fasta for concordance matching) — https://github.com/DNKonanov/Snapper
- **Python** (Implementation language for parsing FASTA motif outputs, matching motifs by exact sequence identity, and writing the concordance CSV)
- **biopython** (Dependency used by Snapper for FASTA parsing of motif outputs)
- **snapper-ont** (pip distribution of Snapper that generates the motif FASTA outputs used as comparison input)
- **PacBio** (Technology that produced the reference motif set (Krebes et al. 2014) against which ONT/Snapper motifs are compared)

## Examples

```
snapper -sample_fast5dir ../HelicobacterMod/fast5/J99_multi/ -control_fast5dir ../HelicobacterMod/fast5/J99_wga_multi/ -reference ../HelicobacterMod/genome/J99.fasta
```

## Evaluation signals

- The concordance table should show the ONT/Snapper motif set as identical to the PacBio reference set except for the TCNNGA motif, which appears as the single discordant entry
- motif_concordance_J99.csv contains separate entries for matched motifs, Snapper-only motifs, and PacBio-only motifs
- Matching uses exact sequence identity, so ambiguous-base notation (e.g. N in TCNNGA) must be preserved and compared literally
- Every motif parsed from final_motifs_[strand]_[contig_name].fasta is accounted for in the table as either a match or a Snapper-only discordance

## Limitations

- Snapper applied to H. pylori J99 ONT data did not recover the TCNNGA motif present in the PacBio reference set, so perfect concordance is not guaranteed even for a well-characterized strain
- Snapper requires minimum genome coverage depth (the J99 run used 100×; the tool default is 40) — insufficient coverage may cause additional motif dropouts
- Snapper's Python dependencies constrain the environment: Python 3.7 is required (later versions might be incompatible because of inner biopython dependencies)

## Evidence

- [results] Snapper applied to H.pylori J99 ONT data with minimum coverage depth 100× discovered the same set of motifs as the PacBio reference set, with the single exception of the TCNNGA motif: [evidence span withheld — anchor status: not_found]
- [abstract] on H.pylori strain J99 studied earlier by the PacBio technology: "on H.pylori strain J99 studied earlier by the PacBio technology"
- [abstract] greedy motif selection algorithm: "to extract methylation motif sequences based on a greedy motif selection algorithm"
- [methods] snapper with sample and control fast5 directories and reference genome: "snapper -sample_fast5dir ../HelicobacterMod/fast5/J99_multi/ -control_fast5dir ../HelicobacterMod/fast5/J99_wga_multi/ -reference ../HelicobacterMod/genome/J99.fasta"
- [readme] python 3.7 later versions might be incompatible because of inner biopython dependencies: "python 3.7 (later versions might be incompatible because of inner biopython dependencies)"
