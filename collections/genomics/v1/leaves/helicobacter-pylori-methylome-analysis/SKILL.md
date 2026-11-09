---
name: helicobacter-pylori-methylome-analysis
description: Use when the goal is to recover H. pylori methylation motifs de novo
  from ONT fast5 data with a WGA control, and to check whether the detected set reproduces
  a published PacBio motif set (all motifs except TCNNGA expected to match).
when_to_use_negative:
- Input is already a list of called methylation motifs or a methylation call table
  — the concordance step applies, but Snapper re-detection of motifs from signal is
  unnecessary.
- Reads are not raw fast5 signal files (e.g. only basecalled FASTQ/POD5 without resquiggling
  support) — Tombo resquiggling is a prerequisite of this workflow.
- The aim is per-site modification calling rather than de novo motif-level methylation
  characterization — Tombo/Remora single-base tools are a better fit.
edam_operation: http://edamontology.org/operation_0231
edam_topics:
- http://edamontology.org/topic_0654
tools:
- name: Snapper
  role: 'Core motif-caller: greedy position-specific motif enrichment from ONT signal
    statistics to extract methylation motifs de novo'
  repo: DNKonanov/Snapper
- name: Tombo
  role: Resquiggles raw fast5 signal to the reference genome before Snapper is run
  repo: github.com/nanoporetech/tombo
- name: ont_fast5_api
  role: Converts resquiggled single-read fast5 files into the multi-read fast5 format
    Snapper expects
  repo: github.com/nanoporetech/ont_fast5_api
- name: snapper-ont
  role: pip distribution package of Snapper used for installation
- name: Python
  role: Implements the concordance matching and CSV table assembly (with biopython
    for FASTA parsing)
- name: PacBio
  role: Source of the reference methylation motif set for H. pylori J99 (Krebes et
    al. 2014) used in the concordance comparison
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/bioinformatics/btad702
    title: 'Snapper: high-sensitive detection of methylation motifs based on Oxford
      Nanopore reads'
  source_content_hash: sha256:7ab5cf74059fdf872a2d5a8ab3a2b2bc6d6f9855ed930501c0fa61fd2401593b
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/DNKonanov/Snapper
  tools:
  - PacBio
  - Python
  - Snapper
  - Tombo
  - biopython
  - ont_fast5_api
  - snapper-ont
  edam_operation: http://edamontology.org/operation_0231
  edam_topics:
  - http://edamontology.org/topic_0654
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7272727272727273
n_claims: 11
---

# helicobacter-pylori-methylome-analysis

## Summary

De novo detection of methylation motifs in Helicobacter pylori (strain J99) from Oxford Nanopore reads using Snapper's greedy position-specific motif enrichment, followed by concordance comparison against the PacBio-derived reference motif set from Krebes et al. 2014.

## When to use

Use when the goal is to recover H. pylori methylation motifs de novo from ONT fast5 data with a WGA control, and to check whether the detected set reproduces a published PacBio motif set (all motifs except TCNNGA expected to match).

## When NOT to use

- Input is already a list of called methylation motifs or a methylation call table — the concordance step applies, but Snapper re-detection of motifs from signal is unnecessary.
- Reads are not raw fast5 signal files (e.g. only basecalled FASTQ/POD5 without resquiggling support) — Tombo resquiggling is a prerequisite of this workflow.
- The aim is per-site modification calling rather than de novo motif-level methylation characterization — Tombo/Remora single-base tools are a better fit.

## Inputs

- Sample ONT fast5 read directory for H. pylori J99 (multi-fast5 format after Tombo resquiggling)
- Control (whole-genome amplified, WGA) fast5 directory
- H. pylori J99 reference genome in FASTA format
- PacBio-derived reference motif set for H. pylori J99 from Krebes et al. 2014

## Outputs

- Snapper final_motifs_[strand]_[contig_name].fasta files listing de novo detected methylation motifs per strand and contig
- motif_concordance_J99.csv listing matched motifs, Snapper-only motifs, and PacBio-only motifs

