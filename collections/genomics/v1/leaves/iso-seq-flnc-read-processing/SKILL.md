---
name: iso-seq-flnc-read-processing
description: Use when you have FLNC CCS reads (FASTQ/FASTA) from targeted Iso-Seq
  sequencing of highly similar multigene family copies and need nucleotide-level-precision
  end-to-end transcripts de novo, without a reference genome.
when_to_use_negative:
- Input reads are unprocessed PacBio subreads that still contain primers, barcodes,
  or chimeras — IsoCon assumes Iso-Seq pre-processing has already been done
- You need to distinguish alleles of distinct gene copies or conclusively assign transcripts
  to individual gene copies — downstream analysis beyond IsoCon is required and transcriptome-only
  data cannot answer gene family size
- Short-read (Illumina) data alone is the input — Illumina-based correction (e.g.
  proovread) is unable to correct most errors in this setting; long reads spanning
  the region end-to-end are required
edam_operation: http://edamontology.org/operation_3218
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_3308
tools:
- name: IsoCon
  role: 'Main pipeline: de novo error correction, clustering, and statistical filtering
    of FLNC reads to derive candidate transcripts (final_candidates.fa)'
  repo: https://github.com/ksahlin/IsoCon
- name: ICE
  role: Comparison/baseline error-correction tool; its transcripts can be imported
    into IsoCon's stat_filter for equivalent statistical evaluation
- name: parasail
  role: Pairwise alignments from each read to the cluster center during the error-correction
    phase
  repo: https://github.com/jeffdaily/parasail-python
- name: edlib
  role: Pairwise alignment library used in downstream evaluation (e.g., counting exact
    matches to Ensembl coding sequences)
  repo: https://github.com/Martinsos/edlib
- name: PacBio Iso-Seq pipeline
  role: 'Upstream pre-processing: primer/barcode removal, chimera filtering, splitting
    reads by primer pair'
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41467-018-06910-x
    title: Deciphering highly similar multigene family transcripts from Iso-Seq data
      with IsoCon
  source_content_hash: sha256:6dee0762551a2e904e78ccfc722d528cd17010ed0f42a0a20e299a4e8a7f6208
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ksahlin/IsoCon
  tools:
  - ICE
  - IsoCon
  - PacBio Iso-Seq pipeline
  - edlib
  - parasail
  edam_operation: http://edamontology.org/operation_3218
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 18
---

# iso-seq-flnc-read-processing

## Summary

Reconstructs finished transcript sequences from FLNC (full-length non-chimeric) PacBio Iso-Seq reads using IsoCon, a de novo error-correction and redundancy-removal algorithm for highly similar gene copies, with quality-aware statistical filtering of candidate transcripts.

## When to use

Use when you have FLNC CCS reads (FASTQ/FASTA) from targeted Iso-Seq sequencing of highly similar multigene family copies and need nucleotide-level-precision end-to-end transcripts de novo, without a reference genome.

## When NOT to use

- Input reads are unprocessed PacBio subreads that still contain primers, barcodes, or chimeras — IsoCon assumes Iso-Seq pre-processing has already been done
- You need to distinguish alleles of distinct gene copies or conclusively assign transcripts to individual gene copies — downstream analysis beyond IsoCon is required and transcriptome-only data cannot answer gene family size
- Short-read (Illumina) data alone is the input — Illumina-based correction (e.g. proovread) is unable to correct most errors in this setting; long reads spanning the region end-to-end are required

## Inputs

