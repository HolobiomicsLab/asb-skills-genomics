---
name: long-read-host-identification-evaluation
description: Use when choosing a host-assignment strategy (e.g., Kraken2, Centrifuger,
  MetaMaps, MEGAN-LR, minimap2 variants, read-overlapping) for ARG-carrying long reads
  and you need quantitative misclassification benchmarks before deployment.
when_to_use_negative:
- When the goal is de novo identification of ARG genomic contexts and hosts in new,
  understudied environments — assembly-based (metaFlye + Kraken2) or binning-based
  (SemiBin2 + GTDB-Tk) methods may be better suited
- When reads fall below ~1x genome copy coverage, since inadequate read overlaps make
  clustering unstable and host classification unreliable
- When the target species is absent from the reference database, since all strategies
  here (including read-overlapping) are inherently reference-based and may misclassify
  unseen species
edam_operation: http://edamontology.org/operation_3672
edam_topics:
- http://edamontology.org/topic_0637
- http://edamontology.org/topic_3174
tools:
- name: NanoSim
  role: Trains HQ/LQ error models and simulates 1,000,000-read metagenome samples
    in metagenome mode
- name: DIAMOND
  role: Frameshift-aware blastx extraction of ARG-containing reads against SARG+ with
    stringent parameters
- name: Kraken2
  role: Per-read k-mer taxonomic classifier benchmarked as a baseline host-assignment
    strategy (v2.1.3)
- name: Centrifuger
  role: Compressed per-read taxonomic classifier benchmarked as a baseline (v1.0.1)
- name: MetaMaps
  role: Long-read metagenomic classifier benchmarked as a baseline (v0.1)
- name: MEGAN-LR
  role: Long-read LCA-based classifier benchmarked on the shared minimap2 index (v6.25.9)
- name: minimap2
  role: Shared '-x map-ont -d' ONT index (v2.28) underlying the minimap2+BH/EM/RO
    and MEGAN-LR strategies
- name: Argo
  role: Provides the minimap2+RO read-overlapping strategy evaluated as the top host-identification
    approach
  repo: https://github.com/xinehc/argo
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41467-025-57088-y
    title: Species-resolved profiling of antibiotic resistance genes in complex metagenomes
      through long-read overlapping with Argo
  source_content_hash: sha256:6bbd2f5ce1f37df5d4e87b4d4c3c5d84325ee86c35aba9af356264f9e49aadd5
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/xinehc/argo
  repo_url: https://github.com/xinehc/argo
  tools:
  - Argo
  - Centrifuger
  - DIAMOND
  - Kraken2
  - MEGAN-LR
  - MetaMaps
  - NanoSim
  - minimap2
  edam_operation: http://edamontology.org/operation_3672
  edam_topics:
  - http://edamontology.org/topic_0637
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 14
---

# long-read-host-identification-evaluation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

A simulation-based benchmark skill for quantifying how accurately different taxonomic assignment strategies identify the hosts of ARG-carrying long reads, by computing per-species and overall misclassification rates in simulated high-quality (HQ) and low-quality (LQ) metagenomes.

## When to use

Use when choosing a host-assignment strategy (e.g., Kraken2, Centrifuger, MetaMaps, MEGAN-LR, minimap2 variants, read-overlapping) for ARG-carrying long reads and you need quantitative misclassification benchmarks before deployment.

## When NOT to use

- When the goal is de novo identification of ARG genomic contexts and hosts in new, understudied environments — assembly-based (metaFlye + Kraken2) or binning-based (SemiBin2 + GTDB-Tk) methods may be better suited
- When reads fall below ~1x genome copy coverage, since inadequate read overlaps make clustering unstable and host classification unreliable
- When the target species is absent from the reference database, since all strategies here (including read-overlapping) are inherently reference-based and may misclassify unseen species

## Inputs

- Zymo D6331 (HQ) and Zymo D6300 (LQ) Nanopore read sets for NanoSim model training
- NCBI RefSeq complete genomes (as of 2024-05-10) for 25 primary/opportunistic pathogens and classifier database construction
- SARG+ curated ARG database (sarg.fa)
- Long reads to be profiled (Nanopore or PacBio)

## Outputs

