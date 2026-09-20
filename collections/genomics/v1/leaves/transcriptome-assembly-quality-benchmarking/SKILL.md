---
name: transcriptome-assembly-quality-benchmarking
description: Use when multiple transcriptome assemblies built from the same reads
  (long-read-only, short-read-only, hybrid merged BAM) must be ranked by Ex90N50 and
  BUSCO completeness to decide which assembly strategy to carry into annotation.
when_to_use_negative:
- No assembled reference genome and no NCBI GTF annotation exist for the organism
  — the genome-guided StringTie/minimap2/HISAT2 strategy cannot be applied
- The goal is annotation quality assessment (F1-score via gffcompare) rather than
  ranking assembly strategies by contiguity and BUSCO completeness
- Only one assembly exists and no competing strategy is being compared — a single
  BUSCO/ExN50 run is descriptive, not a benchmarking exercise
edam_operation: http://edamontology.org/operation_0525
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: minimap2
  role: Aligns PacBio IsoSeq reads against the galGal6 genome (~34x depth) to build
    the long-read-only assembly input
- name: fastp
  role: Automatic trimming of Illumina reads before HISAT2 alignment for the short-read-only
    assembly
- name: HISAT2
  role: Aligns trimmed Illumina reads against the Gallus gallus genome
- name: StringTie
  role: Assembles transcripts from each BAM input (PacBio-only, Illumina-only, merged)
    with identical settings -p 1 -j 2 -c 2 -v -a 4
- name: Salmon
  role: Quantifies transcript abundance per assembly to feed the expression-weighted
    ExN50 calculation
- name: BUSCO
  role: Run in transcriptome mode against aves_odb10 to measure completed (single-copy
    + duplicated) gene counts per assembly
- name: rnaQUAST
  role: Assembly quality assessment tool referenced when choosing the best transcriptome
    assembly
- name: SQANTI
  role: Assembly assessment tool referenced alongside rnaQUAST for evaluating transcriptome
    assemblies
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/gigascience/giac099
    title: 'annotate_my_genomes: an easy-to-use pipeline to improve genome annotation
      and uncover neglected genes by hybrid RNA sequencing'
  source_content_hash: sha256:3e3c1daf0d5d6ab7c2ef344b2848a15533a61ac3f36ec9bfb874156d452a1f05
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
  edam_operation: http://edamontology.org/operation_0525
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8181818181818182
n_claims: 11
---

# transcriptome-assembly-quality-benchmarking

## Summary

Benchmark competing genome-guided transcriptome assembly strategies (PacBio IsoSeq-only, merged PacBio+Illumina, Illumina-only) using expression-weighted Ex90N50 from Salmon quantification and completed-BUSCO counts against the aves_odb10 lineage, as applied to the chicken SCO brain transcriptome (galGal6).

## When to use

Use when multiple transcriptome assemblies built from the same reads (long-read-only, short-read-only, hybrid merged BAM) must be ranked by Ex90N50 and BUSCO completeness to decide which assembly strategy to carry into annotation.

## When NOT to use

- No assembled reference genome and no NCBI GTF annotation exist for the organism — the genome-guided StringTie/minimap2/HISAT2 strategy cannot be applied
- The goal is annotation quality assessment (F1-score via gffcompare) rather than ranking assembly strategies by contiguity and BUSCO completeness
- Only one assembly exists and no competing strategy is being compared — a single BUSCO/ExN50 run is descriptive, not a benchmarking exercise

## Inputs

- PacBio IsoSeq reads (ENA PRJEB36569) aligned to a reference genome (galGal6, GCA_000002315.5) as BAM via minimap2
- Illumina reads (ENA PRJEB36584) trimmed with fastp and aligned with HISAT2 as BAM
- Merged PacBio+Illumina BAM alignment file
- StringTie GTF/FASTA assemblies built with -p 1 -j 2 -c 2 -v -a 4
- Salmon transcript quantification per assembly
- BUSCO aves_odb10 lineage dataset

## Outputs

- Per-assembly Ex90N50 values computed from Salmon quantification via the ExN50 script
- Per-assembly completed-BUSCO (single-copy + duplicated) counts for aves_odb10
- Metrics table (ex90n50_busco_metrics.csv) and Figure 1B–1C-style assembly comparison

## How to apply