- FLNC reads from targeted PacBio Iso-Seq sequencing, ideally FASTQ (quality values used by IsoCon's statistical framework); FASTA also accepted
- Optional CCS BAM file (--ccs) when starting from polymerase reads
- Pre-processed reads with primers, barcodes, and chimeras removed by the Iso-Seq pipeline

## Outputs

- Predicted transcript sequences in /path/to/output/final_candidates.fa
- Reads that could not be corrected or clustered (unclassified reads output)
- Candidate transcripts from get_candidates mode (error correction only, FASTA)

## How to apply

First pre-process reads with the Iso-Seq bioinformatic pipeline to remove primers, barcodes, and chimeras, since IsoCon assumes this pre-processing. Run the full pipeline on per-sample FLNC reads with 'IsoCon pipeline -fl_reads <reads.fastq> -outfolder </path/to/output>'; FASTQ is preferred over FASTA because IsoCon uses quality values for its statistical analysis, and CCS BAM can be supplied via --ccs. IsoCon performs iterative clustering and error correction (pairwise alignments to a cluster center via parasail, correction of the half of variant positions with lowest support), then statistically filters candidates by pairwise hypothesis testing using a Chernoff-type upper bound — candidates with significance value p > alpha (default 0.01) are removed iteratively, at most tau candidates with highest p-values per iteration; candidates differing at more than 10 variant positions are not statistically evaluated, and --min_test_ratio (default 5) omits testing a candidate against one with 5x less support. Predicted transcripts are output in final_candidates.fa; the stat_filter mode can be run alone to statistically filter candidate transcript sets imported from external methods such as ICE. Results in the paper were obtained with IsoCon commit 79589f3.

## Related tools

- **IsoCon** (Main pipeline: de novo error correction, clustering, and statistical filtering of FLNC reads to derive candidate transcripts (final_candidates.fa)) — https://github.com/ksahlin/IsoCon
- **ICE** (Comparison/baseline error-correction tool; its transcripts can be imported into IsoCon's stat_filter for equivalent statistical evaluation)
- **parasail** (Pairwise alignments from each read to the cluster center during the error-correction phase) — https://github.com/jeffdaily/parasail-python
- **edlib** (Pairwise alignment library used in downstream evaluation (e.g., counting exact matches to Ensembl coding sequences)) — https://github.com/Martinsos/edlib
- **PacBio Iso-Seq pipeline** (Upstream pre-processing: primer/barcode removal, chimera filtering, splitting reads by primer pair)

## Examples

```
IsoCon pipeline -fl_reads testis_sample1_flnc.fastq -outfolder out/iscon_sample1
```

## Evaluation signals

- final_candidates.fa exists in the output folder and contains converged candidate transcripts after statistical filtering
- Parameter sanity: default alpha=0.01 significance threshold, tau candidates removed per iteration, >10-variant candidates untested, --min_test_ratio=5 — deviations should be deliberate and documented
- FASTQ input used (not FASTA) so quality values feed the statistical framework; missing quality values degrade the hypothesis testing
- Benchmark invariants from the paper: on two testes samples IsoCon shares 121 transcripts (~32%) across samples vs <2% for ICE/original reads, and yields 21 exact matches to the 61 unique Ensembl coding sequences vs 8 for ICE
- Version pinning: paper results were obtained with IsoCon commit 79589f3; reproducibility checks should record the commit used

## Limitations

- IsoCon cannot distinguish different alleles from distinct gene copies; further downstream analysis is needed for allele-to-copy assignment
- Transcriptome-only sequencing does not provide a definitive answer to the size of each gene family or conclusive gene-copy assignment
- Novel gene copies are difficult to distinguish from splice variants or RNA-editing
- Very low-abundance transcripts might not be captured since deep PacBio sequencing can be cost-prohibitive
- RT-PCR error sources cannot be excluded; two replicate cDNA libraries can be prepared, but further validations will still be needed

## Evidence

- [readme] IsoCon is a tool for reconstructing highly similar sequences present in a dataset of from long noisy reads.: "IsoCon is a tool for reconstructing highly similar sequences present in a dataset of from long noisy reads. Its original use case was transcripts from highly similar gene copies"
- [readme] IsoCon pipeline CLI invocation: "IsoCon pipeline -fl_reads <reads.fastq> -outfolder </path/to/output>"
- [discussion] IsoCon assumes Iso-Seq pre-processing: "IsoCon assumes the reads have been pre-processed with the Iso-Seq bioinformatic pipeline to remove primers, barcodes, and reads that are chimeras"
- [discussion] Pairwise alignments via parasail in correction phase: "We first create pairwise alignments from each read to the center using parasail60"
- [discussion] Statistical filtering threshold alpha default 0.01: "Then, we identify the candidates with pi greater than a significance threshold α. This α is a parameter to our algorithm, set by default to 0.01."
- [discussion] Variant-position threshold for candidate testing: "Candidate transcripts that have more than a threshold of variant positions (default of 10) relative to all other candidate transcripts are not statistically evaluated"
- [readme] Output file location: [evidence span withheld — anchor status: external_doc]
- [methods] stat_filter mode for external candidate sets: "Runs only the statistical filtering of candidates."
- [results] Benchmark comparison outcome: [evidence span withheld — anchor status: not_found]
- [discussion] Version pinning of IsoCon results: "The results of IsoCon in this paper were obtained with commit 79589f3 on GitHub."
