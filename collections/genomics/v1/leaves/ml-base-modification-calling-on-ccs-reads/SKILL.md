---
name: ml-base-modification-calling-on-ccs-reads
description: Use when PacBio CCS/HiFi BAMs with kinetic tags (fi, ri, fp, rp) must
  be analyzed to detect dZ-DNA or call Z vs A bases at single-nucleotide resolution,
  e.g. in hybrid or transformed-organism DNA samples.
when_to_use_negative:
- 'Goal is to reliably classify hybrid/diluted reads as dZ-DNA at the molecule level:
  the SVM read classifier is optimized for fully substituted dZ-genomes and misses
  hybrid reads with diluted kinetic signatures'
- Input reads lack PacBio kinetic tags or come from a non-PacBio platform (e.g. ONT),
  since the A/Z discrimination relies on PW/IPD pulse signals
- Reads are shorter than 500 bp or contain fewer than 100 A+Z bases, which were excluded
  in the paper due to limited A/Z bases
edam_operation: http://edamontology.org/operation_3903
edam_topics:
- http://edamontology.org/topic_0654
- http://edamontology.org/topic_3168
- http://edamontology.org/topic_3511
tools:
- name: Z-Calling
  role: Toolkit providing filter_bam.py, z-calling-base (MLP base calling), z-bam2txt,
    zfreq, z-calling-read, and z-seq executables
  repo: https://github.com/xiaochuanle/Z-Calling
- name: PacBio Circular Consensus Sequencing (CCS)
  role: Source of HiFi reads with pulse width and inter-pulse duration kinetic signals
    (--keep-kinetics)
- name: pbmm2
  role: Aligns filtered CCS reads to the reference (--preset CCS --sort) before base
    calling and coordinate-level frequency aggregation
- name: samtools
  role: Converts filtered BAM to FASTA for the taxonomic source-detection sub-workflow
- name: minimap2
  role: map-hifi mapping for identifying dZ-DNA read sources in mixed datasets
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s42003-026-09849-8
    title: 'Z-Calling: a tool for A/Z (2,6-diaminopurine) base calling and dZ-DNA
      detection using PacBio HiFi reads'
  source_content_hash: sha256:95196997145c9a932d45d239a7a36d0f165b6f456c363544a6f04a750e467e8f
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/xiaochuanle/Z-Calling
  repo_url: https://github.com/xiaochuanle/Z-Calling
  tools:
  - PacBio Circular Consensus Sequencing (CCS)
  - Z-Calling
  - minimap2
  - pbmm2
  - samtools
  - z-bam2txt
  - z-calling-base
  - zfreq
  edam_operation: http://edamontology.org/operation_3903
  edam_topics:
  - http://edamontology.org/topic_0654
  - http://edamontology.org/topic_3168
  - http://edamontology.org/topic_3511
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6363636363636364
n_claims: 11
---

# ml-base-modification-calling-on-ccs-reads

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Machine-learning A/Z (2,6-diaminopurine) single-nucleotide base calling on PacBio CCS/HiFi reads with Z-Calling, which exploits Z-induced polymerase kinetic perturbations (pulse width and inter-pulse duration) to discriminate Z from A bases without chemical processing.

## When to use

Use when PacBio CCS/HiFi BAMs with kinetic tags (fi, ri, fp, rp) must be analyzed to detect dZ-DNA or call Z vs A bases at single-nucleotide resolution, e.g. in hybrid or transformed-organism DNA samples.

## When NOT to use

- Goal is to reliably classify hybrid/diluted reads as dZ-DNA at the molecule level: the SVM read classifier is optimized for fully substituted dZ-genomes and misses hybrid reads with diluted kinetic signatures
- Input reads lack PacBio kinetic tags or come from a non-PacBio platform (e.g. ONT), since the A/Z discrimination relies on PW/IPD pulse signals
- Reads are shorter than 500 bp or contain fewer than 100 A+Z bases, which were excluded in the paper due to limited A/Z bases

## Inputs

- PacBio CCS (HiFi) BAM with preserved kinetics (--keep-kinetics): tags fi, ri, fp, rp, fn, rn, Median_PW, Median_IPD
- Reference FASTA (e.g. combined yeast genome + 2-micron + engineered plasmid sequences) matching the alignment
- Pre-trained models: model/k11-mixed-AZ/scripted_m11.pth (base calling), model/k21-full-ZA/scripted_m21.pth (read detection), model/svm/k21ReadClassifier

