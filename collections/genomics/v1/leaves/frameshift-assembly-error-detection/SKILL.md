---
name: frameshift-assembly-error-detection
description: Use when annotating fern or hornwort plastomes with high RNA editing,
  where internal stop codons may reflect genuine edits or, in excess, assembly/frameshift
  errors needing manual inspection.
when_to_use_negative:
- When the goal is predicting every RNA editing site from RNA-Seq data — ReFernment
  is not intended to predict every RNA editing site like RNA-Seq-comparison packages
- When input annotations are sloppy — ReFernment is not a substitute for manually
  checking gene annotations or a fix for poor annotation
- For GenBank flat files created in Sequin — ReFernment has a known error producing
  annotations for Sequin-created files
edam_operation: http://edamontology.org/operation_0369
edam_topics:
- http://edamontology.org/topic_0085
- http://edamontology.org/topic_0622
tools:
- name: ReFernment
  role: R package that counts internal stop codons per CDS during RNA-editing annotation
    and warns on frameshift/assembly-error candidates
  repo: github.com/TARobison/ReFernment
- name: R
  role: Runtime environment in which ReFernment is installed via devtools::install_github
    and executed
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1002/aps3.1216
    title: 'ReFernment: An R package for annotating <scp>RNA</scp> editing in plastid
      genomes'
  source_content_hash: sha256:5862c6c2490c3b8ed1bd753324a4b5468f41c0ff67309a0fc7d0d6ef5cc6a8fd
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - R
  - ReFernment
  edam_operation: http://edamontology.org/operation_0369
  edam_topics:
  - http://edamontology.org/topic_0085
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.45454545454545453
n_claims: 11
---

# frameshift-assembly-error-detection

## Summary

Detects likely frameshifts, assembly errors, and wrong-frame annotations in plastid coding sequences by counting internal stop codons during ReFernment's RNA-editing-aware annotation run, using a >5 internal stop warning threshold.

## When to use

Use when annotating fern or hornwort plastomes with high RNA editing, where internal stop codons may reflect genuine edits or, in excess, assembly/frameshift errors needing manual inspection.

## When NOT to use

- When the goal is predicting every RNA editing site from RNA-Seq data — ReFernment is not intended to predict every RNA editing site like RNA-Seq-comparison packages
- When input annotations are sloppy — ReFernment is not a substitute for manually checking gene annotations or a fix for poor annotation
- For GenBank flat files created in Sequin — ReFernment has a known error producing annotations for Sequin-created files

## Inputs

- GFF3 file with sequence embedded (required input per genome)
- GenBank flat file (.gb) with nucleotide sequence (optional)
- Separate FASTA nucleotide file (optional)

## Outputs

- Warning messages per gene listing the number of edited stops and gene name when >5 internal stops are detected
- Protein FASTA translation error warnings for CDS spanning the genome origin
- Annotated GenBank output with misc_feature flags at edited stop-codon sites

## How to apply

Run ReFernment on the GFF3 (sequence embedded) plus optional GenBank flat file; the package validates start/final codons (looking within five nearby codons to adjust gene boundaries) and then checks each CDS for internal stops that RNA editing could restore. If a coding sequence has more than 5 internal stops, ReFernment emits a warning ('There are a high number of edited Stops...') flagging two possible problems: an assembly error frameshifting the whole gene, or an annotation error where the proper reading frame was never selected. Especially when more than 10 internal stops are detected, the user should closely inspect that CDS for assembly or annotation-frame errors rather than accept the edits. A second warning about protein-sequence translation failure usually indicates a CDS spanning the genome origin, which should be resolved by shifting the origin rather than treating it as a frameshift.

## Related tools

- **ReFernment** (R package that counts internal stop codons per CDS during RNA-editing annotation and warns on frameshift/assembly-error candidates) — github.com/TARobison/ReFernment
- **R** (Runtime environment in which ReFernment is installed via devtools::install_github and executed)

## Examples

```
install_github("TARobison/ReFernment"); library("ReFernment"); ReFernment(gffFolder, fastaFolderPath=NULL, gbFolderPath=NULL, outputFolder, genomes)
```

## Evaluation signals

- No high-number-of-edited-Stops warnings are emitted for genes with clean frames (≤5 internal stops per CDS)
- Genes with >10 internal stops were manually verified for assembly error or incorrect annotation frame
- cDNA-confirmed test plastomes (AB086179 and AY178864.1) show every nonsense mutation correctly annotated with no false positives
- No protein-translation warnings persist after shifting the genome origin away from CDS overlaps
- Output folder contains the five-column GenBank feature table and protein FASTA with conceptual translations

## Limitations

- More than 5 internal stops in a CDS triggers a warning suggesting manual check of the frame; this can occur legitimately without assembly error, but is relatively rare
- Especially with more than 10 internal stops, the CDS should be closely inspected for assembly or annotation errors
- Protein translation errors usually indicate a CDS spanning the genome origin, resolved by shifting the origin
- ReFernment is not a substitute for manually checking gene annotations, nor a fix for sloppy annotation

## Evidence

- [readme] The warning will appear if a given coding sequence has more than 5 internal stops.: "The warning will appear if a given coding sequence has more than 5 internal stops."
- [readme] Especially if there are more than 10 internal stops the user should take a close look at this coding sequecne to check for assembly error or annotation errors.: "Especially if there are more than 10 internal stops the user should take a close look at this coding sequecne to check for assembly error or annotation errors."
- [readme] 1. potential assembly errors that would result in an entire gene appearing to be frameshifted 2. annotation errors where the proper frame for a gene was not selected to begin with: "potential assembly errors that would result in an entire gene appearing to be frameshifted"
- [abstract] if there are more than five detected internal stops in a gene, ReFernment will produce an error suggesting that the user manually check that gene.: "if there are more than five detected internal stops in a gene, ReFernment will produce an error suggesting that the user manually check that gene."
- [abstract] ReFernment is not a substitute for manually checking gene annotations, nor is ReFernment a fix for sloppy annotation.: "ReFernment is not a substitute for manually checking gene annotations, nor is ReFernment a fix for sloppy annotation."
