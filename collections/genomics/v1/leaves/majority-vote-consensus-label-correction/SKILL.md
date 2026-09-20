---
name: majority-vote-consensus-label-correction
description: Use when HiFi reads mapped to a linear reference (e.g., CHM13 v2.0) at
  low coverage leave misclassified or unclassified ('None') chromosome labels, and
  chromosome-wise de novo assembly requires corrected per-read classes.
when_to_use_negative:
- 'Data is high coverage (e.g., 41x HiFi for HG00438): reference-guided binning overhead
  is unnecessary and assemblers like hifiasm can be run directly'
- No high-quality linear reference genome exists to provide initial chromosome labels
  for the ALG nodes
- Input is noisy long reads (ONT) or short reads rather than high-fidelity (HiFi)
  reads, for which the overlap-degree weighting was designed
edam_operation: http://edamontology.org/operation_3454
edam_topics:
- http://edamontology.org/topic_0102
- http://edamontology.org/topic_0622
tools:
- name: HiFiCCL
  role: Implements the ALG construction, Z-drop-inspired path search, and majority-vote
    class correction (Equations 2, 3, 5, 6)
  repo: https://github.com/zjjbuqi/HiFiCCL
- name: minimap2
  role: Produces '-ax map-hifi' read-to-reference labels and the all-vs-all PAF overlaps
    that seed the ALG
  repo: https://github.com/lh3/minimap2
- name: pysam
  role: Python package required by HiFiCCL for reading SAM/BAM alignment inputs
  repo: https://github.com/pysam-developers/pysam
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1002/advs.202515308
    title: Reference‐Guided Chromosome‐by‐Chromosome <i>de novo</i> Assembly at Scale
      Using Low‐Coverage High‐Fidelity Long‐Reads with HiFiCCL
  source_content_hash: sha256:0b66f6a1710a135fe5ad1d8a9a622e73d5fad3d6b76c082941ceccb9979e75d9
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/zjjbuqi/HiFiCCL
  tools:
  - HiFiCCL
  - Python
  - minimap2
  - pysam
  edam_operation: http://edamontology.org/operation_3454
  edam_topics:
  - http://edamontology.org/topic_0102
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6875
n_claims: 16
---

# majority-vote-consensus-label-correction

## Summary

HiFiCCL's de novo chromosome-class correction: a Z-drop-inspired heuristic search of the Alignment Label Graph (ALG) partitions read overlaps into reliable linear paths, and chromosome labels are re-assigned by majority vote per path and then across M randomized iterations. This corrects misclassified or unaligned read labels at low (~5x) HiFi coverage before chromosome-wise assembly.

## When to use

Use when HiFi reads mapped to a linear reference (e.g., CHM13 v2.0) at low coverage leave misclassified or unclassified ('None') chromosome labels, and chromosome-wise de novo assembly requires corrected per-read classes.

## When NOT to use

- Data is high coverage (e.g., 41x HiFi for HG00438): reference-guided binning overhead is unnecessary and assemblers like hifiasm can be run directly
- No high-quality linear reference genome exists to provide initial chromosome labels for the ALG nodes
- Input is noisy long reads (ONT) or short reads rather than high-fidelity (HiFi) reads, for which the overlap-degree weighting was designed

## Inputs

- PacBio HiFi reads (FASTA/FASTQ, e.g., ~5x coverage human/plant datasets such as HG002 SRR10382244)
- Linear reference genome (e.g., CHM13 v2.0) for initial chromosome labels
- minimap2 read-to-reference alignments (SAM) and all-vs-all read overlaps (PAF)

## Outputs

- chr_by_chr_reads: per-chromosome bins of reads with corrected final chromosome labels
- prefix.map_to_reference.sam: retained read-to-reference alignments
- prefix.all_vs_all.paf: retained all-vs-all pairwise overlaps
- Per-read final chromosome classes determined by majority vote across M iterations

## How to apply

