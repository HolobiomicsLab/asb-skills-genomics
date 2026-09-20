---
name: multigene-family-transcript-simulation
description: Use when you need ground-truth simulated long-read transcript data from
  highly similar gene copies to evaluate de novo error-correction or isoform reconstruction
  tools at varying read depths and abundance profiles.
when_to_use_negative:
- The reads are short Illumina reads rather than long PacBio Iso-Seq reads spanning
  the transcript end-to-end — error correction approaches for short reads cannot leverage
  the linked variant structure IsoCon relies on
- The goal is reference-based isoform quantification rather than de novo reconstruction
  evaluation without a reference genome
- A real experimental dataset with no ground-truth isoform set is used — precision/recall
  against simulated truth is not computable
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0654
tools:
- name: IsoCon
  role: De novo error-correction and redundancy removal of PacBio CCS reads to reconstruct
    transcript candidates from simulated read sets (paper commit 79589f3)
  repo: https://github.com/ksahlin/IsoCon
- name: ICE
  role: PacBio's standard Iso-Seq clustering/correction baseline comparator run on
    identical simulated read sets
- name: IsoCon_Eval
  role: Snakemake workflow and evaluation scripts driving the simulations, ICE comparison,
    and precision/recall scoring
  repo: https://github.com/ksahlin/IsoCon_Eval
- name: parasail
  role: Pairwise alignments of reads to cluster centers during the correction phase
  repo: https://github.com/jeffdaily/parasail-python
- name: edlib
  role: Fast edit-distance alignment library used by IsoCon
  repo: https://github.com/Martinsos/edlib
- name: Snakemake
  role: Workflow engine distributing the simulation and evaluation jobs (cluster execution
    via sbatch)
  repo: http://snakemake.readthedocs.io
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41467-018-06910-x
    title: Deciphering highly similar multigene family transcripts from Iso-Seq data
      with IsoCon
  source_content_hash: sha256:c9318df9b11230885f960eba75631ca5d6e65922ad2b8f2f7a85126b4b859500
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ksahlin/IsoCon
  tools:
  - ICE
  - Iso-Seq
  - IsoCon
  - IsoCon_Eval
  - Snakemake
  - edlib
  - parasail
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0654
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# multigene-family-transcript-simulation

## Summary

Simulates PacBio Iso-Seq read sets for highly similar multigene family transcripts (e.g., TSPY13P, HSFY2, DAZ2) with controlled isoform counts, unequal abundances, mutation rates, and read depths, to benchmark transcript reconstruction methods like IsoCon against ICE using nucleotide-level precision and recall.

## When to use

Use when you need ground-truth simulated long-read transcript data from highly similar gene copies to evaluate de novo error-correction or isoform reconstruction tools at varying read depths and abundance profiles.

## When NOT to use

- The reads are short Illumina reads rather than long PacBio Iso-Seq reads spanning the transcript end-to-end — error correction approaches for short reads cannot leverage the linked variant structure IsoCon relies on
- The goal is reference-based isoform quantification rather than de novo reconstruction evaluation without a reference genome
- A real experimental dataset with no ground-truth isoform set is used — precision/recall against simulated truth is not computable

## Inputs

- Simulated multigene family transcript sets (TSPY, HSFY, DAZ gene families, 30 isoforms each, unequal abundances, three mutation rates)
- Simulated PacBio Iso-Seq full-length read sets (FASTQ/FASTA) at read depths 20, 100, 500, 2500, 12500 with 10 replicate runs
- Config file simulated_experiments. for the Snakemake pipeline (EXPERIMENTS, GENE_MEMBERS, READ_COUNTS, RUNS, ABUNDANCE, TOOLS fields)

## Outputs

- final_candidates.fa — reconstructed transcript candidates from IsoCon
- Nucleotide-level precision and recall scores per method, family, mutation rate, and read depth
- Grouped violin plots of precision and recall as a function of read depth for IsoCon versus ICE

## How to apply

