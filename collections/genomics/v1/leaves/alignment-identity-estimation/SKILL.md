---
name: alignment-identity-estimation
description: Use when performing hybrid long-read correction (e.g., PacBio or Oxford
  Nanopore reads aligned to Illumina short reads) and you must quantify each aligned
  short read's mapping identity and base quality to select an optimal correction policy
  for questionable bases.
when_to_use_negative:
- Bases already covered by at least 90% consensus (η = 0.9) should be handled by HECIL's
  quick correction stage, not this optimization-based correction
- Self-correction scenarios with no second-generation short reads (e.g., HGAP-style
  correction from long reads alone) — this weight scheme requires aligned Illumina
  short reads with PHRED qualities
- If localized per-read information (base quality, mapping identity) is intentionally
  ignored, as in hybrid correctors that do not systematically use it — this skill
  is specifically the localized-weight approach
edam_operation: http://edamontology.org/operation_0292
edam_topics:
- http://edamontology.org/topic_0102
- http://edamontology.org/topic_3168
tools:
- name: HECIL
  role: 'Implements the optimization-based correction: computes quality and mapping-identity
    decision weights, cost J_k, and argmax selection of the winning short read'
  repo: https://github.com/NDBL/HECIL
- name: BWA
  role: Produces the short-read-to-long-read alignments (BWA-MEM) from which exact-match
    counts and mapping identity are derived
- name: Samtools
  role: Processes the BWA alignment output consumed by HECIL for marking questionable
    bases
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41598-018-28364-3
    title: 'HECIL: A Hybrid Error Correction Algorithm for Long Reads with Iterative
      Learning'
  source_content_hash: sha256:c9d74a6ae95639fd395ef0903620491742778b8da4ae613adf57d06307fff75e
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/NDBL/HECIL
  tools:
  - BWA
  - BWA-MEM
  - HECIL
  - Samtools
  edam_operation: http://edamontology.org/operation_0292
  edam_topics:
  - http://edamontology.org/topic_0102
  - http://edamontology.org/topic_3168
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8181818181818182
n_claims: 11
---

# alignment-identity-estimation

## Summary

HECIL's method for estimating a normalized mapping-identity weight (exact matches of an aligned Illumina short read to the long read, divided by the short read's length) that is combined with a PHRED-quality weight into the cost J_k = 1/2*(quality + similarity) used to select the winning correction for questionable long-read bases.

## When to use

Use when performing hybrid long-read correction (e.g., PacBio or Oxford Nanopore reads aligned to Illumina short reads) and you must quantify each aligned short read's mapping identity and base quality to select an optimal correction policy for questionable bases.

## When NOT to use

- Bases already covered by at least 90% consensus (η = 0.9) should be handled by HECIL's quick correction stage, not this optimization-based correction
- Self-correction scenarios with no second-generation short reads (e.g., HGAP-style correction from long reads alone) — this weight scheme requires aligned Illumina short reads with PHRED qualities
- If localized per-read information (base quality, mapping identity) is intentionally ignored, as in hybrid correctors that do not systematically use it — this skill is specifically the localized-weight approach

## Inputs

- Short-read-to-long-read alignment file (SAM/BAM) from BWA-MEM
- Illumina short reads in FASTQ with PHRED quality scores
- Erroneous long reads (PacBio/Oxford Nanopore) in FASTA
- Short read length parameter (-len, e.g., 202 in the README example)

## Outputs

- Winning short read k* and its cost J_k per questionable base
- Corrected base on the long read (replaced by the aligned base of the selected short read)
- Per-correction log (corrected_bases.tsv) feeding confidence assignment and iterative realignment

## How to apply

Load the short-read-to-long-read alignments produced by BWA-MEM (BWA 0.7.15, processed with Samtools 0.1.18 per the HECIL README) and mark positions with mismatches, insertions, or deletions on the long read as questionable bases. For each questionable base, collect the N aligned short reads; compute the normalized similarity weight as the number of exact matches of the short read to the long read divided by the short read's length, and the normalized quality weight by extracting each short read's PHRED quality from the FASTQ and dividing by the maximum quality among the aligning reads. Form the cost J_k as the equal-weight (1/2, 1/2) convex combination of the two normalized weights and select k* = argmax over the aligning reads, breaking ties by choosing the read with the highest quality. Replace the questionable base with the corresponding aligned base of the winning short read, and record each correction (e.g., in corrected_bases.tsv) for downstream confidence assignment and iterative realignment. This is applied only to bases that fall below the 90% consensus threshold (η = 0.9) and are therefore ineligible for HECIL's quick correction stage.

## Related tools

- **HECIL** (Implements the optimization-based correction: computes quality and mapping-identity decision weights, cost J_k, and argmax selection of the winning short read) — https://github.com/NDBL/HECIL
- **BWA** (Produces the short-read-to-long-read alignments (BWA-MEM) from which exact-match counts and mapping identity are derived)
- **Samtools** (Processes the BWA alignment output consumed by HECIL for marking questionable bases)

## Examples

```
python HECIL.py -l LongRead.fa -s ShortRead.fq -len 202 -o Out
```

## Evaluation signals

- Verify J_k equals the equal-weight (1/2, 1/2) convex combination of the normalized quality weight and the normalized similarity weight
- Check both normalized weights lie in [0, 1]: similarity = exact matches / short read length; quality = PHRED score / maximum quality among aligning reads
- Confirm the argmax rule is applied with ties broken by choosing the read with the highest quality
- Confirm each output corrected base equals the aligned base of the selected short read at that position
- Confirm optimization-based correction is only invoked for bases failing the 90% (η = 0.9) consensus quick-correction threshold

## Limitations

- The optimization stage only handles questionable bases that remain after quick correction; it is not a full-read corrector on its own
- Iterative improvement of the correction policy via learned confidence is a conjecture without mathematical proof
- Multi-threaded speedup is described as a future extension; the README version requires external BWA/Samtools binaries to be made executable (chmod +x) after cloning

## Evidence

- [methods] For the remaining questionable bases, we employ an optimization-based correction framework.: "For the remaining questionable bases, we employ an optimization-based correction framework."
- [intro] a core algorithm that selects a correction policy by leveraging an optimal combination of decision weights based on base quality and mapping identity of aligned short reads: "a core algorithm that selects a correction policy by leveraging an optimal combination of decision weights based on base quality and mapping identity of aligned short reads"
- [methods] we adopt a stricter threshold of at least 90% consensus (η = 0.9) to be eligible for quick correction: "we adopt a stricter threshold of at least 90% consensus (η = 0.9) to be eligible for quick correction"
- [intro] Most of the methods listed here do not systematically utilize localized information such as base quality of the short reads or variant information between individuals.: "Most of the methods listed here do not systematically utilize localized information such as base quality of the short reads or variant information between individuals."
- [readme] usage: python HECIL.py [-h] -l LR -s SR -len LENGTH_SR -o OUTPUT [-lc LOW_CONFIDENCE]: "usage: python HECIL.py [-h] -l LR -s SR -len LENGTH_SR -o OUTPUT [-lc LOW_CONFIDENCE]"
