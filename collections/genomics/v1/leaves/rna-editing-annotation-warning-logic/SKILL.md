---
name: rna-editing-annotation-warning-logic
description: Use when annotating plastid genomes for RNA editing and a translated
  CDS accumulates internal stops, to decide whether stops are editable nonsense mutations
  or signs of frameshifted assembly/annotation errors.
when_to_use_negative:
- A gene has 5 or fewer internal stops, all plausibly restorable by RNA editing —
  the guard should not fire and annotation should proceed
- The goal is predicting every RNA editing site from RNA-Seq data — ReFernment explicitly
  is not intended to predict every editing site, unlike RNA-Seq-dependent packages
- The issue is a translation error (e.g., a CDS spanning the genome origin) — use
  ReFernment's separate protein-sequence warning logic and shift the genome origin
  instead
edam_operation: http://edamontology.org/operation_0361
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_0080
tools:
- name: ReFernment
  role: R package that counts internal stops per CDS and implements the >5-stop manual-review
    error/warning
  repo: github.com/TARobison/ReFernment
- name: R
  role: Runtime environment in which ReFernment is installed (install_github) and
    executed
  repo: github.com/TARobison/ReFernment
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1002/aps3.1216
    title: 'ReFernment: An R package for annotating <scp>RNA</scp> editing in plastid
      genomes'
  source_content_hash: sha256:6a5d42ba786f6e914cccae5e3b62715bb1e9e8d882a839710f3c719d168b664e
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - R
  - ReFernment
  edam_operation: http://edamontology.org/operation_0361
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_0080
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5555555555555556
n_claims: 9
---

# rna-editing-annotation-warning-logic

## Summary

A safeguard logic from ReFernment that flags coding sequences with a suspiciously high number of internal stop codons (>5), warning that they may reflect assembly errors or wrong-frame annotation rather than genuine RNA editing.

## When to use

Use when annotating plastid genomes for RNA editing and a translated CDS accumulates internal stops, to decide whether stops are editable nonsense mutations or signs of frameshifted assembly/annotation errors.

## When NOT to use

- A gene has 5 or fewer internal stops, all plausibly restorable by RNA editing — the guard should not fire and annotation should proceed
- The goal is predicting every RNA editing site from RNA-Seq data — ReFernment explicitly is not intended to predict every editing site, unlike RNA-Seq-dependent packages
- The issue is a translation error (e.g., a CDS spanning the genome origin) — use ReFernment's separate protein-sequence warning logic and shift the genome origin instead

## Inputs

- Plastome GFF3 file with sequence embedded (per ReFernment's required input)
- Optionally, a GenBank flat file and separate FASTA file
- Per-gene translated CDS internal stop codon counts

## Outputs

- Per-gene warning message 'There are a high number of edited Stops ( [number] ) in [geneName] manually check to make sure frame is correct' for genes with more than 5 internal stops
- Continued RNA editing annotation for genes with 5 or fewer internal stops
- Optionally, a recorded table of per-gene stop counts and whether the warning fired

## How to apply

Parse the plastome GFF3 (sequence embedded) with the ReFernment R package, extract each coding sequence, translate it, and count internal stop codons per gene. When the count exceeds 5, emit the warning 'There are a high number of edited Stops ( [number] ) in [geneName] manually check to make sure frame is correct' instead of treating all stops as RNA-editable; at or below 5 stops, proceed with RNA editing annotation normally. The rationale is that a large cluster of internal stops usually indicates (1) assembly errors that frameshift an entire gene or (2) annotation in the wrong reading frame, which RNA editing cannot plausibly explain. Counts above 10 internal stops warrant especially close manual inspection of the coding sequence. Boundary behavior must be exact: the warning fires only for counts strictly greater than 5.

## Related tools

- **ReFernment** (R package that counts internal stops per CDS and implements the >5-stop manual-review error/warning) — github.com/TARobison/ReFernment
- **R** (Runtime environment in which ReFernment is installed (install_github) and executed) — github.com/TARobison/ReFernment

## Examples

```
library("ReFernment"); genomes <- c("Asplenium_pek", "Woodwardia_uni"); ReFernment(gffFolder, fastaFolderPath=NULL, gbFolderPath=NULL, outputFolder, genomes)
```

## Evaluation signals

- Warning fires for every gene with more than 5 internal stops and never at exactly 5 or fewer
- The [number] field in the emitted warning matches the actual counted internal stop total for that gene
- The [geneName] field matches the gene whose CDS was translated
- Genes correctly annotated (no assembly/annotation error) produce no false-positive warnings, consistent with ReFernment's no-false-positive finding on clean data
- Boundary tests on synthetic CDS constructs with exactly 5 and 6 internal stops show the threshold is strictly exclusive at 5

## Limitations

- Cases with more than 5 internal stops can still be genuine RNA editing; these are relatively rare, and counts above 10 internal stops especially suggest assembly or annotation error requiring manual review
- ReFernment is not a substitute for manually checking gene annotations, nor a fix for sloppy annotation; output quality depends on input annotation quality
- Known issue: ReFernment does not produce annotations for files created in Sequin, though files exported from Geneious still work
- A separate failure mode exists where translation errors occur when a CDS spans the genome origin, requiring the origin to be shifted

## Evidence

- [methods] Threshold and warning message: "The warning will appear if a given coding sequence has more than 5 internal stops."
- [methods] Warning text: "There are a high number of edited Stops ( [number] ) in [geneName] manually check to make sure frame is correct"
- [readme] Rationale for the guard: "potential assembly errors that would result in an entire gene appearing to be frameshifted"
- [readme] Above-10 guidance: "Especially if there are more than 10 internal stops the user should take a close look at this coding sequecne to check for assembly error or annotation errors."
- [abstract] Article-level finding: "if there are more than five detected internal stops in a gene, ReFernment will produce an error suggesting that the user manually check that gene."
