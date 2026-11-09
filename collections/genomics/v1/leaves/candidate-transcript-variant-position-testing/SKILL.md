---
name: candidate-transcript-variant-position-testing
description: Use when you have already-derived candidate transcripts from error-corrected
  long reads (Iso-Seq/CCS or ONT) and must decide which candidates are statistically
  redundant because their assigned reads show insufficient weighted support to justify
  separate transcripts.
when_to_use_negative:
- Candidates differ by more than the default 10 variant positions relative to all
  other candidates — they are not statistically evaluated and are kept without a p-value
- Candidate c has at least --min_test_ratio (default 5) times more support than t
  — the c-vs-t test is omitted
- Input reads have not been pre-processed with the Iso-Seq pipeline to remove primers,
  barcodes, and chimeras
edam_operation: http://edamontology.org/operation_3227
edam_topics:
- http://edamontology.org/topic_0102
- http://edamontology.org/topic_0080
tools:
- name: IsoCon
  role: 'Implements the statistical filtering: stat_filter runs only the statistical
    filtering of candidates; pipeline runs the full workflow'
  repo: https://github.com/ksahlin/IsoCon
- name: parasail
  role: Performs the pairwise alignments from each read to candidate transcripts used
    for greedy read assignment and variant-position extraction
  repo: https://github.com/jeffdaily/parasail-python
- name: edlib
  role: Sequence-alignment library used within the IsoCon toolchain
  repo: https://github.com/Martinsos/edlib
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41467-018-06910-x
    title: Deciphering highly similar multigene family transcripts from Iso-Seq data
      with IsoCon
  source_content_hash: sha256:d26a0d198d1ac05ef9c8b2ccb827db6f66fd03e1abca3a49307331c5dd017e1d
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ksahlin/IsoCon
  tools:
  - IsoCon
  - edlib
  - parasail
  edam_operation: http://edamontology.org/operation_3227
  edam_topics:
  - http://edamontology.org/topic_0102
  - http://edamontology.org/topic_0080
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# candidate-transcript-variant-position-testing

## Summary

IsoCon's second-stage statistical filtering skill: for each pair of candidate transcripts, greedily assign full-length PacBio reads to their closest candidate, extract the variant positions distinguishing the pair, and test via a weighted-support Bernoulli statistic with a Chernoff-type bound whether both candidates' reads could have come from the same transcript; non-significant candidates (p > alpha, default 0.01) are removed iteratively.

## When to use

Use when you have already-derived candidate transcripts from error-corrected long reads (Iso-Seq/CCS or ONT) and must decide which candidates are statistically redundant because their assigned reads show insufficient weighted support to justify separate transcripts.

## When NOT to use

- Candidates differ by more than the default 10 variant positions relative to all other candidates — they are not statistically evaluated and are kept without a p-value
- Candidate c has at least --min_test_ratio (default 5) times more support than t — the c-vs-t test is omitted
- Input reads have not been pre-processed with the Iso-Seq pipeline to remove primers, barcodes, and chimeras

## Inputs

- Candidate transcripts fasta file from the error-correction step (get_candidates output)
- FLNC reads file (fastq/fasta) with Phred quality values
- Optional CCS bam file (--ccs)
- Parameters: alpha (default 0.01), tau (max removals per iteration), variant-position threshold (default 10), --min_test_ratio X (default 5)

## Outputs

- final_candidates.fa — surviving transcripts after statistical filtering
- cluster_info.tsv — read-to-candidate assignments
- Uncorrected/unclustered reads file in the output folder

## How to apply

After the error-correction stage converges (get_candidates output), greedily assign each FLNC read to its closest-neighbor candidate using pairwise alignments (parasail) and identify the variant positions distinguishing each candidate pair. From each read's Phred quality values derive per-read error probabilities p_i and compute the weighted-support statistic T = prod(1/p_i^{s_i}), log-normalized to t'. Compute the one-sided significance P(T >= t) using the closed-form Chernoff-type upper bound; when observed support is at or below the expected support (t' <= mu), the default p-value is 0.5 because such values are clearly insignificant. Iteratively remove candidates with significance exceeding alpha (default 0.01), removing at most tau highest-p candidates per iteration, and skip candidates differing by more than the default 10 variant positions; the --min_test_ratio parameter (default 5) additionally omits testing candidate c against t when c has 5x more support. Survivors are written to final_candidates.fa with read-to-candidate assignments in cluster_info.tsv; the standalone stat_filter subcommand runs only this filtering step.

## Related tools

- **IsoCon** (Implements the statistical filtering: stat_filter runs only the statistical filtering of candidates; pipeline runs the full workflow) — https://github.com/ksahlin/IsoCon
- **parasail** (Performs the pairwise alignments from each read to candidate transcripts used for greedy read assignment and variant-position extraction) — https://github.com/jeffdaily/parasail-python
- **edlib** (Sequence-alignment library used within the IsoCon toolchain) — https://github.com/Martinsos/edlib

## Examples

```
IsoCon pipeline -fl_reads [path/simulated_pacbio_reads.fa] -outfolder [output path] -candidates <candidate_transcripts.fa>
```

## Evaluation signals

- Running stat_filter on the bundled small test dataset produces final_candidates.fa in the output folder and the file contains transcript sequences
- cluster_info.tsv exists and maps reads to surviving candidate transcripts consistently with the greedy closest-neighbor assignment
- Every removed candidate had a computed significance value above alpha = 0.01, and no more than tau candidates are removed per iteration
- Pairs whose observed support is below expected (t' <= mu) were assigned the default p-value 0.5 and therefore retained in that iteration
- Candidate pairs exceeding the 10-variant-position threshold were excluded from statistical evaluation rather than given p-values

## Limitations

- Candidates with more than a threshold of variant positions (default 10) relative to all other candidates are not statistically evaluated and cannot be filtered by this test
- If more than tau candidates exceed alpha in one iteration, only the top tau with the highest significance values are removed, requiring multiple iterations
- The test assumes per-read error probabilities derived from Phred quality values are reliable; the Chernoff-type bound is an upper bound on the p-value
- IsoCon requires reads pre-processed by the Iso-Seq pipeline (primers, barcodes, chimeras removed); it cannot distinguish alleles from distinct gene copies without downstream analysis

## Evidence

- [discussion] One-sided test P(T >= t) using Chernoff-type upper bound; default p-value 0.5 when observed support below expected: "Such values are clearly insignificant, and our software defaults to a value of 0.5."
- [discussion] Removal of candidates with significance value (p) greater than threshold alpha (default 0.01): "Then, we identify the candidates with pi greater than a significance threshold α. This α is a parameter to our algorithm, set by default to 0.01."
- [discussion] At most tau (τ) candidates with highest p-values removed per iteration: "if there are more than τ candidates with significance value over α, we only remove the top τ candidates with the highest values."
- [methods] stat_filter runs only the statistical filtering step: "Runs only the statistical filtering of candidates."
- [readme] IsoCon outputs predicted transcripts in final_candidates.fa: "predicted transcripts are found in file **/path/to/output/final_candidates.fa**."
