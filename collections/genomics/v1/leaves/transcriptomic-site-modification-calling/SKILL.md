---
name: transcriptomic-site-modification-calling
description: Use when you have CHEUI-solo Model 1 per-read probability distributions
  for transcriptomic sites (coverage >20 reads) and need significant site-level modification
  calls with stoichiometry in a single condition.
when_to_use_negative:
- When the goal is single-molecule (per-read) modification prediction rather than
  site-level aggregation — use CHEUI-solo Model 1 alone instead
- When a control/knockout sample is available for differential testing — use CHEUI-diff
  rather than solo site calling
- For RNA004 chemistry or pseudouridine detection, where SWARM (not CHEUI) is the
  recommended successor
edam_operation: http://edamontology.org/operation_0417
edam_topics:
- http://edamontology.org/topic_0080
- http://edamontology.org/topic_3524
tools:
- name: CHEUI
  role: 'Two-stage deep learning method: Model 1 predicts per-read modification probabilities;
    Model 2 (CNN classifier, Keras/TensorFlow) aggregates them into transcriptomic
    site-level probabilities and stoichiometry'
  repo: https://github.com/comprna/CHEUI-public
- name: nanopore direct RNA sequencing
  role: 'Data source: ionic current signals processed by CHEUI to detect m6A and m5C
    at single-molecule and site resolution'
- name: R2Dtool
  role: 'Post-processing: cheui_to_bed.sh converts CHEUI methylation calls to a bed-like
    format; R2_annotate annotates sites and R2_lift transposes transcriptomic sites
    to genomic coordinates'
  repo: https://github.com/comprna/R2Dtool
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-024-47953-7
    title: Prediction of m6A and m5C at single-molecule resolution reveals a transcriptome-wide
      co-occurrence of RNA modifications
  source_content_hash: sha256:bdd4a579d4d5ca6909dca615eb587ed30881bc84377bd0d965a39abc239e28fe
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/comprna/CHEUI-public
  tools:
  - CHEUI
  - R2Dtool
  - nanopore direct RNA sequencing
  edam_operation: http://edamontology.org/operation_0417
  edam_topics:
  - http://edamontology.org/topic_0080
  - http://edamontology.org/topic_3524
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# transcriptomic-site-modification-calling

## Summary

Call significant m6A/m5C sites at transcriptomic resolution from nanopore direct RNA sequencing (DRS) data using CHEUI-solo Model 2, applying a very strict site-level probability cutoff (>=0.9999 for m6A) calibrated to keep estimated FDR near zero. This skill turns per-read Model 1 predictions into site-level methylation probabilities and stoichiometry estimates.

## When to use

Use when you have CHEUI-solo Model 1 per-read probability distributions for transcriptomic sites (coverage >20 reads) and need significant site-level modification calls with stoichiometry in a single condition.

## When NOT to use

- When the goal is single-molecule (per-read) modification prediction rather than site-level aggregation — use CHEUI-solo Model 1 alone instead
- When a control/knockout sample is available for differential testing — use CHEUI-diff rather than solo site calling
- For RNA004 chemistry or pseudouridine detection, where SWARM (not CHEUI) is the recommended successor

## Inputs

- Nanopore direct RNA sequencing FAST5 files from a single condition (e.g. HEK293 WT DRS replicates)
- Reference transcriptome genome and annotation (GRCh38 and Gencode v38 for human data)
- CHEUI-solo Model 1 per-read probability distributions grouped per transcriptomic site
- 5 x 20 median signal vectors and per-5mer signal distance vectors from CHEUI preprocessing

## Outputs

- Transcriptomic site-level modification calls with methylation probability and stoichiometry (e.g. m6A_sites.tsv)
- Significant site call set filtered at probability >=0.9999 (reference: 10,036 sites on 3905 transcripts, 8776 genomic sites in HEK293 WT)
- Summary metrics including DRACH motif proportion (~85.12%) and IVT-derived FPR (reference 0.0003), e.g. m6A_site_metrics.tsv

## How to apply

