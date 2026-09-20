---
name: genome-background-motif-comparison
description: Use when a set of candidate modified k-mers (seq_set) from ONT reads
  must be scored for over-representation against a reference genome to extract methylation
  motifs de novo.
when_to_use_negative:
- The goal is per-read modified-base calling with pre-trained models (e.g. Tombo alternative
  model) rather than de novo motif enrichment
- No suitable reference genome is available to serve as the negative background for
  enrichment statistics
- Motifs are already known and only need verification rather than de novo discovery
edam_operation: http://edamontology.org/operation_0238
edam_topics:
- http://edamontology.org/topic_3168
- http://edamontology.org/topic_0654
tools:
- name: Snapper
  role: Implements the greedy motif selection algorithm that computes chi-squared
    k-mer enrichment against the reference genome background
  repo: DNKonanov/Snapper
- name: snapper-ont
  role: Pip-installable distribution of Snapper providing the snapper CLI used to
    run the enrichment
- name: scipy
  role: Provides the chi-squared statistic used to score k-mer over-representation
- name: Tombo
  role: Resquiggles raw fast5 signal to the reference upstream of the enrichment step
  repo: https://github.com/nanoporetech/tombo
- name: ont_fast5_api
  role: Converts resquiggled single-read fast5 files into the multi-fast5 format Snapper
    consumes
  repo: https://github.com/nanoporetech/ont_fast5_api
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1093/bioinformatics/btad702
    title: 'Snapper: high-sensitive detection of methylation motifs based on Oxford
      Nanopore reads'
  source_content_hash: sha256:9285ac9d3c0e18580abc03291e8b6912d4b25f01d10e1e51b2c1461f5811c776
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/DNKonanov/Snapper
  tools:
  - Python
  - Snapper
  - Tombo
  - ont_fast5_api
  - scipy
  - snapper-ont
  edam_operation: http://edamontology.org/operation_0238
  edam_topics:
  - http://edamontology.org/topic_3168
  - http://edamontology.org/topic_0654
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 16
---

# genome-background-motif-comparison

## Summary

Computes chi-squared enrichment statistics for candidate k-mers from a Nanopore-derived seq_set against a reference genome used as negative background, driving Snapper's greedy position-specific motif enrichment for methylation motif discovery.

## When to use

Use when a set of candidate modified k-mers (seq_set) from ONT reads must be scored for over-representation against a reference genome to extract methylation motifs de novo.

## When NOT to use

- The goal is per-read modified-base calling with pre-trained models (e.g. Tombo alternative model) rather than de novo motif enrichment
- No suitable reference genome is available to serve as the negative background for enrichment statistics
- Motifs are already known and only need verification rather than de novo discovery

## Inputs

- seq_set of passed-motif k-mers from upstream Nanopore read processing
- reference genome in FASTA format (negative background)
- sample multi-fast5 directory and control (WGA) multi-fast5 directory

## Outputs

- final_motifs.fasta containing a sorted list of extracted methylation motifs with confidence levels
- per-motif chi-squared confidence values

## How to apply

Load the passed-motif k-mer seq_set produced upstream and the reference genome in FASTA format as the negative background. Compute an enrichment statistic (chi-squared via scipy.stats) for each candidate k-mer in the seq_set versus the genome, using odd k-mer sizes (default k_size 15, must be >= 11; long_k_size default 29, must be >= 21). Greedily select the motif with the highest over-representation score, apply position-specific motif adjustment, then filter the seq_set by removing sequences explained by the extracted motif. Repeat until the confidence value (minimal chi-squared statistic, min_conf default 100) is not met or the number of extracted motifs reaches max_motifs (default 20). Sort surviving motifs and write them with confidence levels to final_motifs.fasta.

## Related tools

- **Snapper** (Implements the greedy motif selection algorithm that computes chi-squared k-mer enrichment against the reference genome background) — DNKonanov/Snapper
- **snapper-ont** (Pip-installable distribution of Snapper providing the snapper CLI used to run the enrichment)
- **scipy** (Provides the chi-squared statistic used to score k-mer over-representation)
- **Tombo** (Resquiggles raw fast5 signal to the reference upstream of the enrichment step) — https://github.com/nanoporetech/tombo
- **ont_fast5_api** (Converts resquiggled single-read fast5 files into the multi-fast5 format Snapper consumes) — https://github.com/nanoporetech/ont_fast5_api

## Examples

```
snapper -sample_fast5dir ../HelicobacterMod/fast5/J99_multi/ -control_fast5dir ../HelicobacterMod/fast5/J99_wga_multi/ -reference ../HelicobacterMod/genome/J99.fasta
```

## Evaluation signals

- final_motifs.fasta exists and contains a sorted motif list
- Every emitted motif meets the min_conf threshold (default minimal chi-squared statistic 100)
- The number of extracted motifs does not exceed max_motifs (default 20)
- k_size and long_k_size parameters are odd and respect minimum bounds (>= 11 and >= 21 respectively)
- Each iteration removes seq_set sequences explained by the extracted motif, so extracted motifs do not re-explain the same k-mers

## Limitations

- Requires Python 3.7; later versions might be incompatible because of inner biopython dependencies
- Requires sufficient genome coverage depth (default minimum 40)
- Termination depends on the confidence statistic falling below min_conf, so very weakly enriched true motifs may not be extracted

## Evidence

- [abstract] Greedy enrichment statistic vs genome background: "We present Snapper, a new highly sensitive approach, to extract methylation motif sequences based on a greedy motif selection algorithm."
- [intro] Position-specific motif enrichment is the core feature: "The core feature of Snapper in comparison with other tools is a new high-sensitive greedy algorithm that is used 
for position-specific motif enrichment."
- [intro] Balanced per-k-mer statistics against genome: "Snapper uses balanced approach to compute statistics for each k-mer which is likely to be modified."
- [methods] CLI usage with reference genome input: "snapper -sample_fast5dir ../HelicobacterMod/fast5/J99_multi/ -control_fast5dir ../HelicobacterMod/fast5/J99_wga_multi/ -reference ../HelicobacterMod/genome/J99.fasta"
- [readme] k-mer size constraints from README: "-k_size K_SIZE        k-mer size, must be odd, 
                        should not be less than 11 (default 15)"
