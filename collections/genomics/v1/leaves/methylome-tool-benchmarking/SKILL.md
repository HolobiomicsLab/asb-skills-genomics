---
name: methylome-tool-benchmarking
description: Use when comparing methylation motif callers (Nanodisco, Snapper, MEME+Tombo)
  on ONT sample/WGA-control fast5 pairs against a PacBio-validated methylome, or when
  reproducing known tool failure modes like motif merging.
when_to_use_negative:
- The goal is de novo characterization of a novel strain's methylome rather than tool
  comparison — apply Snapper directly (it requires no manual control during enrichment)
  instead of running benchmark rounds
- Input reads lack a matched WGA control fast5 set, since the tools here rely on sample-versus-control
  signal-level k-mer comparisons
- Motifs are already known and curated, so no de novo discovery or collision/merging
  behavior is being evaluated
edam_operation: http://edamontology.org/operation_0238
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3168
tools:
- name: Nanodisco
  role: 'Tool under benchmark: run in fully automated, non-iterative mode to test
    whether it recovers J99''s CATG and GCGC motifs without motif collision'
- name: Snapper
  role: Higher-sensitivity comparison tool using greedy position-specific motif enrichment,
    deterministic and without manual control during enrichment
  repo: DNKonanov/Snapper
- name: Tombo
  role: Resquiggles raw fast5 signal to the J99 reference before motif calling
  repo: github.com/nanoporetech/tombo
- name: ont_fast5_api
  role: Converts resquiggled single-read fast5 files to the multi-fast5 format required
    downstream
  repo: github.com/nanoporetech/ont_fast5_api
- name: MEME
  role: Baseline motif-discovery comparison; noted to miss H. pylori sites even with
    Nanodisco's iterative approach
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/bioinformatics/btad702
    title: 'Snapper: high-sensitive detection of methylation motifs based on Oxford
      Nanopore reads'
  source_content_hash: sha256:0683063c739f6ef74eba7e56722cb047ee355914dd7d745a1698bf86229dfe37
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/DNKonanov/Snapper
  tools:
  - MEME
  - Nanodisco
  - Snapper
  - Tombo
  - ont_fast5_api
  edam_operation: http://edamontology.org/operation_0238
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3168
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5714285714285714
n_claims: 7
---

# methylome-tool-benchmarking

## Summary

Benchmarks de novo methylation motif detection tools on Oxford Nanopore data against a known ground-truth methylome (H. pylori J99, previously characterized by PacBio), exposing failure modes such as Nanodisco's automated-mode motif collisions that merge CATG and GCGC into GCRYG.

## When to use

Use when comparing methylation motif callers (Nanodisco, Snapper, MEME+Tombo) on ONT sample/WGA-control fast5 pairs against a PacBio-validated methylome, or when reproducing known tool failure modes like motif merging.

## When NOT to use

- The goal is de novo characterization of a novel strain's methylome rather than tool comparison — apply Snapper directly (it requires no manual control during enrichment) instead of running benchmark rounds
- Input reads lack a matched WGA control fast5 set, since the tools here rely on sample-versus-control signal-level k-mer comparisons
- Motifs are already known and curated, so no de novo discovery or collision/merging behavior is being evaluated

## Inputs

- J99 ONT sample multi-fast5 directory (e.g. ../HelicobacterMod/fast5/J99_multi/)
- J99 WGA-control multi-fast5 directory (e.g. ../HelicobacterMod/fast5/J99_wga_multi/)
- J99 reference genome FASTA (../HelicobacterMod/genome/J99.fasta)
- Resquiggled fast5 files produced via Tombo and converted with ont_fast5_api

## Outputs

- nanodisco_automated_motifs.csv containing the automated-mode motif record (merged GCRYG in place of separate CATG and GCGC entries)
- List of most-represented detected motifs per tool for sensitivity comparison

## How to apply

Prepare the H. pylori J99 ONT fast5 datasets (native sample and WGA control) with the J99 reference assembly (J99.fasta), resquiggling with Tombo and converting to multi-fast5 with ont_fast5_api. Run Nanodisco v1.0.3 in fully automated mode — the non-iterative configuration requiring no manual curation between enrichment rounds — on signal-level k-mer sample-vs-control comparisons to de novo detect methylated motifs. Extract the most-represented motifs, which for J99 are expected to be CATG and GCGC, then check whether Nanodisco's automated consensus step merges them into the degenerate IUPAC motif GCRYG. Run Snapper on the same resquiggled multi-fast5 inputs (its greedy motif selection algorithm needs no manual control during enrichment) to serve as the higher-sensitivity comparison. Record the automated-mode motif output to nanodisco_automated_motifs.csv and judge correctness by whether separate CATG and GCGC entries survive or collapse into the merged GCRYG record.

## Related tools

- **Nanodisco** (Tool under benchmark: run in fully automated, non-iterative mode to test whether it recovers J99's CATG and GCGC motifs without motif collision)
- **Snapper** (Higher-sensitivity comparison tool using greedy position-specific motif enrichment, deterministic and without manual control during enrichment) — DNKonanov/Snapper
- **Tombo** (Resquiggles raw fast5 signal to the J99 reference before motif calling) — github.com/nanoporetech/tombo
- **ont_fast5_api** (Converts resquiggled single-read fast5 files to the multi-fast5 format required downstream) — github.com/nanoporetech/ont_fast5_api
- **MEME** (Baseline motif-discovery comparison; noted to miss H. pylori sites even with Nanodisco's iterative approach)

## Examples

```
snapper -sample_fast5dir ../HelicobacterMod/fast5/J99_multi/ -control_fast5dir ../HelicobacterMod/fast5/J99_wga_multi/ -reference ../HelicobacterMod/genome/J99.fasta
```

## Evaluation signals

- nanodisco_automated_motifs.csv exists and contains a motif record
- The record reports the merged degenerate IUPAC motif GCRYG rather than separate CATG and GCGC entries, reproducing the documented collision
- No standalone CATG or GCGC rows appear in the automated-mode output
- Comparison run of Snapper on the same J99_multi / J99_wga_multi inputs resolves the motifs that Nanodisco merged, confirming the failure is tool-specific
- Detected motifs are consistent with the H. pylori J99 methylome previously characterized by PacBio

## Limitations

- In fully automated (non-iterative, no manual curation) mode, Nanodisco fails to correctly discover the most-represented J99 motifs CATG and GCGC, merging them into GCRYG due to excessive motif collisions
- The MEME suit does not extract all Helicobacter pylori methylation sites de novo even using Nanodisco's iterative approach
- Snapper depends on Python 3.7; later versions might be incompatible because of inner biopython dependencies
- Tombo is unsupported and deprecated by Oxford Nanopore in favor of Remora, which may affect long-term reproducibility of the resquiggling step

## Evidence

- [abstract] MEME misses H. pylori sites even with Nanodisco iterative mode: "the MEME suit does not extract all Helicobacter pylori methylation sites de novo even using the iterative approach implemented in the most up-to-date methylation analysis tool Nanodisco"
- [abstract] Snapper as sensitive automated alternative: "We present Snapper, a new highly sensitive approach, to extract methylation motif sequences based on a greedy motif selection algorithm."
- [methods] Grounded J99 comparison invocation: "snapper -sample_fast5dir ../HelicobacterMod/fast5/J99_multi/ -control_fast5dir ../HelicobacterMod/fast5/J99_wga_multi/ -reference ../HelicobacterMod/genome/J99.fasta"
- [readme] Preprocessing requirement before motif calling: "Firstly, fast5 files should be resquiggled using [Tombo](https://github.com/nanoporetech/tombo) software."
