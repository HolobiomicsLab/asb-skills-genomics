---
name: de-novo-motif-enrichment-with-meme
description: Use when reproducing the Tombo+MEME motif-enrichment baseline on ONT
  native/WGA sample pairs, or when a sensitivity benchmark for methylation motif discovery
  on H. pylori J99-like methylomes is needed.
when_to_use_negative:
- When high sensitivity or deterministic de novo motif discovery is required — the
  article states MEME-based approaches do not extract all H. pylori methylation sites
  de novo and return non-deterministic results
- When the goal is to run Snapper's greedy enrichment itself; this skill is only the
  MEME/Tombo baseline, not the Snapper workflow
- When no matched WGA control sample exists, since Tombo model_sample_compare between
  native and control is a prerequisite for defining modified sites
edam_operation: http://edamontology.org/operation_0238
edam_topics:
- http://edamontology.org/topic_3168
tools:
- name: Tombo
  role: Resquiggles fast5 reads against the reference and runs model_sample_compare
    to flag likely modified bases supplying the context sites
  repo: https://github.com/nanoporetech/tombo
- name: ont_fast5_api
  role: Converts resquiggled fast5 files to the multi-fast5 format required downstream
  repo: https://github.com/nanoporetech/ont_fast5_api
- name: MEME
  role: Performs de novo motif enrichment on the 12/18/22-nt context FASTA sets with
    Tombo-documented parameters
- name: Snapper
  role: The high-sensitivity comparator method whose output (five-motif baseline vs
    Snapper) frames this skill as a benchmark
  repo: https://github.com/DNKonanov/Snapper
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/bioinformatics/btad702
    title: 'Snapper: high-sensitive detection of methylation motifs based on Oxford
      Nanopore reads'
  source_content_hash: sha256:4c08c81f9b63f251c202a3afd1cab1282c596aff9627f0b11834ddbaa6f602d0
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/nanoporetech/tombo
  tools:
  - MEME
  - Snapper
  - Tombo
  - ont_fast5_api
  edam_operation: http://edamontology.org/operation_0238
  edam_topics:
  - http://edamontology.org/topic_3168
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# de-novo-motif-enrichment-with-meme

## Summary

Baseline de novo methylation-motif discovery that runs MEME with Tombo-documented parameters on sequence contexts extracted around significantly modified ONT bases; it serves as the low-sensitivity reference point against which Snapper's greedy enrichment is compared.

## When to use

Use when reproducing the Tombo+MEME motif-enrichment baseline on ONT native/WGA sample pairs, or when a sensitivity benchmark for methylation motif discovery on H. pylori J99-like methylomes is needed.

## When NOT to use

- When high sensitivity or deterministic de novo motif discovery is required — the article states MEME-based approaches do not extract all H. pylori methylation sites de novo and return non-deterministic results
- When the goal is to run Snapper's greedy enrichment itself; this skill is only the MEME/Tombo baseline, not the Snapper workflow
- When no matched WGA control sample exists, since Tombo model_sample_compare between native and control is a prerequisite for defining modified sites

## Inputs

- Native ONT fast5 read directory (H. pylori J99)
- WGA control fast5 read directory (H. pylori J99)
- Reference genome in FASTA format (J99.fasta)
- Tombo model_sample_compare per-base modified-base statistics

## Outputs

- Per strand/contig FASTA context sets of 12-, 18-, and 22-nt sequences centered on significantly modified sites
- MEME motif hits per context set
- Filtered list of genome-highly-represented motifs written to meme_motifs.txt (five motifs for H. pylori J99)

## How to apply

First resquiggle the native ONT fast5 reads against the reference genome with Tombo, then convert both the native sample and the WGA control directories to multi-fast5 format with ont_fast5_api. Run Tombo's model_sample_compare between the native sample and the WGA control to obtain per-base statistics identifying likely modified bases. Extract read sequence contexts of 12, 18, and 22 nt centered on significantly modified sites, writing them per strand/contig as FASTA context sets. Run MEME on each context FASTA using the Tombo-documented parameters, then filter MEME motifs to those highly represented in the genome and write the final motif list (e.g. meme_motifs.txt). Rationale: this pipeline is the published baseline that infers only five motifs for H. pylori J99, demonstrating its low sensitivity compared to Snapper.

## Related tools

- **Tombo** (Resquiggles fast5 reads against the reference and runs model_sample_compare to flag likely modified bases supplying the context sites) — https://github.com/nanoporetech/tombo
- **ont_fast5_api** (Converts resquiggled fast5 files to the multi-fast5 format required downstream) — https://github.com/nanoporetech/ont_fast5_api
- **MEME** (Performs de novo motif enrichment on the 12/18/22-nt context FASTA sets with Tombo-documented parameters)
- **Snapper** (The high-sensitivity comparator method whose output (five-motif baseline vs Snapper) frames this skill as a benchmark) — https://github.com/DNKonanov/Snapper

## Evaluation signals

- Tombo resquiggle and multi-fast5 conversion complete for both native and WGA directories before model_sample_compare is run
- Context FASTA files are produced separately for 12-, 18-, and 22-nt windows and per strand/contig
- MEME is invoked with the Tombo-documented parameters rather than ad-hoc settings
- Final motif list contains only genome-highly-represented motifs; for H. pylori J99 the expected output is exactly five motifs (Table 1, ONT+Tombo column)
- Detected motif count is below the full set of J99 methylation motifs, consistent with the reported low sensitivity of the baseline

## Limitations

- Low sensitivity: the Tombo+MEME baseline inferred only five motifs highly represented in the H. pylori J99 genome instead of the full J99 methylation motif set
- The MEME suite does not extract all H. pylori methylation sites de novo even with the iterative approach implemented in Nanodisco
- No existing methylation motif detection algorithms, including this baseline, are sensitive enough and return deterministic results

## Evidence

- [abstract] Tombo+MEME baseline sensitivity: "the MEME suit does not extract all Helicobacter pylori methylation sites de novo even using the iterative approach implemented in the most up-to-date methylation analysis tool Nanodisco"
- [methods] Resquiggle step: "fast5 files should be resquiggled using [Tombo](https://github.com/nanoporetech/tombo) software."
- [methods] Multi-fast5 conversion step: "After resquiggling, fast5 files should be converted to the multi-fast5 format using [ont_fast5_api](https://github.com/nanoporetech/ont_fast5_api)."
- [abstract] Snapper comparator sensitivity claim: "Snapper does not require manual control during the enrichment process and has enrichment sensitivity higher than MEME coupled with Tombo or Nanodisco instruments"
- [readme] Tombo detection algorithms: [evidence span withheld — anchor status: external_doc]
