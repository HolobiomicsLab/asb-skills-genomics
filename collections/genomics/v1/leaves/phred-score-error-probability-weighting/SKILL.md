---
name: phred-score-error-probability-weighting
description: Use when filtering candidate transcripts from PacBio Iso-Seq (or other
  long-read) reads of highly similar gene copies, and you need to test whether reads
  assigned to two candidates could originate from the same transcript using per-base
  error probabilities from Phred scores.
when_to_use_negative:
- Candidate transcript pairs with more than the default threshold of 10 variant positions
  relative to all other candidates are not statistically evaluated
- Candidate c relative to t is omitted from testing when c has more than --min_test_ratio
  X (default 5) times more support
- Short reads or datasets where reads do not span the region of interest end-to-end;
  the statistical framework relies on long reads linking variants across the transcript
edam_operation: http://edamontology.org/operation_3227
edam_topics:
- http://edamontology.org/topic_0102
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3170
tools:
- name: IsoCon
  role: Implements the statistical filtering (stat_filter) that computes the Phred-weighted
    support statistic and iteratively removes non-significant candidates
  repo: https://github.com/ksahlin/IsoCon
- name: parasail
  role: Provides the pairwise alignments used to greedily assign each read to its
    closest-neighbor candidate transcript before hypothesis testing
  repo: https://github.com/jeffdaily/parasail-python
- name: PacBio
  role: Source of long Iso-Seq FLNC reads whose per-base Phred quality values supply
    the error probabilities p_i used to weight the statistic
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41467-018-06910-x
    title: Deciphering highly similar multigene family transcripts from Iso-Seq data
      with IsoCon
  source_content_hash: sha256:1d9b53bdedfee9cfe98feb3336eca5dc4153724d1f3bf3d28479753752155797
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ksahlin/IsoCon
  tools:
  - IsoCon
  - PacBio
  - parasail
  edam_operation: http://edamontology.org/operation_3227
  edam_topics:
  - http://edamontology.org/topic_0102
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8181818181818182
n_claims: 11
---

# phred-score-error-probability-weighting

## Summary

Converts per-read Phred quality values into per-base error probabilities that weight a support statistic T = prod(1/p_i^{s_i}) for IsoCon's second-stage statistical filtering of candidate transcripts. The log-normalized statistic is tested with a one-sided P(T >= t) significance test bounded via a Chernoff-type bound, so reads with lower sequencing error contribute stronger evidence when distinguishing highly similar gene-copy transcripts.

## When to use

Use when filtering candidate transcripts from PacBio Iso-Seq (or other long-read) reads of highly similar gene copies, and you need to test whether reads assigned to two candidates could originate from the same transcript using per-base error probabilities from Phred scores.

## When NOT to use

- Candidate transcript pairs with more than the default threshold of 10 variant positions relative to all other candidates are not statistically evaluated
- Candidate c relative to t is omitted from testing when c has more than --min_test_ratio X (default 5) times more support
- Short reads or datasets where reads do not span the region of interest end-to-end; the statistical framework relies on long reads linking variants across the transcript

## Inputs

- FLNC reads in FASTQ or FASTA format with Phred quality values (e.g., simulated_pacbio_reads.fa from the IsoCon test dataset)
- Already-constructed candidate transcripts from IsoCon's error-correction stage (FASTA, e.g. from the get_candidates step)
- Read-to-candidate assignments from greedy closest-neighbor alignment (parasail)
- Per-variant-position error probabilities p_i derived from read Phred scores

## Outputs

- Per-candidate-pair significance values (one-sided p-value P(T >= t), bounded via Chernoff-type bound; 0.5 when t' <= mu)
- Surviving transcripts written to final_candidates.fa
- Read-to-candidate assignments written to cluster_info.tsv

## How to apply

After IsoCon's error-correction stage converges on candidate transcripts, greedily assign each FLNC read to its closest-neighbor candidate using parasail pairwise alignments. Extract the variant positions distinguishing each candidate pair and derive per-base error probabilities p_i from each read's Phred quality values. Compute the weighted-support statistic T = prod(1/p_i^{s_i}) over these positions and its log-normalized form t'. Evaluate significance with a one-sided test P(T >= t) via a closed-form Chernoff-type upper bound; when observed support falls at or below the expected support (t' <= mu), the value is clearly insignificant and the software defaults the p-value to 0.5. Iteratively remove candidates with significance value above alpha (default 0.01), removing at most tau highest-p candidates per iteration, and skip candidates with more than the default 10 variant positions from statistical evaluation.

## Related tools

- **IsoCon** (Implements the statistical filtering (stat_filter) that computes the Phred-weighted support statistic and iteratively removes non-significant candidates) — https://github.com/ksahlin/IsoCon
- **parasail** (Provides the pairwise alignments used to greedily assign each read to its closest-neighbor candidate transcript before hypothesis testing) — https://github.com/jeffdaily/parasail-python
- **PacBio** (Source of long Iso-Seq FLNC reads whose per-base Phred quality values supply the error probabilities p_i used to weight the statistic)

## Examples

```
IsoCon pipeline -fl_reads [path/simulated_pacbio_reads.fa] -outfolder [output path] -candidates <candidate_transcripts.fa>
```

## Evaluation signals

- Run stat_filter on the bundled small test dataset and verify that final_candidates.fa exists in the output folder and contains transcript sequences
- Verify that candidate pairs with observed support at or below expected support (t' <= mu) receive the default p-value of 0.5
- Check that no surviving candidate has significance value above alpha (default 0.01) after iterative removal
- Confirm candidates with more than the default 10 variant-position threshold were skipped rather than statistically evaluated
- Verify at most tau highest-p candidates were removed per iteration when more than tau exceeded alpha

## Limitations

- Very low-abundance transcripts might not be captured by Iso-Seq, since deep PacBio sequencing can be cost-prohibitive
- IsoCon does not provide a definitive answer to the size of each gene family and does not allow conclusive assignment of each transcript to a gene copy from transcriptome-only sequencing
- Novel gene copies can be difficult to distinguish from splice variants or RNA-editing; distinguishing alleles from distinct copies requires further downstream analysis

## Evidence

- [discussion] One-sided test P(T >= t) using Chernoff-type upper bound; default p-value 0.5 when observed support below expected: "Such values are clearly insignificant, and our software defaults to a value of 0.5."
- [discussion] Removal of candidates with significance value (p) greater than threshold alpha (default 0.01): "Then, we identify the candidates with pi greater than a significance threshold α. This α is a parameter to our algorithm, set by default to 0.01."
- [discussion] At most tau candidates with highest p-values removed per iteration: "if there are more than τ candidates with significance value over α, we only remove the top τ candidates with the highest values."
- [discussion] Candidates with more than a threshold of variant positions (default 10) not statistically evaluated: "Candidate transcripts that have more than a threshold of variant positions (default of 10) relative to all other candidate transcripts are not statistically evaluated"
- [discussion] Pairwise alignments used for read assignment during candidate filtering: "We first create pairwise alignments from each read to the center using parasail60"
- [methods] stat_filter mode runs only the statistical filtering of candidates: [evidence span withheld — anchor status: not_found]
- [methods] Validation run on the small IsoCon test dataset: "You should also be able to run IsoCon on this [small dataset](https://github.com/ksahlin/IsoCon/tree/master/test/data). Simply download the test dataset and run"
- [readme] Predicted transcripts are output to final_candidates.fa: "predicted transcripts are found in file **/path/to/output/final_candidates.fa**."