First align HiFi reads to the reference with minimap2 '-ax map-hifi' for initial labels and run all-vs-all mapping ('-D --dual=no --no-long-join -k19 -w5 -U50,500 --rmq -A1 -B19 -O39,81 -H -e0 -m100 -N20') to obtain overlaps, then build the bidirectional ALG with edge weights od = al^2 * (m/(ql*rl)) * w, collapsing contained shorter-read nodes into longer ones and labeling unaligned nodes 'None'. Run M randomized-order iterations (default --iterations 200) of the Z-drop-inspired greedy bidirectional search from unvisited nodes, truncating each path when Equation (2) w_jq <= theta * w_ij + epsilon is violated (default -W 0.75 weight, -N 3 minimum supporting reads). For each path, update the chromosomal class by majority vote among path nodes (Equation 3) and propagate the winning class only if N(PATH_k) > S_N_min; record per-node class lists in ClassHash across iterations. Determine each read's final class by majority vote across the M iterations (Equation 5), cluster reads into chromosomal bins (Equation 6), and write binned reads to chr_by_chr_reads. The rationale is that nodes on a reliable linear overlap subgraph belong to the same chromosome, so consensus voting recovers labels even when reference alignment fails or is wrong at ~5x coverage.

## Related tools

- **HiFiCCL** (Implements the ALG construction, Z-drop-inspired path search, and majority-vote class correction (Equations 2, 3, 5, 6)) — https://github.com/zjjbuqi/HiFiCCL
- **minimap2** (Produces '-ax map-hifi' read-to-reference labels and the all-vs-all PAF overlaps that seed the ALG) — https://github.com/lh3/minimap2
- **pysam** (Python package required by HiFiCCL for reading SAM/BAM alignment inputs) — https://github.com/pysam-developers/pysam

## Examples

```
minimap2 -D --dual=no --no-long-join -k19 -w5 -U50,500 --rmq -A1 -B19 -O39,81 -H -e0 -m100 -N20 reads.fa reads.fa > prefix.all_vs_all.paf
```

## Evaluation signals

- chr_by_chr_reads output exists and contains per-chromosome read bins covering all input reads
- Every read within each bin carries the same final chromosome label consistent with the majority-vote result
- Each propagated path exceeded the minimum support threshold N(PATH_k) > S_N_min (default -N 3)
- Final per-read classes agree with the cross-iteration majority vote over M=200 iterations recorded in ClassHash
- The retained alignment files *prefix*.map_to_reference.sam and *prefix*.all_vs_all.paf are present and consistent with the ALG edge weights

## Limitations

- At ~5x coverage many genomic regions remain missing, so completeness gaps are unresolved even after label correction
- For plant genomes with higher genetic diversity, only two samples were tested, so correction robustness across diverse species is unverified
- Correction depends on overlap-graph connectivity; at very low coverage, sparse overlaps may leave paths below the S_N_min support threshold, so those reads keep their (possibly wrong) reference-derived labels

## Evidence

- [methods] Inspired by the Z‐drop algorithm, we propose a heuristic algorithm to search the ALG and correct sequence classes, thereby completing the classes correction of reads: "Inspired by the Z‐drop algorithm, we propose a heuristic algorithm to search the ALG and correct sequence classes, thereby completing the classes correction of reads"
- [methods] the chromosomal class is updated based on the majority vote among the nodes in the path: "the chromosomal class is updated based on the majority vote among the nodes in the path"
- [methods] reads are clustered into chromosomal bins based on their final assigned classes: "reads are clustered into chromosomal bins based on their final assigned classes"
- [abstract] HiFiCCL is proposed, the first assembly framework specifically designed for low-coverage high-fidelity reads: "HiFiCCL is proposed, the first assembly framework specifically designed for low-coverage high-fidelity reads"
- [readme] ./minimap2 -ax map-hifi ref.fa pacbio-ccs.fq.gz > aln.sam # PacBio HiFi/CCS genomic reads (v2.19+): [evidence span withheld — anchor status: external_doc]
