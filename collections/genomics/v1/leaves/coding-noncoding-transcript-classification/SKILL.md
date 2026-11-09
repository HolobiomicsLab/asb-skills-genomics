---
name: coding-noncoding-transcript-classification
description: Use when you have de novo transcript candidates (e.g. final_candidates.fa
  from IsoCon) for highly similar gene copies and need exon-level grouping, coding
  vs non-coding classification, and per-family counts.
when_to_use_negative:
- Reads have not been pre-processed with the Iso-Seq pipeline (primer/barcode removal,
  chimera filtering) — IsoCon assumes pre-processed input, so candidates would not
  be trustworthy
- The goal is definitive assignment of each transcript to an individual gene copy
  or determination of gene family size — transcriptome-only sequencing cannot conclusively
  provide this
- Input is already a curated annotation table rather than de novo reconstructed transcript
  sequences
edam_operation: http://edamontology.org/operation_0267
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0080
tools:
- name: IsoCon
  role: Produces the error-corrected, de-redundicated candidate transcripts (final_candidates.fa)
    that are grouped and classified
  repo: https://github.com/ksahlin/IsoCon
- name: parasail
  role: SIMD pairwise sequence alignment used to compare transcript pairs and detect
    exon-level (large-indel-only) differences for grouping
  repo: https://github.com/jeffdaily/parasail-python
- name: IsoCon_Eval
  role: Repository holding the analysis/evaluation scripts (with a snakemake workflow)
    that reproduce the tallies
  repo: https://github.com/ksahlin/IsoCon_Eval
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-018-06910-x
    title: Deciphering highly similar multigene family transcripts from Iso-Seq data
      with IsoCon
  source_content_hash: sha256:65c9dbec79774160ee01404e76c3d72f51bddb46437967138e35ffc384091767
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ksahlin/IsoCon
  tools:
  - IsoCon
  - IsoCon_Eval
  - parasail
  edam_operation: http://edamontology.org/operation_0267
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0080
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# coding-noncoding-transcript-classification

## Summary

Groups near-identical IsoCon-reconstructed transcripts from highly similar multigene family copies, classifies each transcript as coding or non-coding, and tallies per-family and per-group coding/non-coding counts plus novel-transcript and novel-splice-variant counts as reported in Table 3 of the FMR1-family Iso-Seq analysis.

## When to use

Use when you have de novo transcript candidates (e.g. final_candidates.fa from IsoCon) for highly similar gene copies and need exon-level grouping, coding vs non-coding classification, and per-family counts.

## When NOT to use

- Reads have not been pre-processed with the Iso-Seq pipeline (primer/barcode removal, chimera filtering) — IsoCon assumes pre-processed input, so candidates would not be trustworthy
- The goal is definitive assignment of each transcript to an individual gene copy or determination of gene family size — transcriptome-only sequencing cannot conclusively provide this
- Input is already a curated annotation table rather than de novo reconstructed transcript sequences

## Inputs

- final_candidates.fa — IsoCon's converged candidate transcripts in FASTA format from targeted PacBio Iso-Seq data of the two testis samples
- Gene-family membership assignments for each transcript (nine Y chromosome ampliconic gene families)
- NCBI nr/nt databases for novelty screening

## Outputs

- Transcript groups (maximal cliques) where members differ only by large, exon-level indels
- Per-family table of coding vs non-coding transcript counts (72 coding, 49 non-coding over 121 shared transcripts)
- Group tallies: 51 total groups and 38 coding-only groups
- Counts of novel transcripts (94) and novel splice variants (38) relative to nr/nt

## How to apply

Start from the converged candidate transcripts produced by the IsoCon error-correction pipeline (final_candidates.fa). Compute pairwise alignments between all transcript pairs with parasail (e.g. Needleman-Wunsch/semi-global with traceback) and flag pairs whose differences consist only of large indels, i.e. exon-level differences; build a grouping graph over transcripts and extract maximal cliques so that transcripts differing only by large indels fall into the same transcript group. Classify each transcript as coding or non-coding and tally per-family coding/non-coding transcript counts, total group counts, and coding-only group counts. Separately screen each transcript against the NCBI nr/nt databases to count novel transcripts and novel splice variants. Accept the result only when the tallies match the published values: 72 coding and 49 non-coding transcripts among the 121 shared IsoCon predictions, 51 total groups, 38 coding groups, 94 novel transcripts, and 38 novel splice variants; the 49 non-coding transcripts occur in five of the nine Y-chromosome ampliconic gene families while the other four families contain only coding transcripts.

## Related tools

- **IsoCon** (Produces the error-corrected, de-redundicated candidate transcripts (final_candidates.fa) that are grouped and classified) — https://github.com/ksahlin/IsoCon
- **parasail** (SIMD pairwise sequence alignment used to compare transcript pairs and detect exon-level (large-indel-only) differences for grouping) — https://github.com/jeffdaily/parasail-python
- **IsoCon_Eval** (Repository holding the analysis/evaluation scripts (with a snakemake workflow) that reproduce the tallies) — https://github.com/ksahlin/IsoCon_Eval

## Examples

```
import parasail; result = parasail.nw_trace(seq1, seq2, 11, 1, parasail.blosum62); cigar = result.cigar.decode
```

## Evaluation signals

- Derived coding/non-coding tallies equal the published Table 3 values: 72 coding and 49 non-coding among the 121 shared transcripts; any mismatch fails acceptance
- Grouping produces exactly 51 total transcript groups and 38 coding-only groups
- Novelty screening yields 94 novel transcripts and 38 novel splice variants
- The 49 non-coding transcripts map to exactly five of the nine gene families, with the remaining four families coding-only
- Every group member differs from other members only by large indels (exon-level differences), not substitutions or small indels, per the parasail alignment CIGARs

## Limitations

- IsoCon provides no built-in functionality to distinguish alleles from distinct gene copies; downstream analysis is required for that distinction
- Novel gene copies cannot be definitively distinguished from splice variants or RNA-editing effects
- Very low-abundance transcripts might not be captured by Iso-Seq, since deep PacBio sequencing can be cost-prohibitive
- RT-PCR error sources cannot be excluded; replicate cDNA libraries and further validation would still be needed

## Evidence

- [discussion] Pairwise alignment for grouping and exon-level difference detection: "We first create pairwise alignments from each read to the center using parasail60"
- [discussion] IsoCon availability and analyzed version: "IsoCon is open-source and freely available at https://github.com/ksahlin/IsoCon. The results of IsoCon in this paper were obtained with commit 79589f3 on GitHub."
- [readme] Candidate transcript output file location: [evidence span withheld — anchor status: external_doc]
- [discussion] Candidate derivation by iterative error correction and clustering: "an iterative clustering algorithm to error-correct the reads and identify candidate transcripts"
- [readme] IsoCon scope: reconstructing highly similar gene-family transcripts end-to-end: "IsoCon is a tool for reconstructing highly similar sequences present in a dataset of from long noisy reads. Its original use case was transcripts from highly similar gene copies"
