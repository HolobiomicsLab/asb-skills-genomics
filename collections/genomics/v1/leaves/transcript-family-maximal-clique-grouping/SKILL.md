---
name: transcript-family-maximal-clique-grouping
description: Use when you have IsoCon final_candidates.fa transcripts from targeted
  Iso-Seq sequencing of highly similar gene copies (e.g., Y chromosome ampliconic
  families) and need to collapse exon-level isoforms into transcript groups and count
  coding vs non-coding members per family.
when_to_use_negative:
- Transcripts come from distinct, non-duplicated genes where isoform redundancy is
  already resolved by a reference annotation — clique grouping would merge genuinely
  different isoforms or be unnecessary.
- Differences between transcripts include substitutions or short indels rather than
  only large exon-level indels — such pairs must not be grouped together.
- Reads have not been pre-processed by the Iso-Seq pipeline (primer/barcode removal,
  chimera filtering), so IsoCon candidates are not trustworthy inputs.
edam_operation: http://edamontology.org/operation_0292
edam_topics:
- http://edamontology.org/topic_0080
- http://edamontology.org/topic_3308
tools:
- name: IsoCon
  role: Produces the final candidate transcripts (final_candidates.fa) that are grouped
    and tallied by this skill
  repo: https://github.com/ksahlin/IsoCon
- name: parasail
  role: Python/C SIMD library used to compute pairwise alignments between transcript
    pairs to detect large-indel (exon-level) differences
  repo: https://github.com/jeffdaily/parasail-python
- name: IsoCon_Eval
  role: Repository containing the snakemake-driven analysis/evaluation scripts implementing
    the paper's tallies and validation workflow
  repo: https://github.com/ksahlin/IsoCon_Eval
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-018-06910-x
    title: Deciphering highly similar multigene family transcripts from Iso-Seq data
      with IsoCon
  source_content_hash: sha256:b5cc8b250bcf98e40dd9e46ca27d957c9a8deb718f25b6f347e6ace11ebee0da
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ksahlin/IsoCon
  tools:
  - IsoCon
  - IsoCon_Eval
  - parasail
  edam_operation: http://edamontology.org/operation_0292
  edam_topics:
  - http://edamontology.org/topic_0080
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 8
---

# transcript-family-maximal-clique-grouping

## Summary

Groups IsoCon-reconstructed transcripts from highly similar multigene family copies that differ only by exon-level (large-indel) differences, by building a graph over transcript pairs aligned with parasail and extracting maximal cliques, then tallying coding/non-coding transcripts per gene family to reproduce the paper's Table 3 counts.

## When to use

Use when you have IsoCon final_candidates.fa transcripts from targeted Iso-Seq sequencing of highly similar gene copies (e.g., Y chromosome ampliconic families) and need to collapse exon-level isoforms into transcript groups and count coding vs non-coding members per family.

## When NOT to use

- Transcripts come from distinct, non-duplicated genes where isoform redundancy is already resolved by a reference annotation — clique grouping would merge genuinely different isoforms or be unnecessary.
- Differences between transcripts include substitutions or short indels rather than only large exon-level indels — such pairs must not be grouped together.
- Reads have not been pre-processed by the Iso-Seq pipeline (primer/barcode removal, chimera filtering), so IsoCon candidates are not trustworthy inputs.

## Inputs

- final_candidates.fa — IsoCon-predicted (error-corrected) transcripts from targeted Iso-Seq data
- Transcript classification labels (coding vs non-coding) per transcript
- Gene family assignment for each transcript (e.g., nine Y chromosome ampliconic families)
- NCBI nr/nt database for novelty screening

## Outputs

- Transcript groups (maximal cliques of transcripts differing only by large indels)
- Per-family tallies of coding and non-coding transcripts and groups (e.g., 72 coding / 49 non-coding; 51 total groups; 38 coding groups)
- Counts of novel transcripts (94) and novel splice variants (38) flagged against nr/nt

