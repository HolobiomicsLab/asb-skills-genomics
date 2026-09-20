---
name: genome-guided-transcriptome-assembly-stringtie
description: Use when you have long-read (PacBio IsoSeq) and/or short-read (Illumina)
  RNA-seq BAM alignments against a reference genome and need a genome-guided transcript
  assembly in GTF format to feed an annotation pipeline or to compare assembly strategies.
when_to_use_negative:
- The organism of interest lacks an assembled genome and an NCBI GTF annotation, since
  the downstream pipeline is restricted to such organisms
- You only need a de novo transcriptome without a reference genome guide, as this
  skill is explicitly genome-guided
- Your input is already an assembled transcriptome GTF from StringTie — proceed directly
  to annotation (annotate-my-genomes) rather than re-assembling
edam_operation: http://edamontology.org/operation_0310
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: minimap2
  role: Aligns PacBio long reads against the galGal6 genome (~34x depth) to produce
    the long-read BAM
- name: fastp
  role: Automatically trims Illumina reads before HISAT2 alignment
- name: HISAT2
  role: Aligns trimmed Illumina reads against galGal6 to produce the short-read BAM
- name: StringTie
  role: Assembles transcripts from the merged BAM using -p 1 -j 2 -c 2 -v -a 4 to
    output stringtie.gtf
- name: Nextflow
  role: Orchestrates the genome-download and assembly/annotation workflow steps
  repo: https://github.com/cfarkas/annotate_my_genomes
- name: Salmon
  role: Quantifies transcript abundance per assembly for Salmon-based Ex90N50 computation
- name: BUSCO
  role: Transcriptome-mode completeness assessment against the aves_odb10 lineage
    to choose the best assembly
provenance:
  source_task_ids:
  - task_001
  - task_002
  source_papers:
  - doi: 10.1093/gigascience/giac099
    title: 'annotate_my_genomes: an easy-to-use pipeline to improve genome annotation
      and uncover neglected genes by hybrid RNA sequencing'
  source_content_hash: sha256:d2373958bf54f37d3f27ee6e3d99dda30ff7d39cadc1ff393dc9b2670fc954c9
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/cfarkas/annotate_my_genomes
  tools:
  - BUSCO
  - HISAT2
  - IsoSeq
  - Nextflow
  - SQANTI
  - Salmon
  - StringTie
  - fastp
  - minimap2
  - rnaQUAST
  edam_operation: http://edamontology.org/operation_0310
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 12
---

# genome-guided-transcriptome-assembly-stringtie

## Summary

Assembles a genome-guided transcriptome from hybrid PacBio + Illumina RNA-seq alignments by merging coordinate-sorted BAM files against the galGal6 (GCA_000002315.5) reference and running StringTie with fixed parameters (-p 1 -j 2 -c 2 -v -a 4), producing a GTF used for downstream annotation with annotate_my_genomes.

## When to use

Use when you have long-read (PacBio IsoSeq) and/or short-read (Illumina) RNA-seq BAM alignments against a reference genome and need a genome-guided transcript assembly in GTF format to feed an annotation pipeline or to compare assembly strategies.

## When NOT to use

- The organism of interest lacks an assembled genome and an NCBI GTF annotation, since the downstream pipeline is restricted to such organisms
- You only need a de novo transcriptome without a reference genome guide, as this skill is explicitly genome-guided
- Your input is already an assembled transcriptome GTF from StringTie — proceed directly to annotation (annotate-my-genomes) rather than re-assembling

## Inputs

- PacBio long-read RNA-seq dataset (ENA PRJEB36569)
- Illumina RNA-seq dataset (ENA PRJEB36584)
- Unmasked reference genome assembly galGal6 (GCA_000002315.5)
- minimap2-aligned PacBio BAM (~34x depth) and HISAT2-aligned, fastp-trimmed Illumina BAM, merged into one coordinate-sorted alignment file

## Outputs

