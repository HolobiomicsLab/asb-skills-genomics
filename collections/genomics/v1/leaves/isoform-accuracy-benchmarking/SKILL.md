---
name: isoform-accuracy-benchmarking
description: 'Use when comparing transcript-reconstruction tools on targeted Iso-Seq
  data and you need quantitative accuracy: simulated precision/recall vs ground-truth
  isoforms, or fraction of predicted positions supported by >=2 agreeing Illumina
  reads.'
when_to_use_negative:
- Input transcripts already have reference-genome-based validation; the article's
  methods are de novo and assume no reference
- Reads are short Illumina-only contigs rather than end-to-end Iso-Seq/CCS reads spanning
  the region of interest (IsoCon's stated assumption)
- You need to assign transcripts to specific gene copies or distinguish alleles —
  the article states transcriptome-only sequencing does not conclusively answer gene-family
  size or copy assignment
edam_operation: http://edamontology.org/operation_2928
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0622
tools:
- name: IsoCon
  role: Method under benchmark; reconstructs transcripts from full-length reads producing
    final_candidates.fa
  repo: https://github.com/ksahlin/IsoCon
- name: ICE
  role: Baseline comparator (PacBio Iso-Seq clustering/correction tool) run on identical
    simulated read sets
- name: IsoCon_Eval
  role: Snakemake workflow and evaluation scripts driving simulations, tool runs,
    and precision/recall scoring
  repo: https://github.com/ksahlin/IsoCon_Eval
- name: parasail
  role: Pairwise alignment library used by IsoCon for read-to-center alignments underlying
    consensus correction
  repo: https://github.com/jeffdaily/parasail-python
provenance:
  source_task_ids:
  - task_004
  - task_001
  source_papers:
  - doi: 10.1038/s41467-018-06910-x
    title: Deciphering highly similar multigene family transcripts from Iso-Seq data
      with IsoCon
  source_content_hash: sha256:f5bad6542d4a9a3caebbe79ea48e76e1f9b74c52585e67b6b8405ed9c14bcb7f
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ksahlin/IsoCon
  tools:
  - ICE
  - Iso-Seq
  - IsoCon
  - IsoCon_Eval
  - parasail
  edam_operation: http://edamontology.org/operation_2928
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 18
---

# isoform-accuracy-benchmarking

## Summary

Benchmark de novo transcript reconstruction methods (IsoCon vs ICE vs Illumina-corrected/original CCS) by scoring nucleotide-level precision and recall against simulated ground truth, or by measuring Illumina-read support of predicted positions in experimental data.

## When to use

Use when comparing transcript-reconstruction tools on targeted Iso-Seq data and you need quantitative accuracy: simulated precision/recall vs ground-truth isoforms, or fraction of predicted positions supported by >=2 agreeing Illumina reads.

## When NOT to use

- Input transcripts already have reference-genome-based validation; the article's methods are de novo and assume no reference
- Reads are short Illumina-only contigs rather than end-to-end Iso-Seq/CCS reads spanning the region of interest (IsoCon's stated assumption)
- You need to assign transcripts to specific gene copies or distinguish alleles — the article states transcriptome-only sequencing does not conclusively answer gene-family size or copy assignment

## Inputs

- Simulated full-length PacBio Iso-Seq read sets (FASTA/FASTQ) for TSPY, HSFY, DAZ with known ground-truth isoforms
- Predicted transcript sets: IsoCon final_candidates.fa, ICE consensus transcripts, Illumina-corrected PacBio reads, original uncorrected CCS reads
- Illumina Nextera XT MiSeq reads (one indexed library per primer pair-sample combination)

## Outputs

- Per-method, per-depth precision and recall values and grouped violin plots vs read depth
- support_fractions.tsv with per-approach supported-position fractions and fully-supported transcript fractions

## How to apply

For simulated benchmarks, generate PacBio Iso-Seq read sets for gene families such as TSPY, HSFY and DAZ with 30 isoforms, unequal abundances, three mutation rates, and read counts spanning ~20 to 12500 full-length reads; run each tool (e.g. 'IsoCon pipeline -fl_reads <reads> -outfolder <output>' producing final_candidates.fa, plus ICE) on identical inputs and score nucleotide-level precision and recall against the simulated truth using the IsoCon_Eval Snakemake pipeline (config simulated_experiments.json). For experimental validation, align Illumina Nextera XT MiSeq reads to each set of predicted transcripts and count positions covered by at least 2 independent Illumina reads agreeing with the predicted base; report both the overall supported-position fraction and the fraction of transcripts fully supported. Decision point: report precision separately per read depth, since ICE precision collapses to near 0 at depth >=500 while IsoCon stays above 80%.

## Related tools

- **IsoCon** (Method under benchmark; reconstructs transcripts from full-length reads producing final_candidates.fa) — https://github.com/ksahlin/IsoCon
- **ICE** (Baseline comparator (PacBio Iso-Seq clustering/correction tool) run on identical simulated read sets)
- **IsoCon_Eval** (Snakemake workflow and evaluation scripts driving simulations, tool runs, and precision/recall scoring) — https://github.com/ksahlin/IsoCon_Eval
- **parasail** (Pairwise alignment library used by IsoCon for read-to-center alignments underlying consensus correction) — https://github.com/jeffdaily/parasail-python

## Examples

```
IsoCon pipeline -fl_reads <flnc.fastq> -outfolder <output>
```

## Evaluation signals

- ICE precision drops to near 0 at read depth >=500 while IsoCon precision remains above 80% in regenerated violin plots
- Supported-position fractions match reported values: 99% (IsoCon), 93% (ICE), 96% (Illumina-corrected), 79% (original CCS)
- Fully-supported transcript fractions match: 70%, 2%, 15%, and 20% respectively
- IsoCon transcripts in final_candidates.fa match the published reconstruction output
- IsoCon code commit 79589f3 is used, matching the paper's stated version

## Limitations

- Simulated benchmark results depend on the chosen mutation rates, abundance distributions (constant vs exponential), and read counts configured in simulated_experiments.json
- Illumina >=2-agreeing-read support is a proxy for accuracy, not ground truth; RT-PCR error sources cannot be excluded
- Very low-abundance transcripts might not be captured by Iso-Seq, since deep PacBio sequencing can be cost-prohibitive

## Evidence

- [intro] Using simulated data, we demonstrate that IsoCon has substantially higher precision and recall than ICE18 across a wide range of sequencing depths: "Using simulated data, we demonstrate that IsoCon has substantially higher precision and recall than ICE18 across a wide range of sequencing depths"
- [results] IsoCon averaged 99% supported positions: [evidence span withheld — anchor status: not_found]
- [discussion] The results of IsoCon in this paper were obtained with commit 79589f3 on GitHub: "The results of IsoCon in this paper were obtained with commit 79589f3 on GitHub."
- [discussion] Scripts for all analyses are available at https://github.com/ksahlin/IsoCon_Eval. This repository also include a snakemake63 workflow: "Scripts for all analyses are available at https://github.com/ksahlin/IsoCon_Eval. This repository also include a snakemake63 workflow"
- [readme] predicted transcripts are found in file /path/to/output/final_candidates.fa: "predicted transcripts are found in file **/path/to/output/final_candidates.fa**"
