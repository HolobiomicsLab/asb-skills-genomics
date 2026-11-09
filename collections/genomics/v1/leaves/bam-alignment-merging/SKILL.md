---
name: bam-alignment-merging
description: Use when a hybrid RNA-seq dataset (e.g. PacBio PRJEB36569 plus Illumina
  PRJEB36584 aligned to galGal6) has produced separate per-technology BAM files and
  a genome-guided assembly input is needed for StringTie.
when_to_use_negative:
- Only one sequencing technology is available — assemble directly from the single-technology
  BAM (though note the merged assembly had superior BUSCO quality in this study)
- No reference genome assembly exists for genome-guided assembly — the pipeline requires
  a reference genome as a guide
- Transcripts are already assembled in GTF form — proceed directly to annotate_my_genomes
  rather than re-merging alignments
edam_operation: http://edamontology.org/operation_3801
edam_topics:
- http://edamontology.org/topic_0102
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_3569
tools:
- name: minimap2
  role: Aligns PacBio long reads against the galGal6 genome (~34x depth) to produce
    the long-read BAM
- name: fastp
  role: Automatically trims Illumina reads before HISAT2 alignment
- name: HISAT2
  role: Aligns trimmed Illumina short reads against galGal6 to produce the short-read
    BAM
- name: StringTie
  role: Assembles transcripts from the merged BAM with settings -p 1 -j 2 -c 2 -v
    -a 4
- name: Nextflow
  role: Wraps the alignment/assembly workflow for reproducibility
  repo: https://github.com/cfarkas/annotate_my_genomes
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/gigascience/giac099
    title: 'annotate_my_genomes: an easy-to-use pipeline to improve genome annotation
      and uncover neglected genes by hybrid RNA sequencing'
  source_content_hash: sha256:3be8e23915a763945ef8b24b2ab181cb9bdf56faf10cf2fe25322a8501d491c4
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/cfarkas/annotate_my_genomes
  tools:
  - HISAT2
  - Nextflow
  - StringTie
  - fastp
  - minimap2
  edam_operation: http://edamontology.org/operation_3801
  edam_topics:
  - http://edamontology.org/topic_0102
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_3569
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 10
---

# bam-alignment-merging

## Summary

Merge coordinate-sorted BAM alignment files from complementary sequencing technologies — here minimap2-aligned PacBio long reads and HISAT2-aligned fastp-trimmed Illumina short reads — into a single BAM to feed genome-guided transcript assembly with StringTie.

## When to use

Use when a hybrid RNA-seq dataset (e.g. PacBio PRJEB36569 plus Illumina PRJEB36584 aligned to galGal6) has produced separate per-technology BAM files and a genome-guided assembly input is needed for StringTie.

## When NOT to use

- Only one sequencing technology is available — assemble directly from the single-technology BAM (though note the merged assembly had superior BUSCO quality in this study)
- No reference genome assembly exists for genome-guided assembly — the pipeline requires a reference genome as a guide
- Transcripts are already assembled in GTF form — proceed directly to annotate_my_genomes rather than re-merging alignments

## Inputs

- minimap2-aligned PacBio long-read BAM (e.g. from ENA PRJEB36569 vs galGal6, ~34x depth)
- HISAT2-aligned Illumina short-read BAM from fastp-trimmed reads (e.g. from ENA PRJEB36584 vs galGal6)
- Reference genome assembly FASTA (galGal6, GCA_000002315.5)

## Outputs

- Single merged, coordinate-sorted BAM combining PacBio and Illumina alignments
- StringTie transcript assembly GTF (stringtie.gtf) suitable as the -a input of annotate-my-genomes

## How to apply

First generate technology-specific coordinate-sorted BAMs: align PacBio reads against the reference genome (galGal6, GCA_000002315.5) with minimap2 targeting ~34x read depth, and trim Illumina reads with fastp before aligning with HISAT2. Then merge the minimap2 PacBio BAM and the HISAT2 Illumina BAM into a single coordinate-sorted alignment file so that StringTie sees combined long- and short-read junction/coverage evidence. Assemble transcripts from the merged BAM with StringTie using parameters -p 1 -j 2 -c 2 -v -a 4 (1 thread, minimum junction coverage 2, minimum transcript coverage 2, verbose, minimum anchor length 4), writing the assembly as a GTF (stringtie.gtf) for downstream annotate_my_genomes annotation. The rationale is that the merged assembly, while not always best by rnaQUAST/IsoSeq Ex90N50 metrics, achieved BUSCO-assessed quality exceeding either single-technology assembly and was therefore selected for gene annotation.

## Related tools

- **minimap2** (Aligns PacBio long reads against the galGal6 genome (~34x depth) to produce the long-read BAM)
- **fastp** (Automatically trims Illumina reads before HISAT2 alignment)
- **HISAT2** (Aligns trimmed Illumina short reads against galGal6 to produce the short-read BAM)
- **StringTie** (Assembles transcripts from the merged BAM with settings -p 1 -j 2 -c 2 -v -a 4)
- **Nextflow** (Wraps the alignment/assembly workflow for reproducibility) — https://github.com/cfarkas/annotate_my_genomes

## Examples

```
stringtie merged.bam -G galGal6.fa -o stringtie.gtf -p 1 -j 2 -c 2 -v -a 4
```

## Evaluation signals

- Merged BAM is coordinate-sorted and parseable, combining reads from both minimap2 (PacBio) and HISAT2 (Illumina) sources
- Combined read depth approximates the reported ~34x PacBio depth plus Illumina coverage over the target genome
- stringtie.gtf exists, parses as valid GTF, and contains StringTie-assembled transcript records
- stringtie.gtf is accepted as the -a input of annotate-my-genomes.nf
- Downstream BUSCO assessment of the merged assembly exceeds that of Illumina-alone and PacBio-alone assemblies

## Limitations

- IsoSeq PacBio-only assemblies can show higher Ex90N50 values than the merged assembly; the merged assembly was chosen on BUSCO quality, not on all assembly metrics
- The downstream pipeline requires an assembled genome and an NCBI annotation in GTF format; it is not applicable to organisms lacking these
- Assembly quality benchmarks (precision/recall, F1-scores via gffcompare) are only reported in Supplementary Table 2 and require simulation-based benchmarking to reproduce

## Evidence

- [discussion] merged PacBio and Illumina BAMs and assemble with StringTie: "We merged the resulting BAM files from PacBio and Illumina read alignments into a single BAM file and we assembled transcripts from the latter alignment file using StringTie"
- [discussion] StringTie parameters: "using StringTie [57, 58] program with settings: -p 1 -j 2 -c 2 -v -a 4"
- [discussion] minimap2 alignment depth: [evidence span withheld — anchor status: not_found]
- [discussion] Illumina trim/align: "Illumina reads were automatically trimmed using fastp tool [31] and aligned against the referred Gallus gallus genome using HISAT2 aligner"
- [intro] assembly assessment context: "After assessing the best transcriptome assembly with tools such as rnaQUAST [14], SQANTI [15]"
- [intro] IsoSeq comparison caveat: "Well-established PacBio-only based pipelines such as IsoSeq [8, 9] and IsoCon [10] often perform well on these tasks"
