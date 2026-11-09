---
name: statistical-transcript-candidate-filtering
description: Use when de novo error correction (e.g. on targeted Iso-Seq FLNC CCS
  reads) has produced multiple candidate transcript variants and you must decide,
  statistically, which candidates are distinct isoforms versus redundant low-support
  splits.
when_to_use_negative:
- Candidates differ by more than the default threshold of 10 variant positions — these
  are not statistically evaluated and instead retained directly
- Input reads are FASTA without quality values — the weighted-support test requires
  per-read Phred error probabilities
- Reads have not been pre-processed with the Iso-Seq pipeline (primers, barcodes,
  chimeras still present)
edam_operation: http://edamontology.org/operation_3218
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_3308
tools:
- name: IsoCon
  role: Implements the get_candidates and stat_filter stages; stat_filter performs
    the statistical filtering of candidates with alpha=0.01
  repo: https://github.com/ksahlin/IsoCon
- name: parasail
  role: Used to create the pairwise alignments from each read to the cluster center
    used for greedy read-to-candidate assignment
  repo: https://github.com/jeffdaily/parasail-python
- name: edlib
  role: Dependency (>= 1.1.2) for fast edit-distance alignment in IsoCon
  repo: https://github.com/Martinsos/edlib
provenance:
  source_task_ids:
  - task_003
  - task_008
  - task_007
  source_papers:
  - doi: 10.1038/s41467-018-06910-x
    title: Deciphering highly similar multigene family transcripts from Iso-Seq data
      with IsoCon
  source_content_hash: sha256:c260a3b319b3d7b073bf49257ac3f053651dfcc96bcccc97bead89d1d680613d
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
  edam_operation: http://edamontology.org/operation_3218
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7333333333333333
n_claims: 15
---

# statistical-transcript-candidate-filtering

## Summary

IsoCon's second-stage candidate filtering: a weighted-support Bernoulli-style hypothesis test (with a Chernoff-type bound) decides which candidate transcripts from error correction are statistically redundant, so only well-supported distinct transcripts survive. It prevents overcalling isoforms among highly similar gene copies in PacBio Iso-Seq data.

## When to use

Use when de novo error correction (e.g. on targeted Iso-Seq FLNC CCS reads) has produced multiple candidate transcript variants and you must decide, statistically, which candidates are distinct isoforms versus redundant low-support splits.

## When NOT to use

- Candidates differ by more than the default threshold of 10 variant positions — these are not statistically evaluated and instead retained directly
- Input reads are FASTA without quality values — the weighted-support test requires per-read Phred error probabilities
- Reads have not been pre-processed with the Iso-Seq pipeline (primers, barcodes, chimeras still present)

## Inputs

- Candidate transcripts from the error-correction step (get_candidates converged candidates in FASTA)
- Full-length non-chimeric (FLNC) PacBio CCS reads in FASTQ (so Phred quality values are available)
- Pre-processed reads (Iso-Seq pipeline: primers, barcodes, chimeras removed, split by primer pair)

## Outputs

- final_candidates.fa: surviving high-quality transcript sequences
- cluster_info.tsv: read-to-candidate assignments
- not_converged.fa: reads that could not be corrected or clustered

## How to apply

After the get_candidates error-correction step, run stat_filter: greedily assign each FLNC read to its closest-neighbor candidate transcript using pairwise alignments (parasail), then extract the variant positions distinguishing each candidate pair and per-read error probabilities from Phred quality values. For each candidate pair compute the weighted-support statistic T = prod(1/p_i^{s_i}) from the reads' per-base error probabilities, and test P(T >= t) via the one-sided Chernoff-type upper bound; when observed support t' is below expected mu, the test is clearly insignificant and the software defaults the p-value to 0.5. Iteratively remove candidates whose significance value exceeds alpha (default 0.01), removing at most tau highest-p candidates per iteration; candidates differing by more than the default threshold of 10 variant positions are not statistically evaluated, and --min_test_ratio (default 5) omits testing candidate pairs where one has 5x more support. Use FASTQ rather than FASTA input so Phred qualities can feed the statistical analysis; write survivors to final_candidates.fa.

## Related tools

- **IsoCon** (Implements the get_candidates and stat_filter stages; stat_filter performs the statistical filtering of candidates with alpha=0.01) — https://github.com/ksahlin/IsoCon
- **parasail** (Used to create the pairwise alignments from each read to the cluster center used for greedy read-to-candidate assignment) — https://github.com/jeffdaily/parasail-python
- **edlib** (Dependency (>= 1.1.2) for fast edit-distance alignment in IsoCon) — https://github.com/Martinsos/edlib

## Examples

```
IsoCon pipeline -fl_reads simulated_pacbio_reads.fa -outfolder output -candidates candidate_transcripts.fa
```

## Evaluation signals

- Run stat_filter on the bundled small test dataset with -candidates and verify final_candidates.fa exists and contains transcript sequences
- Significance values for pairs where observed support t' <= mu must default to 0.5
- No candidate surviving with a p-value greater than alpha (default 0.01)
- Per-iteration removal caps at tau candidates; verify more than tau high-p candidates were not removed in one iteration
- On the FMR1 dataset, surviving isoform counts should match the paper's reported behavior (average 24 isoforms per sample, exceeding ICE-based ToFU's 20)

## Limitations

- Candidates with more than the default 10 variant positions relative to all others are not statistically evaluated
- IsoCon cannot conclusively distinguish alleles from distinct gene copies; downstream analysis is needed for gene-copy assignment
- Very low-abundance transcripts may be missed since deep PacBio sequencing can be cost-prohibitive
- RT-PCR error sources cannot be excluded; replicate cDNA libraries and further validation are needed

## Evidence

- [discussion] iterative removal of statistically non-significant candidates: [evidence span withheld — anchor status: not_found]
- [discussion] alpha default 0.01: "Then, we identify the candidates with pi greater than a significance threshold α. This α is a parameter to our algorithm, set by default to 0.01."
- [discussion] tau cap per iteration: "if there are more than τ candidates with significance value over α, we only remove the top τ candidates with the highest values."
- [discussion] p-value default 0.5 when support below expectation: "Such values are clearly insignificant, and our software defaults to a value of 0.5."
- [discussion] variant-position threshold of 10: "Candidate transcripts that have more than a threshold of variant positions (default of 10) relative to all other candidate transcripts are not statistically evaluated"
- [methods] stat_filter subcommand: [evidence span withheld — anchor status: not_found]
- [discussion] parasail pairwise alignments: "We first create pairwise alignments from each read to the center using parasail60"
- [readme] README: predicted transcripts location: "predicted transcripts are found in file **/path/to/output/final_candidates.fa**"
