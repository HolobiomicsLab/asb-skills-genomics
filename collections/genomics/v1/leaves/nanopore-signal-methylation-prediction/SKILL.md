---
name: nanopore-signal-methylation-prediction
description: Use when you have nanopore DRS FAST5/eventaligned signals (e.g. HEK293
  WT, HeLa, IVT controls) and need single-molecule m6A/m5C calls, site-level probabilities,
  or stoichiometry in any sequence context without a KO/control sample.
when_to_use_negative:
- You need differential methylation between two conditions rather than single-condition
  calls — use CHEUI-diff instead of CHEUI-solo
- The data use RNA004 chemistry or require pseudouridine detection — SWARM (which
  improves on CHEUI for RNA002 and supports RNA004 plus pseudouridine) is the appropriate
  tool
- You want to detect modifications by comparing two samples (differential-signal tools
  like Nanocompore, Xpore, DRUMMER apply) but lack single-condition signal prediction
  needs — this skill is for single-sample signal-level prediction
edam_operation: http://edamontology.org/operation_0417
edam_topics:
- http://edamontology.org/topic_0080
- http://edamontology.org/topic_0659
tools:
- name: CHEUI
  role: Two-stage deep learning tool (Model 1 per-read prediction on IVT-trained signals;
    Model 2 CNN for site-level probability and stoichiometry) for m6A/m5C detection
    from DRS ionic currents
  repo: https://github.com/comprna/CHEUI
- name: nanopore direct RNA sequencing
  role: Sequencing modality supplying the ionic-current signals that CHEUI processes
    against expected signals
- name: R2Dtool
  role: Converts CHEUI methylation calls to bed-like format (cheui_to_bed.sh), annotates
    sites with isoform-specific metatranscript coordinates (R2_annotate), and lifts
    transcriptomic sites to genomic coordinates (R2_lift)
  repo: https://github.com/comprna/R2Dtool
- name: Nanopolish eventalign
  role: Aligns read signals to matched transcript references during signal preprocessing
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-024-47953-7
    title: Prediction of m6A and m5C at single-molecule resolution reveals a transcriptome-wide
      co-occurrence of RNA modifications
  source_content_hash: sha256:fc53c29d53215e21fe903f9b2202fe5b538bb1df8d0889e1ae7e4c5fb29645e6
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/comprna/CHEUI
  tools:
  - CHEUI
  - Nanopolish eventalign
  - R2Dtool
  - nanopore direct RNA sequencing
  edam_operation: http://edamontology.org/operation_0417
  edam_topics:
  - http://edamontology.org/topic_0080
  - http://edamontology.org/topic_0659
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# nanopore-signal-methylation-prediction

## Summary

Predicts m6A and m5C RNA modifications at single-read and single-site resolution from Oxford Nanopore direct RNA sequencing ionic-current signals using CHEUI's two-stage deep learning pipeline, including site-level significance calling and stoichiometry estimation.

## When to use

Use when you have nanopore DRS FAST5/eventaligned signals (e.g. HEK293 WT, HeLa, IVT controls) and need single-molecule m6A/m5C calls, site-level probabilities, or stoichiometry in any sequence context without a KO/control sample.

## When NOT to use

- You need differential methylation between two conditions rather than single-condition calls — use CHEUI-diff instead of CHEUI-solo
- The data use RNA004 chemistry or require pseudouridine detection — SWARM (which improves on CHEUI for RNA002 and supports RNA004 plus pseudouridine) is the appropriate tool
- You want to detect modifications by comparing two samples (differential-signal tools like Nanocompore, Xpore, DRUMMER apply) but lack single-condition signal prediction needs — this skill is for single-sample signal-level prediction

## Inputs

- Nanopore direct RNA sequencing FAST5 files (observed ionic-current signals)
- Reference transcriptome sequences (e.g. GRCh38/Gencode v38 for human, GRCm39/Ensembl v104 for mouse) for Nanopolish eventalign
- GTF2.2 annotation for R2Dtool annotate/liftover of transcriptomic sites
- In-vitro transcribed (IVT) RNA reads (e.g. Jenjaroenpun et al. IVT reads) as unmodified control for FPR estimation
- Expected (unmodified) signal references per transcript 5-mer

## Outputs

- Per-read m6A/m5C modification probabilities at single-nucleotide resolution (CHEUI-solo Model 1)
- Site-level methylation probabilities, stoichiometry estimates, and significant site call set (e.g. m6A_sites.tsv at probability >=0.9999)
- Genomic-coordinate BED of lifted/annotated modification sites via cheui_to_bed.sh and R2Dtool liftover/annotate
- Summary metrics such as site count, DRACH motif proportion, and IVT-derived m6A FPR (e.g. m6A_site_metrics.tsv)

