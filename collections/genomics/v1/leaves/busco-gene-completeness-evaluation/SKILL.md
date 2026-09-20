---
name: busco-gene-completeness-evaluation
description: 'Use when comparing multiple genome-guided transcriptome assembly strategies
  and you need to decide which is most complete: e.g., choosing between long-read-only
  (IsoSeq PacBio), hybrid merged BAM, and short-read-only assemblies using BUSCO lineage
  scores plus Ex90N50.'
when_to_use_negative:
- The organism lacks an assembled reference genome and NCBI GTF annotation, since
  the genome-guided StringTie workflow feeding this comparison cannot be applied (the
  article restricts applicability to such organisms)
- You need structural/transcript accuracy assessment (precision/recall/F1) rather
  than gene-content completeness; use gffcompare with a truth annotation or rnaQUAST/SQANTI
  instead
- De novo (non-genome-guided) assemblies with no reference alignment — the hybrid-assembly
  comparison here depends on aligning to galGal6 with minimap2/HISAT2
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0622
tools:
- name: BUSCO
  role: Run in transcriptome mode against aves_odb10 to count complete (single-copy
    + duplicated) genes per assembly
- name: StringTie
  role: Build the three compared GTF assemblies with settings -p 1 -j 2 -c 2 -v -a
    4
- name: Salmon
  role: Quantify transcript abundance per assembly to compute the Salmon-based ExN50
    / Ex90N50 metric
- name: minimap2
  role: Align PacBio IsoSeq reads against galGal6 (~34x depth) as the long-read assembly
    input
- name: fastp
  role: Trim Illumina reads before HISAT2 alignment for the Illumina-only assembly
    input
- name: HISAT2
  role: Align trimmed Illumina reads to galGal6 for the Illumina-only assembly input
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/gigascience/giac099
    title: 'annotate_my_genomes: an easy-to-use pipeline to improve genome annotation
      and uncover neglected genes by hybrid RNA sequencing'
  source_content_hash: sha256:874a63e88b16962ff908c238f55033b1ac5a96909dfffe5530ce1a09b43ffbf9
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
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8461538461538461
n_claims: 13
---

# busco-gene-completeness-evaluation

## Summary

Benchmark the completeness of transcriptome assemblies (StringTie GTF/FASTA from IsoSeq PacBio-only, merged PacBio+Illumina, and Illumina-only chicken brain alignments) by running BUSCO in transcriptome mode against the aves_odb10 lineage and comparing completed (single-copy + duplicated) BUSCO counts, alongside Salmon-based expression-weighted Ex90N50 statistics.

## When to use

Use when comparing multiple genome-guided transcriptome assembly strategies and you need to decide which is most complete: e.g., choosing between long-read-only (IsoSeq PacBio), hybrid merged BAM, and short-read-only assemblies using BUSCO lineage scores plus Ex90N50.

## When NOT to use

- The organism lacks an assembled reference genome and NCBI GTF annotation, since the genome-guided StringTie workflow feeding this comparison cannot be applied (the article restricts applicability to such organisms)
- You need structural/transcript accuracy assessment (precision/recall/F1) rather than gene-content completeness; use gffcompare with a truth annotation or rnaQUAST/SQANTI instead
- De novo (non-genome-guided) assemblies with no reference alignment — the hybrid-assembly comparison here depends on aligning to galGal6 with minimap2/HISAT2

## Inputs

- StringTie GTF assemblies built from (a) IsoSeq PacBio-only alignment, (b) merged PacBio+Illumina BAM, (c) Illumina-only HISAT2 alignment (StringTie settings: -p 1 -j 2 -c 2 -v -a 4)
- Reference genome galGal6 (GCA_000002315.5) for alignment inputs
- PacBio reads from ENA PRJEB36569 and Illumina reads from ENA PRJEB36584 (chicken brain)
- BUSCO aves_odb10 lineage dataset
- Salmon transcript abundance quantifications per assembly (for ExN50 computation)

## Outputs

