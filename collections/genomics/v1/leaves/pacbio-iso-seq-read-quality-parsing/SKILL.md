---
name: pacbio-iso-seq-read-quality-parsing
description: Use when running IsoCon's statistical filtering (stat_filter) on targeted
  Iso-Seq FLNC reads in fastq/fasta and per-read error probabilities must be extracted
  from Phred quality values to compute candidate-significance p-values.
when_to_use_negative:
- Reads are raw, unprocessed PacBio output that has not gone through the Iso-Seq pipeline
  for primer/barcode removal and chimera filtering — IsoCon assumes pre-processed
  FLNC reads.
- Reads do not span the region of interest end-to-end (e.g., fragmented or short reads),
  since the weighted-support statistic requires reads spanning variant positions across
  the transcript.
- The goal is general transcript quantification or reference-based isoform annotation
  rather than de novo resolution of highly similar gene-copy transcripts.
edam_operation: http://edamontology.org/operation_3182
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0102
tools:
- name: IsoCon
  role: Main package whose stat_filter step consumes the parsed Phred-derived error
    probabilities and computes the weighted-support statistic and p-values
  repo: https://github.com/ksahlin/IsoCon
- name: parasail
  role: Provides the pairwise alignments of each read to candidate centers used to
    locate variant positions whose quality values are parsed
  repo: https://github.com/jeffdaily/parasail-python
- name: PacBio
  role: Instrument platform whose per-read error model motivates the Phred-based weighting
    of the support statistic
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41467-018-06910-x
    title: Deciphering highly similar multigene family transcripts from Iso-Seq data
      with IsoCon
  source_content_hash: sha256:2e1b085172816f47cef32f83ff1b83a524b8af4d1a7742ffa37b5d3d53f132ee
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ksahlin/IsoCon
  tools:
  - IsoCon
  - PacBio
  - parasail
  edam_operation: http://edamontology.org/operation_3182
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0102
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 11
---

# pacbio-iso-seq-read-quality-parsing

## Summary

Parses per-read base error probabilities from PacBio Iso-Seq FLNC read quality strings (Phred scores) and log-normalizes them to feed IsoCon's weighted-support statistic T = prod(1/p_i^{s_i}) used in candidate hypothesis testing.

## When to use

Use when running IsoCon's statistical filtering (stat_filter) on targeted Iso-Seq FLNC reads in fastq/fasta and per-read error probabilities must be extracted from Phred quality values to compute candidate-significance p-values.

## When NOT to use

- Reads are raw, unprocessed PacBio output that has not gone through the Iso-Seq pipeline for primer/barcode removal and chimera filtering — IsoCon assumes pre-processed FLNC reads.
- Reads do not span the region of interest end-to-end (e.g., fragmented or short reads), since the weighted-support statistic requires reads spanning variant positions across the transcript.
- The goal is general transcript quantification or reference-based isoform annotation rather than de novo resolution of highly similar gene-copy transcripts.

## Inputs

- FLNC PacBio Iso-Seq reads in fastq format (with Phred quality strings) or fasta format
- Candidate transcript sequences in FASTA (e.g., output of IsoCon's get_candidates error-correction step)
- Read-to-candidate assignments from greedy closest-neighbor alignment (parasail pairwise alignments)

## Outputs

- Per-read error probabilities p_i derived from Phred scores at variant positions
- Weighted-support statistic T = prod(1/p_i^{s_i}) and its log-normalized form t' per candidate pair
- One-sided significance values P(T >= t) per candidate pair (default 0.5 when t' <= mu)
- final_candidates.fa (surviving transcripts) and cluster_info.tsv (read-to-candidate assignments)

## How to apply

Load the full-length (FLNC) reads — pre-processed by the Iso-Seq pipeline to remove primers, barcodes, and chimeras — in fastq (preferred, since it carries Phred quality strings) or fasta. Greedily assign each read to its closest-neighbor candidate transcript using pairwise alignments computed with parasail, then extract the variant positions distinguishing each candidate pair. Convert each read's Phred quality values at those variant positions into per-read error probabilities p_i, and compute the weighted-support statistic T = prod(1/p_i^{s_i}) with its log-normalized form t'. This log-normalization and the per-read error weighting are what allow the one-sided test P(T >= t) via a Chernoff-type upper bound to distinguish reads that truly support distinct transcripts from sequencing noise.

## Related tools

- **IsoCon** (Main package whose stat_filter step consumes the parsed Phred-derived error probabilities and computes the weighted-support statistic and p-values) — https://github.com/ksahlin/IsoCon
- **parasail** (Provides the pairwise alignments of each read to candidate centers used to locate variant positions whose quality values are parsed) — https://github.com/jeffdaily/parasail-python
- **PacBio** (Instrument platform whose per-read error model motivates the Phred-based weighting of the support statistic)

## Examples

```
IsoCon stat_filter -fl_reads /path/simulated_pacbio_reads.fa -outfolder /path/output -candidates candidate_transcripts.fa
```

## Evaluation signals

- Running stat_filter on the bundled small test dataset produces final_candidates.fa in the output folder containing transcript sequences.
- cluster_info.tsv in the output folder contains read-to-candidate assignments consistent with the greedy closest-neighbor alignment.
- For any candidate pair where observed support is below expected support (t' <= mu), the assigned p-value is exactly 0.5.
- Candidate pairs with more than 10 variant positions are skipped (not statistically evaluated), so parsing must not emit p-values for them.
- After iterative removal, no surviving candidate has significance value greater than alpha = 0.01 (respecting at most tau removals per iteration).

## Limitations

- Very low-abundance transcripts might not be captured by Iso-Seq, since deep PacBio sequencing can be cost-prohibitive, so quality-derived support may be sparse for rare variants.
- Candidates with more than the default 10 variant-position threshold are never statistically evaluated, so the parsed error probabilities do not inform filtering for those pairs.
- Reads lacking quality strings (fasta-only input) cannot yield per-read error probabilities p_i from Phred scores, weakening the weighting of T.
- IsoCon does not provide a definitive answer to the size of each gene family and does not allow conclusive assignment of each transcript to a gene copy; downstream analysis is still needed.

## Evidence

- [discussion] Chernoff bound / 0.5 default: "Such values are clearly insignificant, and our software defaults to a value of 0.5."
- [discussion] alpha threshold: "Then, we identify the candidates with pi greater than a significance threshold α. This α is a parameter to our algorithm, set by default to 0.01."
- [discussion] variant-position threshold: "Candidate transcripts that have more than a threshold of variant positions (default of 10) relative to all other candidate transcripts are not statistically evaluated"
- [discussion] parasail alignments: "We first create pairwise alignments from each read to the center using parasail60"
- [readme] IsoCon purpose / reads input: "IsoCon is a tool for reconstructing highly similar sequences present in a dataset of from long noisy reads."
- [readme] output file: "predicted transcripts are found in file **/path/to/output/final_candidates.fa**"
