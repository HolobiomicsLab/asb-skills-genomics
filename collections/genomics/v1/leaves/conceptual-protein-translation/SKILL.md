---
name: conceptual-protein-translation
description: Use when annotating fern or hornwort plastid genomes with high RNA editing,
  where internal stop codons must be translated through to produce correct protein
  sequences for GenBank submission.
when_to_use_negative:
- The goal is predicting every RNA editing site from RNA-Seq data — ReFernment is
  explicitly not intended to predict every editing site like RNA-Seq-based packages
- Input annotations are sloppy or unverified — ReFernment is not a substitute for
  manually checking gene annotations
- GenBank files were created in Sequin — a known error where ReFernment does not produce
  annotations for Sequin-created files
edam_operation: http://edamontology.org/operation_0372
edam_topics:
- http://edamontology.org/topic_0085
- http://edamontology.org/topic_0622
tools:
- name: ReFernment
  role: R package that detects RNA-editing-restorable internal stop codons, computes
    conceptual translations, and writes the protein FASTA and GenBank feature table
  repo: github.com/TARobison/ReFernment
- name: R
  role: Runtime environment in which ReFernment is installed via devtools and executed
- name: DOGMA
  role: Example upstream tool whose plastome annotation provides the GFF3 input ReFernment
    corrects
- name: CpGAVAS
  role: Example upstream annotation tool whose output can feed ReFernment
- name: Verdant
  role: Example upstream annotation tool whose output can feed ReFernment
- name: AGORA
  role: Example upstream annotation tool whose output can feed ReFernment
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1002/aps3.1216
    title: 'ReFernment: An R package for annotating <scp>RNA</scp> editing in plastid
      genomes'
  source_content_hash: sha256:9870cde0f2b21e09a200e3ae8a8e8c3d6510e9cabc521c03a41c904590073ffa
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - AGORA
  - CpGAVAS
  - DOGMA
  - R
  - ReFernment
  - Verdant
  edam_operation: http://edamontology.org/operation_0372
  edam_topics:
  - http://edamontology.org/topic_0085
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 15
---

# conceptual-protein-translation

## Summary

Generate conceptual translations for coding sequences whose nonsense mutations are explained by RNA editing, as part of ReFernment's editing-aware plastome annotation workflow, producing a protein FASTA alongside a five-column GenBank feature table.

## When to use

Use when annotating fern or hornwort plastid genomes with high RNA editing, where internal stop codons must be translated through to produce correct protein sequences for GenBank submission.

## When NOT to use

- The goal is predicting every RNA editing site from RNA-Seq data — ReFernment is explicitly not intended to predict every editing site like RNA-Seq-based packages
- Input annotations are sloppy or unverified — ReFernment is not a substitute for manually checking gene annotations
- GenBank files were created in Sequin — a known error where ReFernment does not produce annotations for Sequin-created files

## Inputs

- GFF3 file with nucleotide sequence embedded (required, via gffFolderPath)
- GenBank flat file with nucleotide sequence (optional, via gbFolderPath)
- Separate FASTA file (optional, via fastaFolderPath)
- genomes vector of file basenames without extensions (e.g. c("Asplenium_pek", "Woodwardia_uni"))

## Outputs

- Protein FASTA file containing conceptual translations for coding sequences where RNA editing occurred
- Five-column feature table formatted correctly for submission to GenBank
- Edited GenBank flat file with misc_feature flags marking RNA-edited nonsense mutation sites

## How to apply

Install ReFernment from GitHub with install_github("TARobison/ReFernment") and provide a GFF3 file with the sequence embedded (optionally a GenBank flat file and separate FASTA file). ReFernment first checks start and final codons of each gene, adjusting gene boundaries by looking at nearby codons within five codons, then checks internal stops and whether RNA editing would restore them. For coding sequences with RNA editing, it writes conceptual translations to a protein FASTA file and annotates edited nonsense-mutation sites with misc_feature flags in the edited GenBank flat file. If translation of a protein fails — usually because the CDS spans the genome origin — shift the origin so no CDS overlaps it and rerun; warnings about more than 5 internal stops (especially more than 10) indicate possible frameshift/assembly or annotation-frame errors requiring manual inspection.

## Related tools

- **ReFernment** (R package that detects RNA-editing-restorable internal stop codons, computes conceptual translations, and writes the protein FASTA and GenBank feature table) — github.com/TARobison/ReFernment
- **R** (Runtime environment in which ReFernment is installed via devtools and executed)
- **DOGMA** (Example upstream tool whose plastome annotation provides the GFF3 input ReFernment corrects)
- **CpGAVAS** (Example upstream annotation tool whose output can feed ReFernment)
- **Verdant** (Example upstream annotation tool whose output can feed ReFernment)
- **AGORA** (Example upstream annotation tool whose output can feed ReFernment)

## Examples

```
library("ReFernment"); genomes <- c("Asplenium_pek", "Woodwardia_uni"); ReFernment(gffFolder, fastaFolderPath=NULL, gbFolderPath=NULL, outputFolder, genomes)
```

## Evaluation signals

- Output folder contains both the five-column GenBank-ready feature table and the protein FASTA of conceptual translations
- Edited nonsense-mutation sites are flagged as misc_feature in the GenBank flat file with no false positives (as validated on cDNA-confirmed plastomes AB086179 and AY178864.1)
- Protein FASTA entries translate without internal stop codons at RNA-editing-restorable sites
- No 'problem with the protein sequence' warning appears; if it does, verify no CDS spans the genome origin and shift the origin if needed
- No 'high number of edited Stops' warning (>5 internal stops, investigate especially >10) for any gene

## Limitations

- More than 5 internal stops in a coding sequence triggers a warning to manually check for frameshift/assembly or annotation-frame errors, especially if more than 10
- Translation errors usually arise from a CDS spanning the genome origin; the origin must be shifted
- ReFernment is not a substitute for manually checking gene annotations nor a fix for sloppy annotation
- Known issue: no annotations are produced for files created in Sequin; files exported from Geneious still work
- Input annotation quality (e.g. from DOGMA, CpGAVAS, Verdant, or AGORA) directly limits output quality

## Evidence

- [abstract] ReFernment produces a five‐column feature table, formatted correctly for submission to GenBank, and a protein FASTA file: "ReFernment produces a five‐column feature table, formatted correctly for submission to GenBank, and a protein FASTA file"
- [abstract] internal stops and RNA editing restoration: "Next, ReFernment checks whether a gene has any internal stops, and if so, checks whether RNA editing would restore these nonsense mutations"
- [readme] warning threshold on internal stops: "The warning will appear if a given coding sequence has more than 5 internal stops."
- [readme] translation error cause: "This is usually due to the coding sequence spanning the orgin of the genome. I suggest that you shift the origin of the genome so that it doesn't have a CDS overlapping it."
- [readme] not an every-site predictor: "ReFernment is not intended to predict every RNA editing site, as can other available software packages."
