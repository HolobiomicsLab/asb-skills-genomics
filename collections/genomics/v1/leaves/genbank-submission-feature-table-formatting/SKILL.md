---
name: genbank-submission-feature-table-formatting
description: Use when a plastid genome (e.g., fern or hornwort plastome) with RNA-editing-derived
  internal stop codons must be converted into GenBank-ready submission files after
  nonsense-mutation annotation.
when_to_use_negative:
- When the goal is to predict every RNA editing site using RNA-Seq data (ReFernment
  is not intended for that and relies on GFF3/GenBank annotations instead)
- When the GenBank flat file was created in Sequin — a known bug prevents annotation
  generation; files exported from Geneious do work
- When input annotations are sloppy or unverified — ReFernment is not a substitute
  for manually checking gene annotations
edam_operation: http://edamontology.org/operation_0335
edam_topics:
- http://edamontology.org/topic_0780
- http://edamontology.org/topic_0622
tools:
- name: ReFernment
  role: R package that annotates RNA-editing-restorable nonsense mutations and generates
    the five-column GenBank feature table and protein FASTA
  repo: https://github.com/TARobison/ReFernment
- name: R
  role: Runtime environment in which ReFernment is installed and invoked via library('ReFernment')
- name: DOGMA
  role: Upstream annotation generator whose output (GFF3) ReFernment consumes
- name: CpGAVAS
  role: Upstream annotation generator whose output (GFF3) ReFernment consumes
- name: Verdant
  role: Upstream annotation generator whose output (GFF3) ReFernment consumes
- name: AGORA
  role: Upstream annotation generator whose output (GFF3) ReFernment consumes
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1002/aps3.1216
    title: 'ReFernment: An R package for annotating <scp>RNA</scp> editing in plastid
      genomes'
  source_content_hash: sha256:87788568c94d2f7217ffdc53f367d50c96ae635ebc13e6ea3d1e2517e5581810
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/TARobison/ReFernment
  tools:
  - AGORA
  - CpGAVAS
  - DOGMA
  - R
  - ReFernment
  - Verdant
  edam_operation: http://edamontology.org/operation_0335
  edam_topics:
  - http://edamontology.org/topic_0780
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.625
n_claims: 16
---

# genbank-submission-feature-table-formatting

## Summary

Generates a five-column GenBank feature table and a protein FASTA file of conceptual translations from an RNA-editing-refined GenBank flat file, producing artifacts formatted correctly for direct NCBI GenBank submission of plastid genomes with high RNA editing.

## When to use

Use when a plastid genome (e.g., fern or hornwort plastome) with RNA-editing-derived internal stop codons must be converted into GenBank-ready submission files after nonsense-mutation annotation.

## When NOT to use

- When the goal is to predict every RNA editing site using RNA-Seq data (ReFernment is not intended for that and relies on GFF3/GenBank annotations instead)
- When the GenBank flat file was created in Sequin — a known bug prevents annotation generation; files exported from Geneious do work
- When input annotations are sloppy or unverified — ReFernment is not a substitute for manually checking gene annotations

## Inputs

- GFF3 annotation file with embedded nucleotide sequence (required)
- GenBank flat file (.gb) including nucleotide sequence (optional)
- Separate FASTA nucleotide file (optional)
- Vector of genome basenames without file extensions (e.g., 'Asplenium_pek', 'Woodwardia_uni')
- Paths: gffFolderPath and outputFolderPath (required); gbFolderPath and fastaFolderPath (optional)

## Outputs

- Five-column feature table formatted correctly for submission to GenBank
- Protein FASTA file containing conceptual translations for coding sequences where RNA editing occurred
- Edited GenBank flat file with nonsense-mutation sites annotated as misc_feature flags

## How to apply