Use the IsoCon_Eval Snakemake workflow, configuring simulated_experiments. with gene members (e.g., HSFY2, DAZ2, TSPY13P), read counts (20, 100, 500, 2500, 12500), replicate runs (1–10), abundance models (constant or exponential), and tools (ISOCON, ICE). Generate simulated read sets containing ~30 isoforms per gene family with unequal transcript abundances and multiple mutation rates. Reconstruct transcripts with 'IsoCon pipeline -fl_reads <reads> -outfolder <output>' (yielding final_candidates.fa) and run ICE on the identical read sets as the baseline. Score both methods against the simulated ground truth, computing nucleotide-level precision and recall per family, mutation rate, and read depth, then render grouped violin plots of precision/recall versus depth. The design rationale is that high gene-copy similarity makes error overcorrection the dominant failure mode, so a cautious iterative correction plus statistical candidate filtering (p-value threshold alpha = 0.01, max ~10 variant positions per statistical test) should preserve rare variants that ICE collapses.

## Related tools

- **IsoCon** (De novo error-correction and redundancy removal of PacBio CCS reads to reconstruct transcript candidates from simulated read sets (paper commit 79589f3)) — https://github.com/ksahlin/IsoCon
- **ICE** (PacBio's standard Iso-Seq clustering/correction baseline comparator run on identical simulated read sets)
- **IsoCon_Eval** (Snakemake workflow and evaluation scripts driving the simulations, ICE comparison, and precision/recall scoring) — https://github.com/ksahlin/IsoCon_Eval
- **parasail** (Pairwise alignments of reads to cluster centers during the correction phase) — https://github.com/jeffdaily/parasail-python
- **edlib** (Fast edit-distance alignment library used by IsoCon) — https://github.com/Martinsos/edlib
- **Snakemake** (Workflow engine distributing the simulation and evaluation jobs (cluster execution via sbatch)) — http://snakemake.readthedocs.io

## Examples

```
snakemake --configfile simulated_experiments. --latency-wait 100 --verbose -n ; IsoCon pipeline -fl_reads reads.fastq -outfolder ./output
```

## Evaluation signals

- On simulated data at read depth >= 500, ICE's precision drops close to 0 while IsoCon's precision remains above 80% — reproduce this headline separation in the precision-vs-depth violin plots
- Recall is also substantially higher for IsoCon than ICE across the range of sequencing depths, transcript lengths, similarities, and abundance levels
- The reconstructed IsoCon transcripts in final_candidates.fa match the published reconstruction output on the test dataset
- Configured experiment grid (GENE_MEMBERS x READ_COUNTS x RUNS x ABUNDANCE x TOOLS) completes for all 10 runs without dropped jobs; verify with a snakemake dry run ('snakemake -n --configfile simulated_experiments.')

## Limitations

- Simulations do not address allele-vs-copy ambiguity: IsoCon provides no functionality to distinguish alleles from distinct gene copies, requiring further downstream analysis
- Transcriptome-only sequencing cannot conclusively determine gene family size or per-transcript gene-copy assignment, so simulated gene-copy assignment remains hypothetical
- Very low-abundance transcripts might not be captured since deep PacBio sequencing is cost-prohibitive; simulated abundances may not capture this experimental floor
- RT-PCR and library-preparation error sources present in real data are not simulated, so precision in real targeted experiments may differ

## Evidence

- [intro] Using simulated data, we demonstrate that IsoCon has substantially higher precision and recall than ICE18 across a wide range of sequencing depths: "Using simulated data, we demonstrate that IsoCon has substantially higher precision and recall than ICE18 across a wide range of sequencing depths, as well as of transcript lengths, similarities and"
- [readme] Genes and read counts: ""GENE_MEMBERS" : ["HSFY2", "DAZ2", "TSPY13P"],
    "READ_COUNTS" : [ "20", "100", "500", "2500", "12500"],
    "RUNS" : ["1", "2", "3", "4", "5", "6", "7", "8", "9", "10"]"
- [readme] Snakemake workflow: "Scripts for all analyses are available at https://github.com/ksahlin/IsoCon_Eval. This repository also include a snakemake63 workflow"
- [readme] Pipeline invocation: "IsoCon pipeline -fl_reads <reads.fastq> -outfolder </path/to/output>"
- [discussion] Statistical filtering threshold: "Then, we identify the candidates with pi greater than a significance threshold α. This α is a parameter to our algorithm, set by default to 0.01."
- [discussion] Commit pinning: "IsoCon is open-source and freely available at https://github.com/ksahlin/IsoCon. The results of IsoCon in this paper were obtained with commit 79589f3 on GitHub."
