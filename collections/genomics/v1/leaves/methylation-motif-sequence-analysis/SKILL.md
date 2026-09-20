---
name: methylation-motif-sequence-analysis
description: Use when you have resquiggled multi-fast5 ONT reads for a native sample
  and a matched WGA control plus a reference genome FASTA, and you need to discover
  methylated sequence motifs de novo rather than test known motifs.
when_to_use_negative:
- You only need per-read/per-site methylation calling at known motifs (e.g. dam/dcm)
  rather than de novo motif discovery — Tombo's alternative-model detection may suffice
- Input fast5 files are not resquiggled or are in single-read format — Tombo resquiggling
  and ont_fast5_api multi-fast5 conversion are prerequisites
- No matched WGA control sample exists — the KS-test comparison of native vs WGA signal
  distributions depends on the control
edam_operation: http://edamontology.org/operation_0296
edam_topics:
- http://edamontology.org/topic_0654
- http://edamontology.org/topic_3168
- http://edamontology.org/topic_0749
tools:
- name: Snapper
  role: 'Primary tool: greedy motif selection and position-specific motif enrichment
    from ONT data, installed via pip as snapper-ont'
  repo: https://github.com/DNKonanov/Snapper
- name: Tombo
  role: Resquiggles raw nanopore fast5 signal to the reference genome to index signal
    events by position and strand
  repo: https://github.com/nanoporetech/tombo
- name: ont_fast5_api
  role: Converts resquiggled fast5 files into the multi-fast5 format Snapper consumes
  repo: https://github.com/nanoporetech/ont_fast5_api
- name: scipy
  role: 'Provides the statistical tests used inside Snapper: scipy.stats.ks_2samp
    for the native-vs-WGA KS comparison and scipy.stats chi-squared for motif enrichment'
provenance:
  source_task_ids:
  - task_007
  - task_006
  source_papers:
  - doi: 10.1093/bioinformatics/btad702
    title: 'Snapper: high-sensitive detection of methylation motifs based on Oxford
      Nanopore reads'
  source_content_hash: sha256:99c6b40977c0f818bb7cfe24aeb36bf547e6e7df737469755f61c2f996d4de6b
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
  edam_operation: http://edamontology.org/operation_0296
  edam_topics:
  - http://edamontology.org/topic_0654
  - http://edamontology.org/topic_3168
  - http://edamontology.org/topic_0749
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6363636363636364
n_claims: 11
---

# methylation-motif-sequence-analysis

## Summary

De novo detection of methylation motifs from Oxford Nanopore sequencing data using Snapper's high-sensitive greedy position-specific motif-enrichment algorithm, which compares native vs WGA control k-mer signal distributions and iteratively extracts over-represented motifs via chi-squared statistics.

## When to use

Use when you have resquiggled multi-fast5 ONT reads for a native sample and a matched WGA control plus a reference genome FASTA, and you need to discover methylated sequence motifs de novo rather than test known motifs.

## When NOT to use

- You only need per-read/per-site methylation calling at known motifs (e.g. dam/dcm) rather than de novo motif discovery — Tombo's alternative-model detection may suffice
- Input fast5 files are not resquiggled or are in single-read format — Tombo resquiggling and ont_fast5_api multi-fast5 conversion are prerequisites
- No matched WGA control sample exists — the KS-test comparison of native vs WGA signal distributions depends on the control

## Inputs

- Native sample multi-fast5 directory (resquiggled with Tombo)
- WGA control multi-fast5 directory (resquiggled with Tombo)
- Reference genome in FASTA format
- Optional parameter values: ks_t (default 3), coverage (default 40), threads (default 8), k_size (default 15), long_k_size (default 29), max_motifs (default 20), min_conf (default 100)

## Outputs

- passed_motifs_[strand]_[contig_name].fasta: candidate k-mer seq_set passing the KS-test threshold
- final_motifs.fasta: sorted list of enriched methylation motifs with confidence levels

## How to apply

