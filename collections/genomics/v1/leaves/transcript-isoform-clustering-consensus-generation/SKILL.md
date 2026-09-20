---
name: transcript-isoform-clustering-consensus-generation
description: Use when you have full-length non-chimeric targeted Iso-Seq FASTQ/FASTA
  reads spanning transcripts end-to-end and need nucleotide-level isoform reconstruction
  from highly similar gene copies or close paralog/viral-strain sequences without
  a reference genome.
when_to_use_negative:
- The goal is to distinguish alleles from distinct gene copies or definitively assign
  each transcript to a gene copy — IsoCon provides hypothesized assignment only and
  further downstream analysis is needed
- Reads are short reads or full-length coverage is not guaranteed (e.g., fragments
  rather than end-to-end transcripts) — IsoCon assumes reads span the region(s) of
  interest end-to-end
- The task requires general-purpose RNA-seq transcript quantification against a reference
  genome rather than de novo consensus reconstruction of targeted long-read data
edam_operation: http://edamontology.org/operation_0310
edam_topics:
- http://edamontology.org/topic_0080
- http://edamontology.org/topic_3170
tools:
- name: IsoCon
  role: 'Main tool: iterative clustering, error correction, and statistical filtering
    of Iso-Seq reads into consensus isoform candidates'
  repo: https://github.com/ksahlin/IsoCon
- name: parasail
  role: Pairwise alignment library used to align each read to the cluster center during
    the correction phase
  repo: https://github.com/jeffdaily/parasail-python
- name: edlib
  role: Edit-distance alignment dependency of IsoCon (>= 1.1.2)
  repo: https://github.com/Martinsos/edlib
- name: pysam
  role: Dependency (>= 0.11) for handling BAM/CCS inputs
- name: networkx
  role: Dependency (>= 1.10) supporting the nearest-neighbor graph clustering
- name: IsoCon_Eval
  role: Snakemake-based evaluation/benchmarking pipeline for comparing IsoCon against
    ICE on simulated data
  repo: https://github.com/ksahlin/IsoCon_Eval
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41467-018-06910-x
    title: Deciphering highly similar multigene family transcripts from Iso-Seq data
      with IsoCon
  source_content_hash: sha256:61c8872d17164c2f872b7bd579379e7133a14aad8e750b137396f73235cb717e
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ksahlin/IsoCon
  tools:
  - IsoCon
  - IsoCon_Eval
  - edlib
  - networkx
  - parasail
  - pysam
  edam_operation: http://edamontology.org/operation_0310
  edam_topics:
  - http://edamontology.org/topic_0080
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 12
---

# transcript-isoform-clustering-consensus-generation

## Summary

De novo error correction and redundancy removal of PacBio Iso-Seq circular consensus (CCS) reads using IsoCon, which iteratively clusters reads, builds consensus candidates, and applies statistical filtering (alpha = 0.01) to reconstruct transcript isoforms of highly similar gene copies without a reference genome.

## When to use

Use when you have full-length non-chimeric targeted Iso-Seq FASTQ/FASTA reads spanning transcripts end-to-end and need nucleotide-level isoform reconstruction from highly similar gene copies or close paralog/viral-strain sequences without a reference genome.

## When NOT to use

- The goal is to distinguish alleles from distinct gene copies or definitively assign each transcript to a gene copy — IsoCon provides hypothesized assignment only and further downstream analysis is needed
- Reads are short reads or full-length coverage is not guaranteed (e.g., fragments rather than end-to-end transcripts) — IsoCon assumes reads span the region(s) of interest end-to-end
- The task requires general-purpose RNA-seq transcript quantification against a reference genome rather than de novo consensus reconstruction of targeted long-read data

## Inputs

- Full-length non-chimeric (FLNC) PacBio Iso-Seq reads in FASTQ or FASTA (FASTQ preferred so quality values feed the statistical analysis)
- Optional CCS BAM file passed via --ccs flag
- Reads pre-processed with the Iso-Seq bioinformatic pipeline to remove primers, barcodes, and chimeras

## Outputs

- final_candidates.fa: predicted high-quality transcript sequences
- not_converged.fa: reads that could not be corrected or clustered (e.g., single/low-support exon-differing reads and possible chimeras)
- cluster_info.tsv: mapping of each read to its assigned candidate transcript

## How to apply

