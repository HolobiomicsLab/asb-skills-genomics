---
name: modified-base-kmer-context-extraction
description: Use when you need a baseline, low-automation de novo methylation motif
  caller for native ONT fast5 data with a WGA control (e.g., H. pylori J99) and want
  to compare its sensitivity against Snapper.
when_to_use_negative:
- When the goal is maximal de novo motif sensitivity rather than a baseline — Snapper's
  greedy position-specific motif enrichment has higher sensitivity than MEME coupled
  with Tombo or Nanodisco and does not require manual control
- When no WGA control sample is available, since model_sample_compare requires comparing
  native against control fast5 directories
- When starting from data already processed beyond resquiggled fast5 (e.g., pre-computed
  motif lists or feature tables)
tools:
- name: Tombo
  role: Resquiggles fast5 reads against the reference and runs model_sample_compare
    to produce per-base modification statistics
  repo: https://github.com/nanoporetech/tombo
- name: ont_fast5_api
  role: Converts resquiggled single-read fast5 files to the multi-fast5 format required
    downstream
  repo: https://github.com/nanoporetech/ont_fast5_api
- name: MEME
  role: Discovers enriched methylation motifs from the extracted 12/18/22-nt context
    FASTA sets using Tombo-documented parameters
- name: Snapper
  role: High-sensitivity greedy alternative against which this MEME baseline's motif
    yield is compared
  repo: DNKonanov/Snapper
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/bioinformatics/btad702
    title: 'Snapper: high-sensitive detection of methylation motifs based on Oxford
      Nanopore reads'
  source_content_hash: sha256:52cb832d8149e6a71e5980f45f913dfb8af5c935045485d874294de119fd3a0b
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/nanoporetech/tombo
  tools:
  - MEME
  - Snapper
  - Tombo
  - ont_fast5_api
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# modified-base-kmer-context-extraction

## Summary

Reproduces the Tombo+MEME baseline for de novo methylation motif discovery on Oxford Nanopore data: resquiggle native fast5 reads, run Tombo model_sample_compare against a WGA control, extract 12/18/22-nt k-mer sequence contexts around significantly modified sites, and run MEME to identify genome-highly-represented methylation motifs.

## When to use

Use when you need a baseline, low-automation de novo methylation motif caller for native ONT fast5 data with a WGA control (e.g., H. pylori J99) and want to compare its sensitivity against Snapper.

## When NOT to use

- When the goal is maximal de novo motif sensitivity rather than a baseline — Snapper's greedy position-specific motif enrichment has higher sensitivity than MEME coupled with Tombo or Nanodisco and does not require manual control
- When no WGA control sample is available, since model_sample_compare requires comparing native against control fast5 directories
- When starting from data already processed beyond resquiggled fast5 (e.g., pre-computed motif lists or feature tables)

## Inputs

- Native Oxford Nanopore fast5 read files (e.g., H. pylori J99 native sample)
- Whole-genome-amplified (WGA) control fast5 read files for sample comparison
- Reference genome in FASTA format (e.g., J99.fasta)
- Tombo per-base modification statistics from model_sample_compare

## Outputs

- Per-base modified-site statistics (Tombo model_sample_compare output)
- FASTA files of 12-, 18-, and 22-nt read sequence contexts centered on significantly modified sites, written per strand/contig
- MEME-enriched motif list filtered to genome-highly-represented motifs (e.g., meme_motifs.txt with five J99 motifs)

## How to apply

First resquiggle the native ONT fast5 reads against the reference genome using Tombo, then convert both the native and WGA-control fast5 directories to the multi-fast5 format with ont_fast5_api. Run Tombo's model_sample_compare between the native sample and the WGA control to obtain per-base statistics flagging likely modified bases. From these statistics, extract read sequence contexts of 12, 18, and 22 nt centered on significantly modified sites, writing them per strand/contig as FASTA context sets. Run MEME on each context FASTA using the Tombo-documented parameters, then filter the MEME results to genome-highly-represented motifs — on H. pylori J99 this baseline yields only five motifs, a deliberately low-sensitivity reference point. Rationale: the Snapper paper showed MEME does not extract all H. pylori methylation sites de novo, making this pipeline a quantified negative-control baseline for greedy, higher-sensitivity motif discovery.

## Related tools

- **Tombo** (Resquiggles fast5 reads against the reference and runs model_sample_compare to produce per-base modification statistics) — https://github.com/nanoporetech/tombo
- **ont_fast5_api** (Converts resquiggled single-read fast5 files to the multi-fast5 format required downstream) — https://github.com/nanoporetech/ont_fast5_api
- **MEME** (Discovers enriched methylation motifs from the extracted 12/18/22-nt context FASTA sets using Tombo-documented parameters)
- **Snapper** (High-sensitivity greedy alternative against which this MEME baseline's motif yield is compared) — DNKonanov/Snapper

## Examples

```
snapper -sample_fast5dir ../HelicobacterMod/fast5/J99_multi/ -control_fast5dir ../HelicobacterMod/fast5/J99_wga_multi/ -reference ../HelicobacterMod/genome/J99.fasta
```

## Evaluation signals

- On H. pylori J99 native/WGA data, MEME infers exactly five motifs highly represented in the genome (Table 1, ONT+Tombo column), not the full J99 methylation set
- Context FASTA files exist for each of the three k-mer sizes (12, 18, 22 nt), each per strand/contig
- model_sample_compare ran on both the native multi-fast5 directory and the J99 WGA control directory converted via ont_fast5_api
- Final filtered motif list (meme_motifs.txt) contains only genome-highly-represented motifs after excluding low-representation MEME hits
- Motif yield can be cross-checked against the PacBio-characterized J99 methylome referenced by the Snapper paper

## Limitations

- Low sensitivity: the Tombo+MEME baseline infers only five motifs on H. pylori J99 instead of the full set of J99 methylation motifs
- MEME does not extract all Helicobacter pylori methylation sites de novo, even using Nanodisco's iterative approach, and results are not deterministic
- The pipeline requires manual control during the enrichment/filtering process, unlike Snapper
- Tombo is now unsupported upstream and replaced by Remora, which may affect long-term reproducibility

## Evidence

- [methods] Resquiggle the J99 native ONT fast5 reads against the J99 reference genome using Tombo: "Firstly, fast5 files should be resquiggled using [Tombo](https://github.com/nanoporetech/tombo) software."
- [readme] Convert resquiggled fast5 files to multi-fast5 format with ont_fast5_api: "After resquiggling, fast5 files should be converted to the multi-fast5 format using [ont_fast5_api](https://github.com/nanoporetech/ont_fast5_api)."
- [abstract] MEME fails to extract all H. pylori methylation sites de novo: "the MEME suit does not extract all Helicobacter pylori methylation sites de novo even using the iterative approach implemented in the most up-to-date methylation analysis tool Nanodisco"
- [abstract] Snapper has higher enrichment sensitivity than MEME coupled with Tombo: "Snapper does not require manual control during the enrichment process and has enrichment sensitivity higher than MEME coupled with Tombo or Nanodisco instruments"
- [readme] Tombo is deprecated in favor of Remora: [evidence span withheld — anchor status: external_doc]
