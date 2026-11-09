---
name: genbank-flatfile-formatting
description: Use when a plastid genome (e.g. fern or hornwort) annotation contains
  internal stop codons that could be explained by RNA editing, and a GenBank-submittable
  flat file with conceptual translations is needed.
when_to_use_negative:
- Files created in Sequin — ReFernment has a known error not producing annotations
  for them; use files exported from Geneious instead
- You need prediction of every RNA editing site from RNA-Seq data — ReFernment is
  not intended to predict every site and does not use RNA-Seq comparisons
- The input annotation itself is sloppy or unverified — ReFernment is not a substitute
  for manually checking gene annotations
edam_operation: http://edamontology.org/operation_0334
edam_topics:
- http://edamontology.org/topic_0085
- http://edamontology.org/topic_0780
tools:
- name: ReFernment
  role: R package that checks codon validity, annotates RNA-editing-restored nonsense
    mutations, edits the GenBank flat file, and produces the feature table and protein
    FASTA
  repo: github.com/TARobison/ReFernment
- name: R
  role: Runtime environment in which ReFernment is installed via install_github and
    executed
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1002/aps3.1216
    title: 'ReFernment: An R package for annotating <scp>RNA</scp> editing in plastid
      genomes'
  source_content_hash: sha256:47d41d67e6441a687cdf2435bb38d02eebd44dbd9ae93478085c7c1f09da8ca3
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - R
  - ReFernment
  edam_operation: http://edamontology.org/operation_0334
  edam_topics:
  - http://edamontology.org/topic_0085
  - http://edamontology.org/topic_0780
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 12
---

# genbank-flatfile-formatting

## Summary

Refine existing plastid gene annotations for RNA editing using the ReFernment R package, which fixes nonsense mutations via assumed C-to-U/U-to-C edits and edits the GenBank flat file with 'misc_feature' flags and conceptual translations so it is ready for GenBank submission.

## When to use

Use when a plastid genome (e.g. fern or hornwort) annotation contains internal stop codons that could be explained by RNA editing, and a GenBank-submittable flat file with conceptual translations is needed.

## When NOT to use

- Files created in Sequin — ReFernment has a known error not producing annotations for them; use files exported from Geneious instead
- You need prediction of every RNA editing site from RNA-Seq data — ReFernment is not intended to predict every site and does not use RNA-Seq comparisons
- The input annotation itself is sloppy or unverified — ReFernment is not a substitute for manually checking gene annotations

## Inputs

- GFF3 file with sequence embedded (required)
- GenBank flat file including nucleotide sequence (optional)
- Separate FASTA file (optional)
- genomes vector of file basenames without extensions (e.g. 'Asplenium_pek', 'Woodwardia_uni')

## Outputs

- Refined GenBank flat file with 'misc_feature' flags at RNA-editing sites and conceptual translations, formatted for GenBank submission
- Five-column GenBank feature table
- Protein FASTA file of conceptual translations

## How to apply

Install ReFernment from GitHub (install_github('TARobison/ReFernment')) and load it in R. Declare a 'genomes' vector of file basenames without extensions (e.g. genomes <- c('Asplenium_pek', 'Woodwardia_uni')) and set gffFolder and outputFolder paths (gbFolderPath and fastaFolderPath optional); keep GB and GFF file names identical except for extensions. Call ReFernment(gffFolder, fastaFolderPath=NULL, gbFolderPath=NULL, outputFolder, genomes): it checks start and final codons of each gene, shifting boundaries to valid codons within five codons where needed, then tests each internal stop codon for a restoring RNA edit (e.g. ACG to AUG) and corrects the translation. The tool edits the imputed GenBank flat file, adding 'misc_feature' flags at RNA-editing sites and conceptual translations, and outputs a five-column GenBank feature table plus a protein FASTA file. If more than 5 internal stops appear in a coding sequence, treat the warning as a likely assembly or frame error to check manually; if a protein-sequence translation error occurs (e.g. a CDS spanning the genome origin), shift the origin and rerun.

## Related tools

- **ReFernment** (R package that checks codon validity, annotates RNA-editing-restored nonsense mutations, edits the GenBank flat file, and produces the feature table and protein FASTA) — github.com/TARobison/ReFernment
- **R** (Runtime environment in which ReFernment is installed via install_github and executed)

## Examples

```
library("ReFernment"); genomes <- c("Asplenium_pek", "Woodwardia_uni"); ReFernment(gffFolder, fastaFolderPath=NULL, gbFolderPath=NULL, outputFolder, genomes)
```

## Evaluation signals

- No 'high number of edited Stops' warning was raised — at most 5 internal stops per coding sequence after RNA-edit correction
- No protein-sequence translation error warning was reported for any gene
- Output GenBank file is a five-column feature table formatted for submission to GenBank
- Protein FASTA contains conceptual translations for every coding sequence with RNA editing
- In cDNA-confirmed high-RNA-editing plastomes, every nonsense mutation is annotated with no false positives

## Limitations

- ReFernment currently does not produce annotations for files created in Sequin (known issue); files exported from Geneious still work
- ReFernment is not a substitute for manually checking gene annotations, nor a fix for sloppy annotation
- More than five (especially more than 10) internal stops in a coding sequence usually indicates assembly or annotation frame errors requiring manual inspection
- Protein translation errors can occur when a coding sequence spans the genome origin; shift the origin so no CDS overlaps it and rerun
- No software version number for ReFernment is stated in the article or README

## Evidence

- [abstract] ReFernment requires both a GFF3 (no sequence) file and a GenBank flat file (including nucleotide sequence): "ReFernment requires both a GFF3 (no sequence) file and a GenBank flat file (including nucleotide sequence)"
- [abstract] ReFernment then edits the imputed GenBank flat file, adding conceptual translations and annotations indicating the sites where RNA editing occurred with 'misc_feature' flags: "ReFernment then edits the imputed GenBank flat file, adding conceptual translations and annotations indicating the sites where RNA editing occurred with ‘misc_feature’ flags"
- [abstract] ReFernment produces a five‐column feature table, formatted correctly for submission to GenBank, and a protein FASTA file: "ReFernment produces a five‐column feature table, formatted correctly for submission to GenBank, and a protein FASTA file"
- [readme] The warning will appear if a given coding sequence has more than 5 internal stops.: "The warning will appear if a given coding sequence has more than 5 internal stops."
- [intro] there is a known error where ReFernment is not producing annotations for files created in sequin: "there is a known error where ReFernment is not producing annotations for files created in sequin"
- [readme] This is usually due to the coding sequence spanning the orgin of the genome. I suggest that you shift the origin of the genome so that it doesn't have a CDS overlapping it.: "This is usually due to the coding sequence spanning the orgin of the genome. I suggest that you shift the origin of the genome so that it doesn't have a CDS overlapping it."