- StringTie transcript assembly in GTF format (stringtie.gtf), suitable as the -a input of annotate-my-genomes.nf
- Optional per-strategy assemblies (IsoSeq PacBio-only, merged, Illumina-only) with Ex90N50 and completed-BUSCO (aves_odb10) metrics tabulated in ex90n50_busco_metrics.csv

## How to apply

Download the PacBio long-read dataset (ENA PRJEB36569) and Illumina dataset (ENA PRJEB36584) plus the unmasked galGal6 (GCA_000002315.5) genome. Align PacBio reads against galGal6 with minimap2 (targeting the reported ~34x read depth), trim Illumina reads with fastp and align them with HISAT2, then merge the two coordinate-sorted BAM files into a single alignment file. Assemble transcripts from the merged BAM with StringTie using -p 1 -j 2 -c 2 -v -a 4 (threads 1, min junction coverage 2, min transcript coverage 2, verbose, min anchor length 4), saving the result as stringtie.gtf. To evaluate assembly strategy, build PacBio-only, merged, and Illumina-only assemblies in parallel and compare them with Salmon-based Ex90N50 statistics and BUSCO transcriptome mode against the aves_odb10 lineage; rationale from the paper is that the merged alignment assembly exceeded both single-technology assemblies in completed BUSCOs (~1% over Illumina-alone) despite IsoSeq having superior Ex90N50.

## Related tools

- **minimap2** (Aligns PacBio long reads against the galGal6 genome (~34x depth) to produce the long-read BAM)
- **fastp** (Automatically trims Illumina reads before HISAT2 alignment)
- **HISAT2** (Aligns trimmed Illumina reads against galGal6 to produce the short-read BAM)
- **StringTie** (Assembles transcripts from the merged BAM using -p 1 -j 2 -c 2 -v -a 4 to output stringtie.gtf)
- **Nextflow** (Orchestrates the genome-download and assembly/annotation workflow steps) — https://github.com/cfarkas/annotate_my_genomes
- **Salmon** (Quantifies transcript abundance per assembly for Salmon-based Ex90N50 computation)
- **BUSCO** (Transcriptome-mode completeness assessment against the aves_odb10 lineage to choose the best assembly)

## Examples

```
stringtie merged.bam -p 1 -j 2 -c 2 -v -a 4 -o stringtie.gtf
```

## Evaluation signals

- stringtie.gtf exists and parses as a valid GTF containing StringTie-assembled transcript records suitable as the -a input of annotate-my-genomes.nf
- minimap2 PacBio alignment reaches the reported ~34x read depth against galGal6
- In the assembly comparison, the merged PacBio+Illumina assembly shows a completed-BUSCO (aves_odb10) count approximately 1% higher than the Illumina-only assembly, matching Figure 1B–1C
- Ex90N50 values computed via the Salmon-based ExN50 script are tabulated per assembly (IsoSeq, merged, Illumina-only) in ex90n50_busco_metrics.csv
- StringTie was invoked with exactly -p 1 -j 2 -c 2 -v -a 4 as reported in the paper's workflow

## Limitations

- IsoSeq PacBio-only assembly can have superior Ex90N50 statistics yet lower BUSCO completeness, so Ex90N50 alone is not a sufficient assembly-selection criterion
- SSPO isoform protein products (89-exon and 25-exon isoforms) remain unvalidated despite being assembled
- Benchmark F1-scores and precision/recall values are only available in Supplementary Table 2, not in the main text

## Evidence

- [discussion] StringTie assembly parameters: "using StringTie [57, 58] program with settings: -p 1 -j 2 -c 2 -v -a 4"
- [discussion] Merged BAM assembly: "We merged the resulting BAM files from PacBio and Illumina read alignments into a single BAM file and we assembled transcripts from the latter alignment file using StringTie"
- [discussion] minimap2 alignment depth: [evidence span withheld — anchor status: not_found]
- [supplementary] BUSCO-based selection: "available at European Nucleotide Archive (ENA) Accession Number PRJEB36569 (PacBio)"
- [readme] Pipeline purpose: "annotate_my_genomes is a pipeline that aims to annotate genome-guided transcriptome assemblies from StringTie, coming from long read RNA-Seq alignments in vertebrate genomes"
