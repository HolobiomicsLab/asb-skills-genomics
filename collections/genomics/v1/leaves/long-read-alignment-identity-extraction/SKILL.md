---
name: long-read-alignment-identity-extraction
description: Use when invMap inversion-call outputs must be compared against other
  long-read mapper call sets to isolate invMap-only calls and record their alignment
  identity, length, and anchors, e.g. the novel 231 bp inversion on NA12878 data.
when_to_use_negative:
- Input is an invMap call already reported by all competing mappers — such calls are
  not invMap-only and do not belong in the novel-inversion evidence table
- Analysis goal is mapping reads or generating SAM alignments themselves (use the
  invMap mapping step), not extracting identity/anchor evidence from existing calls
- Reads are short rather than long noisy PacBio/Oxford Nanopore reads, where inversion
  breakpoints are not frequently spanned
edam_operation: http://edamontology.org/operation_0292
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3175
tools:
- name: invMap
  role: primary mapper whose inversion-call outputs and chaining anchors supply the
    candidate novel inversions and alignment identities
  repo: https://github.com/zhang134/invMap.git
- name: Winnowmap2
  role: competing long-read mapper whose call set is used for interval-overlap comparison
    to confirm calls missed outside invMap
- name: minimap2
  role: competing long-read mapper whose call set is used for interval-overlap comparison
    to confirm calls missed outside invMap
- name: NGMLR
  role: competing long-read mapper whose call set is used for interval-overlap comparison
    to confirm calls missed outside invMap
- name: lra
  role: competing long-read mapper whose call set is used for interval-overlap comparison
    to confirm calls missed outside invMap
- name: PacBio
  role: long-read sequencing platform generating the input reads that span inversion
    breakpoints
- name: Oxford Nanopore
  role: long-read sequencing platform generating the input reads that span inversion
    breakpoints
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1093/bioinformatics/btad726
    title: 'invMap: a sensitive mapping tool for long noisy reads with inversion structural
      variants'
  source_content_hash: sha256:dac8ff37ccb9069ce30c7e2fed950e561b81cf4bd16a76b601a1b4a9a549ae47
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/zhang134/invMap.git
  tools:
  - NGMLR
  - Oxford Nanopore
  - PacBio
  - Winnowmap2
  - invMap
  - lra
  - minimap2
  edam_operation: http://edamontology.org/operation_0292
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3175
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7272727272727273
n_claims: 11
---

# long-read-alignment-identity-extraction

## Summary

Extracts alignment identity, inversion length, and supporting anchors for inversion calls produced by invMap that are missed by competing long-read mappers (Winnowmap2, minimap2, NGMLR, lra), building an evidence table of novel (non-ground-truth) inversions such as the 231 bp call from the NA12878 real human genome dataset.

## When to use

Use when invMap inversion-call outputs must be compared against other long-read mapper call sets to isolate invMap-only calls and record their alignment identity, length, and anchors, e.g. the novel 231 bp inversion on NA12878 data.

## When NOT to use

- Input is an invMap call already reported by all competing mappers — such calls are not invMap-only and do not belong in the novel-inversion evidence table
- Analysis goal is mapping reads or generating SAM alignments themselves (use the invMap mapping step), not extracting identity/anchor evidence from existing calls
- Reads are short rather than long noisy PacBio/Oxford Nanopore reads, where inversion breakpoints are not frequently spanned

## Inputs

- invMap inversion-call outputs (SAM-format invMap_aligned.sam and chaining/anchor output) for the NA12878 real human genome sequencing dataset
- inversion call sets from competing long-read mappers: Winnowmap2, minimap2, NGMLR, lra
- simulated ground-truth inversion annotations and Supplementary Table S5 / Figures S3-S4 references

## Outputs

- novel_inversions_evidence.csv with columns: alignment identity, inversion length, supporting anchors, and truth status (ground-truth vs non-ground-truth)
- mean identity statistic for non-ground-truth invMap-only calls (reported value: 81.95%)

## How to apply

Load the invMap inversion-call outputs for the NA12878 real human genome sequencing dataset together with the call sets from the competing mappers (Winnowmap2, minimap2, NGMLR, lra). Compare invMap calls against the other mappers' calls via interval overlap on the reference coordinates to isolate inversions predicted only by invMap. For each unique invMap-only call, record the alignment identity (mean identity of non-ground-truth calls was 81.95%), the inversion length (e.g. the 231 bp novel inversion not part of the simulated ground truth), and the supporting anchors from the invMap chaining output. Cross-reference the extracted calls with Supplementary Table S5 and Figures S3-S4 to label ground-truth-matching versus non-ground-truth calls. Write the records to a CSV (e.g. novel_inversions_evidence.csv) with columns for identity, length, anchors, and truth status.

## Related tools

- **invMap** (primary mapper whose inversion-call outputs and chaining anchors supply the candidate novel inversions and alignment identities) — https://github.com/zhang134/invMap.git
- **Winnowmap2** (competing long-read mapper whose call set is used for interval-overlap comparison to confirm calls missed outside invMap)
- **minimap2** (competing long-read mapper whose call set is used for interval-overlap comparison to confirm calls missed outside invMap)
- **NGMLR** (competing long-read mapper whose call set is used for interval-overlap comparison to confirm calls missed outside invMap)
- **lra** (competing long-read mapper whose call set is used for interval-overlap comparison to confirm calls missed outside invMap)
- **PacBio** (long-read sequencing platform generating the input reads that span inversion breakpoints)
- **Oxford Nanopore** (long-read sequencing platform generating the input reads that span inversion breakpoints)

## Examples

```
./invMap genome.fa reads.fa > invMap_aligned.sam
```

## Evaluation signals

- The evidence table reproduces the novel 231 bp inversion entry that is not part of the simulated ground truth
- The mean alignment identity of non-ground-truth invMap-only calls equals the reported 81.95%
- Every call labeled invMap-only fails interval overlap on reference coordinates against Winnowmap2, minimap2, NGMLR, and lra call sets (all four report alignment rather than the inversion)
- Each record carries non-empty supporting anchors from the invMap chaining output and an inversion length
- Truth-status labels are consistent with Supplementary Table S5 and Figures S3-S4 cross-referencing

## Limitations

- invMap requires a minimum of 16G of RAM for mapping sequence datasets to large reference genomes
- invMap v1.0.0 (2023/09/08) is a single-version release with no changelog found
- Calls detected only by invMap are not part of the simulated ground truth, so their correctness must be argued from identity and anchors rather than a truth label

## Evidence

- [other] For the new 231 bp inversion detected by invMap (which is not part of the simulated ground truth), Winnowmap2, minimap2, NGMLR, and lra fail to discover the new inversion and report the alignment, making it an invMap-only call suitable for the evidence table.: [evidence span withheld — anchor status: not_found]
- [abstract] invMap first locates the aligned region with a specifically designed scoring method for chaining, then checks the remaining anchors in the aligned region to discover potential inversions: "invMap first locates the aligned region with a specifically designed scoring method for chaining, then checks the remaining anchors in the aligned region"
- [abstract] The real human genome sequencing dataset of NA12878 illustrates that invMap can effectively find more candidate variant calls for inversions than the competing methods: "The real human genome sequencing dataset of NA12878 illustrates that invMap can effectively find more candidate variant calls for inversions than the competing methods"
- [intro] invMap requires a minimum of 16G of RAM for mapping sequence datasets to large reference genomes.: "invMap requires a minimum of 16G of RAM for mapping sequence datasets to large reference genomes."
- [readme] The final mapping result is invMap_aligned.sam in SAM format.: "The final mapping result is invMap_aligned.sam in SAM format."