- Per-species misclassification rates for each taxonomic assignment strategy
- Overall misclassification rates split into true-positive, false-positive, and unclassified categories
- Ranked comparison of strategies (e.g., minimap2+RO vs. Kraken2, Centrifuger, MetaMaps, MEGAN-LR, minimap2+BH/EM)

## How to apply

Train two NanoSim (v3.1.0) error models in metagenome mode — one from Zymo D6331 reads (HQ) and one from Zymo D6300 reads (LQ) — then simulate two 1,000,000-read samples over 25 primary/opportunistic pathogens at even abundance, using NCBI RefSeq complete genomes (2024-05-10). Rebuild every classifier database from only RefSeq complete genomes: Kraken2 v2.1.3, Centrifuger v1.0.1, MetaMaps v0.1, and a shared minimap2 v2.28 index built with '-x map-ont -d' for minimap2+BH, minimap2+EM, minimap2+RO, and MEGAN-LR v6.25.9. Extract ARG-containing reads with DIAMOND frameshift-aware blastx against SARG+ ('--evalue 1e-15 --range-culling --frameshift 15 --range-cover 25 --max-target-seqs 25 --max-hsps 0') and classify only those reads with each method (MetaMaps with '--maxmemory 20'). Compute misclassification rate per species and overall, splitting errors into true-positive (misclassified species among the 25), false-positive (outside the 25), and unclassified (no species-level call); the rationale is that per-read classifiers ignore read-overlap context, whereas read-overlapping (Argo's minimap2+RO strategy) aggregates evidence across clusters to reduce host misidentification.

## Related tools

- **NanoSim** (Trains HQ/LQ error models and simulates 1,000,000-read metagenome samples in metagenome mode)
- **DIAMOND** (Frameshift-aware blastx extraction of ARG-containing reads against SARG+ with stringent parameters)
- **Kraken2** (Per-read k-mer taxonomic classifier benchmarked as a baseline host-assignment strategy (v2.1.3))
- **Centrifuger** (Compressed per-read taxonomic classifier benchmarked as a baseline (v1.0.1))
- **MetaMaps** (Long-read metagenomic classifier benchmarked as a baseline (v0.1))
- **MEGAN-LR** (Long-read LCA-based classifier benchmarked on the shared minimap2 index (v6.25.9))
- **minimap2** (Shared '-x map-ont -d' ONT index (v2.28) underlying the minimap2+BH/EM/RO and MEGAN-LR strategies)
- **Argo** (Provides the minimap2+RO read-overlapping strategy evaluated as the top host-identification approach) — https://github.com/xinehc/argo

## Examples

```
diamond blastx --db sarg --query args.fa --evalue 1e-15 --range-culling --frameshift 15 --range-cover 25 --max-target-seqs 25 --max-hsps 0 --outfmt 6
```

## Evaluation signals

- minimap2+RO overall misclassification rate should be approximately 0.19% in the HQ sample and 0.09% in the LQ sample
- Kraken2 worst-case per-species misclassification for Mycobacterium tuberculosis in HQ should approach 16.63%
- Centrifuger overall misclassification should stay at or below 1.32% in both HQ and LQ samples
- Error accounting must partition into true-positive (misclassified among the 25), false-positive (outside the 25), and unclassified (no species-level call) categories summing per strategy
- Simulated HQ and LQ samples must each contain 1,000,000 reads at even taxonomic abundance across the 25 pathogens

## Limitations

- All evaluated strategies are reference-based; they may misclassify unseen species or novel plasmid syntenies
- Results depend on even taxonomic abundance and complete RefSeq genomes; uneven communities or draft references may shift misclassification rates
- Performance at low coverage is unstable — insufficient coverage leads to inadequate read overlaps and ineffective clustering, and Argo marks species below 1x genome copy as unclassified

## Evidence

- [results] minimap2+RO (Argo's read-overlapping strategy) achieved the lowest overall misclassification rates of 0.19% (HQ) and 0.09% (LQ), outperforming Kraken2 (up to 16.63% for M. tuberculosis in HQ), Centrifuger (≤1.32%): [evidence span withheld — anchor status: not_found]
- [methods] DIAMOND frameshift-aware blastx parameters for ARG-read extraction: [evidence span withheld — anchor status: not_found]
