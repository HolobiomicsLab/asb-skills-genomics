---
name: ampliconic-gene-family-transcript-analysis
description: Use when simulating targeted Iso-Seq data of near-identical multigene-family
  transcripts to determine at what sequencing depth (total and per-isoform read counts)
  a de novo corrector like IsoCon reconstructs isoforms with high precision.
when_to_use_negative:
- Input is experimental (not simulated) Iso-Seq data where isoform ground truth is
  unknown, so per-isoform capture/fate classification cannot be validated against
  truth transcripts
- The analysis goal is comparing IsoCon against ICE in precision/recall across sequencing
  depths rather than isolating the per-isoform depth threshold of a single tool
- Reads are short or untargeted and do not span the transcript end-to-end, violating
  IsoCon's assumption that reads cover the region of interest
edam_operation: http://edamontology.org/operation_3731
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0196
tools:
- name: IsoCon
  role: De novo error correction and candidate generation on each simulated read set;
    run with 'IsoCon pipeline -fl_reads ... -outfolder ...' to yield final_candidates.fa,
    not_converged.fa, and cluster_info.tsv
  repo: https://github.com/ksahlin/IsoCon
- name: IsoCon_Eval
  role: Snakemake pipeline that generates the simulated Iso-Seq read sets (10 replicates,
    depth series 20–12500, constant/exponential abundance) and orchestrates the depth
    experiments
  repo: https://github.com/ksahlin/IsoCon_Eval
- name: parasail
  role: SIMD pairwise alignment library used inside IsoCon's correction phase to align
    reads to cluster centers
  repo: https://github.com/jeffdaily/parasail-python
- name: PacBio Iso-Seq
  role: Sequencing protocol whose targeted CCS reads the simulations and IsoCon pre-processing
    assumptions are modeled on
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-018-06910-x
    title: Deciphering highly similar multigene family transcripts from Iso-Seq data
      with IsoCon
  source_content_hash: sha256:35345e5a05f2f19b0620286a44c4bc47a5acfa6dafd659a473fdc94ab4d24880
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ksahlin/IsoCon
  tools:
  - IsoCon
  - IsoCon_Eval
  - PacBio Iso-Seq
  - parasail
  edam_operation: http://edamontology.org/operation_3731
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0196
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9090909090909091
n_claims: 11
---

# ampliconic-gene-family-transcript-analysis

## Summary

A simulation-driven capture-power analysis for transcripts of highly similar ampliconic gene families (e.g., TSPY, HSFY): run IsoCon on simulated PacBio Iso-Seq read sets across total and per-isoform depths and classify each isoform's fate (captured, filtered statistically, lost in error correction) to identify the per-isoform depth threshold for reliable reconstruction.

## When to use

Use when simulating targeted Iso-Seq data of near-identical multigene-family transcripts to determine at what sequencing depth (total and per-isoform read counts) a de novo corrector like IsoCon reconstructs isoforms with high precision.

## When NOT to use

- Input is experimental (not simulated) Iso-Seq data where isoform ground truth is unknown, so per-isoform capture/fate classification cannot be validated against truth transcripts
- The analysis goal is comparing IsoCon against ICE in precision/recall across sequencing depths rather than isolating the per-isoform depth threshold of a single tool
- Reads are short or untargeted and do not span the transcript end-to-end, violating IsoCon's assumption that reads cover the region of interest

## Inputs

- Simulated PacBio Iso-Seq CCS read sets (FASTQ/FASTA) per gene member, depth, abundance model, and replicate (10 replicates per condition)
- simulated_experiments. config specifying GENE_MEMBERS, READ_COUNTS, RUNS, ABUNDANCE, TOOLS, and paths
- cluster_info.tsv with read-to-candidate cluster assignments from each IsoCon run
- final_candidates.fa and not_converged.fa from each IsoCon run

## Outputs

- Per-isoform fate labels (captured / filtered in statistical step / lost in error correction) aggregated across replicates
- Seaborn stripplot of isoform fate versus experiment depth and per-isoform depth (fig_s1_isoform_capture_stripplot.png)
- Estimated per-isoform capture-depth thresholds (≈3 reads for TSPY, ≈10 reads for HSFY)

