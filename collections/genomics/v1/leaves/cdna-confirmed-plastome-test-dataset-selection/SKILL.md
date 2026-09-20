---
name: cdna-confirmed-plastome-test-dataset-selection
description: Use when validating or benchmarking an RNA editing annotation method
  and you need plastid genomes whose editing sites are independently confirmed by
  cDNA data rather than predicted.
when_to_use_negative:
- The tool's purpose is to predict all RNA editing sites from RNA-Seq data — ReFernment
  is explicitly not intended for that, so this dataset cannot validate genome-wide
  editing site prediction
- Input annotation files were created in Sequin — ReFernment has a known error and
  does not produce annotations for Sequin-created files (files exported from Geneious
  still work)
- You need a substitute for manually checking gene annotations — ReFernment is not
  a fix for sloppy annotation and output quality depends on input annotation quality
edam_operation: http://edamontology.org/operation_0362
edam_topics:
- http://edamontology.org/topic_0085
- http://edamontology.org/topic_0622
tools:
- name: ReFernment
  role: R package under test; annotates apparent nonsense mutations explainable by
    RNA editing in the cDNA-confirmed test plastomes
  repo: https://github.com/TARobison/ReFernment
- name: devtools
  role: Used to install the package from GitHub via install_github("TARobison/ReFernment")
  repo: https://github.com/TARobison/ReFernment
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1002/aps3.1216
    title: 'ReFernment: An R package for annotating <scp>RNA</scp> editing in plastid
      genomes'
  source_content_hash: sha256:c37d05e22246dc585e99a13241ea8911617cbc3fd962af44cd0fd68df5127048
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/TARobison/ReFernment
  tools:
  - R
  - ReFernment
  - devtools
  edam_operation: http://edamontology.org/operation_0362
  edam_topics:
  - http://edamontology.org/topic_0085
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 12
---

# cdna-confirmed-plastome-test-dataset-selection

## Summary

Selects plastid genomes with cDNA-confirmed RNA editing sites (e.g., GenBank AB086179 and AY178864.1) as ground-truth test datasets for benchmarking automated RNA editing annotation tools such as ReFernment against manual curation.

## When to use

Use when validating or benchmarking an RNA editing annotation method and you need plastid genomes whose editing sites are independently confirmed by cDNA data rather than predicted.

## When NOT to use

- The tool's purpose is to predict all RNA editing sites from RNA-Seq data — ReFernment is explicitly not intended for that, so this dataset cannot validate genome-wide editing site prediction
- Input annotation files were created in Sequin — ReFernment has a known error and does not produce annotations for Sequin-created files (files exported from Geneious still work)
- You need a substitute for manually checking gene annotations — ReFernment is not a fix for sloppy annotation and output quality depends on input annotation quality

## Inputs

- cDNA-confirmed plastid genome records (GenBank AB086179, AY178864.1)
- GFF3 annotation file with sequence embedded for each test plastome
- Optional GenBank flat file and separate FASTA file
- Genome name vector (without file extensions)

## Outputs

- Correctness assessment: list of annotated nonsense mutations vs. cDNA-confirmed RNA editing sites (false positives / false negatives)
- Wall-clock runtime measurement (elapsed seconds) for the complete annotation run
- ReFernment outputs: five-column GenBank feature table and protein FASTA file with conceptual translations

## How to apply

Choose plastomes whose RNA editing sites have been validated with cDNA data — the ReFernment paper uses GenBank accessions AB086179 and AY178864.1, which are high-RNA-editing fern/hornwort plastomes. Run the annotation tool (e.g., ReFernment installed via devtools from github.com/TARobison/ReFernment) on the GFF3 input with embedded sequence, declaring genome names without file extensions since the package adds extensions automatically. Compare tool output against the cDNA-confirmed editing sites: in the paper's validation, every nonsense mutation was correctly annotated with no false positives. For runtime benchmarking, wrap the ReFernment(gffFolder, fastaFolderPath=NULL, gbFolderPath=NULL, outputFolder, genomes) call in system.time and require the full annotation to complete in under 60 seconds, versus hours for manual annotation.

## Related tools

- **ReFernment** (R package under test; annotates apparent nonsense mutations explainable by RNA editing in the cDNA-confirmed test plastomes) — https://github.com/TARobison/ReFernment
- **devtools** (Used to install the package from GitHub via install_github("TARobison/ReFernment")) — https://github.com/TARobison/ReFernment

## Examples

```
genomes <- c("AB086179"); system.time(ReFernment(gffFolder, fastaFolderPath=NULL, gbFolderPath=NULL, outputFolder, genomes))
```

## Evaluation signals

- Every cDNA-confirmed nonsense mutation in AB086179 and AY178864.1 is annotated by the tool (zero false negatives)
- Zero false-positive RNA editing annotations for data with no sequencing or assembly errors
- Measured wall-clock time via system.time is less than 60 seconds for a single typical fern or hornwort plastome
- Output includes a five-column GenBank feature table and a protein FASTA file with conceptual translations
- No error triggered by the >5 internal stops warning for any gene in the clean test genomes

## Limitations

- ReFernment is not a substitute for manually checking gene annotations, nor a fix for sloppy annotation; output quality depends on the input annotation
- Known issue: ReFernment does not produce annotations for files created in Sequin; files exported from Geneious work
- If a coding sequence has more than 5 internal stops (and especially >10), the user must manually check for assembly or annotation errors
- Sequences with a CDS spanning the genome origin can fail translation; shifting the origin is recommended
- No software version number is stated for ReFernment, so provenance must be recorded via the GitHub install path

## Evidence

- [abstract] confirmed with cDNA data (AB086179 and AY178864.1): "confirmed with cDNA data (AB086179 and AY178864.1)"
- [abstract] every nonsense mutation was correctly annotated, and there were no false positive annotations: "every nonsense mutation was correctly annotated, and there were no false positive annotations"
- [abstract] Manually accounting for RNA edits generally takes hours for a typical fern or hornwort plastid genome, but with ReFernment, this process takes less than a minute.: "Manually accounting for RNA edits generally takes hours for a typical fern or hornwort plastid genome, but with ReFernment, this process takes less than a minute."
- [readme] ReFernment requires only a GFF3 file (with sequence embedded) to work: "ReFernment *requires* only a GFF3 file (with sequence embedded) to work, but you can optionally provide a GenBank flat file and seperate fasta file, if your workflow requires this."
- [readme] The warning will appear if a given coding sequence has more than 5 internal stops.: "The warning will appear if a given coding sequence has more than 5 internal stops."
- [intro] there is a known error where ReFernment is not producing annotations for files created in sequin: "there is a known error where ReFernment is not producing annotations for files created in sequin. I will fix the issue as soon as possible"
