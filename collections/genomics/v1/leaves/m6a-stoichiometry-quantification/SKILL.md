---
name: m6a-stoichiometry-quantification
description: Use when you have raw nanopore fast5 dRNA-Seq reads and need per-site
  m6A stoichiometry estimates (modRatio), either to correlate against a chemical-assay
  reference such as GLORI or to call differential modification between strains.
when_to_use_negative:
- You need predictions at m6A or Ψ motif classes not covered by the original mAFiA
  classifiers — use the successor psi-co-mAFiA, which covers all 18 m6A DRACH motifs
  plus 16 high-frequency Ψ motifs.
- Sites with dRNA-Seq coverage below the minimum threshold of 50 reads — modRatio
  estimates there are out of scope for comparison against GLORI and below the recommended
  precision settings.
- The input is short-read sequencing or already-processed modification calls from
  a different chemistry (e.g. existing CHEUI/m6Anet outputs) — this skill requires
  raw fast5 dRNA-Seq input.
edam_operation: http://edamontology.org/operation_3799
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0625
tools:
- name: RODAN
  role: Fully-convolutional basecaller whose backbone.torch model both basecalls the
    fast5 files and supplies hidden features (768-dim vector from convlayers.conv21)
    for m6A classification
  repo: https://github.com/biodlab/RODAN
- name: mAFiA
  role: Applies pickled logistic regression classifiers to RODAN features to produce
    read-level P(m6A) and site-level modRatio stoichiometry (mAFiA.sites.bed)
  repo: https://github.com/dieterich-lab/mAFiA
- name: minimap2.22
  role: Splice-aware alignment of basecalled reads to the reference genome (GRCh38
    or TAIR10) before modification calling
provenance:
  source_task_ids:
  - task_002
  - task_008
  source_papers:
  - doi: 10.1038/s41467-024-47661-2
    title: Detecting m6A at single-molecular resolution via direct RNA sequencing
      and realistic training data
  source_content_hash: sha256:cb0ec14b18b5a50349fbf6089ec81bf237afd22f779c56e1ad8fda789fef71f3
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/biodlab/RODAN
  tools:
  - RODAN
  - mAFiA
  - minimap2.22
  edam_operation: http://edamontology.org/operation_3799
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0625
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7272727272727273
n_claims: 11
---

# m6a-stoichiometry-quantification

## Summary

Quantify site-level m6A stoichiometry from Oxford Nanopore direct RNA-Seq (dRNA-Seq) data using mAFiA: RODAN-based feature extraction plus logistic regression classifiers yield per-read modification probabilities that are aggregated into per-site modRatio values, which can be benchmarked against GLORI or compared across conditions (e.g. col0 vs vir1).

## When to use

Use when you have raw nanopore fast5 dRNA-Seq reads and need per-site m6A stoichiometry estimates (modRatio), either to correlate against a chemical-assay reference such as GLORI or to call differential modification between strains.

## When NOT to use

- You need predictions at m6A or Ψ motif classes not covered by the original mAFiA classifiers — use the successor psi-co-mAFiA, which covers all 18 m6A DRACH motifs plus 16 high-frequency Ψ motifs.
- Sites with dRNA-Seq coverage below the minimum threshold of 50 reads — modRatio estimates there are out of scope for comparison against GLORI and below the recommended precision settings.
- The input is short-read sequencing or already-processed modification calls from a different chemistry (e.g. existing CHEUI/m6Anet outputs) — this skill requires raw fast5 dRNA-Seq input.

## Inputs

- Raw nanopore dRNA-Seq fast5 files (e.g. HEK293 WT chrX subset, Arabidopsis col0/vir1 replicate 1)
- Reference genome FASTA (GRCh38_96.X for human chrX; TAIR10 for Arabidopsis)
- BED file of candidate m6A motif sites (e.g. GLORI_chrX.bed query modification sites)
- RODAN-based backbone model (backbone.torch) and its backbone.config
- Pickled logistic regression classifier models for the target DRACH motifs

## Outputs

- Basecalled FASTA (rodan.fasta) from RODAN backbone
- Filtered/sorted/indexed BAM (e.g. minimap.q50.bam) of aligned reads at mapq >=50
- modBAM (mAFiA.reads.bam) with MM/ML tags encoding per-read m6A modification probabilities
- mAFiA.sites.bed with columns coverage, modRatio, and confidence per site
- Pearson/Spearman correlation between predicted modRatio and GLORI stoichiometry over matched sites

