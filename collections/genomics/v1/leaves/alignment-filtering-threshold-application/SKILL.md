---
name: alignment-filtering-threshold-application
description: Use when genome FASTA references (e.g. A. thaliana TAIR10 GCF_000001735.4
  or O. sativa IRGSP-1.0) must be scanned for near-identical duplicated regions to
  build paired comparison units for methylation contrasts.
when_to_use_negative:
- When the analysis goal is per-read or per-site methylation calling from Nanopore
  signals — use the DeepSignal-plant call_mods/call_freq pipeline instead
- When studying highly repetitive regions rather than duplicated pairs — bisulfite/alignment
  approaches may not cover them and this pairing step is not designed for that question
- When the input is already a repeat-pair table or methylation-frequency table — proceed
  directly to the |rmet1−rmet2| >= 0.5 differential-methylation filtering step
edam_operation: http://edamontology.org/operation_3695
edam_topics:
- http://edamontology.org/topic_0102
- http://edamontology.org/topic_0622
tools:
- name: MUMmer
  role: Performs the whole-genome self-alignment (nucmer/promer) whose alignments
    are filtered into repeat pairs
- name: Python3
  role: In-house scripts parse the MUMmer self-alignment and apply the length >100
    / identity >0.99 / cytosine-presence filters
- name: DeepSignal-plant
  role: Upstream producer of per-cytosine methylation frequencies (call_mods/call_freq)
    that are compared across the selected repeat pairs
  repo: https://github.com/PengNi/deepsignal-plant
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-021-26278-9
    title: Genome-wide detection of cytosine methylations in plant from Nanopore data
      using deep learning
  source_content_hash: sha256:fafe7abe7d02e1a699e490a84821149a798c811f42d19ededf5dcce68eddbc2a
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/PengNi/deepsignal-plant
  tools:
  - DeepSignal-plant
  - MUMmer
  - Python3
  edam_operation: http://edamontology.org/operation_3695
  edam_topics:
  - http://edamontology.org/topic_0102
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 10
---

# Alignment-filtering threshold application

## Summary

Identifies segmental-duplication repeat pairs by whole-genome self-alignment with MUMmer, retaining aligned region pairs longer than 100 bp with identity score >0.99 that contain at least one cytosine, to serve as the substrate for downstream differential-methylation analysis.

## When to use

Use when genome FASTA references (e.g. A. thaliana TAIR10 GCF_000001735.4 or O. sativa IRGSP-1.0) must be scanned for near-identical duplicated regions to build paired comparison units for methylation contrasts.

## When NOT to use

- When the analysis goal is per-read or per-site methylation calling from Nanopore signals — use the DeepSignal-plant call_mods/call_freq pipeline instead
- When studying highly repetitive regions rather than duplicated pairs — bisulfite/alignment approaches may not cover them and this pairing step is not designed for that question
- When the input is already a repeat-pair table or methylation-frequency table — proceed directly to the |rmet1−rmet2| >= 0.5 differential-methylation filtering step

## Inputs

- Genome reference FASTA (GCF_000001735.4_TAIR10.1_genomic.fna for A. thaliana; IRGSP-1.0 / GCA_001433935.1 for O. sativa)
- MUMmer self-alignment output (nucmer/promer delta/coords) for the reference against itself

## Outputs

- Tab-delimited repeat_pairs.tsv listing repeat-pair regions with length >100, identity >0.99, and at least one cytosine
- Per-pair cytosine counts used in the downstream differential-methylation analysis

## How to apply

Download the genome reference FASTA from NCBI (e.g. GCF_000001735.4_TAIR10.1_genomic.fna for A. thaliana) or EnsemblPlants (IRGSP-1.0 / GCA_001433935.1 for O. sativa), then run a whole-genome self-alignment with MUMmer (nucmer/promer). Parse the self-alignment output with in-house Python scripts, selecting two aligned regions of which length >100 and identity score >0.99 as repeat pairs. Retain only those repeat pairs that contain at least one cytosine in the paired regions, and write the filtered pairs to a tab-delimited repeat_pairs.tsv table. These pairs feed the differential-methylation step, where a cytosine is differentially methylated if |rmet1−rmet2| >= 0.5, and a repeat pair is differential if ≥10% of its cytosines differ.

## Related tools

- **MUMmer** (Performs the whole-genome self-alignment (nucmer/promer) whose alignments are filtered into repeat pairs)
- **Python3** (In-house scripts parse the MUMmer self-alignment and apply the length >100 / identity >0.99 / cytosine-presence filters)
- **DeepSignal-plant** (Upstream producer of per-cytosine methylation frequencies (call_mods/call_freq) that are compared across the selected repeat pairs) — https://github.com/PengNi/deepsignal-plant

## Examples

```
nucmer --prefix self GCF_000001735.4_TAIR10.1_genomic.fna GCF_000001735.4_TAIR10.1_genomic.fna
```

## Evaluation signals

- Every retained repeat pair must have aligned-region length >100 bp and identity score >0.99; spot-check a sample of pairs against the raw MUMmer alignment coordinates
- Every output repeat pair must contain at least one cytosine in the paired regions; pairs with zero cytosines must be dropped
- repeat_pairs.tsv must be tab-delimited and each pair must map back to valid coordinates in the reference FASTA (GCF_000001735.4_TAIR10.1_genomic.fna or IRGSP-1.0)
- Downstream differential call: a cytosine is differentially methylated only if |rmet1−rmet2| >= 0.5, and a pair qualifies only if >=10% of its cytosines differ

## Limitations

- Repeat-pair selection relies on in-house Python scripts with no stated deposition — a reproducibility gap, so the exact parsing logic must be re-implemented
- The thresholds (length >100, identity >0.99) are fixed cutoffs from this paper; they may miss divergent duplications or include repeats below desired stringency
- Key differential-methylation and repeat-pair details are deferred to supplementary materials, limiting full reproduction from the main text alone

## Evidence

- [discussion] Repeat-pair selection thresholds: "we used in-house Python scripts to select two regions of which length >100 and identity score >0.99 as repeat pairs"
- [discussion] Differential-methylation threshold applied to repeat pairs: "the cytosine is said to be differentially methylated if |rmet1−rmet2 | > = 0.5"
- [discussion] Reference provenance for the self-alignment input: "The genome reference of A. thaliana was downloaded from NCBI with the version GCF_000001735.4_TAIR10.140"
