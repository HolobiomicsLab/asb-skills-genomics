---
name: read-to-transcript-cluster-assignment
description: Use when input is FLNC CCS reads (FASTQ/FASTA) from targeted Iso-Seq
  of a multigene family and each read must be assigned to its originating transcript/cluster
  despite near-identical copy sequences.
when_to_use_negative:
- Reads have not been pre-processed with the Iso-Seq pipeline (primers, barcodes,
  and chimeras still present) — IsoCon assumes clean FLNC input
- Reads do not span the region of interest end-to-end (e.g. shotgun transcriptome
  or genomic reads rather than targeted Iso-Seq/amplicon reads spanning the transcript)
- The goal is to definitively distinguish alleles from distinct gene copies or determine
  gene family size — IsoCon provides only hypothetical assignment and further downstream
  analysis would be needed
edam_operation: http://edamontology.org/operation_0291
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0196
tools:
- name: IsoCon
  role: 'Primary tool: iterative clustering, center-based error correction, and greedy
    read-to-candidate assignment with pairwise statistical filtering'
  repo: https://github.com/ksahlin/IsoCon
- name: parasail
  role: SIMD pairwise sequence alignment library used to align each read to its cluster
    center during the correction phase
  repo: https://github.com/jeffdaily/parasail-python
- name: edlib
  role: Edit-distance alignment dependency used by IsoCon (requirement edlib >= 1.1.2)
  repo: https://github.com/Martinsos/edlib
- name: networkx
  role: Graph library dependency supporting nearest-neighbor graph path traversal
    for cluster partitioning (networkx >= 1.10)
- name: pysam
  role: Dependency for reading BAM/CCS inputs (pysam >= 0.11)
provenance:
  source_task_ids:
  - task_003
  - task_007
  source_papers:
  - doi: 10.1038/s41467-018-06910-x
    title: Deciphering highly similar multigene family transcripts from Iso-Seq data
      with IsoCon
  source_content_hash: sha256:7d87efd693496e3d5d932cb02127898aeed3da0a9a37a5480f2747f5528e9a0c
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
  edam_operation: http://edamontology.org/operation_0291
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0196
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7272727272727273
n_claims: 11
---

# read-to-transcript-cluster-assignment

## Summary

Assigns pre-processed FLNC PacBio CCS reads to consensus transcript clusters for highly similar gene copies, via iterative nearest-neighbor graph clustering, center-based error correction, and greedy closest-neighbor candidate assignment with pairwise hypothesis testing (as implemented in IsoCon).

## When to use

Use when input is FLNC CCS reads (FASTQ/FASTA) from targeted Iso-Seq of a multigene family and each read must be assigned to its originating transcript/cluster despite near-identical copy sequences.

## When NOT to use

- Reads have not been pre-processed with the Iso-Seq pipeline (primers, barcodes, and chimeras still present) — IsoCon assumes clean FLNC input
- Reads do not span the region of interest end-to-end (e.g. shotgun transcriptome or genomic reads rather than targeted Iso-Seq/amplicon reads spanning the transcript)
- The goal is to definitively distinguish alleles from distinct gene copies or determine gene family size — IsoCon provides only hypothetical assignment and further downstream analysis would be needed

## Inputs

- FLNC CCS reads from targeted Iso-Seq (FASTQ preferred over FASTA so quality values feed the statistical analysis)
- Optional CCS BAM file for --ccs mode
- Pre-processed reads split by primer pair per sample (primers, barcodes, chimeras already removed)

## Outputs

- final_candidates.fa — high-quality predicted transcript sequences
- cluster_info.tsv — mapping of each read to its assigned candidate cluster
- not_converged.fa — reads that could not be corrected or clustered (e.g. single/low-support exon-differing reads, possible chimeras)

## How to apply

