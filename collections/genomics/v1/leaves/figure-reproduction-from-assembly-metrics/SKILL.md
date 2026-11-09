---
name: figure-reproduction-from-assembly-metrics
description: Decision support for choosing a transcriptome assembly strategy from
  hybrid PacBio+Illumina reads when you must reproduce assembly-quality figures quantified
  by Ex90N50 and completed-BUSCO metrics.
when_to_use_negative:
- The organism has no assembled reference genome available, since all three assemblies
  are genome-guided StringTie assemblies against galGal6
- The goal is de novo transcriptome assembly without a reference genome; this comparison
  relies on genome-guided StringTie GTF output
- Only one sequencing modality is available, so the IsoSeq-vs-merged-vs-Illumina comparison
  cannot be constructed
edam_operation: http://edamontology.org/operation_0362
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_3307
tools:
- name: minimap2
  role: Aligns PacBio reads against galGal6 (~34x depth) for the IsoSeq-only assembly
    input
- name: fastp
  role: Automatic trimming of Illumina reads before HISAT2 alignment
- name: HISAT2
  role: Aligns trimmed Illumina reads to galGal6 for the Illumina-only assembly input
- name: StringTie
  role: Assembles all three transcriptome GTFs with -p 1 -j 2 -c 2 -v -a 4
- name: Salmon
  role: Quantifies transcript abundance per assembly for the Salmon-based ExN50 script
    computing Ex90N50
- name: BUSCO
  role: Transcriptome-mode completeness assessment against the aves_odb10 lineage
- name: IsoSeq
  role: PacBio-only pipeline whose assembly is one arm of the three-way comparison
- name: rnaQUAST
  role: Assembly assessment tool referenced by the article for transcriptome comparison
- name: SQANTI
  role: Assembly assessment tool referenced by the article for transcriptome comparison
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/gigascience/giac099
    title: 'annotate_my_genomes: an easy-to-use pipeline to improve genome annotation
      and uncover neglected genes by hybrid RNA sequencing'
  source_content_hash: sha256:1525d20df77b2d3258b0ca1bdafba29d16447bde5605ea5a8e292ae3755a2e01
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
  edam_operation: http://edamontology.org/operation_0362
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_3307
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 19
---

# figure-reproduction-from-assembly-metrics

## Summary

A benchmark-and-reproduce workflow that rebuilds competing transcriptome assemblies (IsoSeq PacBio-only, merged PacBio+Illumina, Illumina-only) for the chicken SCO brain data, computes expression-weighted Ex90N50 and completed BUSCO (aves_odb10) counts, and reproduces the Figure 1B–1C comparison.

## When to use

Decision support for choosing a transcriptome assembly strategy from hybrid PacBio+Illumina reads when you must reproduce assembly-quality figures quantified by Ex90N50 and completed-BUSCO metrics.

## When NOT to use

- The organism has no assembled reference genome available, since all three assemblies are genome-guided StringTie assemblies against galGal6
- The goal is de novo transcriptome assembly without a reference genome; this comparison relies on genome-guided StringTie GTF output
- Only one sequencing modality is available, so the IsoSeq-vs-merged-vs-Illumina comparison cannot be constructed

## Inputs

- PacBio IsoSeq chicken brain reads (ENA PRJEB36569)
- Illumina chicken brain reads (ENA PRJEB36584)
- galGal6 reference genome (GCA_000002315.5)
- Alignments: minimap2 PacBio BAM, HISAT2 Illumina BAM, merged PacBio+Illumina BAM
- StringTie GTF assemblies (IsoSeq, merged, Illumina-only)
- BUSCO aves_odb10 lineage dataset

## Outputs

- ex90n50_busco_metrics.csv with per-assembly Ex90N50 and completed-BUSCO values
- Reproduction of Figure 1B (completed BUSCOs, aves_odb10) and Figure 1C (Ex90N50) comparison
- BUSCO run results (transcriptome mode) for the three assemblies