## How to apply

Generate simulated PacBio Iso-Seq read sets for representative gene members (e.g., HSFY2, DAZ2, TSPY13P) using the IsoCon_Eval snakemake pipeline configured via simulated_experiments., with READ_COUNTS spanning 20–12500 reads, ABUNDANCE modes constant and exponential, and 10 replicate runs per condition. Run 'IsoCon pipeline -fl_reads <reads> -outfolder <out>' on each simulated set (or restrict tools via the config's TOOLS field) to produce final_candidates.fa, not_converged.fa, and cluster_info.tsv. Use cluster_info.tsv together with per-isoform read counts to classify each simulated isoform as captured (present in final_candidates.fa), removed by the statistical filtering step, or lost during error correction (present only in not_converged.fa). Aggregate fates across replicates as a function of experiment depth and per-isoform depth, then plot isoform fate versus both depth axes with a seaborn stripplot (e.g., fig_s1_isoform_capture_stripplot.png). Validation relies on the minimum per-isoform depth at which isoforms appear in final_candidates.fa matching the reported capture thresholds (~3 reads for TSPY, ~10 reads for HSFY), confirming that depth—not mutation rate—is the dominant factor and that low-depth candidates are mostly lost in the error-correction step.

## Related tools

- **IsoCon** (De novo error correction and candidate generation on each simulated read set; run with 'IsoCon pipeline -fl_reads ... -outfolder ...' to yield final_candidates.fa, not_converged.fa, and cluster_info.tsv) — https://github.com/ksahlin/IsoCon
- **IsoCon_Eval** (Snakemake pipeline that generates the simulated Iso-Seq read sets (10 replicates, depth series 20–12500, constant/exponential abundance) and orchestrates the depth experiments) — https://github.com/ksahlin/IsoCon_Eval
- **parasail** (SIMD pairwise alignment library used inside IsoCon's correction phase to align reads to cluster centers) — https://github.com/jeffdaily/parasail-python
- **PacBio Iso-Seq** (Sequencing protocol whose targeted CCS reads the simulations and IsoCon pre-processing assumptions are modeled on)

## Examples

```
IsoCon pipeline -fl_reads sim_reads.fastq -outfolder out/ && snakemake --configfile simulated_experiments. --latency-wait 100 --verbose -n
```

## Evaluation signals

- Minimum per-isoform depth at which simulated isoforms appear in final_candidates.fa is approximately 3 reads for TSPY and approximately 10 reads for HSFY
- Every simulated isoform across all replicates is assigned exactly one fate (captured / statistically filtered / lost in correction) with no unclassified isoforms
- cluster_info.tsv read counts reconcile with per-isoform depths and the partition of reads across final_candidates.fa and not_converged.fa
- Capture rates depend primarily on per-isoform depth, with mutation rate showing only a minor effect (replicating the paper's finding)
- Snakemake dry run (snakemake -n --configfile simulated_experiments.) resolves all jobs before launching, confirming config paths and experiment fields are valid

## Limitations

- IsoCon assumes reads are pre-processed with the Iso-Seq bioinformatic pipeline (primer/barcode removal, chimera filtering); unprocessed reads invalidate results
- Very low-abundance transcripts may be missed since deep PacBio sequencing is cost-prohibitive — the depth threshold quantifies this limit rather than removing it
- Transcriptome-only sequencing does not provide a definitive gene family size or conclusive per-copy assignment; results are hypothesized copy assignments

## Evidence

- [readme] predicted transcripts are found in file **/path/to/output/final_candidates.fa**: "predicted transcripts are found in file **/path/to/output/final_candidates.fa**"
- [readme] IsoCon pipeline CLI: "IsoCon pipeline -fl_reads <reads.fastq> -outfolder </path/to/output>"
- [discussion] parasail alignment in correction: "We first create pairwise alignments from each read to the center using parasail60"
- [discussion] Iso-Seq pre-processing assumption: "IsoCon assumes the reads have been pre-processed with the Iso-Seq bioinformatic pipeline to remove primers, barcodes, and reads that are chimeras"
- [readme] IsoCon_Eval simulation config fields: [evidence span withheld — anchor status: external_doc]
