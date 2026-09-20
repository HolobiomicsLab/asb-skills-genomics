---
name: iso-seq-long-read-error-correction
description: Use when input is a set of pre-processed FLNC PacBio Iso-Seq reads (FASTA/FASTQ
  with quality values) from targeted RT-PCR sequencing of a gene family with highly
  similar paralogs, and you need de novo transcript reconstruction and isoform counts
  without a reference genome.
when_to_use_negative:
- Reads have not been pre-processed with the Iso-Seq pipeline (primers, barcodes,
  and chimeras still present) — IsoCon assumes pre-processed FLNC input
- Sequences do not span the region of interest end-to-end; IsoCon requires reads covering
  the full transcript/amplicon (unsuited to fragments or reference-guided whole-transcriptome
  assembly)
- The goal is to distinguish alleles from distinct gene copies — IsoCon provides no
  such functionality and downstream analysis is required
edam_operation: http://edamontology.org/operation_3218
edam_topics:
- http://edamontology.org/topic_0080
- http://edamontology.org/topic_0622
tools:
- name: IsoCon
  role: Primary de novo error-correction and clustering tool producing candidate transcripts
    from FLNC Iso-Seq reads; results in the paper obtained with commit 79589f3
  repo: https://github.com/ksahlin/IsoCon
- name: ICE
  role: Standard PacBio de novo error-correction tool (part of the ToFU pipeline)
    run on the same FLNC reads as the baseline for transcript counts
- name: proovread
  role: Baseline corrector using paired Illumina MiSeq reads to correct errors in
    PacBio reads
- name: parasail
  role: SIMD pairwise alignment library used internally by IsoCon to align each read
    to its cluster center
  repo: https://github.com/jeffdaily/parasail-python
- name: edlib
  role: Edit-distance alignment dependency of IsoCon (>= 1.1.2)
  repo: https://github.com/Martinsos/edlib
- name: snakemake
  role: Workflow manager used in IsoCon_Eval to reproduce the simulated and experimental
    benchmark experiments
  repo: https://github.com/ksahlin/IsoCon_Eval
provenance:
  source_task_ids:
  - task_003
  - task_007
  source_papers:
  - doi: 10.1038/s41467-018-06910-x
    title: Deciphering highly similar multigene family transcripts from Iso-Seq data
      with IsoCon
  source_content_hash: sha256:bb8a16d456cba42b0d1b43af23a6f4d62c3b5c5a02152ba1ff8910a8364143fe
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ksahlin/IsoCon
  tools:
  - ICE
  - Iso-Seq
  - IsoCon
  - edlib
  - networkx
  - parasail
  - proovread
  - pysam
  - snakemake
  edam_operation: http://edamontology.org/operation_3218
  edam_topics:
  - http://edamontology.org/topic_0080
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 12
---

# iso-seq-long-read-error-correction

## Summary

De novo error correction and redundancy removal of full-length PacBio CCS (Iso-Seq) reads from targeted sequencing of highly similar multigene families, using IsoCon's iterative clustering plus statistical filtering to recover nucleotide-level transcript isoforms without a reference genome.

## When to use

Use when input is a set of pre-processed FLNC PacBio Iso-Seq reads (FASTA/FASTQ with quality values) from targeted RT-PCR sequencing of a gene family with highly similar paralogs, and you need de novo transcript reconstruction and isoform counts without a reference genome.

## When NOT to use

- Reads have not been pre-processed with the Iso-Seq pipeline (primers, barcodes, and chimeras still present) — IsoCon assumes pre-processed FLNC input
- Sequences do not span the region of interest end-to-end; IsoCon requires reads covering the full transcript/amplicon (unsuited to fragments or reference-guided whole-transcriptome assembly)
- The goal is to distinguish alleles from distinct gene copies — IsoCon provides no such functionality and downstream analysis is required

## Inputs

- Full-length non-chimeric (FLNC) PacBio CCS reads from targeted Iso-Seq, as FASTA or preferably FASTQ (quality values required for statistical filtering)
- Optional CCS BAM file via --ccs flag
- Illumina MiSeq paired-end reads from Nextera XT libraries (only for proovread baseline correction)

## Outputs

- final_candidates.fa — high-quality reconstructed transcript sequences
- not_converged.fa — reads that could not be corrected or clustered (e.g. single/low-support exon-differing reads, possible chimeras)
- cluster_info.tsv — mapping of each read to its assigned candidate transcript
- Per-family, per-sample transcript counts for comparison against ICE/proovread baselines

## How to apply

