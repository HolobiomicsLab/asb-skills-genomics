---
name: plastome-coding-sequence-stop-codon-detection
description: Use when annotating high-RNA-editing plastomes where internal stop codons
  may be nonsense mutations correctable by RNA editing, and when deciding whether
  an apparent frameshift reflects editing or an assembly/annotation error.
when_to_use_negative:
- When predicting every RNA editing site using RNA-Seq data comparison — ReFernment
  is explicitly not intended for this and other packages exist for that purpose
- As a substitute for manually checking gene annotations or a fix for sloppy annotation
- For GFF files created in Sequin, due to a known issue where annotations are not
  produced
edam_operation: http://edamontology.org/operation_0447
edam_topics:
- http://edamontology.org/topic_0659
- http://edamontology.org/topic_0780
tools:
- name: ReFernment
  role: R package that translates plastid CDSs, counts internal stop codons, applies
    the >5-stop manual-review guard, and annotates RNA-editing-corrected nonsense
    mutations
  repo: github.com/TARobison/ReFernment
- name: R
  role: Runtime environment in which ReFernment is installed via install_github('TARobison/ReFernment')
    and executed on plastome GFF3/GenBank inputs
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1002/aps3.1216
    title: 'ReFernment: An R package for annotating <scp>RNA</scp> editing in plastid
      genomes'
  source_content_hash: sha256:3f4ad8eeffd2e5d608674257b03ec61f0fd06dcee9b363a3e05cdb7978ebf68a
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - R
  - ReFernment
  edam_operation: http://edamontology.org/operation_0447
  edam_topics:
  - http://edamontology.org/topic_0659
  - http://edamontology.org/topic_0780
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 10
---

# plastome-coding-sequence-stop-codon-detection

## Summary

Detect and count internal stop codons in plastid genome coding sequences to flag potential RNA editing sites versus assembly or annotation errors, using ReFernment's >5 internal stops threshold to trigger manual review.

## When to use

Use when annotating high-RNA-editing plastomes where internal stop codons may be nonsense mutations correctable by RNA editing, and when deciding whether an apparent frameshift reflects editing or an assembly/annotation error.

## When NOT to use

- When predicting every RNA editing site using RNA-Seq data comparison — ReFernment is explicitly not intended for this and other packages exist for that purpose
- As a substitute for manually checking gene annotations or a fix for sloppy annotation
- For GFF files created in Sequin, due to a known issue where annotations are not produced

## Inputs

- GFF3 file with plastid sequence embedded (required)
- GenBank flat file including nucleotide sequence (optional)
- Separate FASTA file (optional)
- Vector of genome file names without extensions (e.g., Asplenium_pek, Woodwardia_uni)

## Outputs

- Per-gene internal stop codon counts driving the >5 threshold warning
- Warning/error messages listing [number] of edited stops and [geneName] for manual check
- Five-column GenBank feature table with misc_feature RNA-editing flags and conceptual translations
- Protein FASTA file with conceptual translations

## How to apply

Parse a plastome GFF3 file (sequence embedded, optionally with a GenBank flat file and FASTA) using the ReFernment R package, extract each coding sequence, and translate it to count internal stop codons per gene. Apply the safeguard: if a coding sequence has more than 5 internal stops, emit the error 'There are a high number of edited Stops ( [number] ) in [geneName] manually check to make sure frame is correct' prompting manual review of that gene. This threshold distinguishes RNA editing from two common problems: assembly errors that make an entire gene appear frameshifted, and annotation errors from the wrong reading frame. For counts exceeding 10 internal stops, scrutinize the coding sequence closely for assembly or annotation errors since genuine editing at such levels is relatively rare. Stop codons whose reversal by RNA editing (e.g., C-to-U edits) is plausible are then annotated, and conceptual translations are produced.

## Related tools

- **ReFernment** (R package that translates plastid CDSs, counts internal stop codons, applies the >5-stop manual-review guard, and annotates RNA-editing-corrected nonsense mutations) — github.com/TARobison/ReFernment
- **R** (Runtime environment in which ReFernment is installed via install_github('TARobison/ReFernment') and executed on plastome GFF3/GenBank inputs)

## Examples

```
library("ReFernment"); genomes <- c("Asplenium_pek", "Woodwardia_uni"); ReFernment(gffFolder, fastaFolderPath=NULL, gbFolderPath=NULL, outputFolder, genomes)
```

## Evaluation signals

- Warning fires for every coding sequence with more than 5 internal stops and does not fire at exactly 5 or fewer (boundary test with 5- and 6-stop constructs)
- The [number] field in the warning message matches the recorded internal stop count for that gene
- No false positives for data with no sequencing or assembly errors (validated against cDNA-confirmed plastomes AB086179 and AY178864.1)
- Second warning ('There seems to be a problem with the protein sequence of [protein name]') indicates translation errors, typically a CDS spanning the genome origin — shift the origin and re-run
- Output includes a correctly formatted five-column GenBank feature table and protein FASTA with conceptual translations

## Limitations

- Known issue: ReFernment does not produce annotations for files created in Sequin; files exported from Geneious still work
- More than 5 internal stops can occur without assembly error (relatively rare); >10 internal stops strongly suggests a close manual look for assembly or annotation errors
- ReFernment output quality depends on input annotation quality; it is not a substitute for manually checking gene annotations
- Translation errors can occur when a coding sequence spans the genome origin; shifting the origin is recommended

## Evidence

- [readme] Threshold trigger: "The warning will appear if a given coding sequence has more than 5 internal stops."
- [readme] Warning message and purpose: "* There are a high number of edited Stops ( [number] ) in [geneName] manually check to make sure frame is correct"
- [readme] Rationale for the guard: "1. potential assembly errors that would result in an entire gene appearing to be frameshifted 2. annotation errors where the proper frame for a gene was not selected to begin with"
- [abstract] Article finding on the guard: "if there are more than five detected internal stops in a gene, ReFernment will produce an error suggesting that the user manually check that gene."
- [readme] Input format requirement: "ReFernment *requires* only a GFF3 file (with sequence embedded) to work"
