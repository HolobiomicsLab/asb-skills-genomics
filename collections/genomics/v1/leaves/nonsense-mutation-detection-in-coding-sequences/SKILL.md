---
name: nonsense-mutation-detection-in-coding-sequences
description: Use when annotating plastid genomes with high levels of RNA editing where
  internal stop codons appear in CDSs and must be flagged as RNA-edited sites for
  GenBank submission without cDNA data.
when_to_use_negative:
- Input GFF3 files were created in Sequin — ReFernment has a known error and does
  not produce annotations for Sequin files (use Geneious-exported files instead)
- Goal is to predict every RNA editing site genome-wide using RNA-Seq data — ReFernment
  is not intended to predict every RNA editing site, unlike RNA-Seq-based packages
- Genes have more than 5 internal stops — ReFernment errors and asks the user to manually
  check for frameshift or annotation errors rather than auto-annotating
edam_operation: http://edamontology.org/operation_0361
edam_topics:
- http://edamontology.org/topic_0080
tools:
- name: ReFernment
  role: R package that detects internal stop codons, checks whether RNA editing would
    restore them, and annotates the sites with misc_feature flags in the GenBank flat
    file
  repo: github.com/TARobison/ReFernment
- name: R
  role: Runtime environment; ReFernment is installed via devtools::install_github
    and invoked as an R function
  repo: github.com/TARobison/ReFernment
- name: DOGMA
  role: Upstream annotation generator whose GFF3-style annotation serves as ReFernment's
    input
- name: CpGAVAS
  role: Upstream annotation generator whose output can be fed to ReFernment
- name: Verdant
  role: Upstream annotation generator whose output can be fed to ReFernment
- name: AGORA
  role: Upstream annotation generator whose output can be fed to ReFernment
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1002/aps3.1216
    title: 'ReFernment: An R package for annotating <scp>RNA</scp> editing in plastid
      genomes'
  source_content_hash: sha256:b058c0155e90095d3dac1694dd4399484ea7d5cc8d7e3130f8e83891747c3072
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
  edam_operation: http://edamontology.org/operation_0361
  edam_topics:
  - http://edamontology.org/topic_0080
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6428571428571429
n_claims: 14
---

# nonsense-mutation-detection-in-coding-sequences

## Summary

Detect and annotate apparent nonsense mutations (internal stop codons) in plastid coding sequences that can reasonably be explained by RNA editing, using the ReFernment R package to produce GenBank-ready annotations and conceptual translations.

## When to use

Use when annotating plastid genomes with high levels of RNA editing where internal stop codons appear in CDSs and must be flagged as RNA-edited sites for GenBank submission without cDNA data.

## When NOT to use

- Input GFF3 files were created in Sequin — ReFernment has a known error and does not produce annotations for Sequin files (use Geneious-exported files instead)
- Goal is to predict every RNA editing site genome-wide using RNA-Seq data — ReFernment is not intended to predict every RNA editing site, unlike RNA-Seq-based packages
- Genes have more than 5 internal stops — ReFernment errors and asks the user to manually check for frameshift or annotation errors rather than auto-annotating

## Inputs

- GFF3 annotation file with sequence embedded (required)
- GenBank flat file with nucleotide sequence (optional)
- Separate FASTA file (optional)
- genomes vector of file base names without extensions (e.g. c('AB086179','AY178864.1'))

## Outputs

- Edited GenBank flat file with misc_feature RNA-editing flags and conceptual translations
- Five-column GenBank feature table formatted for submission to GenBank
- Protein FASTA file with conceptual translations of coding sequences

## How to apply

Install ReFernment from GitHub (devtools::install_github('TARobison/ReFernment')) and prepare a GFF3 file with the sequence embedded for each plastome, keeping GB and GFF file names identical except for extensions and excluding extensions from the genomes vector. Call ReFernment(gffFolder, fastaFolderPath=NULL, gbFolderPath=NULL, outputFolder, genomes), where genomes is a vector of accession names such as c('AB086179','AY178864.1'). The package checks starting and final codons of each gene, then detects internal stops and determines whether RNA editing would restore them, adding misc_feature RNA-editing flags to the GenBank flat file. Monitor for warnings: 'There are a high number of edited Stops' triggers when a CDS has more than 5 internal stops (possible frameshift or annotation error), and the protein-sequence translation warning usually indicates a CDS spanning the genome origin. Outputs are an edited GenBank flat file, a five-column GenBank feature table, and a protein FASTA of conceptual translations; validate by confirming every nonsense mutation is annotated as RNA-edited with zero false positive annotations.

## Related tools

- **ReFernment** (R package that detects internal stop codons, checks whether RNA editing would restore them, and annotates the sites with misc_feature flags in the GenBank flat file) — github.com/TARobison/ReFernment
- **R** (Runtime environment; ReFernment is installed via devtools::install_github and invoked as an R function) — github.com/TARobison/ReFernment
- **DOGMA** (Upstream annotation generator whose GFF3-style annotation serves as ReFernment's input)
- **CpGAVAS** (Upstream annotation generator whose output can be fed to ReFernment)
- **Verdant** (Upstream annotation generator whose output can be fed to ReFernment)
- **AGORA** (Upstream annotation generator whose output can be fed to ReFernment)

## Examples

```
genomes <- c('AB086179','AY178864.1'); ReFernment(gffFolder, fastaFolderPath=NULL, gbFolderPath=NULL, outputFolder, genomes)
```

## Evaluation signals

- On cDNA-confirmed high-RNA-editing plastomes (AB086179 and AY178864.1), every nonsense mutation is annotated as RNA-edited with zero false positive annotations
- For plastomes with no sequencing or assembly errors, ReFernment produces no false positive annotations of apparent nonsense mutations
- The edited GenBank flat file contains misc_feature flags at the RNA-edited sites and conceptual translations for affected CDSs
- The five-column feature table is formatted correctly for submission to GenBank
- Any CDS with more than 5 internal stops triggered the 'high number of edited Stops' warning and was manually checked before acceptance

## Limitations

- ReFernment does not produce annotations for files created in Sequin (known issue); files exported from Geneious still work
- ReFernment is not a substitute for manually checking gene annotations, nor is it a fix for sloppy annotation — output quality depends on input annotation quality
- A CDS with more than 5 internal stops triggers an error suggesting manual checking; more than 10 internal stops strongly suggests assembly or annotation error
- Protein-sequence translation warnings usually arise from a CDS spanning the genome origin; the origin should be shifted so no CDS overlaps it
- If more than five internal stops are detected, ReFernment will produce an error rather than auto-annotating

## Evidence

- [readme] ReFernment takes as input a GFF3 file (sequence embedded) and generates new annotations for nonsense mutations that can reasonably be explained by RNA editing: "ReFernment takes as input a GFF3 file (sequence embedded) and generates new annotations for nonsense mutations that can reasonably be explained by RNA editing"
- [methods] The warning will appear if a given coding sequence has more than 5 internal stops.: "The warning will appear if a given coding sequence has more than 5 internal stops."
- [abstract] every nonsense mutation was correctly annotated, and there were no false positive annotations.: "every nonsense mutation was correctly annotated, and there were no false positive annotations."
- [intro] there is a known error where ReFernment is not producing annotations for files created in sequin: "there is a known error where ReFernment is not producing annotations for files created in sequin"
- [abstract] ReFernment is not a substitute for manually checking gene annotations, nor is ReFernment a fix for sloppy annotation.: "ReFernment is not a substitute for manually checking gene annotations, nor is ReFernment a fix for sloppy annotation."
