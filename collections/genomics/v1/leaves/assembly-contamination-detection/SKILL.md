---
name: assembly-contamination-detection
description: Use when you need to verify whether public NCBI/GenBank genome assemblies
  carry detectable Nanopore rapid-barcode (SQK-RBK114-96) or adapter-flank sequences,
  and to locate which records and contig positions contain them.
when_to_use_negative:
- Input is raw Nanopore reads that need demultiplexing/trimming rather than a finished
  assembly — use `barbell kit` demux/annotate workflow instead
- The sequencing kit is outside Barbell's tested presets (RNA and Twist kits are exceptions
  to its supported DNA kits)
- Queries are patterns well beyond the ~1000 nt range where Sassy works well (it targets
  short 20–100 nt barcodes/flanks)
edam_operation: http://edamontology.org/operation_0346
edam_topics:
- http://edamontology.org/topic_0085
- http://edamontology.org/topic_3173
tools:
- name: Barbell
  role: Supplies the SQK-RBK114-96 kit preset used to extract barcode (BC86, BC10,
    BC09) and flank query sequences
  repo: https://github.com/rickbeeloo/barbell
- name: Sassy
  role: Performs edit-distance approximate (fuzzy) matching of barcode/flank queries
    against the FASTA assemblies
  repo: https://github.com/RagnarGrootKoerkamp/sassy
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/bioinformatics/btag349
    title: Barbell reveals and resolves demultiplexing and trimming issues in Nanopore
      data
  source_content_hash: sha256:e255533e52d510c46fd0c3f57488a2a553b04e7ce180572bff76967cf51a9f8b
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/rickbeeloo/barbell
  tools:
  - Barbell
  - Sassy
  - cargo
  edam_operation: http://edamontology.org/operation_0346
  edam_topics:
  - http://edamontology.org/topic_0085
  - http://edamontology.org/topic_3173
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# assembly-contamination-detection

## Summary

Screens public genome assemblies (e.g. GenBank records CP131573.1 and CP165501.1) for residual Nanopore rapid-barcode contamination by searching barcode/flank sequences from SQK-RBK114-96 with Sassy-style edit-distance matching.

## When to use

Use when you need to verify whether public NCBI/GenBank genome assemblies carry detectable Nanopore rapid-barcode (SQK-RBK114-96) or adapter-flank sequences, and to locate which records and contig positions contain them.

## When NOT to use

- Input is raw Nanopore reads that need demultiplexing/trimming rather than a finished assembly — use `barbell kit` demux/annotate workflow instead
- The sequencing kit is outside Barbell's tested presets (RNA and Twist kits are exceptions to its supported DNA kits)
- Queries are patterns well beyond the ~1000 nt range where Sassy works well (it targets short 20–100 nt barcodes/flanks)

## Inputs

- NCBI/GenBank assembly records in FASTA format (CP131573.1, CP165501.1)
- SQK-RBK114-96 barcode and flank query sequences extracted from the Barbell kit preset

## Outputs

- barcode_verification.tsv with per-accession match counts, coordinates, and orientations
- List of barcode occurrences (e.g. 11x BC86 in CP131573.1) and terminal overhang positions (BC10/BC09 in CP165501.1)

## How to apply

Download the suspect GenBank records (CP131573.1, CP165501.1) as FASTA, then extract the SQK-RBK114-96 barcode and flank sequences from Barbell's kit preset with `barbell kit -k SQK-RBK114-96` to serve as queries. Scan each assembly with Sassy-style edit-distance approximate matching, allowing the default flank error tolerance, and record every barcode and flank hit with its coordinates and orientation. Count BC86 matches in the 1.43 Mb Photobacterium leiognathi SV5.1 contig and check that BC10/BC09 overhangs in the E. coli plasmid CP165501.1 sit at opposite contig termini, which indicates barcode bleeding. Write per-accession match counts and coordinates to barcode_verification.tsv; if hits appear at random (non-end) locations, lower the flank error threshold as the README advises for excessive `Fflank` matches.

## Related tools

- **Barbell** (Supplies the SQK-RBK114-96 kit preset used to extract barcode (BC86, BC10, BC09) and flank query sequences) — https://github.com/rickbeeloo/barbell
- **Sassy** (Performs edit-distance approximate (fuzzy) matching of barcode/flank queries against the FASTA assemblies) — https://github.com/RagnarGrootKoerkamp/sassy

## Examples

```
barbell kit -k SQK-RBK114-96
```

## Evaluation signals

- barcode_verification.tsv reports exactly 11 BC86 matches in the 1.43 Mb CP131573.1 contig
- BC10 and BC09 overhangs in CP165501.1 are found at opposite contig termini, confirming barcode bleeding
- Match locations are non-random: barcode hits occur at contig ends rather than scattered internal positions
- No excess `Fflank` matches — random-locus flank hits signal the flank error threshold is too lenient
- Counts are stable across Barbell builds up to the documented <0.01% deviation between git and release versions

## Limitations

- Barbell's kit presets were thoroughly tested only for rapid and native Nanopore DNA kits, not RNA or Twist kits
- The Sassy V2-based git version of Barbell is ~5x faster than releases but results can differ slightly (<0.01%)
- A too-lenient `--flank-max-errors` threshold produces random matches that surface as `Fflank` annotations at unexpected locations

## Evidence

- [other] Rapid Nanopore barcode contamination is present in specific public assemblies: barcode BC86 occurs 11 times in the 1.43 Mb Photobacterium leiognathi SV5.1 contig (CP131573.1): [evidence span withheld — anchor status: not_found]
- [other] Extract the SQK-RBK114-96 barcode and flank sequences from the Barbell kit preset (barbell kit -k SQK-RBK114-96) to use as query sequences.: [evidence span withheld — anchor status: not_found]
- [abstract] with Nanopore sequences detected in hundreds of NCBI entries: "with Nanopore sequences detected in hundreds of NCBI entries"
- [readme] Sassy generally works well for patterns/queries up to length 1000, and supports ASCII, DNA, and IUPAC.: [evidence span withheld — anchor status: external_doc]