First preprocess FAST5 DRS reads: basecall with Guppy, align to the reference transcriptome (e.g. GRCh38/Gencode v38) with Minimap2, filter best matches with samtools -F 232457, and eventalign signals with Nanopolish, flipping per-5mer signals to 5'-3' orientation. Run CHEUI-solo Model 1 (trained on IVT read signals) on 5x20 median signal vectors and per-5mer distance vectors to get per-read modification probabilities, then sort/group predictions by site. Run CHEUI-solo Model 2 (CNN classifier, Keras/TensorFlow) on the Model 1 per-read probability distributions for all transcriptomic A sites with coverage >20 reads to obtain site-level methylation probabilities and stoichiometry. Apply the strict significance cutoff (probability >=0.9999 for m6A, established by shuffling read signals across transcriptomic sites to keep estimated FDR at 0), since with the large number of sites tested, very strict cutoffs are required to maintain a low expected FDR at the cost of some sensitivity. Validate by computing DRACH motif enrichment among significant sites (~85.12% expected) and, optionally, estimating the m6A false positive rate on IVT control replicates (reference FPR 0.0003).

## Related tools

- **CHEUI** (Two-stage deep learning method: Model 1 predicts per-read modification probabilities; Model 2 (CNN classifier, Keras/TensorFlow) aggregates them into transcriptomic site-level probabilities and stoichiometry) — https://github.com/comprna/CHEUI-public
- **nanopore direct RNA sequencing** (Data source: ionic current signals processed by CHEUI to detect m6A and m5C at single-molecule and site resolution)
- **R2Dtool** (Post-processing: cheui_to_bed.sh converts CHEUI methylation calls to a bed-like format; R2_annotate annotates sites and R2_lift transposes transcriptomic sites to genomic coordinates) — https://github.com/comprna/R2Dtool

## Examples

```
samtools view -F 232457 aligned.bam | nano eventalign --rnaseq --signal-align; python CHEUI-solo_model2.py -i model1_predictions_sorted.tsv -o m6A_sites.tsv --probability 0.9999
```

## Evaluation signals

- Reproduced HEK293 WT call set at probability >=0.9999 should contain ~10,036 significant m6A transcriptomic sites on ~3905 transcripts (~8776 genomic sites)
- DRACH motif proportion among significant m6A sites should approximate the reported 85.12%
- m6A false positive rate from applying the same cutoff to IVT transcribed HeLa DRS replicates should match the reported 0.0003
- All retained sites must satisfy coverage >20 reads at the transcriptomic A site
- FDR calibration check: shuffling read signals across transcriptomic sites at the same cutoff should yield estimated FDR ~0 for m6A

## Limitations

- Very strict cutoffs (needed to keep expected FDR low across the large number of tested sites) reduce CHEUI sensitivity; many orthogonally-supported sites have high probabilities just below threshold
- Nanopore signal variability limits single-read discrimination, since modified vs non-modified read differences are often comparable to variation among non-modified reads
- Sites detected only from nanopore reads require independent validation experiments

## Evidence

- [discussion] given the large number of sites tested, we used very strict cut-offs to maintain a low expected false discovery rate (FDR). While these strict cutoffs resulted in a reduction of CHEUI sensitivity: "given the large number of sites tested, we used very strict cut-offs to maintain a low expected false discovery rate (FDR). While these strict cutoffs resulted in a reduction of CHEUI sensitivity"
- [methods] we shuffled all read signals across all transcriptomic sites, maintaining the same number of transcriptomic sites: "we shuffled all read signals across all transcriptomic sites, maintaining the same number of transcriptomic sites"
- [methods] the R2Dtool script cheui_to_bed.sh was run with default parameters to convert the CHEUI methylation calls to a bed-like format: "the R2Dtool script cheui_to_bed.sh was run with default parameters to convert the CHEUI methylation calls to a bed-like format"
- [readme] CHEUI can be used to detect m6A and m5C in individual reads at single-nucleotide resolution from any sample (e.g. single condition): [evidence span withheld — anchor status: external_doc]
- [methods] Reads were then aligned to the corresponding reference transcriptome using Minimap2: "Reads were then aligned to the corresponding reference transcriptome using Minimap2"
