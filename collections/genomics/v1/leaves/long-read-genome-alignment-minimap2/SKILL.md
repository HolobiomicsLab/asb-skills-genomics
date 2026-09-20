---
name: long-read-genome-alignment-minimap2
description: Use when PacBio long-read (Iso-Seq style) RNA-seq reads must be mapped
  to a reference genome as the long-read half of a hybrid assembly, prior to merging
  with short-read alignments.
when_to_use_negative:
- Input reads are short Illumina reads — the article aligns those with HISAT2 after
  fastp trimming, not minimap2
- No assembled reference genome exists — the method requires a reference genome as
  a guide
- Goal is de novo transcriptome assembly without a reference; StringTie here is run
  in genome-guided mode on aligned BAMs
edam_operation: http://edamontology.org/operation_3198
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3170
tools:
- name: minimap2
  role: Aligns PacBio long reads against the galGal6 genome at ~34x depth to generate
    the long-read BAM for hybrid merging
- name: Nextflow
  role: Workflow engine implementing the annotate_my_genomes pipeline in which this
    alignment step is embedded
  repo: https://github.com/cfarkas/annotate_my_genomes
- name: StringTie
  role: Consumes the merged PacBio+Illumina BAM downstream to assemble transcripts
    (GTF)
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/gigascience/giac099
    title: 'annotate_my_genomes: an easy-to-use pipeline to improve genome annotation
      and uncover neglected genes by hybrid RNA sequencing'
  source_content_hash: sha256:8e4606b95b8a79615700e6e54bee42dc8798b227f2975aa54a77d7b4e5817d7b
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/cfarkas/annotate_my_genomes
  tools:
  - LongRanger
  - Minimap2
  - Nextflow
  - StringTie
  - fastp
  - minimap2
  - stlfr2supernova
  edam_operation: http://edamontology.org/operation_3198
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
merged_aliases:
- minimap2-whole-genome-alignment
triage_status: promoted
yes_rate: 0.8
n_claims: 5
---

# long-read-genome-alignment-minimap2

## Summary

Align PacBio long-read RNA-seq reads against a reference genome using minimap2 to produce a coordinate-sorted BAM suitable for downstream hybrid transcriptome assembly. In the chicken SCO workflow, minimap2 alignment of PacBio reads achieved ~34x read depth against galGal6.

## When to use

Use when PacBio long-read (Iso-Seq style) RNA-seq reads must be mapped to a reference genome as the long-read half of a hybrid assembly, prior to merging with short-read alignments.

## When NOT to use

- Input reads are short Illumina reads — the article aligns those with HISAT2 after fastp trimming, not minimap2
- No assembled reference genome exists — the method requires a reference genome as a guide
- Goal is de novo transcriptome assembly without a reference; StringTie here is run in genome-guided mode on aligned BAMs

## Inputs

- PacBio long-read RNA-seq reads (ENA accession PRJEB36569)
- Unmasked reference genome assembly in FASTA format (galGal6, GCA_000002315.5)

## Outputs

- minimap2 alignment BAM of PacBio reads against the reference genome, coordinate-sorted and ready for merging with short-read BAM

## How to apply

Download the PacBio long-read dataset (here ENA accession PRJEB36569) and the unmasked reference genome (galGal6, GCA_000002315.5). Align the PacBio reads against the genome with the minimap2 aligner, targeting the reported ~34x read depth, and produce a BAM file. Coordinate-sort the BAM so it can be merged with the HISAT2-aligned, fastp-trimmed Illumina BAM (ENA PRJEB36584) into a single sorted alignment file. The merged BAM then feeds into StringTie assembly (settings -p 1 -j 2 -c 2 -v -a 4) as the genome-guided input for annotate_my_genomes; minimap2 is preferred over short-read aligners because it handles the length and error profile of PacBio reads.

## Related tools

- **minimap2** (Aligns PacBio long reads against the galGal6 genome at ~34x depth to generate the long-read BAM for hybrid merging)
- **Nextflow** (Workflow engine implementing the annotate_my_genomes pipeline in which this alignment step is embedded) — https://github.com/cfarkas/annotate_my_genomes
- **StringTie** (Consumes the merged PacBio+Illumina BAM downstream to assemble transcripts (GTF))

## Examples

```
minimap2 -ax splice -t 1 galGal6.fa pacbio_reads.fastq | samtools sort -o pacbio_vs_galGal6.sorted.bam
```

## Evaluation signals

- Alignment depth reaches approximately the reported ~34x PacBio read depth against the reference genome
- Output BAM is coordinate-sorted and valid (e.g., samtools quickcheck passes) so it can be merged with the HISAT2 Illumina BAM
- Merged PacBio+Illumina BAM supports a StringTie assembly that parses as valid GTF with transcript records for use as the -a input of annotate-my-genomes
- Downstream BUSCO-assessed quality of the merged-assembly annotation exceeds both Illumina-alone and PacBio-alone assemblies, validating the hybrid strategy

## Limitations

- The pipeline requires an assembled reference genome (and an NCBI GTF annotation for the full annotate_my_genomes flow); no results are reported for organisms lacking these
- PacBio-only alignment/assembly yielded higher Ex90N50 in IsoSeq comparisons, so long-read alignment alone is not sufficient for the best-performing annotation — the hybrid merge was selected on BUSCO quality
- Coverage is bounded by input depth; the reported analysis targeted ~34x PacBio read depth against galGal6

## Evidence

- [discussion] We aligned PacBio reads against Gallus gallus genome ... using minimap2 aligner [30] obtaining depths of ∼ 34x: [evidence span withheld — anchor status: not_found]
- [discussion] We merged the resulting BAM files from PacBio and Illumina read alignments into a single BAM file and we assembled transcripts from the latter alignment file using StringTie: "We merged the resulting BAM files from PacBio and Illumina read alignments into a single BAM file and we assembled transcripts from the latter alignment file using StringTie"
- [supplementary] available at European Nucleotide Archive (ENA) Accession Number PRJEB36569 (PacBio): "available at European Nucleotide Archive (ENA) Accession Number PRJEB36569 (PacBio)"
- [intro] This method requires a reference genome as a guide and leads to superior transcriptome assembly and annotation when compared to traditional Illumina or PacBio RNA-seq protocols such as IsoSeq: "This method requires a reference genome as a guide and leads to superior transcriptome assembly and annotation when compared to traditional Illumina or PacBio RNA-seq protocols such as IsoSeq"
