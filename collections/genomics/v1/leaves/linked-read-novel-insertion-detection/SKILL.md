---
name: linked-read-novel-insertion-detection
description: Use when you have a LongRanger/stLFR/TELL-Seq aligned BAM of synthetic
  long reads and need to discover and place novel insertions longer than 300 bp that
  short-read callers miss, without running a whole-genome assembler like SuperNova.
when_to_use_negative:
- Insertions are small (<300 bp) — Novel-X needs a substantial number of distinct
  barcodes per insertion, and small insertions usually lack sufficient coverage for
  anchor assembly
- A whole-genome diploid assembly is already available or desired — SuperNova-based
  placement may be preferable, though it is computationally far more expensive and
  offers no user control over assembler parameters
- Input is short-read data without linked-read barcodes — use short-read NRS methods
  like NovelSeq, MindTheGap, PopIns2 or Pamir instead
edam_operation: http://edamontology.org/operation_3227
edam_topics:
- http://edamontology.org/topic_3673
- http://edamontology.org/topic_0199
tools:
- name: Novel-X
  role: Snakemake pipeline orchestrating the four-step NRS detection and placement
    workflow
  repo: github.com/1dayac/Novel-X
- name: Velvet
  role: De novo assembly of unaligned/poorly-aligned reads into orphan insertion contigs
    (commit 9adf09f, k=63 with MAXKMERLENGTH=63)
  repo: https://github.com/dzerbino/velvet
- name: SPAdes
  role: Reassembly of barcode-recruited reads (k=77, cov-cutoff=3) to build left-anchor--insertion--right-anchor
    contigs
- name: Minimap2
  role: Placement of anchors/orphan contigs on the reference genome and best-alignment
    filtering
- name: LongRanger
  role: Produces the barcode-annotated BAM that serves as Novel-X input directly
- name: SuperNova
  role: Whole-genome diploid assembler baseline against which Novel-X precision (59%
    vs 56%) and recall (37.0% vs 37.4%) were benchmarked
  repo: https://github.com/BGI-Qingdao/stlfr2supernova_pipeline
provenance:
  source_task_ids:
  - task_003
  - task_004
  - task_006
  source_papers:
  - doi: 10.1093/nar/gkac653
    title: Efficient detection and assembly of non-reference DNA sequences with synthetic
      long reads
  source_content_hash: sha256:0faaf5ec39c6325023469062f1931a007d54964c37eb1cac16e2817d6107d573
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - LongRanger
  - Minimap2
  - Novel-X
  - Paftools
  - SPAdes
  - SuperNova
  - Velvet
  edam_operation: http://edamontology.org/operation_3227
  edam_topics:
  - http://edamontology.org/topic_3673
  - http://edamontology.org/topic_0199
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8636363636363636
n_claims: 22
---

# linked-read-novel-insertion-detection

## Summary

A four-step Snakemake pipeline (Novel-X) that detects and genotypes non-reference sequence (NRS) insertions >300 bp in 10x Genomics linked-read data by combining alignment signatures (unmapped/MQ<10/>20% soft-clipped reads) with barcode-guided local reassembly, avoiding whole-genome de novo assembly.

## When to use

Use when you have a LongRanger/stLFR/TELL-Seq aligned BAM of synthetic long reads and need to discover and place novel insertions longer than 300 bp that short-read callers miss, without running a whole-genome assembler like SuperNova.

## When NOT to use

- Insertions are small (<300 bp) — Novel-X needs a substantial number of distinct barcodes per insertion, and small insertions usually lack sufficient coverage for anchor assembly
- A whole-genome diploid assembly is already available or desired — SuperNova-based placement may be preferable, though it is computationally far more expensive and offers no user control over assembler parameters
- Input is short-read data without linked-read barcodes — use short-read NRS methods like NovelSeq, MindTheGap, PopIns2 or Pamir instead

## Inputs

- LongRanger-aligned 10x Genomics BAM with barcode information (e.g. NA19240, CHM1, HG002, Polaris PGx samples)
- Reference genome FASTA (e.g. GRCh38)
- stLFR or TELL-Seq linked-read BAMs (stLFR via stlfr2supernova_pipeline barcode handling)

## Outputs

- VCF of NRS insertion calls >300 bp with anchor lengths summing over 300 bp
- Per-sample VCFs for cohort merging (e.g. 68 Polaris samples merged to 16,691 insertions, 18.2 Mb total, longest insertion 85,908 bp)
- Reassembled anchor--insertion contigs from SPAdes

