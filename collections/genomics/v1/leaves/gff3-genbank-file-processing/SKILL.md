---
name: gff3-genbank-file-processing
description: Use when plastid genome annotations contain apparent nonsense mutations
  (internal stop codons) caused by RNA editing that would be rejected by NCBI, and
  submission-ready GenBank files are needed.
when_to_use_negative:
- Input annotations were created in Sequin — ReFernment has a known error and does
  not produce annotations for Sequin-created files (files exported from Geneious still
  work)
- You need prediction of every RNA editing site from RNA-Seq data — ReFernment is
  not intended to predict every editing site and is not a substitute for manually
  checking gene annotations
edam_operation: http://edamontology.org/operation_0362
edam_topics:
- http://edamontology.org/topic_0080
- http://edamontology.org/topic_0780
tools:
- name: ReFernment
  role: R package that validates codons, detects RNA-editing-restorable internal stops,
    edits the GenBank flat file, and writes the feature table and protein FASTA
  repo: github.com/TARobison/ReFernment
- name: R
  role: Runtime environment in which ReFernment is installed via devtools install_github
    and invoked
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1002/aps3.1216
    title: 'ReFernment: An R package for annotating <scp>RNA</scp> editing in plastid
      genomes'
  source_content_hash: sha256:9ba16e401d16fc84b8aa430cb343e4cfe345e5276e51313ad22d27586587855c
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - R
  - ReFernment
  edam_operation: http://edamontology.org/operation_0362
  edam_topics:
  - http://edamontology.org/topic_0080
  - http://edamontology.org/topic_0780
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5714285714285714
n_claims: 14
---

# gff3-genbank-file-processing

## Summary

Process GFF3 annotations and GenBank flat files for plastid genomes with the ReFernment R package to annotate RNA-editing-restored nonsense mutations and generate GenBank-submission-ready output files.

## When to use

Use when plastid genome annotations contain apparent nonsense mutations (internal stop codons) caused by RNA editing that would be rejected by NCBI, and submission-ready GenBank files are needed.

## When NOT to use

- Input annotations were created in Sequin — ReFernment has a known error and does not produce annotations for Sequin-created files (files exported from Geneious still work)
- You need prediction of every RNA editing site from RNA-Seq data — ReFernment is not intended to predict every editing site and is not a substitute for manually checking gene annotations

## Inputs

- GFF3 file with the nucleotide sequence embedded (required)
- GenBank flat file including nucleotide sequence (optional)
- Separate FASTA file (optional)
- genomes vector of file basenames without extensions (e.g. c("Asplenium_pek", "Woodwardia_uni"))

## Outputs

- Five-column feature table formatted correctly for submission to GenBank
- Protein FASTA file containing conceptual translations for coding sequences where RNA editing occurred
- Edited GenBank flat file with misc_feature flags at RNA-edited nonsense mutation sites

## How to apply

Install ReFernment from GitHub with install_github('TARobison/ReFernment') and provide a GFF3 file with the sequence embedded (a GenBank flat file and separate FASTA file are optional). Set gffFolderPath and outputFolderPath (keep the output directory separate unless overwriting original GB files is intended), name GB and GFF files identically except for extensions, and pass file basenames without extensions in the genomes vector. Run ReFernment(gffFolder, fastaFolderPath, gbFolderPath, outputFolder, genomes); it validates start/final codons (adjusting boundaries by checking nearby codons within five codons), detects internal stop codons that RNA editing can restore, and edits the GenBank flat file with misc_feature flags and conceptual translations. Inspect warnings: more than 5 internal stops in a coding sequence triggers a manual-check warning (especially if more than 10), and a protein-translation error usually means a CDS spans the genome origin and the origin should be shifted. Confirm the output folder contains the five-column GenBank feature table and the protein FASTA of conceptual translations.

## Related tools

- **ReFernment** (R package that validates codons, detects RNA-editing-restorable internal stops, edits the GenBank flat file, and writes the feature table and protein FASTA) — github.com/TARobison/ReFernment
- **R** (Runtime environment in which ReFernment is installed via devtools install_github and invoked)

## Examples

```
install_github("TARobison/ReFernment"); library("ReFernment"); genomes <- c("Asplenium_pek", "Woodwardia_uni"); ReFernment(gffFolder, fastaFolderPath=NULL, gbFolderPath=NULL, outputFolder, genomes)
```

## Evaluation signals

- Output folder contains a five-column feature table formatted correctly for GenBank submission and a protein FASTA file
- Every nonsense mutation explainable by RNA editing is annotated as a misc_feature with no false positive annotations (validated against cDNA-confirmed plastomes AB086179 and AY178864.1)
- Protein FASTA contains conceptual translations for coding sequences where RNA editing occurred
- No unexplained warnings: >5 internal stops per CDS or protein-translation errors should have been resolved (frame check or origin shift) before accepting output
- GB and GFF input files were named identically except for extensions and genomes vector entries contained no file extensions

## Limitations

- ReFernment is not a substitute for manually checking gene annotations, nor is it a fix for sloppy annotation; output quality depends on the quality of the input annotations
- A coding sequence with more than 5 internal stops triggers a warning for possible assembly (frameshift) or annotation-frame errors; more than 10 internal stops warrants close inspection
- Protein translation errors usually indicate a CDS spanning the genome origin; the origin must be shifted so no CDS overlaps it
- Known issue: ReFernment does not produce annotations for files created in Sequin
- ReFernment does not predict every RNA editing site (unlike RNA-Seq-based tools) and requires the GFF3 sequence to be embedded

## Evidence

- [abstract] ReFernment produces a five‐column feature table, formatted correctly for submission to GenBank, and a protein FASTA file: "ReFernment produces a five‐column feature table, formatted correctly for submission to GenBank, and a protein FASTA file"
- [abstract] ReFernment requires both a GFF3 (no sequence) file and a GenBank flat file (including nucleotide sequence): "ReFernment requires both a GFF3 (no sequence) file and a GenBank flat file (including nucleotide sequence)"
- [methods] The warning will appear if a given coding sequence has more than 5 internal stops.: "The warning will appear if a given coding sequence has more than 5 internal stops."
- [readme] ReFernment *requires* only a GFF3 file (with sequence embedded) to work: "ReFernment *requires* only a GFF3 file (with sequence embedded) to work"
- [readme] This is usually due to the coding sequence spanning the orgin of the genome. I suggest that you shift the origin of the genome: "This is usually due to the coding sequence spanning the orgin of the genome. I suggest that you shift the origin of the genome"
