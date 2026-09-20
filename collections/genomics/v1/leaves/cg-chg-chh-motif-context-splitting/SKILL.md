---
name: cg-chg-chh-motif-context-splitting
description: Use when DeepSignal-plant call_freq output (e.g. fast5s.C.call_mods.frequency.tsv)
  contains mixed-context cytosines and downstream analysis needs per-context 5mC counts
  or comparisons, since plants methylate in all three contexts.
when_to_use_negative:
- Input comes from tools that only call CpG-context methylation (e.g. standard nanopolish/Megalodon
  workflows) — there is no CHG/CHH content to split
- Downstream analysis requires per-read methylation probabilities rather than per-site
  aggregated frequencies — split call_mods output logic instead, not the frequency
  file
edam_operation: http://edamontology.org/operation_0335
edam_topics:
- http://edamontology.org/topic_3673
- http://edamontology.org/topic_0622
tools:
- name: DeepSignal-plant
  role: Produces the mixed-context per-site 5mC methylation-frequency file (call_mods
    with --motifs C + call_freq) that is then split by motif; provides split_freq_file_by_5mC_motif.py
  repo: https://github.com/PengNi/deepsignal-plant
- name: Tombo
  role: Re-squiggles raw signals against the genome reference upstream of methylation
    calling, producing the per-site positions whose motif context is determined during
    splitting
  repo: https://github.com/nanoporetech/tombo
provenance:
  source_task_ids:
  - task_007
  - task_008
  source_papers:
  - doi: 10.1038/s41467-021-26278-9
    title: Genome-wide detection of cytosine methylations in plant from Nanopore data
      using deep learning
  source_content_hash: sha256:65ecb314128f2dcd1cd8d3dbc657eed5164d2545de424c175324e09ca9abcd93
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/PengNi/deepsignal-plant
  tools:
  - DeepSignal-plant
  - Guppy
  - Tombo
  edam_operation: http://edamontology.org/operation_0335
  edam_topics:
  - http://edamontology.org/topic_3673
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4
n_claims: 5
---

# CG/CHG/CHH motif context splitting

## Summary

Splits DeepSignal-plant per-site 5mC methylation-frequency output into three sequence-context files (CG, CHG, CHH) so that context-specific site counts, coverage, and per-context analyses (e.g., DMR repeat-pair breakdown) can be computed separately.

## When to use

Use when DeepSignal-plant call_freq output (e.g. fast5s.C.call_mods.frequency.tsv) contains mixed-context cytosines and downstream analysis needs per-context 5mC counts or comparisons, since plants methylate in all three contexts.

## When NOT to use

- Input comes from tools that only call CpG-context methylation (e.g. standard nanopolish/Megalodon workflows) — there is no CHG/CHH content to split
- Downstream analysis requires per-read methylation probabilities rather than per-site aggregated frequencies — split call_mods output logic instead, not the frequency file

## Inputs

- DeepSignal-plant call_freq methylation-frequency table (e.g. fast5s.C.call_mods.frequency.tsv) with chrom, pos, strand, coverage, modification_frequency columns
- Reference genome FASTA (e.g. GCF_000001735.4_TAIR10.1_genomic.fna for A. thaliana or IRGSP-1.0 for O. sativa) to determine the motif context of each cytosine

## Outputs

- Three context-split methylation-frequency files: one each for CG, CHG, and CHH cytosines
- Per-context 5mC site counts used for comparison against bisulfite sequencing (Supplementary Table 1)

## How to apply

Run deepsignal_plant call_mods with --motifs C so that per-read 5mC probabilities are called in all three contexts at once, then aggregate to per-site methylation frequencies with call_freq (optionally --bed --sort for bedMethyl). The frequency table carries chrom, pos, strand, coverage, and modification_frequency columns per cytosine. Apply split_freq_file_by_5mC_motif.py to partition the frequency file into CG, CHG, and CHH files based on each cytosine's downstream sequence motif. This matters in plants because existing Nanopore methods largely detect only CpG, while plant methylation is substantial in CHG and CHH; splitting enables context-resolved counting (e.g., comparison against Supplementary Table 1 bisulfite counts) and context-dominance analysis of DMR repeat pairs. Keep the three context files paired across samples/species so downstream per-context comparisons use matched site sets.

## Related tools

- **DeepSignal-plant** (Produces the mixed-context per-site 5mC methylation-frequency file (call_mods with --motifs C + call_freq) that is then split by motif; provides split_freq_file_by_5mC_motif.py) — https://github.com/PengNi/deepsignal-plant
- **Tombo** (Re-squiggles raw signals against the genome reference upstream of methylation calling, producing the per-site positions whose motif context is determined during splitting) — https://github.com/nanoporetech/tombo

## Examples

```
python split_freq_file_by_5mC_motif.py --input fast5s.C.call_mods.frequency.tsv --output_prefix fast5s.C.call_mods --motifs C CG CHG CHH
```

## Evaluation signals

- Each output context file contains only cytosines of the corresponding motif: CG followed by G; CHG with H∈{A,C,T} followed by G; CHH otherwise
- The union of CG+CHG+CHH rows equals the row count of the original frequency file (no cytosines lost or duplicated)
- Per-context detected 5mC site counts summed across contexts should reproduce the totals compared to bisulfite (1.1% excess in A. thaliana, 5.3% in O. sativa at 100× coverage)
- When used for repeat-pair DMR analysis, the per-context breakdown should show CG dominant in A. thaliana and CHG dominant in O. sativa

## Limitations

- Splitting depends on the genome reference being correctly re-squiggled; a mismatch between reference version (e.g. TAIR10 vs IRGSP-1.0) and the annotation used for motif calling shifts cytosines between contexts
- Sites with low coverage in one context are unaffected by splitting itself, but context-specific comparisons inherit DeepSignal-plant's bisulfite-bounded gold standard, so CHH/CHG calls may carry label noise from replicate unions

## Evidence

- [abstract] existing methods can only detect 5mCs in the CpG context, which limits their application in plants: "existing methods can only detect 5mCs in the CpG context, which limits their application in plants"
- [intro] no current method can profile genome-wide 5mCs in all three contexts: "no current method can profile genome-wide 5mCs in all three contexts with acceptable accuracies using third-generation sequencing data"
- [methods] call_freq function with the call_mods file as input: "A modification-frequency file can be generated by `call_freq` function with the call_mods file as input"
- [discussion] CG context dominant among DMR pairs in A. thaliana and the CHG context dominant in O. sativa: [evidence span withheld — anchor status: not_found]
- [intro] one deep learning model in DeepSignal-plant to detect 5mC sites in all three CpG, CHG, and CHH sequence contexts: "We train one deep learning model in DeepSignal-plant to detect 5mC sites in all three CpG, CHG, and CHH sequence contexts."
