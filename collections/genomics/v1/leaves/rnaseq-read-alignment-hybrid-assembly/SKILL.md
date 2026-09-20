---
name: rnaseq-read-alignment-hybrid-assembly
description: Use when you have PacBio IsoSeq and Illumina RNA-seq reads from the same
  organism and a reference genome, and need to compare IsoSeq-only, merged, and Illumina-only
  transcriptome assemblies.
when_to_use_negative:
- The organism lacks an assembled reference genome and an NCBI annotation in GTF format,
  since the genome-guided assembly and annotation strategy cannot be applied.
- The goal is annotation of predicted proteins or coding/lncRNA classification of
  assembled transcripts — that is handled downstream by the annotate_my_genomes pipeline,
  not by this alignment/assembly step.
- Only de novo (reference-free) assembly is possible because no reference genome is
  available for the reads.
edam_operation: http://edamontology.org/operation_0292
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0196
tools:
- name: minimap2
  role: Aligns PacBio IsoSeq reads against the galGal6 genome, yielding ~34x depth
- name: fastp
  role: Automatically trims Illumina reads before alignment
- name: HISAT2
  role: Aligns trimmed Illumina reads against the galGal6 genome
- name: StringTie
  role: Assembles transcripts from the IsoSeq, merged, and Illumina BAM alignments
    with -p 1 -j 2 -c 2 -v -a 4
- name: IsoSeq
  role: PacBio-only pipeline whose assembly serves as the long-read baseline in the
    Ex90N50/BUSCO comparison
- name: Salmon
  role: Quantifies transcript abundance used to compute expression-weighted ExN50
- name: BUSCO
  role: Runs in transcriptome mode against aves_odb10 to measure assembly completeness
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/gigascience/giac099
    title: 'annotate_my_genomes: an easy-to-use pipeline to improve genome annotation
      and uncover neglected genes by hybrid RNA sequencing'
  source_content_hash: sha256:3bbcc1fee9a71673d9118d34610daaf2b39d5316ec2543d02a1a6bd3fb983b73
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - BUSCO
  - HISAT2
  - IsoSeq
  - SQANTI
  - Salmon
  - StringTie
  - fastp
  - minimap2
  - rnaQUAST
  edam_operation: http://edamontology.org/operation_0292
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0196
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 12
---

# rnaseq-read-alignment-hybrid-assembly

## Summary

Generates a genome-guided hybrid transcriptome assembly by aligning PacBio IsoSeq and trimmed Illumina reads to a reference genome (galGal6), merging the alignments into a single BAM, and assembling transcripts with StringTie for downstream Ex90N50 and BUSCO (aves_odb10) comparison.

## When to use

Use when you have PacBio IsoSeq and Illumina RNA-seq reads from the same organism and a reference genome, and need to compare IsoSeq-only, merged, and Illumina-only transcriptome assemblies.

## When NOT to use

- The organism lacks an assembled reference genome and an NCBI annotation in GTF format, since the genome-guided assembly and annotation strategy cannot be applied.
- The goal is annotation of predicted proteins or coding/lncRNA classification of assembled transcripts — that is handled downstream by the annotate_my_genomes pipeline, not by this alignment/assembly step.
- Only de novo (reference-free) assembly is possible because no reference genome is available for the reads.

## Inputs

- PacBio IsoSeq reads from chicken brain (ENA PRJEB36569)
- Illumina paired-end RNA-seq reads from chicken brain (ENA PRJEB36584)
- Reference genome galGal6 (GCA_000002315.5) FASTA
- Reference annotation in GTF format (galGal6_ncbiRefSeq.gtf)

## Outputs

- PacBio BAM alignment against galGal6 (~34x depth)
- Trimmed Illumina reads and HISAT2 BAM alignment
- Merged PacBio+Illumina single BAM file
- Three StringTie GTF assemblies (IsoSeq-only, merged, Illumina-only)
- Salmon abundance quantifications and Ex90N50 statistics per assembly
- BUSCO aves_odb10 complete (single-copy + duplicated) gene counts per assembly
- ex90n50_busco_metrics.csv tabulating Ex90N50 and completed-BUSCO values

