---
name: splice-variant-novelty-counting
description: Use when you have IsoCon final_candidates.fa transcripts from targeted
  PacBio Iso-Seq sequencing of the nine Y chromosome ampliconic gene families and
  need to tally coding/non-coding counts per family and classify novel transcripts
  vs novel splice variants against NCBI nr/nt.
when_to_use_negative:
- The input reads are raw PacBio CCS/Iso-Seq reads that have not yet been run through
  IsoCon — run IsoCon first to obtain final_candidates.fa before grouping or novelty
  counting.
- The goal is a definitive determination of gene family size or definitive assignment
  of each transcript to an individual gene copy — the paper states transcriptome-only
  sequencing 'does not provide a definitive answer' for either.
- The goal is distinguishing alleles from distinct gene copies — IsoCon output requires
  further downstream analysis for this, so this counting skill alone is insufficient.
edam_operation: http://edamontology.org/operation_0306
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_3168
tools:
- name: IsoCon
  role: Produces the final candidate transcripts (final_candidates.fa) that are grouped
    and classified by this skill
  repo: https://github.com/ksahlin/IsoCon
- name: parasail
  role: Computes the pairwise alignments between transcript pairs used to detect exon-level
    (large-indel) differences for grouping
  repo: https://github.com/jeffdaily/parasail-python
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-018-06910-x
    title: Deciphering highly similar multigene family transcripts from Iso-Seq data
      with IsoCon
  source_content_hash: sha256:c8c8eb082307121b7a559f80d5e4ddcd22b8b384a1be51e4f6e3e8b7be9d2268
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ksahlin/IsoCon
  tools:
  - IsoCon
  - parasail
  edam_operation: http://edamontology.org/operation_0306
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_3168
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# splice-variant-novelty-counting

## Summary

Reproduces the paper's post-IsoCon novelty accounting: transcripts shared between testis samples are grouped by exon-level (large-indel) differences, classified as coding vs non-coding, and screened against NCBI nr/nt to count novel transcripts separately from novel splice variants (Table 3: 72 coding, 49 non-coding, 51 groups, 38 coding groups, 94 novel transcripts, 38 novel splice variants).

## When to use

Use when you have IsoCon final_candidates.fa transcripts from targeted PacBio Iso-Seq sequencing of the nine Y chromosome ampliconic gene families and need to tally coding/non-coding counts per family and classify novel transcripts vs novel splice variants against NCBI nr/nt.

## When NOT to use

- The input reads are raw PacBio CCS/Iso-Seq reads that have not yet been run through IsoCon — run IsoCon first to obtain final_candidates.fa before grouping or novelty counting.
- The goal is a definitive determination of gene family size or definitive assignment of each transcript to an individual gene copy — the paper states transcriptome-only sequencing 'does not provide a definitive answer' for either.
- The goal is distinguishing alleles from distinct gene copies — IsoCon output requires further downstream analysis for this, so this counting skill alone is insufficient.

## Inputs

- final_candidates.fa — high-quality transcript output of the IsoCon pipeline (121 shared transcripts between the two testis samples)
- NCBI nr/nt databases (for novelty screening)
- Coding/non-coding classification per transcript (e.g., from coding-potential assessment)

## Outputs

- Per-family tallies of coding vs non-coding transcripts (72 coding, 49 non-coding; non-coding present in 5 of 9 families)
- Transcript group counts: 51 total groups, 38 coding-only groups
- Novelty counts: 94 novel transcripts and 38 novel splice variants

## How to apply

Load the 121 final candidate transcripts produced by the IsoCon pipeline (output file final_candidates.fa). Compute pairwise alignments between all transcript pairs with parasail and identify pairs whose differences consist only of large indels (i.e., exon-level differences); build a grouping graph over the transcripts and extract maximal cliques so that transcripts differing only by large indels land in the same transcript group (51 groups total, 38 coding-only). Classify each transcript as coding or non-coding (72 coding, 49 non-coding), tally per-family counts across the nine gene families, and note that the 49 non-coding transcripts fall in only five of the nine families. Separately screen each transcript against the NCBI nr/nt databases to flag novel transcripts and novel splice variants, counting the two categories independently (94 and 38 respectively). Acceptance requires the derived tallies to match the published Table 3 values exactly.

## Related tools

- **IsoCon** (Produces the final candidate transcripts (final_candidates.fa) that are grouped and classified by this skill) — https://github.com/ksahlin/IsoCon
- **parasail** (Computes the pairwise alignments between transcript pairs used to detect exon-level (large-indel) differences for grouping) — https://github.com/jeffdaily/parasail-python

## Examples

```
import parasail; aln = parasail.sg_trace(t1, t2, 11, 1, parasail.blosum62); # group pairs whose CIGAR shows only large indels, then screen sequences vs NCBI nr/nt
```

## Evaluation signals

- Coding/non-coding tallies over the 121 shared transcripts must equal 72 coding and 49 non-coding (Table 3).
- Group counts must equal 51 total groups and 38 coding-only groups.
- Novelty screening must yield exactly 94 novel transcripts and 38 novel splice variants.
- The 49 non-coding transcripts must be distributed across exactly five of the nine gene families, with the other four families containing only coding transcripts.
- Every clique in the grouping graph should contain only transcript pairs whose parasail alignment differences are exclusively large indels (exon-level differences), not substitutions or small indels.

## Limitations

- Novel gene copies are difficult to distinguish from splice variants or RNA-editing in this screen.
- Transcriptome-only sequencing does not provide a definitive answer to gene family size nor conclusively assign each transcript to a gene copy.
- Very low-abundance transcripts might not be captured by Iso-Seq, since deep PacBio sequencing can be cost-prohibitive.
- RT-PCR error sources cannot be excluded; replicate cDNA libraries and further validation would still be needed.

## Evidence

- [discussion] parasail used for pairwise alignments underlying grouping: "We first create pairwise alignments from each read to the center using parasail60"
- [discussion] novel gene copies cannot be cleanly distinguished from splice variants: "it is difficult to distinguish novel gene copies from splice variants or RNA-editing."
- [discussion] transcriptome-only data cannot definitively size families or assign copies: "does not provide a definitive answer to the size of each gene family and does not allow us to conclusively assign each transcript to a gene copy."
- [readme] IsoCon output file holding the transcripts to be grouped/counted: "predicted transcripts are found in file **/path/to/output/final_candidates.fa**"
- [methods] candidate transcripts converge to a fasta output from the error-correction step: "Runs only the error correction step. The output is the converged candidates in a fasta file."
- [intro] scope: nine Y chromosome ampliconic gene families from two men: "We use a targeted design to isolate and sequence transcripts from all nine Y chromosome ampliconic gene families from the testes of two men."