## How to apply

Basecall the fast5 files with the RODAN-based backbone model (basecall.py, batchsize 4096, GPU), which also yields the 768-dimensional feature vector extracted from the last convolution layer ('convlayers.conv21') used as classifier input. Align the basecalled reads with minimap2.22 (--secondary=no -ax splice -uf -k14 --cs), filter with samtools view -q50, sort and index the BAM. Run read-level prediction (process_reads/test_mAFiA) against the BAM, fast5 directory, reference, the RODAN backbone.torch model, and the pickled logistic regression classifiers with mod_prob_thresh 0.5 and max_num_reads 1000, producing a modBAM (mAFiA.reads.bam) with MM/ML tags. Aggregate with the pileup step using min_coverage 50 to produce mAFiA.sites.bed, whose modRatio column is the fraction of reads at the site with modification probability >=0.5. For benchmarking, match sites to GLORI_chrX.bed by genomic coordinate (chromosome, position, strand) and compute Pearson/Spearman correlation over the matched set; for cross-condition comparison (e.g. Arabidopsis col0 vs vir1 against TAIR10), compare modRatio values per site and optionally visualize reads in IGV.

## Related tools

- **RODAN** (Fully-convolutional basecaller whose backbone.torch model both basecalls the fast5 files and supplies hidden features (768-dim vector from convlayers.conv21) for m6A classification) — https://github.com/biodlab/RODAN
- **mAFiA** (Applies pickled logistic regression classifiers to RODAN features to produce read-level P(m6A) and site-level modRatio stoichiometry (mAFiA.sites.bed)) — https://github.com/dieterich-lab/mAFiA
- **minimap2.22** (Splice-aware alignment of basecalled reads to the reference genome (GRCh38 or TAIR10) before modification calling)

## Examples

```
python3 ${mafia}/RODAN/basecall.py --fast5dir ${fast5dir} --model ${backbone} --batchsize 4096 --outdir ${output}
```

## Evaluation signals

- mAFiA.sites.bed contains only sites with coverage at or above the min_coverage threshold of 50
- On HEK293 WT, coordinate-matched comparison against GLORI_chrX.bed yields 5925 sites with Pearson correlation S_mAFiA/S_GLRORI = 0.86 (vs m6Anet 0.80 and CHEUI 0.64 on the same set)
- Read-level outputs are thresholded at P_n(m6A) = 0.5, so modRatio at a site equals the fraction of reads with modification probability >=0.5 (check consistency against mAFiA.reads.bam MM/ML tags)
- In differential comparisons, the mutant (e.g. vir1) modRatio at candidate sites is lower than the wild-type (col0) modRatio, reproducing reported transcriptome-wide m6A down-regulation
- Confidence column should be high (>=0.8 recommended for high-precision measurements) at reported sites

## Limitations

- The original mAFiA classifiers cover only a subset of DRACH motifs; mAFiA is superseded by psi-co-mAFiA, which expands coverage to all 18 m6A DRACH motifs and 16 Ψ motifs.
- IGV (as of version 2.17.4) does not support RNA modifications and renders them as 'Other'; tags must be relabelled (e.g. m6A ChEBI 21891 recast as 6mA) for informative snapshots.
- Benchmarking against GLORI requires a minimum dRNA-Seq coverage of 50 per site; discontinuous alignments are handled by keeping the read section with the longest continuous mapping, which can drop reads with complex splicing.

## Evidence

- [results] Feature extraction from RODAN convolution layer: [evidence span withheld — anchor status: not_found]
- [results] Read-level probability threshold: [evidence span withheld — anchor status: not_found]
- [results] Minimum coverage for GLORI comparison: "A minimum dRNA-Seq coverage of 50 is required for a site to be compared against GLORI measurements"
- [methods] Classifier definition: "Logistic regression (see methods) is then applied"
- [readme] modRatio column definition in mAFiA.sites.bed: [evidence span withheld — anchor status: not_found]
- [readme] Recommended precision settings for stoichiometry: [evidence span withheld — anchor status: external_doc]
- [readme] IGV RNA modification limitation: [evidence span withheld — anchor status: external_doc]