## Outputs

- ZP-tagged BAM with per-base Z probability for each A/T position
- Tab-delimited z-bam2txt table (mapped: QueryName, ReadID, Strand, QueryPos, Base, RefName, RefPos, Probability)
- zfreq frequency report with columns chr, coor, num_records_above_threshold, num_records_below_threshold, ratio_above_threshold
- Read-level SVM classification TSV (+ dZ-DNA, - canonical, otherwise unclassified)

## How to apply

First filter the raw CCS BAM with py/filter_bam.py, requiring kinetic tags (fi, ri, fp, rp), a Median_PW/Median_IPD ratio between 0.3 and the 99th percentile, and at least 3 passes on both strands (fn >= 3, rn >= 3); optionally align filtered reads with pbmm2 align --preset CCS --sort. Then run the MLP base-calling module build/z-calling-base -k 11 with model/k11-mixed-AZ/scripted_m11.pth, which appends a ZP tag recording the Z probability for each A/T base; use -k 21 with model/k21-full-ZA/scripted_m21.pth only for whole-read dZ-DNA detection. For site-level quantification, convert the ZP-tagged BAM to a TSV with build/z-bam2txt -zp, supplying the exact same reference FASTA used for alignment, and aggregate per-position Z ratios with build/zfreq, which computes ratio_above_threshold as Z count / (A + Z count). The k11 model was chosen because Z incorporation primarily perturbs PW signals in a sequence-context-dependent (21 bp k-mer / 11-mer) manner, and the module reports AUCs of 0.94-0.96 with F1 of 0.85-0.92; a read-level SVM (build/z-calling-read, min_len >= 500, intervals = 100) is reserved for fully-substituted dZ-genome classification.

## Related tools

- **Z-Calling** (Toolkit providing filter_bam.py, z-calling-base (MLP base calling), z-bam2txt, zfreq, z-calling-read, and z-seq executables) — https://github.com/xiaochuanle/Z-Calling
- **PacBio Circular Consensus Sequencing (CCS)** (Source of HiFi reads with pulse width and inter-pulse duration kinetic signals (--keep-kinetics))
- **pbmm2** (Aligns filtered CCS reads to the reference (--preset CCS --sort) before base calling and coordinate-level frequency aggregation)
- **samtools** (Converts filtered BAM to FASTA for the taxonomic source-detection sub-workflow)
- **minimap2** (map-hifi mapping for identifying dZ-DNA read sources in mixed datasets)

## Examples

```
build/z-calling-base -k 11 -t 16 FilteredMappingBAM.bam model/k11-mixed-AZ/scripted_m11.pth ZP_tagged.bam && build/z-bam2txt -zp -t 16 ZP_tagged.bam REF.fasta > Result.tsv && build/zfreq -i Result.tsv -o Output_Freq.tsv
```

## Evaluation signals

- Filtered BAM obeys the stated thresholds: Median_PW/Median_IPD within [0.3, 99th percentile] and fn >= 3, rn >= 3
- zfreq output schema contains exactly chr, coor, num_records_above_threshold, num_records_below_threshold, and ratio_above_threshold columns
- Aggregate genomic Z/(A+Z) fraction matches an independent nucleotide-composition measurement (e.g. ~26.4% estimated vs 24.7% by HPLC-UV for transformed yeast hybrid DNA)
- Per-base Z probabilities are present as ZP tags on A/T bases in the output BAM; AUC on benchmarked A/Z discrimination is 0.942-0.952
- z-bam2txt must be run with the identical reference FASTA used for alignment, otherwise mapping coordinates in the zfreq table will not match

## Limitations

- Inherent noise of PacBio kinetic measurements: the subtle Z-induced shift (0.5-1.5-fold) overlaps substantially with canonical Adenine variance, limiting hybrid-sample quantification
- The SVM read classifier is optimized for fully substituted dZ-genomes and can miss hybrid reads with diluted kinetic signatures
- Lack of 'ground truth' datasets with verified modifications at defined loci; future enzymatic generation of hybrid training data is required
- Read-level SVM requires min_len >= 500 and exactly 100 intervals; shorter or unclassifiable reads are reported as unclassified

## Evidence