## How to apply

Preprocess FAST5 files by basecalling with Guppy, aligning reads to the reference transcriptome with Minimap2, filtering best matches with samtools -F 232457, and eventaligning signals with Nanopolish; flip signals per 5-mer to 5'–3' orientation before concatenating overlapping 5-mers. Run CHEUI-solo Model 1 (trained on IVT read signals) on the 5 x 20 median signal vectors to get per-read modification probabilities, then CHEUI-solo Model 2 (CNN classifier, Keras/TensorFlow) on Model 1 per-read probability distributions for transcriptomic A sites with coverage >20 reads to obtain site-level methylation probabilities and stoichiometry. For site calls, apply the strict probability >=0.9999 cutoff, which was calibrated by shuffling read signals across transcriptomic sites so that the estimated m6A FDR is 0 (in HEK293 WT this yields ~10,036 significant sites on 3905 transcripts, 8776 genomic sites, 85.12% DRACH). To estimate the m6A false-positive rate, run Models 1 and 2 with the same cutoff on in-vitro transcribed RNA reads (reference FPR 0.0003). Convert calls to genomic/annotated coordinates with R2Dtool (cheui_to_bed.sh, R2_annotate, R2_lift) using the relevant GTF annotation (e.g. GRCh38/Gencode v38).

## Related tools

- **CHEUI** (Two-stage deep learning tool (Model 1 per-read prediction on IVT-trained signals; Model 2 CNN for site-level probability and stoichiometry) for m6A/m5C detection from DRS ionic currents) — https://github.com/comprna/CHEUI
- **nanopore direct RNA sequencing** (Sequencing modality supplying the ionic-current signals that CHEUI processes against expected signals)
- **R2Dtool** (Converts CHEUI methylation calls to bed-like format (cheui_to_bed.sh), annotates sites with isoform-specific metatranscript coordinates (R2_annotate), and lifts transcriptomic sites to genomic coordinates (R2_lift)) — https://github.com/comprna/R2Dtool
- **Nanopolish eventalign** (Aligns read signals to matched transcript references during signal preprocessing)

## Examples

```
samtools view -F 232457 aligned.bam | nanopolicy eventalign; python CHEUI-solo_model1.py -i signals.tsv --mod m6A && python CHEUI-solo_model2.py --kmer_models models/ --input sorted_predictions --mod m6A -o m6A_sites.tsv
```

## Evaluation signals

- Reproducing the HEK293 WT call set at probability >=0.9999 should yield ~10,036 significant m6A transcriptomic sites on 3905 transcripts (8776 genomic sites)
- The DRACH motif proportion among significant m6A sites should be ~85.12%
- Running Models 1 and 2 on the three IVT WT HeLa replicates should give an m6A false-positive rate matching the reported 0.0003
- Shuffling read signals across transcriptomic sites at the chosen cutoff should keep the estimated m6A FDR at 0
- Output BED must contain valid transcript IDs with coverage, strand, N_valid_cov, and fraction_modified columns compatible with R2Dtool liftover against the Gencode v38 GTF

## Limitations

- Very strict cutoffs are needed on cell transcriptome sites to maintain low expected FDR, which reduces CHEUI sensitivity; many sites found by orthogonal methods have high CHEUI probabilities below the thresholds
- Nanopore signal variability limits detection where differences between modified and non-modified reads are comparable to variation among non-modified reads; independent validation is needed for sites detected only from nanopore reads
- Signal interference at distances <5 nt is exacerbated in densely modified RNAs such as ribosomal RNA

## Evidence

- [abstract] CHEUI processes observed and expected nanopore direct RNA sequencing signals to achieve high single-molecule, transcript-site, and stoichiometry accuracies: "CHEUI processes observed and expected nanopore direct RNA sequencing signals to achieve high single-molecule, transcript-site, and stoichiometry accuracies"
- [methods] eventalign aligns read signals: "eventalign was then used to align the read signals to the matched transcript references"
- [methods] we shuffled all read signals across all transcriptomic sites, maintaining the same number of transcriptomic sites: "we shuffled all read signals across all transcriptomic sites, maintaining the same number of transcriptomic sites"
- [discussion] very strict cut-offs to maintain a low expected false discovery rate (FDR): "given the large number of sites tested, we used very strict cut-offs to maintain a low expected false discovery rate (FDR). While these strict cutoffs resulted in a reduction of CHEUI sensitivity"
- [readme] README on CHEUI scope: [evidence span withheld — anchor status: not_found]
