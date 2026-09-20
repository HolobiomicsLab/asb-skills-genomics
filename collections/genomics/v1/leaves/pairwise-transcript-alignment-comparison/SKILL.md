---
name: pairwise-transcript-alignment-comparison
description: Use when you have error-corrected long-read transcript sequences from
  highly similar gene copies and need to detect pairs whose differences consist only
  of large indels (exon-level differences) rather than point substitutions or small
  indels.
when_to_use_negative:
- Transcripts from a well-annotated reference genome where a reference-based isoform
  comparison (e.g., exon-coordinate overlap) is available instead of de novo pairwise
  alignment
- Alignment of highly divergent sequences where differences are not limited to large
  indels — grouping by cliques would merge unrelated transcripts
- Uncorrected or error-prone long reads (e.g., raw PacBio CCS inputs) — the comparison
  assumes transcripts already error-corrected by IsoCon so that only true exon-level
  differences remain
edam_operation: http://edamontology.org/operation_0294
edam_topics:
- http://edamontology.org/topic_0080
- http://edamontology.org/topic_3308
tools:
- name: parasail
  role: SIMD pairwise sequence alignment library used to compute pairwise alignments
    between transcript pairs and inspect difference patterns (large indels only vs.
    substitutions)
  repo: https://github.com/jeffdaily/parasail-python
- name: IsoCon
  role: Produces the error-corrected, redundant-collapsed transcript set (final_candidates.fa)
    that serves as input for pairwise comparison and grouping
  repo: https://github.com/ksahlin/IsoCon
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-018-06910-x
    title: Deciphering highly similar multigene family transcripts from Iso-Seq data
      with IsoCon
  source_content_hash: sha256:3d46a214fb887c40fa2e2b7ec254a17446d948f60a1d58670379eec4a2cf7629
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/jeffdaily/parasail-python
  tools:
  - IsoCon
  - parasail
  edam_operation: http://edamontology.org/operation_0294
  edam_topics:
  - http://edamontology.org/topic_0080
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 4
---

# Pairwise transcript alignment comparison

## Summary

Computes pairwise alignments between reconstructed transcripts (e.g., IsoCon final_candidates.fa output) using parasail, and inspects alignment differences to determine whether two transcripts differ only by large indels (exon-level differences), enabling grouping of highly similar transcripts from multigene family Iso-Seq data.

## When to use

Use when you have error-corrected long-read transcript sequences from highly similar gene copies and need to detect pairs whose differences consist only of large indels (exon-level differences) rather than point substitutions or small indels.

## When NOT to use

- Transcripts from a well-annotated reference genome where a reference-based isoform comparison (e.g., exon-coordinate overlap) is available instead of de novo pairwise alignment
- Alignment of highly divergent sequences where differences are not limited to large indels — grouping by cliques would merge unrelated transcripts
- Uncorrected or error-prone long reads (e.g., raw PacBio CCS inputs) — the comparison assumes transcripts already error-corrected by IsoCon so that only true exon-level differences remain

## Inputs

- Error-corrected transcript sequences from the IsoCon pipeline output file final_candidates.fa (FASTA format)
- Gene family labels for each transcript (for per-family tallies)
- Coding/non-coding classification per transcript

## Outputs

- Set of transcript pairs whose alignment differences consist only of large indels (exon-level differences)
- Grouping graph over transcripts with maximal cliques defining transcript groups
- Per-family tallies of coding/non-coding transcript counts, total group counts, and coding-only group counts (e.g., Table 3 values)

## How to apply

Load the error-corrected transcripts from the IsoCon output file final_candidates.fa. Run pairwise alignments between all transcript pairs using parasail, examining each alignment's differences. Retain pairs whose differences consist only of large indels — these represent exon-level differences (e.g., exon deletion or retention within highly similar gene copies) rather than true point-variant transcripts. Build a grouping graph where an edge connects every such pair and extract maximal cliques so that transcripts differing only by large indels are assigned to the same transcript group. The rationale is that IsoCon-derived transcripts from ampliconic gene families (e.g., the nine Y chromosome gene families sequenced from two testis samples) can differ from each other only by whole-exon events; pairwise alignment inspection with parasail is what distinguishes these from genuinely distinct sequences. The grouped transcripts can then be tallied per family and classified as coding vs non-coding for comparison against published counts.

## Related tools

- **parasail** (SIMD pairwise sequence alignment library used to compute pairwise alignments between transcript pairs and inspect difference patterns (large indels only vs. substitutions)) — https://github.com/jeffdaily/parasail-python
- **IsoCon** (Produces the error-corrected, redundant-collapsed transcript set (final_candidates.fa) that serves as input for pairwise comparison and grouping) — https://github.com/ksahlin/IsoCon

## Examples

```
import parasail; aln = parasail.nw_trace(s1, s2, 11, 1, parasail.blosum62); print(aln.cigar.decode)
```

## Evaluation signals

- Applying the grouping to the 121 transcripts IsoCon predicted identically in both testis samples should yield 72 coding and 49 non-coding transcripts (Table 3)
- Total number of extracted groups should equal 51, of which 38 contain only coding transcripts
- Non-coding transcripts should occur in exactly five of the nine gene families; the remaining four families should have only coding transcripts
- Downstream novelty screening against NCBI nr/nt should count 94 novel transcripts and 38 novel splice variants
- Each clique in the grouping graph should contain only transcripts whose pairwise parasail alignments show differences consisting solely of large indels — no substitution or small-indel differences

## Limitations

- IsoCon provides no functionality to distinguish alleles from distinct gene copies; further downstream analysis would be needed to assign transcripts to individual copies
- The analysis cannot conclusively determine gene family size or per-transcript gene-copy assignment from transcriptome-only sequencing
- Novel gene copies are difficult to distinguish from splice variants or RNA-editing effects
- Very low-abundance transcripts might not be captured by Iso-Seq, since deep PacBio sequencing can be cost-prohibitive

## Evidence

- [discussion] Pairwise alignments via parasail used to group transcripts: "We first create pairwise alignments from each read to the center using parasail60"
- [full_text] Grouping criterion and Table 3 tallies: [evidence span withheld — anchor status: not_found]
- [readme] IsoCon output file used as input for grouping: "predicted transcripts are found in file **/path/to/output/final_candidates.fa**"
- [readme] parasail Python API for pairwise alignment: [evidence span withheld — anchor status: external_doc]