Install IsoCon (paper results use GitHub commit 79589f3) in a Python 3.8 environment with dependencies edlib >= 1.1.2, networkx >= 1.10, parasail, and pysam >= 0.11 from requirements.txt. Provide full-length non-chimeric reads in FASTQ (preferred over FASTA so quality values feed the statistical testing) and run 'IsoCon pipeline -fl_reads <reads.fastq> -outfolder <out>'. IsoCon first runs get_candidates — an iterative clustering/error-correction phase that builds a nearest-neighbor graph, picks cluster centers, aligns reads to centers with parasail, and corrects half of the lowest-support variant positions per iteration — then runs stat_filter, a greedy read-to-candidate assignment followed by pairwise Chernoff-type hypothesis testing that removes candidates with significance value above alpha (default 0.01), removing at most tau candidates with the highest p-values per iteration; candidates differing at more than 10 variant positions are not statistically evaluated. Collect final_candidates.fa (high-quality transcripts), not_converged.fa (uncorrected/low-support or possible-chimera reads), and cluster_info.tsv (read-to-candidate assignments). Validate against a known isoform set, e.g., on the FMR1 targeted Iso-Seq data IsoCon detected on average 24 isoforms per sample versus 20 for the ICE-based ToFU pipeline against 52 published isoforms.

## Related tools

- **IsoCon** (Main tool: iterative clustering, error correction, and statistical filtering of Iso-Seq reads into consensus isoform candidates) — https://github.com/ksahlin/IsoCon
- **parasail** (Pairwise alignment library used to align each read to the cluster center during the correction phase) — https://github.com/jeffdaily/parasail-python
- **edlib** (Edit-distance alignment dependency of IsoCon (>= 1.1.2)) — https://github.com/Martinsos/edlib
- **pysam** (Dependency (>= 0.11) for handling BAM/CCS inputs)
- **networkx** (Dependency (>= 1.10) supporting the nearest-neighbor graph clustering)
- **IsoCon_Eval** (Snakemake-based evaluation/benchmarking pipeline for comparing IsoCon against ICE on simulated data) — https://github.com/ksahlin/IsoCon_Eval

## Examples

```
IsoCon pipeline -fl_reads fmr1_flnc.fastq -outfolder ./out
```

## Evaluation signals

- final_candidates.fa must contain converged high-quality candidates while not_converged.fa captures only uncorrectable/low-support reads — a large fraction of reads in not_converged.fa suggests clustering failed to converge
- Every FLNC read should appear in cluster_info.tsv mapped to a candidate; missing or contradictory mappings indicate errors
- On the FMR1 benchmark, per-sample isoform recovery must exceed the ICE-based ToFU result (~24 vs ~20 isoforms per sample) and recover the ToFU-missed isoforms among the 52 published FMR1 isoforms
- Filter parameters should match paper defaults: significance threshold alpha = 0.01, max ~10 variant positions before a candidate is exempt from statistical testing, --min_test_ratio default 5
- On the bundled test dataset (IsoCon/tree/master/test/data) the pipeline should converge and produce final output files without errors

## Limitations

- Very low-abundance transcripts may be missed since deep PacBio sequencing can be cost-prohibitive
- Transcriptome-only sequencing does not definitively determine gene family size or conclusively assign each transcript to a gene copy; novel copies are difficult to distinguish from splice variants or RNA-editing
- RT-PCR error sources cannot be excluded; replicate cDNA libraries and further validation are needed
- IsoCon assumes reads are pre-processed by the Iso-Seq pipeline (primer/barcode removal, chimera filtering); uncorrectable reads, single/low-support exon-differing reads, and possible chimeras land in not_converged.fa
- Results in the paper are tied to a specific version (commit 79589f3); Python support spans 2.7 and 3.6-3.8

## Evidence

- [intro] we develop IsoCon, a de novo algorithm for error-correcting and removing redundancy of PacBio circular consensus sequence (CCS) reads: "we develop IsoCon, a de novo algorithm for error-correcting and removing redundancy of PacBio circular consensus sequence (CCS) reads"
- [readme] IsoCon pipeline command and outputs: [evidence span withheld — anchor status: not_found]
- [discussion] statistical filtering threshold and iterative removal: "Then, we identify the candidates with pi greater than a significance threshold α. This α is a parameter to our algorithm, set by default to 0.01."
- [discussion] read-to-center alignment in correction phase: "We first create pairwise alignments from each read to the center using parasail60"
- [discussion] pre-processing assumption: "IsoCon assumes the reads have been pre-processed with the Iso-Seq bioinformatic pipeline to remove primers, barcodes, and reads that are chimeras"
- [methods] input format preference and statistical analysis: "Runs only the statistical filtering of candidates."
- [discussion] filter exemption threshold: "Candidate transcripts that have more than a threshold of variant positions (default of 10) relative to all other candidate transcripts are not statistically evaluated"
- [discussion] version pinning: "The results of IsoCon in this paper were obtained with commit 79589f3 on GitHub."
