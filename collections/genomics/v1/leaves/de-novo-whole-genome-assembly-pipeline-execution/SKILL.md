---
name: de-novo-whole-genome-assembly-pipeline-execution
description: 'Use when benchmarking a targeted NRS-insertion caller against the whole-genome-assembly
  paradigm: 10x linked-read BAM/FASTQ data are available for a diploid genome and
  insertion calls must be derived from a full assembly rather than a local-assembly
  caller.'
when_to_use_negative:
- The goal is efficient NRS insertion detection itself — the article shows a targeted
  local-assembly approach (Novel-X) is substantially more accurate and computationally
  cheaper than whole-genome de novo assembly
- Input reads are standard short reads without barcode/linked-read information, so
  a synthetic-long-read assembler like SuperNova cannot be applied
- The sample is haploid (CHM1/CHM13) or the goal is small insertions — SuperNova defaults
  are tuned for whole human genome diploid assembly and small insertions lack sufficient
  barcodes for anchor assembly
edam_operation: http://edamontology.org/operation_0525
edam_topics:
- http://edamontology.org/topic_0196
tools:
- name: SuperNova
  role: Whole-genome diploid de novo assembler run on the 10x dataset to produce the
    assembly from which novel insertions are extracted
- name: Paftools
  role: Extracts novel insertion calls from the assembly-to-reference alignment
- name: Minimap2
  role: Aligns the SuperNova assembly contigs to the GRCh38 reference to generate
    the PAF consumed by Paftools
- name: LongRanger
  role: Produces the 10x-aligned BAM that defines the dataset the assembly is benchmarked
    on
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/nar/gkac653
    title: Efficient detection and assembly of non-reference DNA sequences with synthetic
      long reads
  source_content_hash: sha256:028acac0b5b318c63015d388d965d8ef60669ba8d7455c0018c770f32ea7ba35
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - LongRanger
  - Minimap2
  - Paftools
  - SuperNova
  edam_operation: http://edamontology.org/operation_0525
  edam_topics:
  - http://edamontology.org/topic_0196
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 10
---

# De novo whole-genome assembly pipeline execution

## Summary

Runs a whole-genome diploid de novo assembler (SuperNova) on synthetic long-read (10x Genomics linked-read) data and extracts novel insertion sequences from the assembly-to-reference alignment, serving as the computationally expensive whole-genome-assembly baseline that Novel-X avoids.

## When to use

Use when benchmarking a targeted NRS-insertion caller against the whole-genome-assembly paradigm: 10x linked-read BAM/FASTQ data are available for a diploid genome and insertion calls must be derived from a full assembly rather than a local-assembly caller.

## When NOT to use

- The goal is efficient NRS insertion detection itself — the article shows a targeted local-assembly approach (Novel-X) is substantially more accurate and computationally cheaper than whole-genome de novo assembly
- Input reads are standard short reads without barcode/linked-read information, so a synthetic-long-read assembler like SuperNova cannot be applied
- The sample is haploid (CHM1/CHM13) or the goal is small insertions — SuperNova defaults are tuned for whole human genome diploid assembly and small insertions lack sufficient barcodes for anchor assembly

## Inputs

- 10x Genomics linked-read dataset (e.g. LongRanger-aligned NA19240 BAM and raw FASTQs)
- GRCh38 reference genome FASTA
- PacBio long-read validation callset for the same sample (ground-truth insertions)

## Outputs

- Whole-genome diploid assembly (SuperNova contigs/megabubbles)
- PAF alignment of the assembly to the reference genome (Minimap2)
- VCF of novel insertion calls (length > 300 bp) extracted via Paftools
- Precision/recall benchmark table (e.g. benchmark_table.csv) comparing the assembly route to the targeted caller

## How to apply

Run the SuperNova whole-genome diploid assembler on the same 10x Genomics dataset used by the targeted caller (e.g. the LongRanger-aligned NA19240 dataset with GRCh38 as reference), accepting its default parameters since the assembler exposes no user control. Align the resulting assembly contigs/scaffolds to the reference genome with Minimap2 to produce a PAF alignment. Use Paftools on that alignment to extract novel insertion calls relative to the reference. Restrict calls to insertions longer than 300 bp to match the comparator's output scope, then match calls against a PacBio long-read validation callset counting a match when breakpoints differ by ≤100 bp and insertion lengths differ by ≤5%. The rationale is that whole-genome assembly was previously the only synthetic-long-read route to NRS detection, but it is computationally expensive because it assembles the entire genome rather than the small fraction of informative linked-reads.

## Related tools

- **SuperNova** (Whole-genome diploid de novo assembler run on the 10x dataset to produce the assembly from which novel insertions are extracted)
- **Paftools** (Extracts novel insertion calls from the assembly-to-reference alignment)
- **Minimap2** (Aligns the SuperNova assembly contigs to the GRCh38 reference to generate the PAF consumed by Paftools)
- **LongRanger** (Produces the 10x-aligned BAM that defines the dataset the assembly is benchmarked on)

## Evaluation signals

- The assembly-to-reference PAF alignment covers the assembled contigs against GRCh38 without excessive soft/hard-clipped alignments (>= 20% clipping indicates low-quality alignments)
- Extracted insertion VCF is restricted to insertions > 300 bp, matching the comparator's output filter
- Benchmark precision/recall against the PacBio validation callset uses the defined match criteria (breakpoints within 100 bp, lengths within 5%)
- Reported metrics for the SuperNova/Paftools route on NA19240-10x reproduce the paper's values: precision 56%, recall 37.4%
- Comparison table shows Novel-X achieves slightly better precision (59% vs 56%) but slightly lower recall (37.0% vs 37.4%) than the SuperNova/Paftools assembly route

## Limitations

- SuperNova offers the user no control over assembler parameters; its defaults are maximally tuned for whole human genome diploid assembly
- Whole-genome assembly is computationally expensive compared to targeted approaches that process only the small fraction of informative linked-reads
- Insertions shorter than 300 bp are out of scope; small insertions usually have too few associated barcodes for anchor assembly regardless of method

## Evidence

- [discussion] SuperNova assembler offers no user control over parameters; defaults tuned for whole human genome diploid assembly: "the user has no control over any assembler parameters, and default parameters are maximally tuned for whole human genome diploid assembly"
- [intro] Whole-genome assembly was the expensive prior route for NRS detection from synthetic long reads: "our method is computationally more efficient compared with those synthetic long-read approaches that use whole-genome de novo assembly such as (9,20)"
- [methods] Output restricted to insertions longer than 300 bp with anchor sum over 300 bp: [evidence span withheld — anchor status: not_found]
- [readme] stlfr2supernova pipeline assembles stLFR raw reads with the SuperNova assembler: [evidence span withheld — anchor status: external_doc]
