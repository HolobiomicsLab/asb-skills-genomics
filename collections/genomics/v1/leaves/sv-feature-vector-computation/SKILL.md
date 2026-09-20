---
name: sv-feature-vector-computation
description: Use when you have putative SV calls (VCF/BED from dysgu or gold sets
  such as GIAB HG001/HG002 or pbsv PacBio calls) and need feature vectors plus quality
  probabilities for ML-based SV classification.
when_to_use_negative:
- The task is SV calling itself (signal detection from BAMs) rather than scoring already-detected
  events — use dysgu's run/fetch/call pipeline instead
- You need to benchmark SV calls against truth sets rather than compute classification
  features — use svbench (svbench REFERENCE_VCF QUERY_VCFS...) instead
- Only read-support filtering is needed without sequence metrics — the --contigs False
  model strips all sequence-related metrics, making full feature computation unnecessary
edam_topics:
- http://edamontology.org/topic_0199
- http://edamontology.org/topic_3474
tools:
- name: dysgu
  role: Source of the feature-vector computation (DysguSV python API, load_dysgu_vcf,
    load_bed, merge_intervals, merge_dysgu_df, parameter presets) and the LightGBM
    quality-score model
  repo: https://github.com/kcleal/dysgu
- name: nanovar
  role: Source of the NanoVar-derived insertion call set used as a labelled training
    resource; also cited as prior art using a neural network classifier over 14 input
    SV features
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1093/nar/gkac039
    title: 'Dysgu: efficient structural variant calling using short or long reads'
  source_content_hash: sha256:1336f4918d358052af06507b8761a239b95accbf8168f58c46b9871218aba4b2
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/kcleal/dysgu
  tools:
  - dysgu
  - nanovar
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0199
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 5
---

# sv-feature-vector-computation

## Summary

Computes per-event structural variant feature vectors (read support, coverage-based metrics such as fold change in coverage (FCC), and sequence/contig metrics) from putative SV calls so they can be scored by dysgu's LightGBM gradient-boosting quality classifier.

## When to use

Use when you have putative SV calls (VCF/BED from dysgu or gold sets such as GIAB HG001/HG002 or pbsv PacBio calls) and need feature vectors plus quality probabilities for ML-based SV classification.

## When NOT to use

- The task is SV calling itself (signal detection from BAMs) rather than scoring already-detected events — use dysgu's run/fetch/call pipeline instead
- You need to benchmark SV calls against truth sets rather than compute classification features — use svbench (svbench REFERENCE_VCF QUERY_VCFS...) instead
- Only read-support filtering is needed without sequence metrics — the --contigs False model strips all sequence-related metrics, making full feature computation unnecessary

## Inputs

- Dysgu VCF of putative SV calls (loadable via load_dysgu_vcf)
- BED-format gold-set call files, e.g. HG001 svclassify/Personalis deletions (≥50 bp) and NanoVar insertion calls (loadable via load_bed)
- pbsv HG005 PacBio deletion/insertion VCF from the GIAB FTP site
- Sorted, indexed BAM/CRAM of PE (bwa mem) or long-read (minimap2/ngmlr) alignments plus indexed reference FASTA
- dysgu_default_args / dysgu_preset_args parameter presets

## Outputs

- Per-SV feature vector table with probability/quality-score column
- dysgu_model.pickle (serialized LightGBM 'dart' model)
- feature_importance.csv with sorted feature ranking (SV length first, FCC second)
- VCF with per-event model probability for downstream filtering

## How to apply

Load putative calls with dysgu's python API (load_dysgu_vcf for VCFs, load_bed for BED-format truth sets like the HG001 svclassify deletion set of ≥50 bp deletions) and reconcile overlapping gold regions with merge_intervals / merge_dysgu_df to assign positive labels. Compute feature vectors for each event using the DysguSV API with parameters taken from dysgu_default_args / dysgu_preset_args; features derive from alignment cigar information, discordant/split-read mappings, consensus contigs, and per-site read-depth, with FCC (fold change in coverage) expected to rank just below SV length in importance. The vectors feed a LightGBM 'dart' gradient-boosting classifier tuned by recursive feature elimination plus 5-fold stratified cross-validated grid search. Call probabilities can then be used to filter events (≥0.5 for PacBio, ≥0.35 for ONT combinations) instead of manual filtering.

## Related tools

- **dysgu** (Source of the feature-vector computation (DysguSV python API, load_dysgu_vcf, load_bed, merge_intervals, merge_dysgu_df, parameter presets) and the LightGBM quality-score model) — https://github.com/kcleal/dysgu
- **nanovar** (Source of the NanoVar-derived insertion call set used as a labelled training resource; also cited as prior art using a neural network classifier over 14 input SV features)

## Examples

```
from dysgu.api import load_dysgu_vcf, load_bed, merge_intervals, merge_dysgu_df; df = load_dysgu_vcf('svs.vcf'); gold = load_bed('giab_hg001_dels.bed')
```

## Evaluation signals

- dysgu_model.pickle loads successfully through the DysguSV python API
- Exported feature_importance.csv ranks FCC immediately after SV length in importance
- Feature/hyperparameter selection completes via recursive feature elimination combined with 5-fold stratified cross-validated grid search
- A script_runs check on the training script passes
- Model probabilities stratify calls at the documented thresholds: maximum model probability ≥0.5 (PacBio) or ≥0.35 (ONT combinations)

## Limitations

- Manually labelled training data induces observer bias ('the correctness is a matter of opinion of the human observer')
- The current feature list is non-exhaustive and heuristics (span position distance, minimizer support) are unoptimized
- Training data availability/versioning can be underspecified; classifier training data was hosted at https://zenodo.org/record/4761527
- In PE data, dysgu's insertion size estimates are expected to be less accurate than de novo assembly-based callers such as manta and gatk

## Evidence

- [intro] A machine learning classifier is then employed to generate a useful quality score which can be used to prioritize variants.: "A machine learning classifier is then employed to generate a useful quality score which can be used to prioritize variants."
- [abstract] Dysgu detects signals from alignment gaps, discordant and supplementary mappings, and generates consensus contigs, before classifying events using machine learning: "Dysgu detects signals from alignment gaps, discordant and supplementary mappings, and generates consensus contigs, before classifying events using machine learning"
- [discussion] Manually labelling induces an obvious bias in the training set, where the correctness is a matter of opinion of the human observer.: "Manually labelling induces an obvious bias in the training set, where the correctness is a matter of opinion of the human observer."
- [results] filtering calls with a maximum model probability ≥0.5 for PacBio, or ≥0.35 for ONT combinations: "filtering calls with a maximum model probability ≥0.5 for PacBio, or ≥0.35 for ONT combinations"
- [discussion] Putative events are then labelled with a useful probability value using a gradient boosting classifier (30).: "Putative events are then labelled with a useful probability value using a gradient boosting classifier (30)."
- [intro] The nanovar utilizes a neural network classifier trained on simulated datasets, where 14 input features of each putative SV are used to classify events: "The LR caller nanovar utilizes a neural network classifier trained on simulated datasets, where 14 input features of each putative SV are used to classify events (17)"
