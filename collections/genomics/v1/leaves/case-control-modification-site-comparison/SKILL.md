---
name: case-control-modification-site-comparison
description: Use when two dRNA-Seq datasets (case vs control, e.g. a modification-writer
  mutant vs wild-type fast5 raw reads) must be compared at single m6A sites to find
  sites with significantly altered modification stoichiometry.
when_to_use_negative:
- Site coverage is below the min_coverage 50 threshold — the site will not appear
  in mAFiA.sites.bed and modRatio comparison is unreliable
- You need Ψ or motifs beyond the m6A DRACH subset covered by mAFiA classifiers —
  use the superseding psi-co-mAFiA model set instead
- You have no per-read modification calls (e.g. only bulk chemical-assay stoichiometry)
  — this skill requires read-level MM/ML tags from mAFiA modBAMs
edam_topics:
- http://edamontology.org/topic_0092
- http://edamontology.org/topic_3308
tools:
- name: mAFiA
  role: Predicts read-level m6A probabilities and aggregates them into per-site modRatio
    stoichiometry (mAFiA.sites.bed) for the case/control comparison
  repo: https://github.com/dieterich-lab/mAFiA
- name: RODAN
  role: Backbone model for basecalling raw fast5 dRNA-Seq reads and extracting hidden
    features for mAFiA
  repo: https://github.com/biodlab/RODAN
- name: minimap2.22
  role: Aligns basecalled reads to the reference (TAIR10/GRCh38) with splice-aware,
    strand-correct parameters before q50 filtering
- name: IGV
  role: Visualization of case/control modBAM reads with MM/ML tags to snapshot a significantly
    down-regulated m6A site
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41467-024-47661-2
    title: Detecting m6A at single-molecular resolution via direct RNA sequencing
      and realistic training data
  source_content_hash: sha256:df8891cae8f932591cfcdb9bca1e857a5d85b2ad1ea7857d77e3be26d8e2b926
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/dieterich-lab/mAFiA
  tools:
  - IGV
  - RODAN
  - mAFiA
  - minimap2.22
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0092
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 12
---

# case-control-modification-site-comparison

## Summary

Compares per-site m6A stoichiometry (modRatio) from mAFiA between a case and control dRNA-Seq sample — e.g. Arabidopsis thaliana vir1 mutant vs col0 wild-type — to identify transcriptome-wide down- or up-regulation of m6A at otherwise highly modified sites.

## When to use

Use when two dRNA-Seq datasets (case vs control, e.g. a modification-writer mutant vs wild-type fast5 raw reads) must be compared at single m6A sites to find sites with significantly altered modification stoichiometry.

## When NOT to use

- Site coverage is below the min_coverage 50 threshold — the site will not appear in mAFiA.sites.bed and modRatio comparison is unreliable
- You need Ψ or motifs beyond the m6A DRACH subset covered by mAFiA classifiers — use the superseding psi-co-mAFiA model set instead
- You have no per-read modification calls (e.g. only bulk chemical-assay stoichiometry) — this skill requires read-level MM/ML tags from mAFiA modBAMs

## Inputs

- Raw fast5 dRNA-Seq reads for case sample (e.g. ENA PRJEB32782 vir1 replicate 1)
- Raw fast5 dRNA-Seq reads for control sample (e.g. ENA PRJEB32782 col0 replicate 1)
- Reference genome FASTA (TAIR10 for Arabidopsis, GRCh38 for human)
- mAFiA models: backbone.torch (RODAN-based) and pickled logistic regression classifiers
- minimap2.22 and samtools installed; GPU machine for basecalling

## Outputs

- Per-sample sorted, indexed, q50-filtered BAM files
- mAFiA.sites.bed per sample with coverage, modRatio, and confidence columns
- List of candidate sites with significant modRatio change between case and control (e.g. vir1 down-regulated vs col0)
- Optional IGV snapshot of a down-regulated site with MM/ML modification tags on reads

## How to apply

Process both strains through the identical pipeline: basecall raw fast5 files with the RODAN-based backbone model (basecall.py, batchsize 4096), align with minimap2.22 (--secondary=no -ax splice -uf -k14 --cs), filter with samtools view -q50, then sort and index the BAMs. Run test_mAFiA per BAM with min_coverage 50, max_num_reads 1000, mod_prob_thresh 0.5, the backbone.torch model and pickled logistic regression classifiers to obtain per-site modRatio values. Compare site-level modRatio between case and control only at sites meeting the coverage threshold (mAFiA.sites.bed should contain only sites with coverage >= 50), flagging sites where the mutant modRatio is markedly lower than wild-type as down-regulated m6A. For high-precision calls prefer sites with confidence >= 80%, since modRatio is the fraction of reads with modification probability >= 0.5.

## Related tools

- **mAFiA** (Predicts read-level m6A probabilities and aggregates them into per-site modRatio stoichiometry (mAFiA.sites.bed) for the case/control comparison) — https://github.com/dieterich-lab/mAFiA
- **RODAN** (Backbone model for basecalling raw fast5 dRNA-Seq reads and extracting hidden features for mAFiA) — https://github.com/biodlab/RODAN
- **minimap2.22** (Aligns basecalled reads to the reference (TAIR10/GRCh38) with splice-aware, strand-correct parameters before q50 filtering)
- **IGV** (Visualization of case/control modBAM reads with MM/ML tags to snapshot a significantly down-regulated m6A site)

## Examples

```
pileup --bam_file ${out_dir}/mAFiA.reads.bam --sites ${sites} --min_coverage 50 --out_dir ${out_dir} --num_jobs 36
```

## Evaluation signals

- mAFiA.sites.bed contains only sites with coverage at or above the min_coverage threshold of 50
- At candidate down-regulated sites, the vir1 modRatio is lower than the col0 modRatio, reproducing the reported mutant down-regulation
- modRatio values fall in [0,1] and match the fraction of reads with modification probability >= 0.5 at the site
- BAM files pass samtools view -q50 filtering, are sorted and indexed without errors
- For high-precision sites, confidence is at or above the recommended 80%

## Limitations

- mAFiA (as of Jan 2025) is superseded by psi-co-mAFiA, which extends coverage to all 18 m6A DRACH and 16 high-frequency Ψ motifs
- IGV (as of version 2.17.4) does not support RNA modification tags and shows them as 'Other'; tags must be relabelled (e.g. N+21891 to N+a) for meaningful color visualization
- modRatio depends on the mod_prob_thresh 0.5 cutoff; low-coverage or low-confidence sites give unstable stoichiometry estimates

## Evidence

- [results] Reads with mapq below 50 are filtered out.: "Reads with mapq below 50 are filtered out."
- [results] A minimum dRNA-Seq coverage of 50 is required for a site to be compared against GLORI measurements: "A minimum dRNA-Seq coverage of 50 is required for a site to be compared against GLORI measurements"
- [readme] modRatio definition: [evidence span withheld — anchor status: not_found]
- [readme] coverage and confidence recommendation: [evidence span withheld — anchor status: external_doc]
- [readme] MM/ML tags in modBAM: [evidence span withheld — anchor status: external_doc]
