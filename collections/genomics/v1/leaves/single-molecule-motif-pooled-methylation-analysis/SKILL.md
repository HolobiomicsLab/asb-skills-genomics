---
name: single-molecule-motif-pooled-methylation-analysis
description: Use when you have long (10Kb+) SMRT library preps where each subread
  spans many distinct motif sites, and you need per-molecule methylation calls (SMp
  scores) to detect subpopulations of cells with active/inactive phase-variable MTases
  or intracellular stochastic methylation.
when_to_use_negative:
- Short (~250bp) sequencing library preps where each motif site is covered by many
  passes — use the SMsn single-molecule, single-nucleotide protocol instead, since
  SMp reliability increases with the number of distinct motif sites per subread
- You only need population-level consensus methylome calls without single-molecule
  resolution — SMALR is explicitly designed to address the limitation that consensus-based
  methods lack the resolution to observe epigenetic heterogeneity
- Detecting modifications whose kinetic signatures extend into surrounding bases (e.g.
  some non-canonically positioned modifications) — the article states these are beyond
  the scope of the current algorithms
edam_operation: http://edamontology.org/operation_2409
edam_topics:
- http://edamontology.org/topic_3168
- http://edamontology.org/topic_0157
tools:
- name: SMALR
  role: Core pipeline; run in SMp mode to pool all motif sites on a molecule and score
    molecule-wide methylation from native vs WGA IPD kinetics
  repo: https://github.com/fanglab/smalr
- name: Python 2.7
  role: Runtime dependency and scripting for aggregating per-contig outputs and plotting
    SMp score distributions
  repo: https://github.com/fanglab/smalr
- name: samtools
  role: Dependency (>= 0.1.18) for handling BAM alignments of native and WGA reads
- name: R
  role: Dependency (>= 3.0) for statistical analysis/plotting of SMp distributions
provenance:
  source_task_ids:
  - task_004
  - task_007
  source_papers:
  - doi: 10.1038/ncomms8438
    title: Single molecule-level detection and long read-based phasing of epigenetic
      variations in bacterial methylomes
  source_content_hash: sha256:9eab359f47bddd0dc038c3bb187044279cc896d6bd60b6881048ec23136b0fae
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/fanglab/smalr
  tools:
  - Python
  - Python 2.7
  - R
  - SMALR
  - samtools
  edam_operation: http://edamontology.org/operation_2409
  edam_topics:
  - http://edamontology.org/topic_3168
  - http://edamontology.org/topic_0157
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# single-molecule-motif-pooled-methylation-analysis

## Summary

SMALR's SMp protocol pools all motif sites on each long SMRT sequencing molecule and scores molecule-wide methylation status for a given motif, using the subread-normalized ln(IPD) native score minus the WGA (unmethylated control) score. It detects per-molecule epigenetic heterogeneity—phase-variable or stochastic MTase activity—in bacterial methylomes that population-consensus methods miss.

## When to use

Use when you have long (10Kb+) SMRT library preps where each subread spans many distinct motif sites, and you need per-molecule methylation calls (SMp scores) to detect subpopulations of cells with active/inactive phase-variable MTases or intracellular stochastic methylation.

## When NOT to use

- Short (~250bp) sequencing library preps where each motif site is covered by many passes — use the SMsn single-molecule, single-nucleotide protocol instead, since SMp reliability increases with the number of distinct motif sites per subread
- You only need population-level consensus methylome calls without single-molecule resolution — SMALR is explicitly designed to address the limitation that consensus-based methods lack the resolution to observe epigenetic heterogeneity
- Detecting modifications whose kinetic signatures extend into surrounding bases (e.g. some non-canonically positioned modifications) — the article states these are beyond the scope of the current algorithms

## Inputs

- Native SMRT long-read alignments (aligned_reads.cmph5 or BAM) with IPD kinetics
- Matching WGA (whole-genome-amplified, unmethylated control) alignments (aligned_reads.cmp.h5 or BAM)
- Reference genome FASTA
- SMALR inputs_file (native_cmph5, wga_cmph5, ref, fastq=NONE lines)
- Known methylation motif and modified-base position (e.g. --motif=GATC --mod_pos=2)

## Outputs

- Per-molecule SMp score distributions per motif (native score minus WGA score)
- SMp.out nine-column table per contig: contig strand, motif position, SMp score, molecule ID, native score, WGA score, native/WGA data-point counts, mean subread length
- Per-contig output directories with a log detailing motif positions (forward and reverse strand)
- IPD-shuffled negative-control score distribution
- Classification of methylation heterogeneity (uniform vs phase-variable vs stochastic)

