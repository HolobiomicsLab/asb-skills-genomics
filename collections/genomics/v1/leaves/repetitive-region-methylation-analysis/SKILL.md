---
name: repetitive-region-methylation-analysis
description: Use when a plant genome (e.g., A. thaliana TAIR10.1 or O. sativa IRGSP-1.0)
  has Nanopore-derived per-cytosine methylation frequencies and you must quantify
  5mC in highly repetitive regions where short-read bisulfite sequencing fails to
  map reads.
when_to_use_negative:
- Your goal is single-base 5mC calling from raw Nanopore signals rather than comparing
  methylation between duplicated regions — use DeepSignal-plant call_mods/call_freq
  directly instead.
- The genome has few segmental duplications under the 100 bp / 0.99 identity threshold,
  so the repeat-pair table would be too sparse for pair-level statistics.
- You lack per-cytosine methylation frequencies from both technologies being compared;
  repeat pairs alone cannot support the |rmet1 - rmet2| >= 0.5 differential call.
edam_operation: http://edamontology.org/operation_3232
edam_topics:
- http://edamontology.org/topic_0797
- http://edamontology.org/topic_0776
tools:
- name: MUMmer
  role: Whole-genome self-alignment (nucmer/promer) of the reference FASTA to generate
    candidate repeat pairs
- name: DeepSignal-plant
  role: Produces the per-cytosine Nanopore methylation frequencies (call_mods + call_freq)
    evaluated within each repeat pair
  repo: https://github.com/PengNi/deepsignal-plant
- name: Python3
  role: In-house scripts parse MUMmer alignments and filter region pairs by length,
    identity, and cytosine content
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-021-26278-9
    title: Genome-wide detection of cytosine methylations in plant from Nanopore data
      using deep learning
  source_content_hash: sha256:47ff0986fc5b91a2009ef21293922451db74c938bd283c89c28bbd96a46037aa
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/PengNi/deepsignal-plant
  tools:
  - DeepSignal-plant
  - MUMmer
  - Python3
  edam_operation: http://edamontology.org/operation_3232
  edam_topics:
  - http://edamontology.org/topic_0797
  - http://edamontology.org/topic_0776
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 6
---

# repetitive-region-methylation-analysis

## Summary

Identifies segmental-duplication repeat pairs by whole-genome self-alignment with MUMmer (length >= 100, identity >= 0.99, containing cytosines), then tests cytosine methylation differences between paired regions to compare Nanopore-based 5mC profiling (via DeepSignal-plant) against bisulfite sequencing in repetitive genome regions.

## When to use

Use when a plant genome (e.g., A. thaliana TAIR10.1 or O. sativa IRGSP-1.0) has Nanopore-derived per-cytosine methylation frequencies and you must quantify 5mC in highly repetitive regions where short-read bisulfite sequencing fails to map reads.

## When NOT to use

- Your goal is single-base 5mC calling from raw Nanopore signals rather than comparing methylation between duplicated regions — use DeepSignal-plant call_mods/call_freq directly instead.
- The genome has few segmental duplications under the 100 bp / 0.99 identity threshold, so the repeat-pair table would be too sparse for pair-level statistics.
- You lack per-cytosine methylation frequencies from both technologies being compared; repeat pairs alone cannot support the |rmet1 - rmet2| >= 0.5 differential call.

## Inputs

- Genome reference FASTA (e.g., GCF_000001735.4_TAIR10.1_genomic.fna from NCBI for A. thaliana; IRGSP-1.0 / GCA_001433935.1 from EnsemblPlants for O. sativa)
- Per-cytosine methylation frequency files from DeepSignal-plant call_freq (Nanopore methylation frequencies per site)
- Per-cytosine methylation frequencies from bisulfite sequencing for comparison

## Outputs

- repeat_pairs.tsv: tab-delimited table of filtered repeat pairs (length > 100, identity > 0.99, at least one cytosine) for the differential-methylation analysis
- Per-repeat-pair differential-methylation statistics (fraction of cytosines with |rmet1 - rmet2| >= 0.5)

## How to apply

Self-align each genome reference FASTA (e.g., GCF_000001735.4_TAIR10.1_genomic.fna for A. thaliana) against itself using MUMmer (nucmer/promer), then parse the alignment with in-house Python scripts to select region pairs with length >100 and identity score >0.99 as repeat pairs; retain only pairs containing at least one cytosine and write them to a tab-delimited repeat_pairs.tsv. For each cytosine in a retained pair, compute methylation frequencies rmet1 and rmet2 from DeepSignal-plant call_freq output (per-site counts of methylated calls over total mapped reads) and the bisulfite equivalents; call a cytosine differentially methylated when |rmet1 - rmet2| >= 0.5. Declare a repeat pair differentially methylated when at least 10% of its cytosines are differentially methylated. The rationale is that Nanopore long reads profile 5mCs in repetitive regions inaccessible to short-read bisulfite sequencing, so these pair-level statistics quantify the coverage advantage.

## Related tools

- **MUMmer** (Whole-genome self-alignment (nucmer/promer) of the reference FASTA to generate candidate repeat pairs)
- **DeepSignal-plant** (Produces the per-cytosine Nanopore methylation frequencies (call_mods + call_freq) evaluated within each repeat pair) — https://github.com/PengNi/deepsignal-plant
- **Python3** (In-house scripts parse MUMmer alignments and filter region pairs by length, identity, and cytosine content)

## Examples

```
nucmer --maxmatch GCF_000001735.4_TAIR10.1_genomic.fna GCF_000001735.4_TAIR10.1_genomic.fna; python select_repeat_pairs.py --delta out.delta --min_len 100 --min_id 0.99 --out repeat_pairs.tsv
```

## Evaluation signals

- Every retained repeat pair has alignment length > 100 and identity score > 0.99 in the parsed MUMmer output.
- repeat_pairs.tsv exists, is tab-delimited, and contains no pair lacking at least one cytosine.
- Per-cytosine differential calls all satisfy the |rmet1 - rmet2| >= 0.5 threshold; a repeat pair is flagged only when >= 10% of its cytosines are differentially methylated.
- The same self-alignment procedure reproduces pairs independently for A. thaliana and O. sativa references, confirming genome-agnostic behavior.
- Cytosine counts within pairs match the number of cytosines covered by DeepSignal-plant methylation frequency calls for those coordinates.

## Limitations

- Repeat-pair selection depends on in-house Python scripts with no stated public deposition, limiting reproducibility.
- The definition is threshold-sensitive (length > 100, identity > 0.99); different cutoffs would change the set of pairs analyzed.
- Comparison quality is bounded by bisulfite sequencing, which cannot profile 5mCs in repetitive genome regions — the very regions being compared.
- Key differential-methylation details are deferred to supplementary materials not included in the main document.

## Evidence

- [discussion] repeat pair threshold definition: "select two regions of which length >100 and identity score >0.99 as repeat pairs"
- [discussion] cytosine differential methylation criterion: "the cytosine is said to be differentially methylated if |rmet1−rmet2 | > = 0.5"
- [intro] bisulfite limitation in repetitive regions: "due to short-read sequencing, bisulfite sequencing cannot profile 5mCs in the repetitive genome regions"
- [discussion] MUMmer tool and Python parsing: "we used in-house Python scripts to select two regions of which length >100 and identity score >0.99 as repeat pairs"
