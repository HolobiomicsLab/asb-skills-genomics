---
name: repeat-pair-identification
description: 'Use when comparing methylation states between paralogous duplicated
  regions of a genome: input is a reference FASTA (e.g. TAIR10 or IRGSP-1.0) and the
  goal is paired regions suitable for differential-methylation analysis.'
when_to_use_negative:
- The goal is de novo repeat-family annotation or transposable-element classification
  rather than paired segmental duplications for methylation comparison
- The genome reference is not yet available or not self-aligned with MUMmer — no alignment
  output exists to filter
- Analyzing methylation at single sites without a duplicated-region comparator; this
  skill only produces paired regions for differential-methylation testing
edam_operation: http://edamontology.org/operation_0292
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3673
tools:
- name: MUMmer
  role: Whole-genome self-alignment (nucmer/promer) of the reference FASTA to generate
    candidate aligned region pairs
- name: Python3
  role: In-house scripts to parse the self-alignment and select repeat pairs with
    length >100, identity >0.99, and at least one cytosine
  repo: https://github.com/PengNi/deepsignal-plant
- name: DeepSignal-plant
  role: Upstream tool whose per-site methylation frequencies are counted within the
    identified repeat pairs for the differential-methylation step
  repo: https://github.com/PengNi/deepsignal-plant
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-021-26278-9
    title: Genome-wide detection of cytosine methylations in plant from Nanopore data
      using deep learning
  source_content_hash: sha256:cdbc41c67585647fef137aebea4279fe8abc6c08875f31d1c4a035b00b5bc3a7
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/PengNi/deepsignal-plant
  tools:
  - DeepSignal-plant
  - MUMmer
  - Python3
  edam_operation: http://edamontology.org/operation_0292
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3673
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8125
n_claims: 16
---

# repeat-pair-identification

## Summary

Identifies segmental-duplication repeat pairs by whole-genome self-alignment with MUMmer, retaining aligned region pairs with length >100 and identity score >0.99 that contain at least one cytosine, for downstream differential-methylation analysis in the DeepSignal-plant pipeline.

## When to use

Use when comparing methylation states between paralogous duplicated regions of a genome: input is a reference FASTA (e.g. TAIR10 or IRGSP-1.0) and the goal is paired regions suitable for differential-methylation analysis.

## When NOT to use

- The goal is de novo repeat-family annotation or transposable-element classification rather than paired segmental duplications for methylation comparison
- The genome reference is not yet available or not self-aligned with MUMmer — no alignment output exists to filter
- Analyzing methylation at single sites without a duplicated-region comparator; this skill only produces paired regions for differential-methylation testing

## Inputs

- Genome reference FASTA (e.g. GCF_000001735.4_TAIR10.1_genomic.fna for A. thaliana from NCBI, or IRGSP-1.0 / GCA_001433935.1 for O. sativa from EnsemblPlants)
- MUMmer self-alignment output (nucmer/promer) of the reference against itself

## Outputs

- Tab-delimited table of filtered repeat pairs (repeat_pairs.tsv) containing aligned region pairs with length >100, identity >0.99, and at least one cytosine, for use in the differential-methylation analysis

## How to apply

Download the genome reference from NCBI or EnsemblPlants (e.g. GCF_000001735.4_TAIR10.1_genomic.fna for A. thaliana) and run a whole-genome self-alignment with MUMmer (nucmer/promer) on the reference FASTA. Parse the self-alignment output with Python3 scripts to select pairs of aligned regions whose length >100 and identity score >0.99 as repeat pairs. Retain only those repeat pairs containing at least one cytosine in the paired regions, then write the filtered pairs to a tab-delimited table (repeat_pairs.tsv). These pairs are later used to count cytosines and call differentially methylated cytosines (|rmet1−rmet2| >= 0.5, with a repeat pair called differentially methylated when >=10% of its cytosines differ). Note the paper flags that repeat-pair selection relied on in-house Python scripts with no stated deposition, so reimplementation of the parsing step may be required.

## Related tools

- **MUMmer** (Whole-genome self-alignment (nucmer/promer) of the reference FASTA to generate candidate aligned region pairs)
- **Python3** (In-house scripts to parse the self-alignment and select repeat pairs with length >100, identity >0.99, and at least one cytosine) — https://github.com/PengNi/deepsignal-plant
- **DeepSignal-plant** (Upstream tool whose per-site methylation frequencies are counted within the identified repeat pairs for the differential-methylation step) — https://github.com/PengNi/deepsignal-plant

## Examples

```
nucmer -maxmatch GCF_000001735.4_TAIR10.1_genomic.fna GCF_000001735.4_TAIR10.1_genomic.fna; delta-filter -i 0.99 -l 100 out.delta; python select_repeat_pairs.py --min-len 100 --min-ident 0.99 --out repeat_pairs.tsv
```

## Evaluation signals

- Every retained repeat pair has aligned-region length >100 and identity score >0.99 in the parsed output
- Each output repeat pair contains at least one cytosine in the paired regions; pairs lacking cytosines are dropped
- The output repeat_pairs.tsv is a tab-delimited table whose entries correspond to region pairs found in the MUMmer self-alignment
- Self-alignment is run against the correct reference version (GCF_000001735.4_TAIR10.1 for A. thaliana; IRGSP-1.0 / GCA_001433935.1 for O. sativa)

## Limitations

- Repeat-pair selection relies on in-house Python scripts that were not stated as publicly deposited, limiting reproducibility
- Thresholds are strict (length >100, identity >0.99), so more divergent duplications below these cutoffs will not be captured
- The workflow depends on the chosen reference assembly version; changing the reference (e.g. TAIR10.1 vs IRGSP-1.0) changes the repeat pairs produced

## Evidence

- [discussion] Repeat pair criteria with thresholds: "select two regions of which length >100 and identity score >0.99 as repeat pairs"
- [discussion] In-house scripts and reproducibility gap: "we used in-house Python scripts to select two regions of which length >100 and identity score >0.99 as repeat pairs"
- [discussion] Differential methylation call on repeat pairs: "the cytosine is said to be differentially methylated if |rmet1−rmet2 | > = 0.5"
- [discussion] Reference genome download versions: "The genome reference of A. thaliana was downloaded from NCBI with the version GCF_000001735.4_TAIR10.1"
- [discussion] DeepSignal-plant implementation language: "DeepSignal-plant is implemented in Python3 and PyTorch (version 1.2.0)"
