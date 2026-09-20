---
name: intron-chain-extraction
description: Use when you need to test whether transcript isoforms in a GTF annotation
  (e.g., GENCODE v39) are uniquely defined by their intron chains, or to count splice-junction
  signatures shared by multiple transcript start/end combinations.
when_to_use_negative:
- Input is raw long-read data (HiFi reads, subreads.bam) rather than an annotated
  transcript model — run the isoseq > minimap2/pbmm2 > SQANTI3 pipeline first to obtain
  isoform coordinates
- The goal is cross-sample differential isoform abundance or read-support deconvolution
  rather than isoform identity via splice junctions — isoSeQL's comparison and visualization
  features are the right fit
- Transcripts are single-exon (e.g., many NNC models), for which no intron chain can
  be derived
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: isoSeQL
  role: Downstream tool that identifies common and unique isoforms across samples
    by compiling long-read transcriptomic data; intron chains underpin its cross-sample
    isoform matching
  repo: https://github.com/christine-liu/isoSeQL
- name: Python 3.9.6 / pandas
  role: 'Implementation language and GTF parsing: load transcript/exon records, group
    exons, and tally intron-chain to transcript mappings'
- name: SQANTI3
  role: Produces the annotated isoform classification output (PB.X.Y IDs) that isoSeQL
    consumes; intron-chain identity substitutes for the non-matching sample-specific
    IDs
  repo: https://github.com/ConesaLab/SQANTI3
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/bioinformatics/btaf680
    title: 'isoSeQL: comparing long-read isoforms across multiple datasets'
  source_content_hash: sha256:6a86dbd796fc838bec5aeda46f7379de89aec29b52dbd126f4d5ffff74d754a2
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/christine-liu/isoSeQL
  tools:
  - Python
  - Python 3.9.6 / pandas
  - SQANTI3
  - isoSeQL
  - numpy
  - pandas
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.42857142857142855
n_claims: 14
---

# intron-chain-extraction

## Summary

Extracts the ordered splice-junction (intron chain) signature of every transcript in an annotation such as GENCODE v39 and quantifies how uniquely each chain identifies a transcript, showing that 244,939 transcripts collapse to 215,652 unique intron chains with only 1.03% shared by multiple transcripts.

## When to use

Use when you need to test whether transcript isoforms in a GTF annotation (e.g., GENCODE v39) are uniquely defined by their intron chains, or to count splice-junction signatures shared by multiple transcript start/end combinations.

## When NOT to use

- Input is raw long-read data (HiFi reads, subreads.bam) rather than an annotated transcript model — run the isoseq > minimap2/pbmm2 > SQANTI3 pipeline first to obtain isoform coordinates
- The goal is cross-sample differential isoform abundance or read-support deconvolution rather than isoform identity via splice junctions — isoSeQL's comparison and visualization features are the right fit
- Transcripts are single-exon (e.g., many NNC models), for which no intron chain can be derived

## Inputs

- GENCODE v39 GTF annotation file (transcript and exon records)
- Transcript attributes: gene ID, transcript ID, chromosome, strand, exon start/end coordinates

## Outputs

- Dictionary of unique intron chains mapping to sets of (transcript start, transcript end) combinations
- Summary CSV (gencode_intron_chain_summary.csv) with total transcripts, unique intron chains, and chains with multiple transcripts

## How to apply

Load the GENCODE v39 GTF with pandas and filter for transcript and exon records, parsing gene_id/transcript_id, chromosome, strand, and exon coordinates from the attribute column. For each transcript, group its exons in genomic order and derive the intron chain as the ordered tuple of splice junction coordinates between consecutive exons. Build a dictionary keyed by intron chain, mapping each chain to the set of (transcript start, transcript end) coordinate combinations observed, while tallying total transcripts processed. Count unique intron chains and identify chains associated with more than one start/end combination, then compute the fraction of chains with multiple transcripts (2223 of 215,652, i.e. 1.03%). Write the summary to gencode_intron_chain_summary.csv. This chain identity is also the cross-sample key needed because isoform ID numbers (PB.X.Y) from SQANTI2/SQANTI3-analyzed Iso-Seq runs cannot be matched across samples.

## Related tools

- **isoSeQL** (Downstream tool that identifies common and unique isoforms across samples by compiling long-read transcriptomic data; intron chains underpin its cross-sample isoform matching) — https://github.com/christine-liu/isoSeQL
- **Python 3.9.6 / pandas** (Implementation language and GTF parsing: load transcript/exon records, group exons, and tally intron-chain to transcript mappings)
- **SQANTI3** (Produces the annotated isoform classification output (PB.X.Y IDs) that isoSeQL consumes; intron-chain identity substitutes for the non-matching sample-specific IDs) — https://github.com/ConesaLab/SQANTI3

## Examples

```
gtf = pd.read_csv('gencode.v39.annotation.gtf', sep='\t', comment='#', names=['seqname','source','feature','start','end','score','strand','frame','attribute'])
```

## Evaluation signals

- gencode_intron_chain_summary.csv exists and reports 244,939 total transcripts processed
- Number of unique intron chains equals 215,652
- Exactly 2223 intron chains have more than one (transcript start, transcript end) combination, approximately 1.03%
- Over 98% of intron chains map to a single transcript (singular isoforms)
- Every derived intron chain is an ordered tuple of junction coordinates between consecutive exons in genomic order, with no chains computed from unsorted exon groups

## Limitations

- Intron chains ignore transcript start and end sites, so chains shared by multiple (start, end) combinations are counted as non-unique even though the splice junctions themselves are identical
- Single-exon transcripts yield no intron chain and cannot be represented by this signature
- The isoSeQL repository provides no changelog, so version-specific behavior changes are not documented

## Evidence

- [other] Per-transcript intron chain derivation and counting procedure: [evidence span withheld — anchor status: not_found]
- [other] GENCODE v39 intron-chain uniqueness result: [evidence span withheld — anchor status: not_found]
- [abstract] isoSeQL's role in identifying unique vs common isoforms: "We developed isoSeQL, a tool for compiling long-read transcriptomic data, identifying common and unique isoforms across multiple samples, and extracting and visualizing various metrics."
- [readme] Cross-sample comparison context from the README: "Database tool for comparing across many Iso-Seq runs analyzed through SQANTI3"