Start with FLNC CCS reads already processed by the Iso-Seq bioinformatic pipeline (primers, barcodes, chimeras removed; split by primer pair), preferably FASTQ so quality values feed the statistical analysis. Run 'IsoCon pipeline -fl_reads <reads.fastq> -outfolder <out>', which executes the error-correction step (get_candidates) — iterative clustering via nearest-neighbor graph path traversal, parasail pairwise alignments to cluster centers, and multi-alignment consensus with correction of half of low-support variant positions — followed by stat_filter, which iteratively removes statistically non-significant candidates using pairwise hypothesis testing with significance threshold alpha = 0.01 (default), a per-iteration cap tau on removed candidates, and a default threshold of 10 variant positions for statistical evaluation; use --min_test_ratio (default 5) to omit testing candidates with far more support. Collect final_candidates.fa (high-quality transcripts), not_converged.fa (uncorrected/unclustered reads, possible chimeras), and cluster_info.tsv (read-to-candidate assignment). Benchmark against baselines ICE (standard PacBio de novo correction, ToFU pipeline) and proovread (Illumina MiSeq-based correction) by counting predicted transcripts per gene family and per sample.

## Related tools

- **IsoCon** (Primary de novo error-correction and clustering tool producing candidate transcripts from FLNC Iso-Seq reads; results in the paper obtained with commit 79589f3) — https://github.com/ksahlin/IsoCon
- **ICE** (Standard PacBio de novo error-correction tool (part of the ToFU pipeline) run on the same FLNC reads as the baseline for transcript counts)
- **proovread** (Baseline corrector using paired Illumina MiSeq reads to correct errors in PacBio reads)
- **parasail** (SIMD pairwise alignment library used internally by IsoCon to align each read to its cluster center) — https://github.com/jeffdaily/parasail-python
- **edlib** (Edit-distance alignment dependency of IsoCon (>= 1.1.2)) — https://github.com/Martinsos/edlib
- **snakemake** (Workflow manager used in IsoCon_Eval to reproduce the simulated and experimental benchmark experiments) — https://github.com/ksahlin/IsoCon_Eval

## Examples

```
IsoCon pipeline -fl_reads fmr1_flnc.fastq -outfolder out
```

## Evaluation signals

- Compiled transcript totals per family and sample must match reported benchmark values: 406 IsoCon vs 1406 ICE transcripts in sample 1, and 344 vs 2003 in sample 2 across the nine Y ampliconic families
- On the public FMR1 targeted Iso-Seq dataset, IsoCon recovers an average of 24 isoforms per sample versus 20 from the ICE-based ToFU pipeline, including recovery of previously missing isoforms
- final_candidates.fa contains converged candidates; not_converged.fa and cluster_info.tsv are populated and consistent (every read mapped in cluster_info.tsv belongs to a candidate)
- stat_filter parameters behave as specified: alpha = 0.01 significance threshold, at most tau candidates removed per iteration, candidates with more than 10 variant positions not statistically evaluated
- On simulated data (IsoCon_Eval: MEMBER/ISOFORM/COMBINED experiments, gene families HSFY2, DAZ2, TSPY13P; read counts 20–12500), IsoCon shows substantially higher precision and recall than ICE

## Limitations

- IsoCon does not distinguish alleles from distinct gene copies; further downstream analysis would be needed for copy assignment
- Transcriptome-only sequencing does not provide a definitive answer to gene family size or conclusive assignment of each transcript to a gene copy
- Novel gene copies are difficult to distinguish from splice variants or RNA-editing
- RT-PCR error sources cannot be excluded; replicate cDNA libraries and further validation are still needed
- Very low-abundance transcripts might not be captured, since deep PacBio sequencing can be cost-prohibitive (targeted RT-PCR sequencing mitigates this)

## Evidence

- [readme] Usage: IsoCon pipeline command: "IsoCon pipeline -fl_reads <reads.fastq> -outfolder </path/to/output>"
- [readme] Output files: "predicted transcripts are found in file **/path/to/output/final_candidates.fa**"
- [discussion] Pre-processing assumption: "IsoCon assumes the reads have been pre-processed with the Iso-Seq bioinformatic pipeline to remove primers, barcodes, and reads that are chimeras"
- [discussion] Statistical filtering default threshold: "we identify the candidates with pi greater than a significance threshold α. This α is a parameter to our algorithm, set by default to 0.01."
- [discussion] Variant-position threshold: "Candidate transcripts that have more than a threshold of variant positions (default of 10) relative to all other candidate transcripts are not statistically evaluated"
- [discussion] Alignment engine: "We first create pairwise alignments from each read to the center using parasail60"
- [results] Benchmark numbers on Y families: [evidence span withheld — anchor status: not_found]
- [discussion] Paper commit pin and availability: "IsoCon is open-source and freely available at https://github.com/ksahlin/IsoCon. The results of IsoCon in this paper were obtained with commit 79589f3"
