---
name: nanopore-drna-quantification-output-parsing
description: Use when you have LR-Kallisto long-read quantification output from Dogme-processed
  dRNA replicates and need to determine how many detected transcripts are known isoforms
  versus novel (ISM, NIC, NNC), and whether classification is reproducible across
  replicates.
when_to_use_negative:
- Input is short-read RNA-seq data without full-length transcript paths — splice-junction-chain
  classification against the catalog is not meaningful
- The goal is modification detection (m6A, m5C, inosine, pseudouridine, Nm) from modkit
  bedMethyl outputs rather than isoform identity classification
- Input is single-cell cDNA kb/bustools quantification rather than bulk full-length
  dRNA transcript quantification
edam_operation: http://edamontology.org/operation_2990
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: Dogme
  role: Nextflow pipeline whose kallisto and annotateRNA entry points produce the
    quantification and TALON-compatible transcript annotations parsed and classified
    by this skill
  repo: https://github.com/mortazavilab/dogme
- name: LR-Kallisto
  role: Quantifies full-length transcript isoforms for dRNA and cDNA; its per-replicate
    output is the input to isoform classification
- name: minimap2
  role: Performs spliced alignment (GTF converted to junction BED, --splice-max 500000)
    so read splice-junction chains can be compared to the reference annotation
- name: MinION
  role: Instrument used to generate the three direct RNA C2C12 myoblast replicates
    whose outputs are classified
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/bioinformatics/btag066
    title: 'Dogme: a nextflow pipeline for reprocessing nanopore RNA and DNA modifications'
  source_content_hash: sha256:a468d1987e438346b5c8037f0816a30529edb0cfc974528d751889e3cc866ee7
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/mortazavilab/dogme
  tools:
  - Dogme
  - LR-Kallisto
  - MinION
  - minimap2
  edam_operation: http://edamontology.org/operation_2990
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.42857142857142855
n_claims: 14
---

# nanopore-drna-quantification-output-parsing

## Summary

Parses Dogme's LR-Kallisto transcript quantification outputs from mouse C2C12 direct RNA (dRNA) MinION replicates and classifies each detected full-length transcript as known, ISM, NIC, or NNC, aggregating reproducible per-replicate isoform class counts.

## When to use

Use when you have LR-Kallisto long-read quantification output from Dogme-processed dRNA replicates and need to determine how many detected transcripts are known isoforms versus novel (ISM, NIC, NNC), and whether classification is reproducible across replicates.

## When NOT to use

- Input is short-read RNA-seq data without full-length transcript paths — splice-junction-chain classification against the catalog is not meaningful
- The goal is modification detection (m6A, m5C, inosine, pseudouridine, Nm) from modkit bedMethyl outputs rather than isoform identity classification
- Input is single-cell cDNA kb/bustools quantification rather than bulk full-length dRNA transcript quantification

## Inputs

- LR-Kallisto transcript quantification output per dRNA replicate (from Dogme RNA workflow)
- Mapped BAM files from minimap2 spliced alignment of Dorado-basecalled reads
- Reference transcript annotation GTF (used to auto-generate junction BED via scripts/gtf_to_junction_bed.py)
- Reference genome and GTF entries in genome_annot_refs config

## Outputs

- transcript_class_counts.csv with transcript counts per class (known, ISM, NIC, NNC) and per replicate
- TALON-compatible transcript annotation outputs from annotateRNA.py
- QC summary CSVs per genome from the annotateRNA workflow

## How to apply

Load the LR-Kallisto transcript quantification output for each of the three separate mouse C2C12 myoblast dRNA replicates sequenced on MinION flow cells. Match each detected full-length transcript against the reference transcript annotation (GTF) to identify transcripts that correspond to known isoforms; Dogme's annotateRNA.py annotates mapped BAMs with transcript information and produces TALON-compatible outputs, which is the basis for this splice-junction-chain comparison. For transcripts not matching a known isoform, classify their splice junction chains as Incomplete Splice Match (ISM), Novel in Catalog (NIC), or Novel Not in Catalog (NNC) based on splice-site matching rules. Aggregate the results into a transcript_class_counts.csv table of transcript counts per class (known, ISM, NIC, NNC) and per replicate. The rationale is that dRNA reads are full-length, so splice-junction chains can be compared directly against the annotation using minimap2 spliced alignment (with GTF-to-junction-BED conversion and --splice-max 500000) rather than inferred from coverage.

## Related tools

- **Dogme** (Nextflow pipeline whose kallisto and annotateRNA entry points produce the quantification and TALON-compatible transcript annotations parsed and classified by this skill) — https://github.com/mortazavilab/dogme
- **LR-Kallisto** (Quantifies full-length transcript isoforms for dRNA and cDNA; its per-replicate output is the input to isoform classification)
- **minimap2** (Performs spliced alignment (GTF converted to junction BED, --splice-max 500000) so read splice-junction chains can be compared to the reference annotation)
- **MinION** (Instrument used to generate the three direct RNA C2C12 myoblast replicates whose outputs are classified)

## Examples

```
nextflow run mortazavilab/dogme -entry annotateRNA --sample C2C12_rep1
```

## Evaluation signals

- transcript_class_counts.csv exists and contains exactly the four class labels (known, ISM, NIC, NNC) and all three replicates
- Per-replicate class counts sum to the total number of detected transcripts in that replicate
- Known and ISM classes dominate detected transcripts, consistent with the article's finding that the majority were known isoforms and ISM
- Class assignments are highly concordant across the three biological replicates (same transcripts recur with the same labels)
- Annotated BAMs and QC summary CSVs are produced for each genome by annotateRNA.py without errors

## Limitations

- Classification depends on the quality of the supplied GTF reference and kallisto index; when kallistoIndex and t2g are null, the pipeline auto-builds indexes from the genome/GTF in genome_annot_refs, so an incomplete GTF shifts transcripts between known and NNC/NIC
- ISM/NIC/NNC boundary calls follow splice-site matching rules, so partial-length or degraded reads can be misclassified as ISM
- The article does not record specific GTF/reference versions or commit hashes, so exact classification counts may vary with reference build

## Evidence

- [abstract] while concurrently quantifying full-length transcript isoforms LR-Kallisto for transcript quantification for dRNA and cDNA: "while concurrently quantifying full-length transcript isoforms LR-Kallisto for transcript quantification for dRNA and cDNA"
- [abstract] three replicates: "three separate mouse C2C12 myoblast replicates using direct RNA sequencing on MinION flow cells"
- [readme] annotateRNA description: [evidence span withheld — anchor status: external_doc]
- [readme] junction bed conversion: [evidence span withheld — anchor status: external_doc]
- [readme] entry points: [evidence span withheld — anchor status: external_doc]
