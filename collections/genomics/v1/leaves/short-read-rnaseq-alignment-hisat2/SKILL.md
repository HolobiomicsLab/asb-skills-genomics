---
name: short-read-rnaseq-alignment-hisat2
description: Use when short-read Illumina RNA-seq data (e.g. ENA PRJEB36584) must
  be aligned to a reference genome as part of a hybrid (PacBio + Illumina) genome-guided
  transcriptome assembly.
when_to_use_negative:
- No assembled reference genome is available — the pipeline is genome-guided and requires
  a reference genome as a guide
- Only PacBio long-read data is available — use a PacBio-only pipeline such as IsoSeq/IsoCon
  instead of hybrid alignment
- Input alignment BAMs already exist and only transcript assembly is needed — proceed
  directly to StringTie
edam_operation: http://edamontology.org/operation_0292
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0622
tools:
- name: fastp
  role: Automatically trims Illumina reads before alignment
- name: HISAT2
  role: Aligns trimmed Illumina short reads against the galGal6 genome to generate
    a splice-aware BAM
- name: minimap2
  role: Aligns PacBio long reads (~34x depth) against galGal6; its BAM is merged with
    the HISAT2 BAM
- name: StringTie
  role: Assembles transcripts from the merged BAM with settings -p 1 -j 2 -c 2 -v
    -a 4
- name: Nextflow
  role: Workflow engine implementing the annotate-my-genomes pipeline steps
  repo: https://github.com/cfarkas/annotate_my_genomes
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/gigascience/giac099
    title: 'annotate_my_genomes: an easy-to-use pipeline to improve genome annotation
      and uncover neglected genes by hybrid RNA sequencing'
  source_content_hash: sha256:931de9c5a681f9d45f9e6059d3e72083231aa39fbaaa050da65cc01e10063a29
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/cfarkas/annotate_my_genomes
  tools:
  - GMAP
  - HISAT2
  - Nextflow
  - StringTie
  - TAGET
  - fastp
  - minimap2
  - samtools
  edam_operation: http://edamontology.org/operation_0292
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
merged_aliases:
- short-read-rnaseq-alignment-qc
triage_status: promoted
yes_rate: 1.0
n_claims: 10
---

# short-read-rnaseq-alignment-hisat2

## Summary

Trim paired-end Illumina RNA-seq reads with fastp and align them to a reference genome (galGal6, GCA_000002315.5) using HISAT2, producing a BAM file that is merged with a minimap2-aligned PacBio BAM for downstream StringTie transcript assembly.

## When to use

Use when short-read Illumina RNA-seq data (e.g. ENA PRJEB36584) must be aligned to a reference genome as part of a hybrid (PacBio + Illumina) genome-guided transcriptome assembly.

## When NOT to use

- No assembled reference genome is available — the pipeline is genome-guided and requires a reference genome as a guide
- Only PacBio long-read data is available — use a PacBio-only pipeline such as IsoSeq/IsoCon instead of hybrid alignment
- Input alignment BAMs already exist and only transcript assembly is needed — proceed directly to StringTie

## Inputs

- Paired-end Illumina RNA-seq reads (FASTQ) from ENA accession PRJEB36584
- Unmasked reference genome assembly galGal6 (GCA_000002315.5)
- minimap2-aligned PacBio BAM (from ENA PRJEB36569, ~34x depth) for the merge step

## Outputs

- Coordinate-sorted HISAT2 Illumina alignment BAM
- Merged PacBio+Illumina coordinate-sorted BAM
- StringTie transcript assembly in GTF format (stringtie.gtf)

## How to apply

Download the Illumina paired-end reads from ENA accession PRJEB36584 and the unmasked reference genome (galGal6, GCA_000002315.5). Automatically trim the reads with fastp, then align the trimmed reads against galGal6 with HISAT2 to produce a coordinate-sorted BAM. This BAM is subsequently merged with the minimap2-aligned PacBio BAM (targeting ~34x read depth from ENA PRJEB36569) into a single coordinate-sorted alignment file. The merged BAM is assembled into transcripts by StringTie with the settings -p 1 -j 2 -c 2 -v -a 4 (threads 1, min junction coverage 2, min transcript coverage 2, verbose, min anchor length 4), yielding stringtie.gtf for annotation by annotate-my-genomes. The rationale is that Illumina short reads add splice-junction accuracy that complements PacBio long reads, and the merged assembly was selected because its BUSCO-assessed quality exceeded either single-technology assembly.

## Related tools

- **fastp** (Automatically trims Illumina reads before alignment)
- **HISAT2** (Aligns trimmed Illumina short reads against the galGal6 genome to generate a splice-aware BAM)
- **minimap2** (Aligns PacBio long reads (~34x depth) against galGal6; its BAM is merged with the HISAT2 BAM)
- **StringTie** (Assembles transcripts from the merged BAM with settings -p 1 -j 2 -c 2 -v -a 4)
- **Nextflow** (Workflow engine implementing the annotate-my-genomes pipeline steps) — https://github.com/cfarkas/annotate_my_genomes

## Examples

```
fastp -i R1.fq.gz -I R2.fq.gz -o trimmed_R1.fq.gz -O trimmed_R2.fq.gz; hisat2 -x galGal6 -1 trimmed_R1.fq.gz -2 trimmed_R2.fq.gz | samtools sort -o illumina_hisat2.bam
```

## Evaluation signals

- stringtie.gtf exists and parses as a valid GTF containing StringTie-assembled transcript records suitable as the -a input of annotate-my-genomes
- The PacBio minimap2 alignment reaches the reported ~34x read depth against galGal6
- The merged BAM is coordinate-sorted and contains reads from both the HISAT2 Illumina and minimap2 PacBio alignments
- BUSCO quality of the merged PacBio-Illumina assembly exceeds the Illumina-alone and PacBio-alone assemblies
- StringTie settings match -p 1 -j 2 -c 2 -v -a 4 (threads 1, min junction coverage 2, min transcript coverage 2, verbose, min anchor length 4)

## Limitations

- The downstream pipeline can only be used for organisms that have an assembled genome and an NCBI available annotation in GTF format
- StringTie GTF assembly quality alone (e.g. higher Ex90N50 for IsoSeq) does not guarantee selection — the merged assembly was chosen for exceeding BUSCO-assessed quality of single-technology assemblies

## Evidence

- [discussion] Illumina reads were automatically trimmed using fastp tool [31] and aligned against the referred Gallus gallus genome using HISAT2 aligner: "Illumina reads were automatically trimmed using fastp tool [31] and aligned against the referred Gallus gallus genome using HISAT2 aligner"
- [discussion] using StringTie [57, 58] program with settings: -p 1 -j 2 -c 2 -v -a 4: "using StringTie [57, 58] program with settings: -p 1 -j 2 -c 2 -v -a 4"
- [supplementary] Trim the Illumina reads with fastp and align the trimmed reads to galGal6 using HISAT2: [evidence span withheld — anchor status: not_found]
- [discussion] We aligned PacBio reads against Gallus gallus genome ... using minimap2 aligner [30] obtaining depths of ∼ 34x: [evidence span withheld — anchor status: not_found]