Install ReFernment from GitHub via install_github('TARobison/ReFernment') and load it with library('ReFernment'). Prepare input files so the GenBank (.gb) and GFF3 (.gff) files share identical basenames except for extensions, and declare a genomes vector of basenames without extensions (e.g., c('Asplenium_pek', 'Woodwardia_uni')). Set gffFolderPath (required, GFF3 with embedded sequence) and outputFolderPath (a directory separate from input folders unless overwriting original GB files is intended), optionally adding gbFolderPath and fastaFolderPath. Call the ReFernment function; during the run it validates start/final codons (adjusting gene boundaries by checking nearby codons within five codons), detects internal stop codons restorable by RNA editing, edits the GenBank flat file with misc_feature flags, and emits the five-column feature table and protein FASTA. Inspect warnings: more than 5 internal stops in a coding sequence suggests assembly or annotation-frame errors (especially above 10), and protein-translation errors usually indicate a CDS spanning the genome origin, resolved by shifting the origin.

## Related tools

- **ReFernment** (R package that annotates RNA-editing-restorable nonsense mutations and generates the five-column GenBank feature table and protein FASTA) — https://github.com/TARobison/ReFernment
- **R** (Runtime environment in which ReFernment is installed and invoked via library('ReFernment'))
- **DOGMA** (Upstream annotation generator whose output (GFF3) ReFernment consumes)
- **CpGAVAS** (Upstream annotation generator whose output (GFF3) ReFernment consumes)
- **Verdant** (Upstream annotation generator whose output (GFF3) ReFernment consumes)
- **AGORA** (Upstream annotation generator whose output (GFF3) ReFernment consumes)

## Examples

```
library("ReFernment"); genomes <- c("Asplenium_pek", "Woodwardia_uni"); ReFernment(gffFolder, fastaFolderPath=NULL, gbFolderPath=NULL, outputFolder, genomes)
```

## Evaluation signals

- Output folder contains both a five-column feature table formatted correctly for GenBank submission and a protein FASTA file of conceptual translations
- Nonsense-mutation sites in the flat file are annotated with misc_feature flags without false positive annotations, as verified on cDNA-confirmed plastomes AB086179 and AY178864.1
- Protein FASTA conceptual translations show RNA-editing-restored codons with no residual internal stop codons for correctly annotated genes
- No warning about 'a high number of edited Stops' (more than 5 internal stops) was emitted for any coding sequence
- No protein-translation error warnings indicating a CDS spanning the genome origin

## Limitations

- ReFernment is not a substitute for manually checking gene annotations, nor a fix for sloppy annotation; output quality depends on input annotation quality
- Known error: no annotations are produced for files created in Sequin (fix planned); files exported from Geneious still work
- More than 5 internal stops in a coding sequence triggers a warning to manually check for frameshift/assembly or annotation-frame errors; more than 10 internal stops strongly suggests assembly error
- Protein-translation errors usually arise from a coding sequence spanning the genome origin; the user must shift the genome origin
- Execution can take several minutes when annotating a large number of plastomes

## Evidence

- [abstract] ReFernment produces a five-column feature table, formatted correctly for submission to GenBank, and a protein FASTA file: "ReFernment produces a five‐column feature table, formatted correctly for submission to GenBank, and a protein FASTA file"
- [methods] install_github TARobison ReFernment: "install_github("TARobison/ReFernment")"
- [readme] genomes vector without extensions: "genomes should **not** contain the file extension. ReFernment will add the file extensions later."
- [readme] internal stop warning threshold: "The warning will appear if a given coding sequence has more than 5 internal stops."
- [readme] translation error due to CDS spanning origin: "This is usually due to the coding sequence spanning the orgin of the genome. I suggest that you shift the origin of the genome"
- [abstract] no false positives for cDNA-confirmed plastomes: "every nonsense mutation was correctly annotated, and there were no false positive annotations."
- [abstract] not a substitute for manual checking: "ReFernment is not a substitute for manually checking gene annotations, nor is ReFernment a fix for sloppy annotation."
