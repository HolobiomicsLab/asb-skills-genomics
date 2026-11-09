---
name: genome-self-alignment-analysis
description: 'Use when comparing methylation states between duplicated/paralogous
  genomic regions: you have a genome reference FASTA and need paired homologous regions
  to test differential cytosine methylation.'
when_to_use_negative:
- You need interspecies (not intra-genome) synteny or whole-genome alignment between
  two different genomes rather than segmental-duplication pairs within one reference.
- You want repeat annotation/classification (transposable elements, tandem repeats)
  rather than paired aligned regions for pairwise methylation comparison.
- Your genome reference FASTA has not been downloaded/decided — thresholds and cytosine
  filtering are defined per reference build (TAIR10.1, IRGSP-1.0).
edam_operation: http://edamontology.org/operation_0491
edam_topics:
- http://edamontology.org/topic_0102
tools:
- name: MUMmer
  role: Whole-genome self-alignment (nucmer/promer) of the reference FASTA to generate
    candidate aligned region pairs
- name: Python3
  role: Parse MUMmer self-alignment output, filter pairs by length >100 and identity
    >0.99, retain cytosine-containing pairs, write repeat_pairs.tsv
- name: DeepSignal-plant
  role: Upstream methylation caller whose per-site methylation frequencies are compared
    between the repeat pairs
  repo: https://github.com/PengNi/deepsignal-plant
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-021-26278-9
    title: Genome-wide detection of cytosine methylations in plant from Nanopore data
      using deep learning
  source_content_hash: sha256:f02ccaa3853125e73c5a024ed4cfd032f0c732c06fc9b651209a6725f6c5f1e9
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/PengNi/deepsignal-plant
  tools:
  - DeepSignal-plant
  - MUMmer
  - Python3
  edam_operation: http://edamontology.org/operation_0491
  edam_topics:
  - http://edamontology.org/topic_0102
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 5
---

# genome-self-alignment-analysis

## Summary

Identify intra-genomic repeat pairs by self-aligning a plant genome reference (e.g., A. thaliana TAIR10 or O. sativa IRGSP-1.0) with MUMmer, keeping aligned region pairs of length >100 with identity score >0.99 that contain at least one cytosine, for downstream differential-methylation analysis in DeepSignal-plant.

## When to use

Use when comparing methylation states between duplicated/paralogous genomic regions: you have a genome reference FASTA and need paired homologous regions to test differential cytosine methylation.

## When NOT to use

- You need interspecies (not intra-genome) synteny or whole-genome alignment between two different genomes rather than segmental-duplication pairs within one reference.
- You want repeat annotation/classification (transposable elements, tandem repeats) rather than paired aligned regions for pairwise methylation comparison.
- Your genome reference FASTA has not been downloaded/decided — thresholds and cytosine filtering are defined per reference build (TAIR10.1, IRGSP-1.0).

## Inputs

- Genome reference FASTA (e.g., GCF_000001735.4_TAIR10.1_genomic.fna for A. thaliana; IRGSP-1.0/GCA_001433935.1 for O. sativa)
- Per-site methylation frequencies from DeepSignal-plant call_freq output
- In-house Python parsing scripts

## Outputs

- Tab-delimited repeat_pairs.tsv listing cytosine-containing repeat pairs (aligned region pairs, length >100, identity >0.99)
- Per-repeat-pair differential-methylation summary (fraction of differentially methylated cytosines)

## How to apply

Download the genome reference (e.g., GCF_000001735.4_TAIR10.1_genomic.fna for A. thaliana from NCBI, or IRGSP-1.0/GCA_001433935.1 for O. sativa from EnsemblPlants) and run a whole-genome self-alignment with MUMmer. Parse the self-alignment output with Python scripts, selecting pairs of aligned regions with length >100 and identity score >0.99 as repeat pairs. Retain only repeat pairs containing at least one cytosine in the paired regions, and write the filtered pairs to a tab-delimited repeat_pairs.tsv for the differential-methylation step, where a cytosine is differentially methylated if |rmet1−rmet2| >= 0.5 and a repeat pair is differential if >=10% of its cytosines differ.

## Related tools

- **MUMmer** (Whole-genome self-alignment (nucmer/promer) of the reference FASTA to generate candidate aligned region pairs)
- **Python3** (Parse MUMmer self-alignment output, filter pairs by length >100 and identity >0.99, retain cytosine-containing pairs, write repeat_pairs.tsv)
- **DeepSignal-plant** (Upstream methylation caller whose per-site methylation frequencies are compared between the repeat pairs) — https://github.com/PengNi/deepsignal-plant

## Examples

```
nucmer -maxmatch GCF_000001735.4_TAIR10.1_genomic.fna GCF_000001735.4_TAIR10.1_genomic.fna; delta-filter -i 0.99 -l 100 out.delta | python scripts/select_repeat_pairs.py --min-len 100 --min-identity 0.99 --out repeat_pairs.tsv
```

## Evaluation signals

- Every retained repeat pair has aligned-region length >100 and alignment identity score >0.99.
- Every retained repeat pair contains at least one cytosine in the paired regions (pairs with no cytosines are removed).
- Output file is a tab-delimited repeat_pairs.tsv with valid reference coordinates parseable for the differential-methylation step.
- Self-alignment covers the full genome reference (whole-genome nucmer/promer run, not a subset).
- Downstream differential-methylation output flags a cytosine as differential only when |rmet1−rmet2| >= 0.5 and a pair when >=10% of its cytosines differ.

## Limitations

- Repeat-pair selection relies on in-house Python scripts with no stated deposition, limiting reproducibility.
- The whole-genome self-alignment with MUMmer can be computationally expensive for large genomes.
- Thresholds (>100 bp, >0.99 identity, cytosine presence) are heuristics; the source notes details deferred to supplementary materials, so parameter provenance may be incomplete.

## Evidence

- [discussion] Repeat pair criteria: length >100, identity score >0.99, and at least one cytosine: "select two regions of which length >100 and identity score >0.99 as repeat pairs"
- [discussion] In-house scripts used for repeat-pair selection: "we used in-house Python scripts to select two regions of which length >100 and identity score >0.99 as repeat pairs"
- [discussion] Differentially methylated cytosine threshold: "the cytosine is said to be differentially methylated if |rmet1−rmet2 | > = 0.5"
- [discussion] Reference genome source for A. thaliana: "The genome reference of A. thaliana was downloaded from NCBI with the version GCF_000001735.4_TAIR10.1"
- [discussion] Reference genome source for O. sativa: "The genome reference and gene annotation of O. sativa were downloaded from EnsemblPlants with the version IRGSP-1.0"
