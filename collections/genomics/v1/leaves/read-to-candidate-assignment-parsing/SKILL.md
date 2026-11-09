---
name: read-to-candidate-assignment-parsing
description: Use when simulating Iso-Seq reads of multigene-family transcripts (e.g.,
  TSPY, HSFY) and you need to determine, per isoform and per depth, whether IsoCon
  captured it, removed it in statistical filtering, or failed to converge it during
  error correction.
when_to_use_negative:
- IsoCon has not been run and no cluster_info.tsv, final_candidates.fa, or not_converged.fa
  exist — assignment parsing requires the converged clustering output
- Reads come from real (not simulated) data without isoform truth labels — fate classification
  per simulated isoform cannot be computed
- The goal is to distinguish transcripts from different gene copies post-reconstruction
  — IsoCon does not provide definitive gene-copy assignment ('further downstream analysis
  would be needed')
edam_operation: http://edamontology.org/operation_2995
edam_topics:
- http://edamontology.org/topic_0080
- http://edamontology.org/topic_3170
tools:
- name: IsoCon
  role: Generates final_candidates.fa, not_converged.fa, and cluster_info.tsv via
    iterative clustering, error correction, and greedy read-to-candidate assignment
    whose parsing enables the fate classification
  repo: https://github.com/ksahlin/IsoCon
- name: parasail
  role: Underlying SIMD pairwise aligner used by IsoCon for pairwise alignments of
    each read to the cluster center during error correction
  repo: https://github.com/jeffdaily/parasail-python
- name: edlib
  role: Fast edit-distance library used in IsoCon's read-to-candidate distance computations
    during clustering
  repo: https://github.com/Martinsos/edlib
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-018-06910-x
    title: Deciphering highly similar multigene family transcripts from Iso-Seq data
      with IsoCon
  source_content_hash: sha256:85c383f3dd3fe8a961c1cf6baf158908d1969caae05980bfb77f083b928b864d
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ksahlin/IsoCon
  tools:
  - IsoCon
  - edlib
  - parasail
  edam_operation: http://edamontology.org/operation_2995
  edam_topics:
  - http://edamontology.org/topic_0080
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5714285714285714
n_claims: 7
---

# read-to-candidate-assignment-parsing

## Summary

Parses IsoCon's per-read clustering output (cluster_info.tsv) together with final_candidates.fa and not_converged.fa to classify each simulated isoform's fate — captured, filtered in the statistical step, or lost in error correction — as a function of per-isoform read depth.

## When to use

Use when simulating Iso-Seq reads of multigene-family transcripts (e.g., TSPY, HSFY) and you need to determine, per isoform and per depth, whether IsoCon captured it, removed it in statistical filtering, or failed to converge it during error correction.

## When NOT to use

- IsoCon has not been run and no cluster_info.tsv, final_candidates.fa, or not_converged.fa exist — assignment parsing requires the converged clustering output
- Reads come from real (not simulated) data without isoform truth labels — fate classification per simulated isoform cannot be computed
- The goal is to distinguish transcripts from different gene copies post-reconstruction — IsoCon does not provide definitive gene-copy assignment ('further downstream analysis would be needed')

## Inputs

- Simulated PacBio Iso-Seq FASTQ/FASTA read sets (10 replicates per depth condition, e.g., READ_COUNTS 20/100/500/2500/12500)
- IsoCon output directory per run: final_candidates.fa, not_converged.fa, cluster_info.tsv
- Truth table of simulated isoforms (gene family membership, e.g., TSPY13P, HSFY2, DAZ2) and per-isoform read counts

## Outputs

- Per-isoform fate classification table (captured / filtered in statistical step / lost in error correction) per replicate and depth
- Aggregated isoform fate counts as a function of experiment depth and per-isoform depth
- fig_s1_isoform_capture_stripplot.png (seaborn stripplot of isoform fate vs. depth)

## How to apply

Run 'IsoCon pipeline -fl_reads <reads> -outfolder <dir>' on each simulated PacBio Iso-Seq read set (10 replicates per condition from the IsoCon_Eval snakemake pipeline), producing final_candidates.fa, not_converged.fa, and cluster_info.tsv. Parse cluster_info.tsv to obtain the greedy read-to-candidate assignment — the article describes 'greedy assignment of reads to closest-neighbor candidates' followed by pairwise hypothesis testing in the statistical filter. Join the assigned read IDs with the known simulated isoform truth labels and per-isoform read counts, then classify each isoform: present in final_candidates.fa = captured; absent from candidates but its reads tested against other candidates = filtered in the statistical step (p-value threshold alpha=0.01); present only in not_converged.fa = lost in error correction. Aggregate isoform fates across replicates as a function of experiment depth and per-isoform depth, then plot fate versus depth (seaborn stripplot). The rationale is that low-depth isoforms mostly fail in error correction rather than being overfiltered, which the fate decomposition makes explicit.

## Related tools

- **IsoCon** (Generates final_candidates.fa, not_converged.fa, and cluster_info.tsv via iterative clustering, error correction, and greedy read-to-candidate assignment whose parsing enables the fate classification) — https://github.com/ksahlin/IsoCon
- **parasail** (Underlying SIMD pairwise aligner used by IsoCon for pairwise alignments of each read to the cluster center during error correction) — https://github.com/jeffdaily/parasail-python
- **edlib** (Fast edit-distance library used in IsoCon's read-to-candidate distance computations during clustering) — https://github.com/Martinsos/edlib

## Examples

```
IsoCon pipeline -fl_reads sim_reads_rep1.fastq -outfolder runs/rep1 && python classify_isoform_fates.py --cluster_info runs/rep1/cluster_info.tsv --candidates runs/rep1/final_candidates.fa --not_converged runs/rep1/not_converged.fa --truth simulated_isoforms.tsv
```

## Evaluation signals

- Minimum per-isoform depth at which captured isoforms appear matches reported thresholds: ~3 reads for TSPY and ~10 reads for HSFY
- Every simulated isoform in each replicate is assigned exactly one fate (captured / stat-filtered / not converged) with no missing labels
- Isoforms recorded as captured must appear in final_candidates.fa; isoforms labeled lost-in-correction must appear only in not_converged.fa
- Counts of read assignments in cluster_info.tsv sum to the number of input reads per replicate (no unassigned reads unaccounted for)
- Aggregate fate distributions across the 10 replicates per condition show lower-depth candidates predominantly lost in the error correction step rather than in statistical filtering

## Limitations

- Low-depth candidates are mostly lost in error correction, so the statistical-filter fate category is sparsely populated at very low per-isoform depths
- IsoCon does not provide definitive assignment of transcripts to individual gene copies or gene family size; downstream analysis would be needed
- Results depend on the IsoCon commit used (paper used commit 79589f3) and on reads being pre-processed with the Iso-Seq pipeline (primer/barcode removal, chimera filtering)
- Candidate transcripts with more than a threshold of variant positions relative to others (default 10) are not statistically evaluated, complicating the stat-filter fate label

## Evidence

- [discussion] Candidate filtering: greedy assignment of reads to closest-neighbor candidates and pairwise hypothesis testing: [evidence span withheld — anchor status: not_found]
- [readme] Simulated data configuration in IsoCon_Eval: ""GENE_MEMBERS" : ["HSFY2", "DAZ2", "TSPY13P"],
    "READ_COUNTS" : [ "20", "100", "500", "2500", "12500"],
    "RUNS" : ["1", "2", "3", "4", "5", "6", "7", "8", "9", "10"]"
- [readme] IsoCon output files: [evidence span withheld — anchor status: external_doc]
- [discussion] Statistical filter p-value threshold: "Then, we identify the candidates with pi greater than a significance threshold α. This α is a parameter to our algorithm, set by default to 0.01."
- [results] Depth thresholds finding for TSPY and HSFY: [evidence span withheld — anchor status: not_found]
