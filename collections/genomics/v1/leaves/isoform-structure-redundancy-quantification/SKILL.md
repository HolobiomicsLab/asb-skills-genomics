---
name: isoform-structure-redundancy-quantification
description: Use when a transcript annotation (GTF) or collapsed long-read isoform
  set needs redundancy profiling — e.g. testing whether intron chains uniquely identify
  isoforms before matching PB.X.Y IDs across samples.
when_to_use_negative:
- When the goal is cross-sample comparison of already-annotated Iso-Seq isoforms with
  full metrics — use isoSeQL directly instead of reimplementing chain counting
- When input is raw subreads or uncollapsed alignments (pre-isoseq cluster2/collapse)
  — transcript structures do not exist yet
- When splice structure alone is insufficient (e.g. TSS/TES polyA diversity is the
  question, since intron chains ignore start/end position except as redundancy tallies)
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: isoSeQL
  role: Tool for compiling long-read transcriptomic data and identifying common and
    unique isoforms across multiple samples; the intron-chain uniqueness statistic
    is a conceptual basis for its cross-sample matching
  repo: https://github.com/christine-liu/isoSeQL
- name: SQANTI3
  role: Annotates high-quality Iso-Seq isoforms whose classification output feeds
    isoSeQL comparison
  repo: https://github.com/ConesaLab/SQANTI3
- name: cDNA_Cupcake
  role: Collapse step (collapse_isoforms_by_sam.py) in the legacy isoseq > minimap2
    > cDNA_cupcake > SQANTI2 pipeline producing collapsed isoform structures
  repo: https://github.com/Magdoll/cDNA_Cupcake
- name: pandas
  role: Parses the GTF transcript/exon records and builds the intron-chain dictionary
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/bioinformatics/btaf680
    title: 'isoSeQL: comparing long-read isoforms across multiple datasets'
  source_content_hash: sha256:12be0693a8714eaa88d73658bb37a971a19e74a6e9094ffe8a90306628aae3f1
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/christine-liu/isoSeQL
  tools:
  - Python
  - SQANTI3
  - cDNA_Cupcake
  - isoSeQL
  - numpy
  - pandas
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5555555555555556
n_claims: 9
---

# isoform-structure-redundancy-quantification

## Summary

Quantifies how uniquely isoforms are described by their splice structure (intron chain) in a transcript reference: it parses a GTF (e.g. GENCODE v39), derives the ordered intron chain per transcript, and counts how many chains map to more than one transcript start/end combination. This is the same unique/common isoform comparison logic that motivates isoSeQL for long-read Iso-Seq data analyzed through SQANTI3.

## When to use

Use when a transcript annotation (GTF) or collapsed long-read isoform set needs redundancy profiling — e.g. testing whether intron chains uniquely identify isoforms before matching PB.X.Y IDs across samples.

## When NOT to use

- When the goal is cross-sample comparison of already-annotated Iso-Seq isoforms with full metrics — use isoSeQL directly instead of reimplementing chain counting
- When input is raw subreads or uncollapsed alignments (pre-isoseq cluster2/collapse) — transcript structures do not exist yet
- When splice structure alone is insufficient (e.g. TSS/TES polyA diversity is the question, since intron chains ignore start/end position except as redundancy tallies)

## Inputs

- GENCODE v39 GTF annotation file (transcript and exon records with gene/transcript IDs, chromosome, strand, coordinates)
- Optionally, SQANTI3-annotated long-read isoform output (classification.txt / collapsed GTF with PB.X.Y IDs)

## Outputs

- gencode_intron_chain_summary.csv with total transcripts (244,939), unique intron chains (215,652), and chains with multiple start/end combinations (2223; 1.03%)
- Dictionary mapping each intron chain to its set of (transcript start, transcript end) coordinate combinations

## How to apply

Load the GENCODE v39 GTF (or SQANTI3-annotated isoform output) and filter transcript and exon records with pandas, extracting gene/transcript IDs, chromosome, strand, and exon coordinates. For each transcript, group its exons in genomic order and derive the intron chain as the ordered tuple of splice junction coordinates between consecutive exons. Build a dictionary keyed by intron chain that maps each chain to the set of observed (transcript start, transcript end) combinations while tallying total transcripts processed (expected 244,939 for GENCODE v39). Count unique intron chains (215,652) and identify chains with more than one start/end combination (2223, ≈1.03%), then write the summary to a CSV. The rationale is that over 98% of intron chains represent singular isoforms, so intron chains can serve as stable keys for cross-sample isoform comparison — the exact problem isoSeQL solves when PB.X.Y identifiers cannot be matched between samples.

## Related tools

- **isoSeQL** (Tool for compiling long-read transcriptomic data and identifying common and unique isoforms across multiple samples; the intron-chain uniqueness statistic is a conceptual basis for its cross-sample matching) — https://github.com/christine-liu/isoSeQL
- **SQANTI3** (Annotates high-quality Iso-Seq isoforms whose classification output feeds isoSeQL comparison) — https://github.com/ConesaLab/SQANTI3
- **cDNA_Cupcake** (Collapse step (collapse_isoforms_by_sam.py) in the legacy isoseq > minimap2 > cDNA_cupcake > SQANTI2 pipeline producing collapsed isoform structures) — https://github.com/Magdoll/cDNA_Cupcake
- **pandas** (Parses the GTF transcript/exon records and builds the intron-chain dictionary)

## Examples

```
python -c "import pandas as pd; gtf=pd.read_csv('gencode.v39.annotation.gtf',sep='\t',comment='#',names=['chr','src','type','start','end','score','strand','frame','attr']); ex=gtf[gtf.type=='exon']"
```

## Evaluation signals

- Summary CSV exists and reports exactly 244,939 total transcripts and 215,652 unique intron chains for GENCODE v39
- Number of intron chains with more than one (start, end) combination equals 2223, i.e. approximately 1.03% of unique chains
- Total transcript count in the tally matches the number of transcript records in the GTF (sanity check against the source annotation)
- Every intron-chain key's value set is non-empty and all splice junction coordinates are internally consistent (donor > acceptor, correct strand ordering)
- Fraction of multi-transcript chains is below ~2%, consistent with the finding that over 98% of intron chains represent singular isoforms

## Limitations

- Intron chains ignore transcript start/end coordinates as identity keys, so transcripts sharing a chain but differing at their termini are counted as redundant only in the (start, end) tally
- Single-exon transcripts have no intron chain and are excluded from chain-based identity comparisons
- Counts are annotation-specific; running on other GENCODE releases or a SQANTI3-collapsed sample will produce different totals and percentages

## Evidence

- [abstract] isoSeQL, a tool for compiling long-read transcriptomic data, identifying common and unique isoforms across multiple samples: "We developed isoSeQL, a tool for compiling long-read transcriptomic data, identifying common and unique isoforms across multiple samples"
- [intro] isoform ID #s in each sample couldn't be matched: "the isoform ID #s (PB.X.Y) in each sample couldn't be matched"
- [intro] isoSeQL was created to work with the output of SQANTI2 (now SQANTI3): "isoSeQL was created to work with the output of SQANTI2 (now SQANTI3)"
- [intro] running the analysis pipeline (isoseq > minimap2 > cDNA_cupcake > SQANTI2): "running the analysis pipeline (isoseq > minimap2 > cDNA_cupcake > SQANTI2) on that single "mega sample""
- [readme] SQANTI3 is the newest version of the SQANTI tool: [evidence span withheld — anchor status: external_doc]