First resquiggle the fast5 files with Tombo and convert them to multi-fast5 format using ont_fast5_api. Run snapper with -sample_fast5dir, -control_fast5dir, and -reference pointing to the native reads, WGA control reads, and reference genome FASTA; for every odd-sized k-mer of the reference (k_size default 15, must be odd and >= 11; long_k_size default 29, >= 21), Snapper aggregates normalized signal events per genome position and strand into a hash table, draws balanced equal-size (N=200) signal samples from native and WGA to avoid coverage bias, and runs a two-sample Kolmogorov-Smirnov test (scipy.stats.ks_2samp), retaining k-mers whose -log KS p-value exceeds ks_t (default 3) with confidence above min_conf (default 100) into passed_motifs_[strand]_[contig_name].fasta. The greedy enrichment stage then iteratively selects the candidate motif most over-represented in the seq_set versus the reference genome background using chi-squared statistics, performs position-specific motif adjustment, removes sequences containing the extracted motif from the seq_set, and repeats until the confidence falls below min_conf or max_motifs (default 20) motifs have been extracted, writing a sorted motif list with confidence levels to final_motifs.fasta. Rationale: greedy iterative extraction with seq_set filtering lets each round target signals not yet explained by previously extracted motifs, giving deterministic, high-sensitivity results without manual control, unlike MEME coupled with Tombo or Nanodisco.

## Related tools

- **Snapper** (Primary tool: greedy motif selection and position-specific motif enrichment from ONT data, installed via pip as snapper-ont) — https://github.com/DNKonanov/Snapper
- **Tombo** (Resquiggles raw nanopore fast5 signal to the reference genome to index signal events by position and strand) — https://github.com/nanoporetech/tombo
- **ont_fast5_api** (Converts resquiggled fast5 files into the multi-fast5 format Snapper consumes) — https://github.com/nanoporetech/ont_fast5_api
- **scipy** (Provides the statistical tests used inside Snapper: scipy.stats.ks_2samp for the native-vs-WGA KS comparison and scipy.stats chi-squared for motif enrichment)

## Examples

```
snapper -sample_fast5dir ../HelicobacterMod/fast5/J99_multi/ -control_fast5dir ../HelicobacterMod/fast5/J99_wga_multi/ -reference ../HelicobacterMod/genome/J99.fasta
```

## Evaluation signals

- final_motifs.fasta exists and contains a sorted motif list where every emitted motif has confidence (chi-squared statistic) at or above min_conf (default 100)
- Number of motifs in final_motifs.fasta does not exceed max_motifs (default 20)
- All k-mer sizes used are odd and respect the minima: k_size >= 11, long_k_size >= 21
- Candidate seq_set entries in passed_motifs files each have -log KS-test p-value above ks_t (default 3) and confidence above min_conf (default 100)
- For H.pylori J99, recovered motifs should match motifs characterized previously by PacBio technology; for novel strains, expect motifs not previously described

## Limitations

- Requires Python 3.7; later versions might be incompatible because of inner biopython dependencies
- Requires a matched WGA control sample and minimum genome coverage depth (default 40)
- Tombo is deprecated/unsupported by Oxford Nanopore (superseded by Remora), so the upstream resquiggling dependency may be hard to maintain

## Evidence

- [abstract] greedy motif selection with chi-squared and termination: "We present Snapper, a new highly sensitive approach, to extract methylation motif sequences based on a greedy motif selection algorithm."
- [intro] position-specific motif enrichment: "The core feature of Snapper in comparison with other tools is a new high-sensitive greedy algorithm that is used 
for position-specific motif enrichment."
- [results] KS-test signal comparison in the signal-comparison stage: [evidence span withheld — anchor status: not_found]
- [results] greedy algorithm details: chi-squared, filtering, termination: [evidence span withheld — anchor status: not_found]
- [methods] snapper CLI invocation: "snapper -sample_fast5dir ../HelicobacterMod/fast5/J99_multi/ -control_fast5dir ../HelicobacterMod/fast5/J99_wga_multi/ -reference ../HelicobacterMod/genome/J99.fasta"
- [readme] pip install snapper-ont: "Snapper is implemented using Python and is freely available as a pip package named "snapper-ont.""
