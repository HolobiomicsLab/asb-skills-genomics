---
name: cross-sample-transcript-sequence-matching
description: Use when two or more samples' reconstructed transcript sets (e.g., IsoCon
  vs ICE outputs as FASTA) must be intersected by exact sequence identity, or when
  counting exact nucleotide matches to known reference CDSs (e.g., 61 Ensembl coding
  sequences of nine Y ampliconic gene families).
when_to_use_negative:
- Transcripts need assignment to individual gene copies or family-size estimation
  — the article notes transcriptome-only sequencing does not conclusively assign transcripts
  to gene copies and further downstream analysis is required
- Input reads are not pre-processed Iso-Seq reads — IsoCon assumes primers, barcodes,
  and chimeras have already been removed by the Iso-Seq pipeline
- Sequences are too divergent for exact/near-exact identity matching to be meaningful,
  or only approximate homology quantification is needed rather than exact nucleotide
  matches
edam_operation: http://edamontology.org/operation_0292
edam_topics:
- http://edamontology.org/topic_0654
- http://edamontology.org/topic_3170
tools:
- name: IsoCon
  role: De novo error correction and clustering of PacBio FLNC reads to produce per-sample
    candidate transcripts for cross-sample comparison
  repo: https://github.com/ksahlin/IsoCon
- name: ICE
  role: Baseline standard Iso-Seq tool producing candidate transcript sets compared
    against IsoCon's outputs
- name: parasail
  role: SIMD pairwise sequence alignment library used to align shared transcripts
    to reference CDSs and detect exact matches
  repo: https://github.com/jeffdaily/parasail-python
- name: edlib
  role: Edit-distance alignment library usable for nucleotide-level exact-match detection
    against Ensembl CDSs
  repo: https://github.com/Martinsos/edlib
- name: PacBio
  role: Source of long Iso-Seq FLNC/CCS reads from targeted RT-PCR sequencing of the
    two testes samples
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41467-018-06910-x
    title: Deciphering highly similar multigene family transcripts from Iso-Seq data
      with IsoCon
  source_content_hash: sha256:f2c792947721ab558c28abc9da01e78a07361edabb9fabafad19d2aeaba0827a
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ksahlin/IsoCon
  tools:
  - ICE
  - IsoCon
  - PacBio
  - edlib
  - parasail
  edam_operation: http://edamontology.org/operation_0292
  edam_topics:
  - http://edamontology.org/topic_0654
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 6
---

# cross-sample-transcript-sequence-matching

## Summary

Compares reconstructed transcript sets across biological samples and against a reference CDS set by exact nucleotide sequence identity, using pairwise alignment (parasail/edlib) to quantify reproducibility and reference-level precision of methods like IsoCon and ICE.

## When to use

Use when two or more samples' reconstructed transcript sets (e.g., IsoCon vs ICE outputs as FASTA) must be intersected by exact sequence identity, or when counting exact nucleotide matches to known reference CDSs (e.g., 61 Ensembl coding sequences of nine Y ampliconic gene families).

## When NOT to use

- Transcripts need assignment to individual gene copies or family-size estimation — the article notes transcriptome-only sequencing does not conclusively assign transcripts to gene copies and further downstream analysis is required
- Input reads are not pre-processed Iso-Seq reads — IsoCon assumes primers, barcodes, and chimeras have already been removed by the Iso-Seq pipeline
- Sequences are too divergent for exact/near-exact identity matching to be meaningful, or only approximate homology quantification is needed rather than exact nucleotide matches

## Inputs

- Per-sample predicted transcript sets in FASTA (e.g., IsoCon final_candidates.fa from two testes samples)
- Candidate transcript sets from alternative methods (ICE, original PacBio reads, Illumina/proovread-corrected reads)
- Reference coding sequences: 61 unique Ensembl CDSs of the nine Y chromosome ampliconic gene families
- FLNC FASTQ reads per sample (fastq preferred over fasta since IsoCon uses quality values)

## Outputs

