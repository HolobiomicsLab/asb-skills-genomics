---
name: ncbi-nrnt-novelty-screening
description: Use when a set of de novo reconstructed transcripts (e.g., IsoCon final_candidates.fa
  from targeted Iso-Seq of multigene families) must be checked for prior existence
  in public sequence databases, distinguishing wholly novel transcripts from novel
  splice variants of known ones.
when_to_use_negative:
- The goal is to assign transcripts to individual gene copies or determine gene family
  size — transcriptome-only sequencing cannot conclusively do this, per the paper
- The goal is to distinguish alleles from distinct gene copies — IsoCon output requires
  further downstream analysis beyond this screening for that purpose
- The input reads have not been pre-processed with the Iso-Seq pipeline (primer/barcode
  removal, chimera filtering) and error-corrected by IsoCon, so the transcript set
  is not yet defined
edam_operation: http://edamontology.org/operation_0292
edam_topics:
- http://edamontology.org/topic_0080
tools:
- name: IsoCon
  role: Produces the final_candidates.fa transcript set that is screened against nr/nt;
    run at commit 79589f3 for the paper's results
  repo: https://github.com/ksahlin/IsoCon
- name: parasail
  role: Builds pairwise alignments used to group transcripts differing only by large
    indels prior to per-family novelty tallying
  repo: https://github.com/jeffdaily/parasail-python
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-018-06910-x
    title: Deciphering highly similar multigene family transcripts from Iso-Seq data
      with IsoCon
  source_content_hash: sha256:5b771e2806146d8c56bfd9553e91a2860c85750c5e63795039a87401b8071bac
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ksahlin/IsoCon
  tools:
  - IsoCon
  - parasail
  edam_operation: http://edamontology.org/operation_0292
  edam_topics:
  - http://edamontology.org/topic_0080
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5714285714285714
n_claims: 7
---

# ncbi-nrnt-novelty-screening

## Summary

Screens IsoCon-derived transcript sequences (final_candidates.fa) against the NCBI nr/nt databases to flag novel transcripts and novel splice variants, counting the two classes separately. In the paper this yielded 94 novel transcripts and 38 novel splice variants among the Y chromosome ampliconic gene family transcripts.

## When to use

Use when a set of de novo reconstructed transcripts (e.g., IsoCon final_candidates.fa from targeted Iso-Seq of multigene families) must be checked for prior existence in public sequence databases, distinguishing wholly novel transcripts from novel splice variants of known ones.

## When NOT to use

- The goal is to assign transcripts to individual gene copies or determine gene family size — transcriptome-only sequencing cannot conclusively do this, per the paper
- The goal is to distinguish alleles from distinct gene copies — IsoCon output requires further downstream analysis beyond this screening for that purpose
- The input reads have not been pre-processed with the Iso-Seq pipeline (primer/barcode removal, chimera filtering) and error-corrected by IsoCon, so the transcript set is not yet defined

## Inputs

- final_candidates.fa — FASTA of IsoCon-reconstructed candidate transcripts (high-quality output of the IsoCon pipeline)
- NCBI nr/nt sequence databases
- Optionally, transcript-group definitions from parasail-based large-indel grouping across samples (e.g., the 121 shared testis transcripts)

## Outputs

- Per-transcript novelty flags distinguishing novel transcripts from novel splice variants
- Separate counts of novel transcripts and novel splice variants (paper values: 94 and 38)
- Per-gene-family tallies of coding/non-coding transcripts and group counts consistent with Table 3 (72 coding / 49 non-coding transcripts, 51 total groups, 38 coding groups)

## How to apply

Start from the high-quality transcript output of the IsoCon pipeline, final_candidates.fa, which contains candidate transcripts that survived statistical filtering (significance threshold alpha = 0.01, candidates with more than 10 variant positions not statistically evaluated, --min_test_ratio default 5). For cross-sample analyses, first group transcripts that differ only by large indels (exon-level differences) using parasail pairwise alignments and maximal-clique extraction on a grouping graph, as done for the 121 transcripts shared between the two testis samples. Classify each transcript as coding or non-coding (the 121 shared transcripts split into 72 coding and 49 non-coding), then screen each transcript against the NCBI nr/nt databases. Flag transcripts with no database support as novel transcripts and transcripts supported as genes but with unmatched splice structure as novel splice variants, tallying the two categories separately. Acceptance requires the derived tallies to equal the published Table 3 values, including 94 novel transcripts and 38 novel splice variants.

## Related tools

- **IsoCon** (Produces the final_candidates.fa transcript set that is screened against nr/nt; run at commit 79589f3 for the paper's results) — https://github.com/ksahlin/IsoCon
- **parasail** (Builds pairwise alignments used to group transcripts differing only by large indels prior to per-family novelty tallying) — https://github.com/jeffdaily/parasail-python

## Examples

```
IsoCon pipeline -fl_reads <reads.fastq> -outfolder </path/to/output>   # screen /path/to/output/final_candidates.fa against NCBI nr/nt
```

## Evaluation signals

- Derived counts of novel transcripts and novel splice variants equal the published Table 3 values (94 novel transcripts, 38 novel splice variants); any mismatch fails acceptance
- Novel transcripts and novel splice variants are counted as separate categories, never merged into a single novelty tally
- Upstream invariant holds: the 121 shared transcripts between the two testis samples split into 72 coding and 49 non-coding, with the 49 non-coding transcripts occurring in five of the nine gene families
- Every transcript in final_candidates.fa receives a screen result (novel transcript, novel splice variant, or known) with no unclassified leftovers

## Limitations

- It is difficult to distinguish novel gene copies from splice variants or RNA-editing, so novelty calls may conflate these origins
- Very low-abundance transcripts might not be captured by Iso-Seq, since deep PacBio sequencing can be cost-prohibitive, so the novelty screen can miss rare isoforms
- IsoCon has no functionality to distinguish alleles from distinct gene copies; further downstream analysis would be needed for that

## Evidence

- [abstract] IsoCon has allowed us to detect an unprecedented number of novel isoforms: "IsoCon has allowed us to detect an unprecedented number of novel isoforms"
- [readme] predicted transcripts are found in final_candidates.fa: "predicted transcripts are found in file **/path/to/output/final_candidates.fa**"
- [other] Screen each transcript against the NCBI nr/nt databases to flag novel transcripts and novel splice variants, counting them separately: [evidence span withheld — anchor status: not_found]
- [discussion] it is difficult to distinguish novel gene copies from splice variants or RNA-editing: "it is difficult to distinguish novel gene copies from splice variants or RNA-editing."
- [discussion] IsoCon is open-source and results obtained with commit 79589f3: "IsoCon is open-source and freely available at https://github.com/ksahlin/IsoCon. The results of IsoCon in this paper were obtained with commit 79589f3 on GitHub."
