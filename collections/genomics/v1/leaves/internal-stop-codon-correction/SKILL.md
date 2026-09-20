---
name: internal-stop-codon-correction
description: Use when a plastid genome (e.g., fern or hornwort, high RNA editing)
  has annotations with internal stop codons that would be rejected by NCBI, and no
  cDNA/RNA-Seq data is available to confirm editing sites.
when_to_use_negative:
- Files created in Sequin — a known error means ReFernment does not produce annotations
  for them (use files exported from Geneious instead)
- The goal is predicting every RNA editing site from RNA-Seq data — ReFernment is
  not intended for that and relies on assumed C-to-U/U-to-C edits without cDNA
- The input annotation itself is sloppy — ReFernment is not a substitute for manually
  checking gene annotations nor a fix for sloppy annotation
edam_operation: http://edamontology.org/operation_0361
edam_topics:
- http://edamontology.org/topic_0085
- http://edamontology.org/topic_0622
tools:
- name: ReFernment
  role: R package that checks start/final codons, detects internal stops, tests C-to-U/U-to-C
    RNA-edit restoration, and writes corrected GenBank files with conceptual translations
  repo: github.com/TARobison/ReFernment
- name: R
  role: Runtime environment; ReFernment is installed via devtools::install_github
    and called as an R function
- name: DOGMA
  role: Upstream annotation program whose GFF3/GenBank output serves as ReFernment's
    input
- name: Geneious
  role: File export source; ReFernment works with files exported from Geneious (unlike
    Sequin files)
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1002/aps3.1216
    title: 'ReFernment: An R package for annotating <scp>RNA</scp> editing in plastid
      genomes'
  source_content_hash: sha256:ec78544bc4000e06187c731e6cbe18dd2f4b3cca1d28db9897e64b9e20867a0b
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - DOGMA
  - Geneious
  - R
  - ReFernment
  edam_operation: http://edamontology.org/operation_0361
  edam_topics:
  - http://edamontology.org/topic_0085
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5384615384615384
n_claims: 13
---

# internal-stop-codon-correction

## Summary

Refines existing plastid gene annotations by detecting internal stop codons and correcting those explainable by C-to-U/U-to-C RNA editing, producing GenBank-ready files with conceptual translations using the ReFernment R package.

## When to use

Use when a plastid genome (e.g., fern or hornwort, high RNA editing) has annotations with internal stop codons that would be rejected by NCBI, and no cDNA/RNA-Seq data is available to confirm editing sites.

## When NOT to use

- Files created in Sequin — a known error means ReFernment does not produce annotations for them (use files exported from Geneious instead)
- The goal is predicting every RNA editing site from RNA-Seq data — ReFernment is not intended for that and relies on assumed C-to-U/U-to-C edits without cDNA
- The input annotation itself is sloppy — ReFernment is not a substitute for manually checking gene annotations nor a fix for sloppy annotation

## Inputs

- GFF3 file with sequence embedded (required)
- GenBank flat file including nucleotide sequence (optional)
- Separate FASTA file (optional)
- Vector of genome file names without file extensions (e.g., genomes <- c('Asplenium_pek', 'Woodwardia_uni'))
- Paths to GFF folder, output folder, and optionally GB and FASTA folders

## Outputs

- Edited GenBank flat file with misc_feature flags at RNA-editing sites and conceptual translations
- Five-column GenBank feature table formatted for submission
- Protein FASTA file of conceptual translations
- Warning messages for genes with >5 internal stops or translation failures

## How to apply

Install ReFernment from GitHub (install_github('TARobison/ReFernment')) and load it in R. Prepare a GFF3 file with sequence embedded (optionally a GenBank flat file and separate FASTA file), keeping GB and GFF filenames identical except for extensions, and declare a `genomes` vector without file extensions. Call ReFernment(gffFolder, fastaFolderPath, gbFolderPath, outputFolder, genomes): it first checks the starting and final codons of each gene, shifting gene boundaries to valid codons within five codons where needed, then checks each gene for internal stops and tests whether an assumed RNA edit (e.g., ACG to AUG) restores a sense codon, correcting the annotation accordingly. If a coding sequence has more than 5 internal stops, treat the warning as a likely assembly or frame error and manually inspect that gene; if translation fails (e.g., a CDS spanning the genome origin), shift the origin and rerun. Validation succeeds when no internal-stop warning is raised and no protein-sequence translation error is reported, indicating the output GenBank flat file with misc_feature flags at editing sites and the protein FASTA are ready for GenBank submission.

## Related tools

- **ReFernment** (R package that checks start/final codons, detects internal stops, tests C-to-U/U-to-C RNA-edit restoration, and writes corrected GenBank files with conceptual translations) — github.com/TARobison/ReFernment
- **R** (Runtime environment; ReFernment is installed via devtools::install_github and called as an R function)
- **DOGMA** (Upstream annotation program whose GFF3/GenBank output serves as ReFernment's input)
- **Geneious** (File export source; ReFernment works with files exported from Geneious (unlike Sequin files))

## Examples

```
library("ReFernment"); genomes <- c("Asplenium_pek", "Woodwardia_uni"); ReFernment(gffFolder, fastaFolderPath=NULL, gbFolderPath=NULL, outputFolder, genomes)
```

## Evaluation signals

- No warning of the form 'There are a high number of edited Stops ([number]) in [geneName]' was emitted — i.e., every coding sequence has at most 5 internal stops
- No warning of the form 'There seems to be a problem with the protein sequence of ([protein name])' was emitted
- Output GenBank flat file contains misc_feature flags at RNA-editing sites and conceptual translations for edited CDS
- Protein FASTA translations of edited genes are complete sense sequences with no residual internal stop codons
- Confirmed against cDNA data (e.g., GenBank AB086179 and AY178864.1) every nonsense mutation was correctly annotated with no false positives

## Limitations

- If more than five internal stops are detected in a gene, ReFernment errors and asks the user to manually check that gene (possible assembly or frame error)
- Protein-sequence translation errors usually arise when a coding sequence spans the origin of the genome; the user should shift the origin and rerun
- ReFernment is not a substitute for manually checking gene annotations, nor a fix for sloppy annotation; output quality depends on input annotation quality
- Known issue: ReFernment does not produce annotations for files created in Sequin
- No software version is stated; install via install_github('TARobison/ReFernment')

## Evidence

- [abstract] Next, ReFernment checks whether a gene has any internal stops, and if so, checks whether RNA editing would restore these nonsense mutations: "ReFernment checks whether a gene has any internal stops, and if so, checks whether RNA editing would restore these nonsense mutations"
- [abstract] ReFernment requires both a GFF3 (no sequence) file and a GenBank flat file (including nucleotide sequence): "ReFernment requires both a GFF3 (no sequence) file and a GenBank flat file (including nucleotide sequence)"
- [methods] The warning will appear if a given coding sequence has more than 5 internal stops.: "The warning will appear if a given coding sequence has more than 5 internal stops."
- [methods] ReFernment takes as input a GFF3 file (sequence embedded) and generates new annotations for nonsense mutations that: "ReFernment takes as input a GFF3 file (sequence embedded) and generates new annotations for nonsense mutations"
- [readme] This is usually due to the coding sequence spanning the orgin of the genome. I suggest that you shift the origin of the genome so that it doesn't have a CDS overlapping it.: "This is usually due to the coding sequence spanning the orgin of the genome. I suggest that you shift the origin of the genome so that it doesn't have a CDS overlapping it."
