---
name: native-versus-wga-modification-calling
description: Use when native Oxford Nanopore fast5 reads and a matched WGA control
  are available for the same strain and the goal is de novo detection of methylated
  bases and methylation motifs against a reference genome.
when_to_use_negative:
- Only a native ONT sample is available with no matched WGA control and no canonical
  base model suitable for the condition, since the sample-compare contrast requires
  the control
- The goal is methyltransferase gene annotation or validation rather than de novo
  motif detection from ONT signal
- Short-read (e.g. PacBio or Illumina) data is the input; Snapper and the Tombo resquiggle
  pipeline are designed for Oxford Nanopore fast5 reads
edam_operation: http://edamontology.org/operation_3186
edam_topics:
- http://edamontology.org/topic_3673
- http://edamontology.org/topic_0654
tools:
- name: Tombo
  role: Resquiggles raw fast5 reads against the reference genome and performs sample-comparison
    modified-base detection between native and WGA samples
  repo: https://github.com/nanoporetech/tombo
- name: ont_fast5_api
  role: Converts resquiggled single-read fast5 files into the multi-read fast5 format
    required downstream
  repo: https://github.com/nanoporetech/ont_fast5_api
- name: Snapper
  role: 'Replaces the Tombo+MEME enrichment step: takes sample/control multi-fast5
    directories and the reference FASTA and greedily extracts position-specific methylation
    motifs with higher sensitivity'
  repo: https://github.com/DNKonanov/Snapper
- name: MEME
  role: Baseline motif-discovery tool run on extracted modified-site context FASTA
    sets with Tombo-documented parameters
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/bioinformatics/btad702
    title: 'Snapper: high-sensitive detection of methylation motifs based on Oxford
      Nanopore reads'
  source_content_hash: sha256:a3358fad0985f7603dd56647cb852f324eb3a799a2ca85ae2efb09e61e3529ed
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/nanoporetech/tombo
  tools:
  - MEME
  - Snapper
  - Tombo
  - ont_fast5_api
  edam_operation: http://edamontology.org/operation_3186
  edam_topics:
  - http://edamontology.org/topic_3673
  - http://edamontology.org/topic_0654
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 5
---

# native-versus-wga-modification-calling

## Summary

Detect methylation sites from ONT data by running Tombo's model_sample_compare between native and WGA-control multi-fast5 reads resquiggled against a reference genome, then discovering enriched motifs from the extracted sequence contexts (Tombo+MEME baseline or Snapper). This comparison-based contrast distinguishes true modifications from sequence-dependent signal effects.

## When to use

Use when native Oxford Nanopore fast5 reads and a matched WGA control are available for the same strain and the goal is de novo detection of methylated bases and methylation motifs against a reference genome.

## When NOT to use

- Only a native ONT sample is available with no matched WGA control and no canonical base model suitable for the condition, since the sample-compare contrast requires the control
- The goal is methyltransferase gene annotation or validation rather than de novo motif detection from ONT signal
- Short-read (e.g. PacBio or Illumina) data is the input; Snapper and the Tombo resquiggle pipeline are designed for Oxford Nanopore fast5 reads

## Inputs

- Native ONT fast5 read directory (single-read or multi-read format) for the strain of interest
- Matched WGA (whole-genome-amplified) control fast5 read directory
- Reference genome FASTA (e.g. H. pylori J99 genome)
- Snapper parameters such as k_size (odd, >=11, default 15), coverage (default 40), ks_t (default 3)

## Outputs

- Per-base modification statistics for the native-vs-WGA comparison
- Sequence context FASTA sets (12/18/22 nt) centered on significantly modified sites, per strand/contig
- Methylation motif list (e.g. meme_motifs.txt) filtered to genome-highly-represented motifs

## How to apply

First resquiggle the raw native and WGA fast5 files against the reference genome with Tombo, then convert the resquiggled files to multi-fast5 format using ont_fast5_api for both the sample and control directories. Run Tombo's model_sample_compare (or Snapper, which takes -sample_fast5dir, -control_fast5dir and -reference) to obtain per-base statistics contrasting native against WGA coverage-normalized signal. For the Tombo+MEME baseline, extract 12-, 18-, and 22-nt read sequence contexts centered on significantly modified sites per strand/contig as FASTA sets, run MEME with Tombo-documented parameters on each context FASTA, and filter to genome-highly-represented motifs. Rationale: comparing native to WGA removes the sequence-specific ion-current baseline so that per-base deviations reflect modifications; on H. pylori J99 this baseline yields only five genome-highly-represented motifs, whereas Snapper's greedy position-specific motif enrichment detects the full set with higher sensitivity and without manual control.

## Related tools

- **Tombo** (Resquiggles raw fast5 reads against the reference genome and performs sample-comparison modified-base detection between native and WGA samples) — https://github.com/nanoporetech/tombo
- **ont_fast5_api** (Converts resquiggled single-read fast5 files into the multi-read fast5 format required downstream) — https://github.com/nanoporetech/ont_fast5_api
- **Snapper** (Replaces the Tombo+MEME enrichment step: takes sample/control multi-fast5 directories and the reference FASTA and greedily extracts position-specific methylation motifs with higher sensitivity) — https://github.com/DNKonanov/Snapper
- **MEME** (Baseline motif-discovery tool run on extracted modified-site context FASTA sets with Tombo-documented parameters)

## Examples

```
snapper -sample_fast5dir ../HelicobacterMod/fast5/J99_multi/ -control_fast5dir ../HelicobacterMod/fast5/J99_wga_multi/ -reference ../HelicobacterMod/genome/J99.fasta
```

## Evaluation signals

- Both native and WGA multi-fast5 directories were produced from resquiggled reads before comparison, per the documented Tombo + ont_fast5_api steps
- On H. pylori J99, the Tombo+MEME baseline should recover approximately five genome-highly-represented motifs (Table 1, ONT+Tombo column); recovering far more may indicate filtering was skipped
- Context extraction produced FASTA sets at three context lengths (12, 18, 22 nt) split per strand/contig
- Snapper run completed with coverage at or above the default minimal genome coverage depth of 40 and odd k_size >= 11
- Resulting motifs are 'highly represented in the genome' (coverage of motif occurrences across the reference), not just enriched in the modified-site subset

## Limitations

- The Tombo+MEME baseline has rather low sensitivity: it inferred only five motifs for H. pylori J99 instead of the full set of J99 methylation motifs
- MEME results require manual filtering to genome-highly-represented motifs, whereas Snapper does not require manual control during enrichment
- Snapper expects Python 3.7; later versions might be incompatible because of inner biopython dependencies
- Tombo is deprecated: the repository is unsupported and Oxford Nanopore recommends Remora instead

## Evidence

- [methods] workflow step 1: "Firstly, fast5 files should be resquiggled using [Tombo](https://github.com/nanoporetech/tombo) software."
- [methods] workflow step 2: "After resquiggling, fast5 files should be converted to the multi-fast5 format using [ont_fast5_api](https://github.com/nanoporetech/ont_fast5_api)."
- [methods] snapper invocation shape: "snapper -sample_fast5dir ../HelicobacterMod/fast5/J99_multi/ -control_fast5dir ../HelicobacterMod/fast5/J99_wga_multi/ -reference ../HelicobacterMod/genome/J99.fasta"
- [results] baseline sensitivity finding: [evidence span withheld — anchor status: not_found]
- [abstract] Snapper advantage: "Snapper does not require manual control during the enrichment process and has enrichment sensitivity higher than MEME coupled with Tombo or Nanodisco instruments"
- [readme] README dependency constraint: "python 3.7 (later versions might be incompatible because of inner biopython dependencies)"