Build all candidate assemblies with StringTie using identical settings (-p 1 -j 2 -c 2 -v -a 4) so comparisons are confounded only by input read type, not assembler parameters: (a) PacBio IsoSeq reads aligned to galGal6 with minimap2 (~34x depth), (b) Illumina reads trimmed with fastp and aligned with HISAT2, and (c) the merged PacBio+Illumina single BAM file. Quantify transcript abundance in each assembly with Salmon, then run the Salmon-based ExN50 script to compute the expression-weighted Ex90N50 (N50 over transcripts contributing 90% of total expressed signal), which weights assembly contiguity by biological expression rather than raw length. Run BUSCO in transcriptome mode against the aves_odb10 lineage on each assembly and record complete (single-copy + duplicated) gene counts. Tabulate Ex90N50 and completed-BUSCO values per assembly (e.g. ex90n50_metrics.csv) to reproduce a Figure 1B–1C-style comparison; in this dataset the IsoSeq assembly won on Ex90N50, but the merged and Illumina assemblies had higher completed-BUSCO counts, with the merged strategy ~1% more complete than Illumina-alone.

## Related tools

- **minimap2** (Aligns PacBio IsoSeq reads against the galGal6 genome (~34x depth) to build the long-read-only assembly input)
- **fastp** (Automatic trimming of Illumina reads before HISAT2 alignment for the short-read-only assembly)
- **HISAT2** (Aligns trimmed Illumina reads against the Gallus gallus genome)
- **StringTie** (Assembles transcripts from each BAM input (PacBio-only, Illumina-only, merged) with identical settings -p 1 -j 2 -c 2 -v -a 4)
- **Salmon** (Quantifies transcript abundance per assembly to feed the expression-weighted ExN50 calculation)
- **BUSCO** (Run in transcriptome mode against aves_odb10 to measure completed (single-copy + duplicated) gene counts per assembly)
- **rnaQUAST** (Assembly quality assessment tool referenced when choosing the best transcriptome assembly)
- **SQANTI** (Assembly assessment tool referenced alongside rnaQUAST for evaluating transcriptome assemblies)

## Examples

```
stringtie merged_PacBio_Illumina.bam -p 1 -j 2 -c 2 -v -a 4 -o merged_assembly.gtf
```

## Evaluation signals

- PacBio alignment against galGal6 reaches approximately 34x depth before assembly
- All three assemblies are built with identical StringTie settings (-p 1 -j 2 -c 2 -v -a 4), so only the input BAM differs
- Ex90N50 is computed from Salmon quantification with the expression-coverage threshold set at 90% of total expressed signal
- The merged PacBio+Illumina assembly shows a completed-BUSCO count approximately 1% higher than the Illumina-only assembly for aves_odb10, matching Figure 1B–1C
- IsoSeq-only assembly reports superior Ex90N50 statistics even though it ranks lower on BUSCO completeness — both metrics must be tabulated per assembly

## Limitations

- BUSCO completeness and Ex90N50 can disagree: IsoSeq won on Ex90N50 while Illumina and merged assemblies won on completed BUSCOs, so no single metric determines the best assembly
- The comparison is genome-guided and requires a reference genome (galGal6) plus available RNA-seq data from the same organism (chicken brain); results may not transfer to other tissues or species
- The ~1% completeness gain of the merged strategy over Illumina-alone is small and was validated specifically for the chicken SCO transcriptome

## Evidence

- [discussion] We aligned PacBio reads against Gallus gallus genome ... using minimap2 aligner [30] obtaining depths of ∼ 34x: [evidence span withheld — anchor status: not_found]
- [discussion] Illumina reads were automatically trimmed using fastp tool and aligned against the referred Gallus gallus genome using HISAT2 aligner: "Illumina reads were automatically trimmed using fastp tool [31] and aligned against the referred Gallus gallus genome using HISAT2 aligner"
- [discussion] using StringTie program with settings: -p 1 -j 2 -c 2 -v -a 4: "using StringTie [57, 58] program with settings: -p 1 -j 2 -c 2 -v -a 4"
- [discussion] We merged the resulting BAM files from PacBio and Illumina read alignments into a single BAM file and we assembled transcripts from the latter alignment file using StringTie: "We merged the resulting BAM files from PacBio and Illumina read alignments into a single BAM file and we assembled transcripts from the latter alignment file using StringTie"
- [supplementary] available at European Nucleotide Archive (ENA) Accession Number PRJEB36569 (PacBio): "available at European Nucleotide Archive (ENA) Accession Number PRJEB36569 (PacBio)"