- ex90n50_busco_metrics.csv tabulating Ex90N50 and completed-BUSCO values per assembly strategy
- BUSCO transcriptome-mode results (complete single-copy, complete duplicated, fragmented, missing gene counts) for each of the three assemblies
- Figure 1B–1C comparison of Ex90N50 vs. completed BUSCOs across assembly strategies

## How to apply

Build all candidate assemblies with StringTie using identical parameters (-p 1 -j 2 -c 2 -v -a 4) on each alignment input: (a) PacBio IsoSeq reads aligned to galGal6 with minimap2 (~34x depth), (b) a merged PacBio+Illumina single BAM, and (c) Illumina reads trimmed with fastp and aligned with HISAT2. Quantify transcript abundance in each assembly with Salmon and run a Salmon-based ExN50 script to obtain the expression-weighted Ex90N50 (N50 of transcripts contributing 90% of expressed signal). Then run BUSCO in transcriptome mode against the aves_odb10 lineage on each of the three GTF/FASTA assemblies and record complete gene counts (single-copy + duplicated). Tabulate Ex90N50 and completed-BUSCO values per assembly into ex90n50_busco_metrics.csv and reproduce the Figure 1B–1C comparison. Rationale: the article found IsoSeq had superior Ex90N50 statistics, but Illumina and merged assemblies achieved higher completed-BUSCO completeness, with the merged strategy improving completeness over Illumina-alone by ~1% — so neither metric alone should drive the assembly-strategy decision.

## Related tools

- **BUSCO** (Run in transcriptome mode against aves_odb10 to count complete (single-copy + duplicated) genes per assembly)
- **StringTie** (Build the three compared GTF assemblies with settings -p 1 -j 2 -c 2 -v -a 4)
- **Salmon** (Quantify transcript abundance per assembly to compute the Salmon-based ExN50 / Ex90N50 metric)
- **minimap2** (Align PacBio IsoSeq reads against galGal6 (~34x depth) as the long-read assembly input)
- **fastp** (Trim Illumina reads before HISAT2 alignment for the Illumina-only assembly input)
- **HISAT2** (Align trimmed Illumina reads to galGal6 for the Illumina-only assembly input)

## Examples

```
busco -i merged_stringtie_assembly.fasta -l aves_odb10 -m transcript -o busco_merged
```

## Evaluation signals

- The merged PacBio+Illumina assembly shows a completed-BUSCO count approximately 1% higher than the Illumina-only assembly (aves_odb10), matching Figure 1B–1C
- The IsoSeq PacBio-only assembly shows the best Ex90N50 statistic while ranking lower on completed BUSCOs — Ex90N50 and BUSCO rankings must not be conflated
- Each assembly was built with identical StringTie parameters (-p 1 -j 2 -c 2 -v -a 4), so BUSCO differences reflect input alignments, not assembly settings
- ex90n50_busco_metrics.csv contains one Ex90N50 value and one completed-BUSCO count per assembly strategy (three rows)
- PacBio alignment depth is ~34x against galGal6; a substantially different depth indicates incorrect read subsetting or indexing

## Limitations

- IsoSeq assembly had superior Ex90N50 statistics but lower BUSCO completeness, so completeness metrics from different quality dimensions must be reported together
- Reported precision/recall and F1 values are deferred to Supplementary Table 2 rather than the main text, limiting in-text validation of structural accuracy
- Protein products of assembled SSPO isoforms (89-exon and 25-exon) were not validated, so transcript-level completeness does not guarantee confirmed protein products

## Evidence

- [discussion] using StringTie [57, 58] program with settings: -p 1 -j 2 -c 2 -v -a 4: "using StringTie [57, 58] program with settings: -p 1 -j 2 -c 2 -v -a 4"
- [discussion] minimap2 alignment depth: [evidence span withheld — anchor status: not_found]
- [supplementary] ENA accessions: "available at European Nucleotide Archive (ENA) Accession Number PRJEB36569 (PacBio)"
- [discussion] Illumina trimming and alignment: "Illumina reads were automatically trimmed using fastp tool [31] and aligned against the referred Gallus gallus genome using HISAT2 aligner"
- [abstract] repo availability: "The code and datasets are publicly available at https://github.com/cfarkas/annotate_my_genomes"
