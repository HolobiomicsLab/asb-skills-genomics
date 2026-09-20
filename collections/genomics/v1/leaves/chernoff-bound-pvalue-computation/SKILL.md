---
name: chernoff-bound-pvalue-computation
description: Use when filtering candidate transcripts from Iso-Seq/PacBio reads of
  highly similar gene copies, to test whether reads assigned to two candidates could
  have originated from the same transcript via a weighted-support hypothesis test.
when_to_use_negative:
- Candidate transcript pairs differ by more than the default threshold of 10 variant
  positions — they are not statistically evaluated
- Candidate c has X times (default 5, via --min_test_ratio) more support than t, in
  which case testing is omitted
- Input is raw un-corrected reads rather than IsoCon candidate transcripts; run the
  error-correction stage first
edam_operation: http://edamontology.org/operation_2239
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_3168
tools:
- name: IsoCon
  role: Implements the statistical filtering (stat_filter) using the Chernoff-bound
    p-value computation; open-source at the referenced repo
  repo: https://github.com/ksahlin/IsoCon
- name: parasail
  role: Provides pairwise alignments of each read to candidates for greedy closest-neighbor
    assignment preceding the hypothesis test
  repo: https://github.com/jeffdaily/parasail-python
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41467-018-06910-x
    title: Deciphering highly similar multigene family transcripts from Iso-Seq data
      with IsoCon
  source_content_hash: sha256:ee9a48dc9ec3de46f08209374aec5a156863a107c60f1fbc6df015743915f5ad
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ksahlin/IsoCon
  tools:
  - IsoCon
  - parasail
  edam_operation: http://edamontology.org/operation_2239
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_3168
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# chernoff-bound-pvalue-computation

## Summary

Computes one-sided significance values P(T >= t) for a weighted-support statistic T (built from per-read Phred-derived error probabilities) in IsoCon's candidate filtering, using a closed-form Chernoff-type upper bound and defaulting the p-value to 0.5 when observed support falls below expectation (t' <= mu).

## When to use

Use when filtering candidate transcripts from Iso-Seq/PacBio reads of highly similar gene copies, to test whether reads assigned to two candidates could have originated from the same transcript via a weighted-support hypothesis test.

## When NOT to use

- Candidate transcript pairs differ by more than the default threshold of 10 variant positions — they are not statistically evaluated
- Candidate c has X times (default 5, via --min_test_ratio) more support than t, in which case testing is omitted
- Input is raw un-corrected reads rather than IsoCon candidate transcripts; run the error-correction stage first

## Inputs

- FLNC PacBio Iso-Seq reads (fastq/fasta) with Phred quality values
- Candidate transcripts from the error-correction stage (fasta, e.g. from get_candidates)
- Per-candidate-pair variant positions and per-read error probabilities derived from quality scores

## Outputs

- Surviving transcripts written to final_candidates.fa
- Read-to-candidate assignments written to cluster_info.tsv
- Per-candidate-pair p-values used for iterative filtering

## How to apply

After greedily assigning each FLNC read to its closest-neighbor candidate transcript (parasail pairwise alignments), extract the variant positions distinguishing each candidate pair and per-read error probabilities p_i from Phred quality values. Compute the weighted-support statistic T = prod(1/p_i^{s_i}) and its log-normalized form t' for each candidate pair. Evaluate significance with the one-sided test P(T >= t) via the Chernoff-type upper bound; whenever t' <= mu (observed support below expected), the result is clearly insignificant and the software defaults to a p-value of 0.5. Iteratively remove candidates whose p exceeds alpha (default 0.01), removing at most tau highest-p candidates per iteration, and skip candidates differing by more than the default 10 variant positions. This method works because the log-normalized statistic bounds the tail probability in closed form, making the test scalable across many candidate pairs.

## Related tools

- **IsoCon** (Implements the statistical filtering (stat_filter) using the Chernoff-bound p-value computation; open-source at the referenced repo) — https://github.com/ksahlin/IsoCon
- **parasail** (Provides pairwise alignments of each read to candidates for greedy closest-neighbor assignment preceding the hypothesis test) — https://github.com/jeffdaily/parasail-python

## Examples

```
IsoCon pipeline -fl_reads [path/simulated_pacbio_reads.fa] -outfolder [output path] -candidates <candidate_transcripts.fa>
```

## Evaluation signals

- p-value equals 0.5 exactly for all candidate pairs whose log-normalized support t' <= mu
- Candidates with p > alpha = 0.01 are removed iteratively, and at most tau candidates with the highest p-values are removed per iteration
- Pairs differing at more than 10 variant positions are skipped (not statistically evaluated)
- On the bundled small test dataset, stat_filter produces a final_candidates.fa containing transcript sequences in the output folder
- Read-to-candidate assignments are reported in cluster_info.tsv

## Limitations

- Very low-abundance transcripts might not be captured by Iso-Seq, since deep PacBio sequencing can be cost-prohibitive
- The p-value of 0.5 for t' <= mu is a conservative default, not a true tail probability
- IsoCon does not distinguish alleles from distinct gene copies; further downstream analysis would be needed

## Evidence

- [discussion] One-sided test with Chernoff-type bound and 0.5 default: "Such values are clearly insignificant, and our software defaults to a value of 0.5."
- [discussion] Significance threshold alpha = 0.01 for candidate removal: "Then, we identify the candidates with pi greater than a significance threshold α. This α is a parameter to our algorithm, set by default to 0.01."
- [discussion] Variant-position threshold of 10 exempts candidates from testing: "Candidate transcripts that have more than a threshold of variant positions (default of 10) relative to all other candidate transcripts are not statistically evaluated"
- [methods] stat_filter runs the statistical filtering stage only: "Runs only the statistical filtering of candidates."
- [methods] Small test dataset available for running IsoCon: "You should also be able to run IsoCon on this [small dataset](https://github.com/ksahlin/IsoCon/tree/master/test/data). Simply download the test dataset and run"
