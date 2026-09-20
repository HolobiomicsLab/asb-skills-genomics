---
name: conceptual-cds-translation
description: Use when a plastid (or similar organellar) genome annotation contains
  internal stop codons that could be explained by RNA editing, and you need corrected
  conceptual protein translations plus GenBank-ready annotation of the edited sites.
when_to_use_negative:
- You need to predict every RNA editing site using RNA-Seq/cDNA evidence — ReFernment
  is not intended to predict every editing site, only those explaining nonsense mutations
- Input files were created in Sequin — ReFernment has a known error producing annotations
  for Sequin-created files (Geneious-exported files still work)
- The gene annotation is sloppy or unverified — ReFernment is not a substitute for
  manually checking gene annotations nor a fix for sloppy annotation
edam_operation: http://edamontology.org/operation_0372
edam_topics:
- http://edamontology.org/topic_0085
- http://edamontology.org/topic_0622
tools:
- name: ReFernment
  role: R package that detects internal stops, tests C-to-U/U-to-C edits that restore
    sense codons, adds misc_feature flags, and writes conceptual translations to a
    protein FASTA
  repo: github.com/TARobison/ReFernment
- name: R
  role: Runtime environment in which ReFernment is installed via devtools::install_github
    and executed
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1002/aps3.1216
    title: 'ReFernment: An R package for annotating <scp>RNA</scp> editing in plastid
      genomes'
  source_content_hash: sha256:16e820f579ab85dae757926f02fb5403688eaef27b0e3e139d45582b08634a35
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - R
  - ReFernment
  edam_operation: http://edamontology.org/operation_0372
  edam_topics:
  - http://edamontology.org/topic_0085
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4444444444444444
n_claims: 9
---

# conceptual-cds-translation

## Summary

ReFernment's conceptual translation step converts RNA-edited plastid coding sequences into corrected amino-acid sequences: internal stops that a C-to-U/U-to-C edit can restore are translated as sense codons, and the protein sequences are exported for GenBank submission.

## When to use

Use when a plastid (or similar organellar) genome annotation contains internal stop codons that could be explained by RNA editing, and you need corrected conceptual protein translations plus GenBank-ready annotation of the edited sites.

## When NOT to use

- You need to predict every RNA editing site using RNA-Seq/cDNA evidence — ReFernment is not intended to predict every editing site, only those explaining nonsense mutations
- Input files were created in Sequin — ReFernment has a known error producing annotations for Sequin-created files (Geneious-exported files still work)
- The gene annotation is sloppy or unverified — ReFernment is not a substitute for manually checking gene annotations nor a fix for sloppy annotation

## Inputs

- GFF3 annotation file with nucleotide sequence embedded (required)
- GenBank flat file including nucleotide sequence (optional, per workflow)
- Separate FASTA file (optional, per workflow)
- genomes vector of file basenames without extensions (e.g., c('Asplenium_pek', 'Woodwardia_uni'))

## Outputs

- Refined GenBank flat file with 'misc_feature' flags at RNA-editing sites and conceptual translations embedded
- Five-column GenBank feature table formatted for GenBank submission
- Protein FASTA file of conceptual translations for coding sequences with RNA editing

## How to apply

After ReFernment has checked start/final codons and shifted gene boundaries (within five codons) to valid start/stop positions, it scans each CDS for internal stop codons and tests whether an assumed C-to-U or U-to-C RNA edit (e.g., ACG to AUG) would restore a sense codon; if so, it adjusts the translation accordingly. Each resolved editing site is flagged in the GenBank flat file with a 'misc_feature' RNA-editing flag, and the edited CDS is translated to produce a conceptual amino-acid sequence. Translations are written to a protein FASTA file alongside a five-column GenBank feature table formatted for submission. If translation of a protein fails—typically because the CDS spans the genome origin—shift the origin so no CDS overlaps it and rerun. Conceptual translation here is intentionally not a prediction of every RNA editing site from RNA-Seq data; it only resolves nonsense mutations reasonably attributable to editing.

## Related tools

- **ReFernment** (R package that detects internal stops, tests C-to-U/U-to-C edits that restore sense codons, adds misc_feature flags, and writes conceptual translations to a protein FASTA) — github.com/TARobison/ReFernment
- **R** (Runtime environment in which ReFernment is installed via devtools::install_github and executed)

## Examples

```
library(ReFernment); ReFernment(gffFolder, fastaFolderPath=NULL, gbFolderPath=NULL, outputFolder, genomes)
```

## Evaluation signals

- No 'There seems to be a problem with the protein sequence of ([protein name])' warning was emitted, indicating all CDSs translated successfully
- No 'There are a high number of edited Stops ([number]) in [geneName]' warning appeared, i.e., every coding sequence has at most 5 internal stops
- Protein FASTA contains a conceptual translation for every coding sequence flagged with RNA editing
- GenBank flat file contains 'misc_feature' flags at the RNA-editing sites and a five-column feature table ready for submission
- Translations agree with cDNA-confirmed benchmarks — the paper reports every nonsense mutation correctly annotated with no false positives on two cDNA-confirmed high-RNA-editing plastomes

## Limitations

- A coding sequence with more than 5 internal stops triggers a warning to manually check for assembly or frame errors; >10 internal stops strongly suggests assembly or annotation error
- Translation errors usually arise when a CDS spans the genome origin — shift the origin so no CDS overlaps it and rerun
- No software version is stated and there is no changelog for ReFernment; install via install_github('TARobison/ReFernment')
- Known issue: ReFernment does not produce annotations for files created in Sequin

## Evidence

- [abstract] ReFernment then edits the imputed GenBank flat file, adding conceptual translations and annotations indicating the sites where RNA editing occurred with 'misc_feature' flags: "ReFernment then edits the imputed GenBank flat file, adding conceptual translations and annotations indicating the sites where RNA editing occurred with ‘misc_feature’ flags"
- [abstract] ReFernment produces a five-column feature table, formatted correctly for submission to GenBank, and a protein FASTA file: "ReFernment produces a five‐column feature table, formatted correctly for submission to GenBank, and a protein FASTA file"
- [methods] ReFernment takes as input a GFF3 file (sequence embedded) and generates new annotations for nonsense mutations: "ReFernment takes as input a GFF3 file (sequence embedded) and generates new annotations for nonsense mutations that"
- [readme] There seems to be a problem with the protein sequence of ( [protein name] ) please check its sequence in the protein fasta manually: "There seems to be a problem with the protein sequence of ( [protein name] ) please check its sequence in the protein fasta manually"
- [readme] This is usually due to the coding sequence spanning the orgin of the genome. I suggest that you shift the origin of the genome so that it doesn't have a CDS overlapping it.: "This is usually due to the coding sequence spanning the orgin of the genome. I suggest that you shift the origin of the genome so that it doesn't have a CDS overlapping it."
- [abstract] ReFernment checks for internal stops and determines whether RNA editing would restore them.: "Next, ReFernment checks whether a gene has any internal stops, and if so, checks whether RNA editing would restore these nonsense mutations"