## How to apply

Download the PacBio chicken brain reads from ENA PRJEB36569 and Illumina reads from ENA PRJEB36584, align PacBio reads to galGal6 with minimap2 (~34x depth), and trim Illumina reads with fastp followed by HISAT2 alignment. Build three StringTie assemblies with -p 1 -j 2 -c 2 -v -a 4: from the IsoSeq PacBio-only BAM, a merged PacBio+Illumina single BAM, and the Illumina-only BAM. Quantify transcripts with Salmon and run the Salmon-based ExN50 script to compute the Ex90N50 (N50 of transcripts contributing 90% of total expressed signal), then run BUSCO in transcriptome mode against aves_odb10 and record complete (single-copy + duplicated) counts. Tabulate per-assembly Ex90N50 and completed-BUSCO values into ex90n50_busco_metrics.csv to reproduce Figure 1B–1C; validation requires the merged assembly to show completed BUSCOs ~1% higher than Illumina-only.

## Related tools

- **minimap2** (Aligns PacBio reads against galGal6 (~34x depth) for the IsoSeq-only assembly input)
- **fastp** (Automatic trimming of Illumina reads before HISAT2 alignment)
- **HISAT2** (Aligns trimmed Illumina reads to galGal6 for the Illumina-only assembly input)
- **StringTie** (Assembles all three transcriptome GTFs with -p 1 -j 2 -c 2 -v -a 4)
- **Salmon** (Quantifies transcript abundance per assembly for the Salmon-based ExN50 script computing Ex90N50)
- **BUSCO** (Transcriptome-mode completeness assessment against the aves_odb10 lineage)
- **IsoSeq** (PacBio-only pipeline whose assembly is one arm of the three-way comparison)
- **rnaQUAST** (Assembly assessment tool referenced by the article for transcriptome comparison)
- **SQANTI** (Assembly assessment tool referenced by the article for transcriptome comparison)

## Examples

```
stringtie merged_PB_IL.bam -p 1 -j 2 -c 2 -v -a 4 -o merged_assembly.gtf; busco -i merged_assembly.fa -m transcriptome -l aves_odb10 -o busco_merged
```

## Evaluation signals

- The merged PacBio+Illumina assembly shows a completed-BUSCO count approximately 1% higher than the Illumina-only assembly, matching Figure 1B–1C
- The IsoSeq PacBio-only assembly exhibits the superior Ex90N50 statistic across the three assemblies
- completed-BUSCO values are recorded as complete (single-copy + duplicated) counts against the aves_odb10 lineage for all three GTF/FASTA assemblies
- ex90n50_busco_metrics.csv contains one Ex90N50 and one completed-BUSCO row/value per assembly strategy (IsoSeq, merged, Illumina-only)
- minimap2 PacBio alignment depth against galGal6 is approximately 34x before assembly

## Limitations

- The trade-off is metric-dependent: IsoSeq wins on Ex90N50 while merged/Illumina assemblies win on BUSCO completeness, so no single assembly dominates on all criteria
- Precision/recall benchmarking values are only available in Supplementary Table 2, not in the main text
- Protein products of assembled SSPO isoforms remain to be validated, so assembly completeness does not guarantee protein-level correctness

## Evidence

- [discussion] minimap2 alignment depth: [evidence span withheld — anchor status: not_found]
- [discussion] merged BAM StringTie assembly: "We merged the resulting BAM files from PacBio and Illumina read alignments into a single BAM file and we assembled transcripts from the latter alignment file using StringTie"
- [discussion] StringTie parameters: "using StringTie [57, 58] program with settings: -p 1 -j 2 -c 2 -v -a 4"
- [other] Ex90N50 vs BUSCO trade-off: [evidence span withheld — anchor status: not_found]
- [intro] assessment tools: "After assessing the best transcriptome assembly with tools such as rnaQUAST [14], SQANTI [15]"
