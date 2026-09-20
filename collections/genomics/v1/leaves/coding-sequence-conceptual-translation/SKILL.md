---
name: coding-sequence-conceptual-translation
description: Use when a fern or hornwort plastome annotation contains internal stop
  codons that are plausibly explainable by C-to-U RNA editing, and NCBI would reject
  the annotation without RNA-editing flags and corrected translations.
when_to_use_negative:
- You need to predict every RNA editing site from RNA-Seq evidence — ReFernment is
  not intended to predict every site and does not rely on RNA-Seq data
- Input annotations were created in Sequin — ReFernment currently does not produce
  annotations for Sequin-created files (known issue); files exported from Geneious
  still work
- The gene annotation itself is sloppy or the genome has assembly errors — ReFernment
  is not a substitute for manually checking gene annotations nor a fix for bad annotation
edam_operation: http://edamontology.org/operation_0310
edam_topics:
- http://edamontology.org/topic_0085
- http://edamontology.org/topic_0780
tools:
- name: ReFernment
  role: R package that detects nonsense mutations restorable by RNA editing, adds
    misc_feature flags, and writes conceptual translations plus GenBank-ready feature
    tables
  repo: https://github.com/TARobison/ReFernment
- name: R
  role: Runtime environment; ReFernment is installed via devtools::install_github
    and invoked as library('ReFernment')
  repo: https://github.com/TARobison/ReFernment
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1002/aps3.1216
    title: 'ReFernment: An R package for annotating <scp>RNA</scp> editing in plastid
      genomes'
  source_content_hash: sha256:40ad74b17241b6a9a08c20c12e52251184ccc3193dd8832a2f40fa19d9234136
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/TARobison/ReFernment
  tools:
  - R
  - ReFernment
  edam_operation: http://edamontology.org/operation_0310
  edam_topics:
  - http://edamontology.org/topic_0085
  - http://edamontology.org/topic_0780
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# coding-sequence-conceptual-translation

## Summary

Annotates apparent nonsense mutations (internal stop codons) in high-RNA-editing plastid genomes using ReFernment, an R package that adds misc_feature RNA-editing flags to GenBank records and produces conceptual (edited) protein translations ready for GenBank submission.

## When to use

Use when a fern or hornwort plastome annotation contains internal stop codons that are plausibly explainable by C-to-U RNA editing, and NCBI would reject the annotation without RNA-editing flags and corrected translations.

## When NOT to use

- You need to predict every RNA editing site from RNA-Seq evidence — ReFernment is not intended to predict every site and does not rely on RNA-Seq data
- Input annotations were created in Sequin — ReFernment currently does not produce annotations for Sequin-created files (known issue); files exported from Geneious still work
- The gene annotation itself is sloppy or the genome has assembly errors — ReFernment is not a substitute for manually checking gene annotations nor a fix for bad annotation

## Inputs

- GFF3 annotation file with nucleotide sequence embedded (required)
- GenBank flat file (.gb) with nucleotide sequence (optional)
- Separate FASTA nucleotide file (optional)
- genomes vector of file names without extensions (e.g., c('AB086179', 'AY178864.1'))
- gffFolderPath, outputFolderPath, and optionally gbFolderPath / fastaFolderPath directory paths

## Outputs

- Edited GenBank flat file with misc_feature RNA-editing flags and conceptual translations
- Five-column GenBank feature table formatted for GenBank submission
- Protein FASTA file with conceptual translations of RNA-edited coding sequences

## How to apply

Install ReFernment from GitHub in R (devtools::install_github('TARobison/ReFernment')) and supply a GFF3 file with the sequence embedded as the only required input; a GenBank flat file and separate FASTA are optional. Keep GB and GFF file names identical except for extensions (e.g., AB086179.gb and AB086179.gff), exclude extensions from the genomes vector, and direct outputFolder to a directory separate from the inputs to avoid overwriting original GB files. ReFernment checks starting and final codons (adjusting boundaries using RNA editing and nearby codons within five codons), then checks whether internal stops can be restored by RNA editing, editing the GenBank flat file with 'misc_feature' flags and conceptual translations. It warns when a coding sequence has more than 5 internal stops (check for assembly or annotation errors, especially above 10) and when protein-sequence translation fails (usually a CDS spanning the genome origin; shift the origin). Outputs are the edited GenBank flat file, a five-column GenBank feature table, and a protein FASTA with conceptual translations.

## Related tools

- **ReFernment** (R package that detects nonsense mutations restorable by RNA editing, adds misc_feature flags, and writes conceptual translations plus GenBank-ready feature tables) — https://github.com/TARobison/ReFernment
- **R** (Runtime environment; ReFernment is installed via devtools::install_github and invoked as library('ReFernment')) — https://github.com/TARobison/ReFernment

## Examples

```
ReFernment(gffFolder, fastaFolderPath=NULL, gbFolderPath=NULL, outputFolder, genomes=c('AB086179','AY178864.1'))
```

## Evaluation signals

- Every internal stop codon in each gene is annotated as RNA-edited, and no false positive annotations are produced (validated zero false positives on cDNA-confirmed plastomes AB086179 and AY178864.1)
- The edited GenBank flat file contains misc_feature flags at each RNA-editing site and conceptual translations replacing the stop-containing translations
- No warning 'There are a high number of edited Stops' fired unexamined; any gene flagged for >5 internal stops (especially >10) was manually inspected for frameshift or annotation error
- No 'There seems to be a problem with the protein sequence' warning fired, or any flagged protein (typically a CDS spanning the genome origin) was manually checked in the protein FASTA
- The five-column feature table is present and formatted correctly for GenBank submission, and per-accession stop counts were compiled (e.g., refernment_nonsense_mutations.csv)

## Limitations

- More than five internal stops in a coding sequence triggers an error asking the user to manually check that gene; >10 internal stops strongly suggests assembly or annotation error
- Protein-sequence translation errors usually occur when a CDS spans the genome origin; the user should shift the origin before re-running
- ReFernment is not a substitute for manually checking gene annotations, nor a fix for sloppy annotation — output quality depends on input annotation quality
- Known issue: ReFernment does not produce annotations for files created in Sequin; files exported from Geneious work
- No software version number is stated for ReFernment in the article or README, complicating reproducibility pinning

## Evidence

- [abstract] ReFernment then edits the imputed GenBank flat file, adding conceptual translations and annotations indicating the sites where RNA editing occurred with ‘misc_feature’ flags: "ReFernment then edits the imputed GenBank flat file, adding conceptual translations and annotations indicating the sites where RNA editing occurred with ‘misc_feature’ flags"
- [methods] The warning will appear if a given coding sequence has more than 5 internal stops.: "The warning will appear if a given coding sequence has more than 5 internal stops."
- [readme] This is usually due to the coding sequence spanning the orgin of the genome. I suggest that you shift the origin of the genome: "This is usually due to the coding sequence spanning the orgin of the genome. I suggest that you shift the origin of the genome so that it doesn't have a CDS overlapping it."
- [readme] ReFernment is not intended to predict every RNA editing site, as can other available software packages. These packages rely on RNA-Seq data: "It should be made clear that ReFernment is not intended to predict every RNA editing site, as can other available software packages. These packages rely on RNA-Seq data"
- [intro] there is a known error where ReFernment is not producing annotations for files created in sequin.: "there is a known error where ReFernment is not producing annotations for files created in sequin. I will fix the issue as soon as possible"
