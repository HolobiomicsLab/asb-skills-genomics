---
name: simulated-long-read-transcriptome-generation
description: Use when benchmarking genome-annotation pipelines across species and
  real long-read RNA-seq data is unavailable — simulate PacBio reads from a reference
  coding GTF to obtain standardized transcriptome inputs.
when_to_use_negative:
- Real PacBio/Illumina hybrid sequencing data are already available for the organism
  — use real alignments (minimap2 + HISAT2 BAM merge) rather than simulated reads.
- The organism lacks an assembled genome and NCBI GTF annotation — the paper states
  the pipeline 'can be used for any organism that has an assembled genome and an NCBI
  available annotation in GTF format', so simulation from a reference annotation is
  not possible.
- The goal is biological discovery (e.g., novel transcript identification) rather
  than pipeline benchmarking — simulated reads will not reveal genuinely missing genes.
edam_topics:
- http://edamontology.org/topic_0102
- http://edamontology.org/topic_3308
tools:
- name: IsoSeqSim
  role: Simulates PacBio Iso-Seq long reads from the reference annotation to create
    transcriptome inputs for benchmarking
  repo: https://github.com/yunhaowang/IsoSeqSim
- name: StringTie
  role: Assembles the simulated PacBio reads into GTF transcriptome assemblies with
    settings -p 1 -j 2 -c 2 -v -a 4
- name: gffcompare
  role: Compares each pipeline's GTF (built from the simulated transcriptome) against
    the reference annotation to obtain tmap files and precision/recall/F1 statistics
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/gigascience/giac099
    title: 'annotate_my_genomes: an easy-to-use pipeline to improve genome annotation
      and uncover neglected genes by hybrid RNA sequencing'
  source_content_hash: sha256:0e06910ef9f2f173b0ba2bbbb7131f24da9526e25d3cdce740ef6285dded1eb5
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/yunhaowang/IsoSeqSim
  tools:
  - AGAT
  - AUGUSTUS
  - BRAKER1
  - BRAKER2
  - IsoSeqSim
  - StringTie
  - TSEBRA
  - gffcompare
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0102
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# Simulated long-read transcriptome generation

## Summary

Generates PacBio Iso-Seq-like long-read transcriptome inputs by simulating reads with IsoSeqSim from each reference genome's annotation and assembling them with StringTie, enabling fair F1-score benchmarking of annotate_my_genomes against BRAKER1/2, TSEBRA, and AUGUSTUS on controlled inputs.

## When to use

Use when benchmarking genome-annotation pipelines across species and real long-read RNA-seq data is unavailable — simulate PacBio reads from a reference coding GTF to obtain standardized transcriptome inputs.

## When NOT to use

- Real PacBio/Illumina hybrid sequencing data are already available for the organism — use real alignments (minimap2 + HISAT2 BAM merge) rather than simulated reads.
- The organism lacks an assembled genome and NCBI GTF annotation — the paper states the pipeline 'can be used for any organism that has an assembled genome and an NCBI available annotation in GTF format', so simulation from a reference annotation is not possible.
- The goal is biological discovery (e.g., novel transcript identification) rather than pipeline benchmarking — simulated reads will not reveal genuinely missing genes.

## Inputs

- Reference genome assembly FASTA for each species (e.g., galGal6 GCA_000002315.5, hg38, mm10, danRer11, ce11)
- Matched reference coding GTF annotation (used to seed IsoSeqSim simulation)
- IsoSeqSim configuration parameters for PacBio Iso-Seq read simulation

## Outputs

- Simulated PacBio long-read sequencing files per genome
- StringTie transcriptome assemblies (GTF) built from the simulated reads
- gffcompare accuracy statistics (precision, recall, F1) per genome/pipeline compiled into a summary table matching Supplementary Table 2

## How to apply

For each benchmarked genome (galGal6, hg38, mm10, danRer11, ce11), use IsoSeqSim to simulate PacBio Iso-Seq reads derived from the genome's reference coding GTF annotation. Assemble the simulated long reads with StringTie using the same relaxed settings applied elsewhere in the study (-p 1 -j 2 -c 2 -v -a 4) to obtain a simulated transcriptome assembly in GTF format for every genome. Feed each simulated StringTie GTF as input to annotate_my_genomes (annotate-my-genomes.nf / add-ncbi-annotation.nf) and to the comparator pipelines (BRAKER1, BRAKER2, TSEBRA, AUGUSTUS), then run gffcompare against the reference GTF to extract precision, recall, and harmonic-mean F1 at base, exon, intron, intron-chain, transcript, and locus levels. The rationale is that a controlled simulated transcriptome isolates pipeline performance from sequencing-platform variability, and the compiled F1 table must reproduce Supplementary Table 2 with annotate_my_genomes performing equal or better on StringTie de-novo assemblies.

## Related tools

- **IsoSeqSim** (Simulates PacBio Iso-Seq long reads from the reference annotation to create transcriptome inputs for benchmarking) — https://github.com/yunhaowang/IsoSeqSim
- **StringTie** (Assembles the simulated PacBio reads into GTF transcriptome assemblies with settings -p 1 -j 2 -c 2 -v -a 4)
- **gffcompare** (Compares each pipeline's GTF (built from the simulated transcriptome) against the reference annotation to obtain tmap files and precision/recall/F1 statistics)

## Examples

```
IsoSeqSim -g galGal6_genome.fa -a galGal6_ncbiRefSeq.gtf -o pacbio_sim ; stringtie -p 1 -j 2 -c 2 -v -a 4 pacbio_sim.bam -o simulated_assembly.gtf
```

## Evaluation signals

- gffcompare stats parse cleanly into precision, recall, and harmonic-mean F1 at base, exon, intron, intron-chain, transcript, and locus levels for every genome × pipeline combination
- The compiled F1 table reproduces Supplementary Table 2 of the article
- annotate_my_genomes achieves equal or better F1 than BRAKER1, BRAKER2, TSEBRA, and AUGUSTUS under the StringTie de-novo (simulated) condition
- A .tmap transcript map is produced for each pipeline's GTF against the corresponding reference annotation
- Identical StringTie parameters (-p 1 -j 2 -c 2 -v -a 4) are applied across all simulated conditions so comparisons are not confounded by assembly settings

## Limitations

- Precision/recall/F1 values are not reported in the article main text — they are deferred entirely to Supplementary Table 2, so direct numeric verification requires the supplementary file.
- Only PacBio reads were simulated via IsoSeqSim; the simulated condition does not cover the hybrid PacBio+Illumina condition used elsewhere in the study.
- The approach presumes an organism with an assembled genome and an existing NCBI GTF annotation to seed simulation; organisms lacking these cannot be simulated this way.

## Evidence

- [discussion] IsoSeqSim used for PacBio read simulation: "We simulated PacBio reads using IsoSeqSim tool (https://github.com/yunhaowang/IsoSeqSim)"
- [readme] IsoSeqSim purpose per README: [evidence span withheld — anchor status: external_doc]
- [discussion] StringTie assembly parameters used to build transcriptome inputs: "using StringTie [57, 58] program with settings: -p 1 -j 2 -c 2 -v -a 4"
- [discussion] Pipeline scope constraint on reference annotation availability: "This pipeline can be used for any organism that has an assembled genome and an NCBI available annotation in GTF format"
