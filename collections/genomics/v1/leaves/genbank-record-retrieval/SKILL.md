---
name: genbank-record-retrieval
description: Use when you need to verify whether specific public NCBI/GenBank assemblies
  contain detectable Nanopore barcode or flank sequences, or when a contamination
  hypothesis must be checked against concrete accession numbers.
when_to_use_negative:
- Input is raw Nanopore FASTQ reads that need demultiplexing/trimming rather than
  assembled records — use Barbell's annotate/filter/trim workflow on reads instead
- No candidate accession numbers or contamination hypothesis exists and a database-wide
  contamination survey is required instead of targeted record verification
edam_operation: http://edamontology.org/operation_2422
edam_topics:
- http://edamontology.org/topic_0622
tools:
- name: Barbell
  role: Provides the SQK-RBK114-96 kit preset from which barcode and flank query sequences
    are extracted for the search
  repo: https://github.com/rickbeeloo/barbell
- name: Sassy
  role: SIMD-accelerated edit-distance approximate matcher used to locate barcode
    and flank hits within the downloaded assemblies
  repo: https://github.com/RagnarGrootKoerkamp/sassy
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/bioinformatics/btag349
    title: Barbell reveals and resolves demultiplexing and trimming issues in Nanopore
      data
  source_content_hash: sha256:7e1ad0a41637be5196e842c7aa8bf743d7b1a029fa5ed07d47f5467b0b040327
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/rickbeeloo/barbell
  tools:
  - Barbell
  - Sassy
  - cargo
  edam_operation: http://edamontology.org/operation_2422
  edam_topics:
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# genbank-record-retrieval

## Summary

Retrieve specific public GenBank/NCBI nucleotide records (e.g. CP131573.1, CP165501.1) in FASTA format and screen them for Nanopore rapid-barcoding contamination using Barbell's SQK-RBK114-96 preset queries and Sassy-style edit-distance matching.

## When to use

Use when you need to verify whether specific public NCBI/GenBank assemblies contain detectable Nanopore barcode or flank sequences, or when a contamination hypothesis must be checked against concrete accession numbers.

## When NOT to use

- Input is raw Nanopore FASTQ reads that need demultiplexing/trimming rather than assembled records — use Barbell's annotate/filter/trim workflow on reads instead
- No candidate accession numbers or contamination hypothesis exists and a database-wide contamination survey is required instead of targeted record verification

## Inputs

- NCBI/GenBank accession numbers of target nucleotide records (e.g. CP131573.1, CP165501.1)
- FASTA-format assembly contig sequences downloaded from NCBI
- SQK-RBK114-96 barcode and flank query sequences from the Barbell kit preset

## Outputs

- Per-accession barcode/flank match counts (e.g. BC86 occurring 11 times in CP131573.1)
- Match positions and orientations for barcode overhangs (e.g. BC10 and BC09 in CP165501.1)
- barcode_verification.tsv summarizing match counts and coordinates

## How to apply

Download the target GenBank records (CP131573.1, CP165501.1) from NCBI as FASTA. Extract the SQK-RBK114-96 barcode and flank sequences from Barbell's built-in kit preset (`barbell kit -k SQK-RBK114-96`) to serve as query sequences, since Barbell ships presets for many Nanopore kits. Scan each assembly with Sassy-style edit-distance approximate matching, keeping the default flank error tolerance rather than relaxing `--flank-max-errors` so only genuine hits are recovered. Count per-accession matches — e.g. BC86 hits in the 1.43 Mb Photobacterium leiognathi SV5.1 contig — and record positions/orientations to check whether BC10 and BC09 overhangs sit at opposite contig termini of the E. coli plasmid, which indicates barcode bleeding. Write per-accession match counts and coordinates to a TSV (barcode_verification.tsv) for validation against published findings.

## Related tools

- **Barbell** (Provides the SQK-RBK114-96 kit preset from which barcode and flank query sequences are extracted for the search) — https://github.com/rickbeeloo/barbell
- **Sassy** (SIMD-accelerated edit-distance approximate matcher used to locate barcode and flank hits within the downloaded assemblies) — https://github.com/RagnarGrootKoerkamp/sassy

## Examples

```
curl -s 'https://eutils.ncbi.nlm.nih.gov/entrez/eutils/efetch.fcgi?db=nuccore&id=CP131573.1,CP165501.1&rettype=fasta&retmode=text' > assemblies.fasta && barbell kit -k SQK-RBK114-96 -i reads.fastq -o out --maximize
```

## Evaluation signals

- barcode_verification.tsv reports BC86 occurring exactly 11 times in the 1.43 Mb P. leiognathi SV5.1 contig CP131573.1
- BC10 and BC09 overhangs are found at opposite ends of plasmid contig CP165501.1, consistent with barcode bleeding
- All reported hits are valid barcode/flank configurations rather than random Fflank-type matches
- Retrieved FASTA headers/accessions in the downloaded records exactly match the queried NCBI accessions

## Limitations

- Relaxing the flank error threshold (e.g. --flank-max-errors 5) can introduce random matches that appear as Fflank hits at non-end locations; defaults should be kept and results inspected
- Sassy/barbell results can differ slightly (<0.01%) between release versions and the current git version using Sassy V2

## Evidence

- [methods] Download the GenBank records CP131573.1 and CP165501.1 from NCBI in FASTA format.: [evidence span withheld — anchor status: not_found]
- [methods] Extract the SQK-RBK114-96 barcode and flank sequences from the Barbell kit preset: [evidence span withheld — anchor status: not_found]
- [methods] Scan each assembly with Sassy-style edit-distance approximate matching to locate all barcode and flank hits: [evidence span withheld — anchor status: not_found]
- [abstract] Nanopore barcoding contamination is widespread in public databases, detected in hundreds of NCBI entries: "with Nanopore sequences detected in hundreds of NCBI entries, some of which are responsible for artificial taxonomic connections."
- [readme] Barbell includes built-in kit *presets* for many Nanopore kits: "Barbell includes built-in kit *presets* for many Nanopore kits (most DNA kits; RNA and Twist kits are an exception)."