## How to apply

Load the final candidate transcripts from final_candidates.fa, the high-quality output of the IsoCon pipeline. Compute pairwise alignments between all transcript pairs using parasail and identify pairs whose differences consist only of large indels, i.e., exon-level differences rather than point substitutions. Build a grouping graph over the transcripts with edges connecting transcripts differing only by large indels, and extract maximal cliques so that such transcripts are assigned to the same transcript group. Classify each transcript as coding or non-coding and tally per-family coding/non-coding transcript counts, total group counts, and coding-only group counts. Screen transcripts against NCBI nr/nt to separately count novel transcripts and novel splice variants. Validate against the published Table 3 values: 72 coding and 49 non-coding of the 121 shared transcripts, 51 total groups, 38 coding groups, 94 novel transcripts, and 38 novel splice variants.

## Related tools

- **IsoCon** (Produces the final candidate transcripts (final_candidates.fa) that are grouped and tallied by this skill) — https://github.com/ksahlin/IsoCon
- **parasail** (Python/C SIMD library used to compute pairwise alignments between transcript pairs to detect large-indel (exon-level) differences) — https://github.com/jeffdaily/parasail-python
- **IsoCon_Eval** (Repository containing the snakemake-driven analysis/evaluation scripts implementing the paper's tallies and validation workflow) — https://github.com/ksahlin/IsoCon_Eval

## Examples

```
import parasail; result = parasail.nw_trace(tx1, tx2, 10, 1, parasail.nucleotide); cigar = result.cigar.decode.decode()
```

## Evaluation signals

- Tallies must equal published Table 3 values: 72 coding and 49 non-coding transcripts among the 121 shared transcripts, 51 total groups, and 38 coding groups; any mismatch fails acceptance.
- The 49 non-coding transcripts must be distributed over five of the nine gene families, with the other four families containing only coding transcripts.
- Novelty counts against NCBI nr/nt must yield 94 novel transcripts and 38 novel splice variants, counted separately.
- Every pair within a clique should differ only by large indels (exon-level differences); any pair with substitutions or short indels indicates an incorrect grouping edge.
- Group partition sanity: transcripts belong to exactly one maximal clique/group after extraction from the grouping graph.

## Limitations

- Transcriptome-only sequencing does not provide a definitive answer to the size of each gene family and does not allow conclusively assigning each transcript to a gene copy.
- IsoCon has no functionality to distinguish alleles from distinct gene copies; further downstream analysis is needed for that purpose.
- Novel gene copies are difficult to distinguish from splice variants or RNA-editing, so novelty tallies may conflate these categories.
- Very low-abundance transcripts might not be captured by Iso-Seq, since deep PacBio sequencing can be cost-prohibitive.

## Evidence

- [discussion] We first create pairwise alignments from each read to the center using parasail60: "We first create pairwise alignments from each read to the center using parasail60"
- [abstract] We apply IsoCon to nine Y chromosome ampliconic gene families and show that it outperforms existing methods on both experimental and simulated data.: "We apply IsoCon to nine Y chromosome ampliconic gene families and show that it outperforms existing methods on both experimental and simulated data."
- [intro] We use a targeted design to isolate and sequence transcripts from all nine Y chromosome ampliconic gene families from the testes of two men.: "We use a targeted design to isolate and sequence transcripts from all nine Y chromosome ampliconic gene families from the testes of two men."
- [readme] predicted transcripts are found in file **/path/to/output/final_candidates.fa**: "predicted transcripts are found in file **/path/to/output/final_candidates.fa**"
- [discussion] IsoCon assumes the reads have been pre-processed with the Iso-Seq bioinformatic pipeline to remove primers, barcodes, and reads that are chimeras: "IsoCon assumes the reads have been pre-processed with the Iso-Seq bioinformatic pipeline to remove primers, barcodes, and reads that are chimeras"