## How to apply

Resquiggle the raw fast5 files with Tombo, then convert them to multi-fast5 format using ont_fast5_api. Run snapper with a sample fast5 directory, a WGA (whole-genome amplified) control fast5 directory, and the J99 reference genome FASTA, using a minimum genome coverage depth of 100x for the J99 run (tool default is 40). Collect the de novo motifs from the Snapper output files final_motifs_[strand]_[contig].fasta per strand and contig. Load the PacBio reference motif set for H. pylori J99 from Krebes et al. 2014 and match motifs by exact sequence identity including ambiguous-base notation, recording matches and discordances separately. Write the concordance table (matched motifs, Snapper-only motifs, PacBio-only motifs) as motif_concordance_J99.csv, using Python with biopython where needed.

## Related tools

- **Snapper** (Core motif-caller: greedy position-specific motif enrichment from ONT signal statistics to extract methylation motifs de novo) — DNKonanov/Snapper
- **Tombo** (Resquiggles raw fast5 signal to the reference genome before Snapper is run) — github.com/nanoporetech/tombo
- **ont_fast5_api** (Converts resquiggled single-read fast5 files into the multi-read fast5 format Snapper expects) — github.com/nanoporetech/ont_fast5_api
- **snapper-ont** (pip distribution package of Snapper used for installation)
- **Python** (Implements the concordance matching and CSV table assembly (with biopython for FASTA parsing))
- **PacBio** (Source of the reference methylation motif set for H. pylori J99 (Krebes et al. 2014) used in the concordance comparison)

## Examples

```
snapper -sample_fast5dir ../HelicobacterMod/fast5/J99_multi/ -control_fast5dir ../HelicobacterMod/fast5/J99_wga_multi/ -reference ../HelicobacterMod/genome/J99.fasta -coverage 100
```

## Evaluation signals

- The concordance table (motif_concordance_J99.csv) shows the Snapper and PacBio motif sets as identical except for the TCNNGA motif, which appears as the single discordant entry.
- Each expected final_motifs_[strand]_[contig_name].fasta output file is present for every strand and contig of the J99 reference.
- The run achieved at least the required minimum genome coverage depth of 100x on the J99 data.
- Motif matching uses exact sequence identity including ambiguous-base notation, so TCNNGA-family motifs are not spuriously matched.
- Snapper completes without manual intervention — the article states it 'does not require manual control during the enrichment process'.

## Limitations

- The ONT+Snapper motif set for J99 did not reproduce the PacBio TCNNGA motif (reported in Table 1), so this motif must be expected as a known discordance.
- Snapper depends on Python 3.7; the README warns that later versions might be incompatible because of inner biopython dependencies.
- Tombo, required for resquiggling, is deprecated by Oxford Nanopore in favor of Remora.
- Existing motif-detection algorithms (e.g. MEME with Nanodisco's iterative approach) lack sensitivity and determinism, which is why Snapper's greedy algorithm is used; results from such tools should not be pooled uncritically.

## Evidence

- [readme] Snapper uses balanced approach to compute statistics for each k-mer which is likely to be modified.: "Snapper uses balanced approach to compute statistics for each k-mer which is likely to be modified."
- [intro] Core greedy algorithm feature: "The core feature of Snapper in comparison with other tools is a new high-sensitive greedy algorithm that is used 
for position-specific motif enrichment."
- [methods] Reference snapper invocation on J99: "snapper -sample_fast5dir ../HelicobacterMod/fast5/J99_multi/ -control_fast5dir ../HelicobacterMod/fast5/J99_wga_multi/ -reference ../HelicobacterMod/genome/J99.fasta"
- [abstract] Snapper validated against PacBio J99 methylome: "on H.pylori strain J99 studied earlier by the PacBio technology"
- [abstract] Snapper superiority over MEME/Tombo/Nanodisco: "Snapper does not require manual control during the enrichment process and has enrichment sensitivity higher than MEME coupled with Tombo or Nanodisco instruments"