- Per-method count of transcripts shared between the two samples by exact sequence identity (e.g., 121 for IsoCon)
- Per-method count of exact nucleotide-level matches to the 61 Ensembl CDSs (e.g., 21 for IsoCon vs 8 for ICE)
- Comparison table of shared-transcript percentages and Ensembl exact-match counts across methods

## How to apply

Run each transcript-reconstruction method (IsoCon on FLNC FASTQ reads via 'IsoCon pipeline -fl_reads', fastq preferred because IsoCon uses quality values for its statistical filtering; ICE on the same per-sample reads, importing its candidates via IsoCon's stat_filter-style evaluation since external candidate sets need not originate from IsoCon's error correction). For each method, intersect the two testes samples' candidate transcript sets (final_candidates.fa) by exact sequence identity to count transcripts identically predicted in both samples. Then align the shared transcripts against the 61 unique Ensembl coding sequences of the nine ampliconic gene families and count exact (nucleotide-level) sequence matches using pairwise alignment with parasail (or edlib). Validate against the reported benchmark: IsoCon yields ~121 shared transcripts (~32% of transcripts, vs 11% for Illumina-corrected reads and <2% for ICE and original reads) and 21 Ensembl exact matches versus 8 for ICE.

## Related tools

- **IsoCon** (De novo error correction and clustering of PacBio FLNC reads to produce per-sample candidate transcripts for cross-sample comparison) — https://github.com/ksahlin/IsoCon
- **ICE** (Baseline standard Iso-Seq tool producing candidate transcript sets compared against IsoCon's outputs)
- **parasail** (SIMD pairwise sequence alignment library used to align shared transcripts to reference CDSs and detect exact matches) — https://github.com/jeffdaily/parasail-python
- **edlib** (Edit-distance alignment library usable for nucleotide-level exact-match detection against Ensembl CDSs) — https://github.com/Martinsos/edlib
- **PacBio** (Source of long Iso-Seq FLNC/CCS reads from targeted RT-PCR sequencing of the two testes samples)

## Examples

```
IsoCon pipeline -fl_reads reads.fastq -outfolder out_sample1; IsoCon pipeline -fl_reads reads.fastq -outfolder out_sample2; python compare_exact_matches.py out_sample1/final_candidates.fa out_sample2/final_candidates.fa ensembl_cds.fa
```

## Evaluation signals

- IsoCon should recover ~121 transcripts shared between the two testes samples, ~32% of transcripts (vs 11% for Illumina-corrected reads and <2% for ICE and original reads)
- IsoCon shared transcripts should exactly match 21 of the 61 Ensembl coding sequences, versus only 8 exact matches for ICE
- Shared-transcript sets must be derived by exact sequence identity, so any reported shared transcript must be byte-identical between the two samples' FASTA outputs
- Cross-sample consistency: sequences in final_candidates.fa should only be counted as shared if they pass IsoCon's statistical filtering (significance threshold alpha = 0.01)

## Limitations

- Low-abundance transcripts may be missed because deep PacBio sequencing can be cost-prohibitive
- IsoCon cannot distinguish alleles from distinct gene copies; downstream analysis is needed to separate them
- Exact-match counts cannot conclusively assign transcripts to gene copies, determine gene family size, or distinguish novel gene copies from splice variants or RNA-editing
- RT-PCR error sources cannot be excluded; replicate cDNA libraries and further validation are still needed

## Evidence

- [results] IsoCon shared transcripts exactly matching the Ensembl sequences: [evidence span withheld — anchor status: not_found]
- [readme] IsoCon pipeline command: "IsoCon pipeline -fl_reads <reads.fastq> -outfolder </path/to/output>"
- [discussion] Pairwise alignment tool: "We first create pairwise alignments from each read to the center using parasail60"
- [discussion] Statistical filtering threshold: "we identify the candidates with pi greater than a significance threshold α. This α is a parameter to our algorithm, set by default to 0.01."
- [readme] Output transcript file: "predicted transcripts are found in file **/path/to/output/final_candidates.fa**"