First ensure reads are pre-processed with the Iso-Seq bioinformatic pipeline to remove primers, barcodes, and chimeras and split by primer pair, since IsoCon assumes this. Run the full 'IsoCon pipeline -fl_reads <reads.fastq> -outfolder <out>' which performs error correction (get_candidates) followed by statistical filtering (stat_filter, alpha = 0.01 by default, tau candidates removed per iteration, candidates with >10 variant positions relative to others not statistically evaluated). Internally, IsoCon builds a nearest-neighbor graph over reads, creates clusters by picking read x with the highest number of vertices that can reach it, and corrects reads to the center using pairwise alignments (parasail) followed by a multi-alignment consensus, correcting only the half of variant positions with lowest support. In the filtering phase, reads are greedily assigned to the closest-neighbor candidates and each candidate pair is tested for sufficient evidence that their assigned reads did not originate from the same transcript; one-sided tests use a Chernoff-type upper bound with default p-value 0.5 when observed support is below expected. Output cluster membership maps each read to its assigned candidate transcript, enabling per-copy isoform counts that can be compared against ICE or proovread baselines.

## Related tools

- **IsoCon** (Primary tool: iterative clustering, center-based error correction, and greedy read-to-candidate assignment with pairwise statistical filtering) — https://github.com/ksahlin/IsoCon
- **parasail** (SIMD pairwise sequence alignment library used to align each read to its cluster center during the correction phase) — https://github.com/jeffdaily/parasail-python
- **edlib** (Edit-distance alignment dependency used by IsoCon (requirement edlib >= 1.1.2)) — https://github.com/Martinsos/edlib
- **networkx** (Graph library dependency supporting nearest-neighbor graph path traversal for cluster partitioning (networkx >= 1.10))
- **pysam** (Dependency for reading BAM/CCS inputs (pysam >= 0.11))

## Examples

```
IsoCon pipeline -fl_reads fmr1_flnc.fastq -outfolder out
```

## Evaluation signals

- cluster_info.tsv should map every FLNC read to exactly one candidate (or to not_converged.fa if unassignable); no reads silently dropped
- Number of predicted transcripts in final_candidates.fa should be lower than the read count and consistent with per-gene-family expectations (e.g. IsoCon yielded 406 transcripts vs 1406 for ICE in testes sample 1, 344 vs 2003 in sample 2)
- Changing the statistical filtering parameters (alpha from 0.01, tau cap, 10-variant-position threshold) should predictably alter the number of retained candidates; candidate pairs are retained only when there is sufficient evidence their reads did not come from the same transcript
- Per-family transcript counts should be reproducible when the pipeline is rerun on the same FLNC FASTQ (paper used IsoCon commit 79589f3)
- Assignment sanity check: reads in a cluster should align end-to-end to their center with fewer variant positions than the candidate-pair variant threshold

## Limitations

- IsoCon does not provide functionality to distinguish alleles from distinct gene copies; distinguishing alleles, splice variants, or RNA-editing effects requires further downstream analysis
- Transcriptome-only sequencing does not give a definitive answer to gene family size or allow conclusive assignment of each transcript to a gene copy
- Very low-abundance transcripts may not be captured at all, since deep PacBio sequencing can be cost-prohibitive
- RT-PCR error sources cannot be excluded; replicate cDNA libraries and further validation are still needed

## Evidence

- [readme] IsoCon is a tool for reconstructing highly similar sequences present in a dataset of from long noisy reads.: "IsoCon is a tool for reconstructing highly similar sequences present in a dataset of from long noisy reads."
- [discussion] we identify a read x in G with the highest number of vertices that can reach x. We create a new cluster with x as the center: "we identify a read x in G with the highest number of vertices that can reach x. We create a new cluster with x as the center"
- [discussion] We first create pairwise alignments from each read to the center using parasail60: "We first create pairwise alignments from each read to the center using parasail60"
- [discussion] for every pair we check whether there is sufficient evidence that their assigned reads did not in fact originate from the same transcript.: "for every pair we check whether there is sufficient evidence that their assigned reads did not in fact originate from the same transcript."
- [discussion] IsoCon assumes the reads have been pre-processed with the Iso-Seq bioinformatic pipeline to remove primers, barcodes, and reads that are chimeras: "IsoCon assumes the reads have been pre-processed with the Iso-Seq bioinformatic pipeline to remove primers, barcodes, and reads that are chimeras"