## How to apply

Run the Novel-X Snakemake pipeline (novel-x.py run --10x --high-coverage, or clone with --recursive since submodules are required) on a LongRanger-aligned BAM against GRCh38. Step 1 assembles reads with at least one unmapped end, mapping quality <10, or >20% soft-clipped bases, keeping only reads with average base quality >20, into orphan contigs using Velvet (k=63, MAXKMERLENGTH=63 for 10x data). Step 2 extracts barcode lists per contig, discarding contigs recruiting fewer than t=5 barcodes and alignments with >=20% soft/hard-clipped bases. Step 3 reassembles all reads carrying those barcodes with SPAdes (k=77, cov-cutoff=3) to build left-anchor--insertion--right-anchor contigs, retaining only contigs whose best Minimap2 match corresponds to an orphan contig. Step 4 places anchors on the reference with Minimap2, applies QUAST-like best-alignment-subset filtering, and outputs a VCF of insertions longer than 300 bp whose anchor lengths sum to >300 bp to prevent false calls.

## Related tools

- **Novel-X** (Snakemake pipeline orchestrating the four-step NRS detection and placement workflow) — github.com/1dayac/Novel-X
- **Velvet** (De novo assembly of unaligned/poorly-aligned reads into orphan insertion contigs (commit 9adf09f, k=63 with MAXKMERLENGTH=63)) — https://github.com/dzerbino/velvet
- **SPAdes** (Reassembly of barcode-recruited reads (k=77, cov-cutoff=3) to build left-anchor--insertion--right-anchor contigs)
- **Minimap2** (Placement of anchors/orphan contigs on the reference genome and best-alignment filtering)
- **LongRanger** (Produces the barcode-annotated BAM that serves as Novel-X input directly)
- **SuperNova** (Whole-genome diploid assembler baseline against which Novel-X precision (59% vs 56%) and recall (37.0% vs 37.4%) were benchmarked) — https://github.com/BGI-Qingdao/stlfr2supernova_pipeline

## Examples

```
git clone --recursive git@github.com:1dayac/Novel-X.git && python novel-x.py run --10x --high-coverage
```

## Evaluation signals

- VCF contains only insertions longer than 300 bp with sum of anchor lengths exceeding 300 bp
- On NA19240-10x, benchmarked precision/recall should approximate Novel-X 59% precision / 37.0% recall vs SuperNova/Paftools 56% / 37.4%, using a match window of ≤100 bp breakpoint difference and ≤5% length difference against PacBio validation callsets
- On CHM1-10x downsampled to 40x, the callset should contain 226 insertions with annotation counts matching the paper: 40 RepeatMasker-repetitive, 24 (k100) and 93 (k50) in non-uniquely mappable regions, 111 inside GENCODE v.24 genes (2 exonic protein-coding)
- On the 68-sample Polaris HiSeq 4000 PGx cohort, the merged callset should yield ~16,691 insertions totaling 18.2 Mb with a longest insertion of 85,908 bp
- Reassembled contigs whose best Minimap2 match does not correspond to an orphan contig must be discarded

## Limitations

- Small insertions often have too few associated barcodes, so coverage is not sufficient for anchor assembly
- Novel-X requires a substantial number of distinct barcodes per insertion candidate (default t=5)
- No dedicated local assembler for SV detection in synthetic long-read data exists; a generic assembler (Velvet/SPAdes) is used, and assembly of other classes of complex SVs is unexplored future work
- A known problem exists on the filter_target_contig stage, and a contamination removal procedure (as in Pamir) may be needed

## Evidence

- [readme] Novel-X detects and genotypes novel sequence insertions in 10X sequencing dataset using non-trivial read alignment signatures and barcode information.: "Novel-X detects and genotypes novel sequence insertions in 10X sequencing dataset using non-trivial read alignment signatures and barcode information."
- [methods] read selection filters: "We choose paired-end reads with at least one end not aligned to the reference genome, or aligned but with a mapping quality below 10, or that have > 20% of soft-clipped bases"
- [methods] output filter: [evidence span withheld — anchor status: not_found]
- [methods] SPAdes parameters: "k=77 (that empirically works better than other standard single k-mer lengths) and cov-cutoff = 3 (to remove low-covered spurious sequences)"
- [methods] barcode threshold: [evidence span withheld — anchor status: not_found]
- [abstract] Polaris cohort result: "Novel-X discovered 16 691 NRS insertions of size > 300 bp (total length 18.2 Mb)."