## How to apply

Download the PacBio reads (ENA PRJEB36569) and Illumina reads (ENA PRJEB36584), then align PacBio reads against the galGal6 genome with minimap2 to a depth of ~34x. Trim Illumina reads with fastp and align them with HISAT2 against the same genome. Merge the PacBio and Illumina BAM files into a single BAM and assemble transcripts from three alignments — PacBio-only, merged, and Illumina-only — using StringTie with settings -p 1 -j 2 -c 2 -v -a 4. Quantify abundance with Salmon, compute the expression-weighted Ex90N50, and run BUSCO in transcriptome mode against aves_odb10 to score completeness. Rationale: hybrid merging improves BUSCO completeness over Illumina-alone (~1%), even though the IsoSeq assembly attains superior Ex90N50 statistics.

## Related tools

- **minimap2** (Aligns PacBio IsoSeq reads against the galGal6 genome, yielding ~34x depth)
- **fastp** (Automatically trims Illumina reads before alignment)
- **HISAT2** (Aligns trimmed Illumina reads against the galGal6 genome)
- **StringTie** (Assembles transcripts from the IsoSeq, merged, and Illumina BAM alignments with -p 1 -j 2 -c 2 -v -a 4)
- **IsoSeq** (PacBio-only pipeline whose assembly serves as the long-read baseline in the Ex90N50/BUSCO comparison)
- **Salmon** (Quantifies transcript abundance used to compute expression-weighted ExN50)
- **BUSCO** (Runs in transcriptome mode against aves_odb10 to measure assembly completeness)

## Examples

```
stringtie -p 1 -j 2 -c 2 -v -a 4 merged_pacbio_illumina.sorted.bam -o merged_stringtie.gtf
```

## Evaluation signals

- PacBio alignment against galGal6 reaches a depth of approximately 34x as reported with minimap2.
- The merged PacBio+Illumina assembly shows a completed-BUSCO count (aves_odb10) approximately 1% higher than the Illumina-only assembly.
- The IsoSeq PacBio-only assembly yields the superior Ex90N50 statistic among the three assemblies, matching Figure 1B-1C.
- All three StringTie assemblies were produced with identical settings (-p 1 -j 2 -c 2 -v -a 4) so the comparison is parameter-controlled.
- Ex90N50 and completed-BUSCO values for each assembly are present in ex90n50_busco_metrics.csv with one row per assembly strategy.

## Limitations

- The comparison is specific to the chicken brain transcriptome on galGal6; Ex90N50 advantages for IsoSeq may not transfer to other tissues or genomes.
- Assembly completeness measured by BUSCO and continuity measured by Ex90N50 rank the strategies differently, so a single metric does not determine the 'best' assembly.
- The downstream annotate_my_genomes pipeline requires organisms with an assembled genome and NCBI annotation in GTF format, constraining where this hybrid assembly strategy is useful.

## Evidence

- [discussion] Align PacBio reads against genome with minimap2: [evidence span withheld — anchor status: not_found]
- [discussion] Trim and align Illumina reads: "Illumina reads were automatically trimmed using fastp tool [31] and aligned against the referred Gallus gallus genome using HISAT2 aligner"
- [discussion] Merge BAMs and assemble with StringTie: "We merged the resulting BAM files from PacBio and Illumina read alignments into a single BAM file and we assembled transcripts from the latter alignment file using StringTie"
- [discussion] StringTie assembly parameters: "using StringTie [57, 58] program with settings: -p 1 -j 2 -c 2 -v -a 4"
- [supplementary] Dataset accessions: "available at European Nucleotide Archive (ENA) Accession Number PRJEB36569 (PacBio)"
- [readme] Pipeline scope from README: "annotate genome-guided transcriptome assemblies from StringTie, coming from long read RNA-Seq alignments in vertebrate genomes"