## How to apply

Build an inputs_file pointing to the native long-read alignments (native_cmph5; cmp.h5 or BAM), the WGA control alignments (wga_cmph5), and the reference FASTA, specifying NONE for the deprecated fastq path. Run SMALR in SMp mode per motif (e.g. 'smalr -i --SMp --motif=GATC --mod_pos=2 --procs=4 -c 5 input.txt'), applying default filters: per-molecule/strand native coverage threshold (--nativeCovThresh, 10), aggregate WGA coverage threshold (--wgaCovThresh, 10), minimum subread length (--minSubreadLen, 100), and minimum subread accuracy (--minAcc, 0.8). For each molecule, pool subread-normalized ln(IPD) values across all motif sites it spans to compute the native score, then subtract the aggregate WGA score to obtain the SMp score per molecule. Generate an IPD-shuffled negative control by permuting IPD values across positions before scoring to establish the background distribution. Aggregate per-contig output directories into per-motif score tables, then plot SMp score distributions to classify heterogeneity: a unimodal distribution indicates uniform methylation (e.g. H. pylori J99 GATC, ~0.07% non-methylated fraction), an SMP≈2 subpopulation indicates a minor wild-type subpopulation with active phase-variable MTase (GWCAY/M.Hpy99XXI), and a peak near SMP≈0.9 indicates intracellular stochastic methylation (C. salexigens RGATCY, ~60% M.CsaI activity).

## Related tools

- **SMALR** (Core pipeline; run in SMp mode to pool all motif sites on a molecule and score molecule-wide methylation from native vs WGA IPD kinetics) — https://github.com/fanglab/smalr
- **Python 2.7** (Runtime dependency and scripting for aggregating per-contig outputs and plotting SMp score distributions) — https://github.com/fanglab/smalr
- **samtools** (Dependency (>= 0.1.18) for handling BAM alignments of native and WGA reads)
- **R** (Dependency (>= 3.0) for statistical analysis/plotting of SMp distributions)

## Examples

```
smalr -i --SMp --motif=GATC --mod_pos=2 --procs=4 -c 5 input.txt
```

## Evaluation signals

- SMp.out exists and contains the documented nine-column header (contig strand, motif position, SMp score, molecule ID, native score, WGA score, native/WGA data-point counts, mean subread length)
- IPD-shuffled negative control yields a background SMp score distribution against which the real distribution is judged
- Fully methylated motif control (e.g. H. pylori J99 GATC) gives a unimodal distribution with ~0.07% non-methylated fraction
- Phase-variable MTase case reproduces a subpopulation centered at SMP~2 (GWCAY in H. pylori J99); stochastic case peaks near SMP~0.9 (C. salexigens RGATCY) with methylated fraction consistent with ~60% M.CsaI activity
- Install check: both SMsn and SMp test protocols complete successfully after setup

## Limitations

- SMp reliability increases with the number of distinct motif sites contained in a subread, so motifs that are sparse in the genome or short subreads yield unreliable scores
- Requires a matched WGA (unmethylated) control alignment to compute the SMp score
- Detection depends on the number of repeated observations per single molecule; for a given read length there is a tradeoff between library size and single-molecule accuracy
- Modifications creating kinetic signatures extending into surrounding bases are beyond the scope of the current work and would require more sophisticated algorithms
- Pipeline depends on Python 2.7 and legacy SMRT formats (cmp.h5); the fastq input path is deprecated (specify NONE)

## Evidence

- [intro] SMp pools all motif sites on a molecule and assesses molecule-wide methylation status for long 10Kb+ preps: "All motif sites on a sequencing molecule are pooled together and the molecule-wide methylation status for the given motif is assessed."
- [readme] SMp designed for long 10Kb+ preps where each subread spans many distinct motif sites: "This is designed for use with long (10Kb+) sequencing library preps, where each single long subread can span many distinct motif sites."
- [methods] Filter thresholds: native coverage 10, WGA coverage 10, min subread length 100: "Per mol/strand coverage threshold below which to ignore molecules [10]"
- [discussion] SMP scores reveal small cell fractions with active/inactive MTases and estimate processivity: "the SMP scores can reveal small fractions of cells in a bacterial population that contain active or inactive MTases, and quantitatively estimate the processivity of an MTase of interest."
- [abstract] Prior consensus-based SMRT methods lack single-cell resolution needed to see epigenetic heterogeneity: "existing SMRT sequencing-based methods for studying bacterial methylomes rely on a population-level consensus that lacks the single-cell resolution required to observe epigenetic heterogeneity"
