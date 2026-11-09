---
name: busco-gene-completeness-assessment
description: Use when comparing assemblies of downsampled (~5x) HiFi human datasets
  — e.g. HiFiCCL-Hifiasm vs hifiasm --primary — to verify that reduced misassembled
  contig length does not cost single-copy ortholog completeness.
when_to_use_negative:
- Input is raw ~5x HiFi reads rather than assembled contig FASTA — BUSCO genome mode
  requires assemblies, which are produced first by the HiFiCCL/hifiasm assembly step
- Evaluating base-level accuracy or k-mer completeness/QV alone — the paper pairs
  BUSCO with Merqury completeness/QV and QUAST misassembled contig length, so BUSCO
  alone is insufficient
- Non-vertebrate genomes (e.g. the rice and Arabidopsis thaliana plant datasets in
  this study) — the vertebrate ortholog database used here is the wrong lineage; a
  taxon-appropriate BUSCO dataset is required
edam_topics:
- http://edamontology.org/topic_0102
tools:
- name: BUSCO
  role: Runs in genome mode with the vertebrate conserved single-copy ortholog database
    (v5.4.7) to score complete/fragmented/missing orthologs for each assembly
- name: hifiasm
  role: Produces the primary contig assemblies (both HiFiCCL-Hifiasm and baseline)
    that BUSCO evaluates; contigs are extracted from the bp.p_ctg.gfa output
  repo: https://github.com/chhylp123/hifiasm
- name: HiFiCCL
  role: Reference-guided chromosome-by-chromosome framework that regenerates hifiasm-based
    assemblies expected to show improved BUSCO values at ~5x coverage
  repo: https://github.com/zjjbuqi/HiFiCCL.git
- name: seqtk
  role: Downsamples the HiFi read sets to ~5x coverage (v1.3-r106) before assembly
    so BUSCO comparisons are made on matched inputs
  repo: https://github.com/lh3/seqtk
- name: QUAST
  role: Provides the paired misassembled contig length, NG50 and NGA50 metrics (v5.2.0)
    against which BUSCO completeness is jointly interpreted
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1002/advs.202515308
    title: Reference‐Guided Chromosome‐by‐Chromosome <i>de novo</i> Assembly at Scale
      Using Low‐Coverage High‐Fidelity Long‐Reads with HiFiCCL
  source_content_hash: sha256:b4403b0c84100b3246192f2b5c01f4cbb3acf68596e9bd15660fa71652aa419b
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/chhylp123/hifiasm
  tools:
  - BUSCO
  - HiFiCCL
  - Hifiasm
  - Pilon
  - QUAST
  - Racon
  - hifiasm
  - minimap2
  - seqtk
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0102
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 10
---

# busco-gene-completeness-assessment

## Summary

Assess gene completeness of de novo assemblies from low-coverage (~5x) HiFi read pipelines (HiFiCCL-Hifiasm vs hifiasm alone) using BUSCO 5.4.7 with the vertebrate conserved single-copy ortholog database, so that misassembly reductions can be judged against ortholog recovery.

## When to use

Use when comparing assemblies of downsampled (~5x) HiFi human datasets — e.g. HiFiCCL-Hifiasm vs hifiasm --primary — to verify that reduced misassembled contig length does not cost single-copy ortholog completeness.

## When NOT to use

- Input is raw ~5x HiFi reads rather than assembled contig FASTA — BUSCO genome mode requires assemblies, which are produced first by the HiFiCCL/hifiasm assembly step
- Evaluating base-level accuracy or k-mer completeness/QV alone — the paper pairs BUSCO with Merqury completeness/QV and QUAST misassembled contig length, so BUSCO alone is insufficient
- Non-vertebrate genomes (e.g. the rice and Arabidopsis thaliana plant datasets in this study) — the vertebrate ortholog database used here is the wrong lineage; a taxon-appropriate BUSCO dataset is required

## Inputs

- Assembled primary contig FASTA from HiFiCCL-Hifiasm (hifiasm --primary backend)
- Assembled primary contig FASTA from baseline hifiasm (0.19.5-r592, -t20 --primary)
- BUSCO 5.4.7 vertebrate conserved single-copy ortholog database (lineage dataset)
- Context: HiFi read sets downsampled to ~5x coverage with seqtk 1.3-r106

## Outputs

- Per-assembly BUSCO complete (single-copy + duplicated), fragmented and missing percentages
- BUSCO deltas between HiFiCCL-Hifiasm and hifiasm baseline assemblies
- Rows in assembly_evaluation_metrics.csv alongside QUAST misassembled-contig-length/NG50/NGA50 and Merqury completeness/QV values

## How to apply

Run BUSCO 5.4.7 in genome mode on the primary contig FASTA produced by each assembler (for hifiasm, extract contigs from the bp.p_ctg.gfa; for HiFiCCL, the contigs of its hifiasm --primary output), using the vertebrate conserved single-copy ortholog database as the lineage. Apply the identical BUSCO configuration to both pipelines on the same seqtk-downsampled ~5x dataset so deltas are comparable. Record complete (single-copy plus duplicated), fragmented and missing ortholog percentages, then compute BUSCO deltas alongside QUAST v5.2.0 metrics (misassembled contig length, NG50, NGA50) and Merqury completeness/QV in step 6 of the workflow. Write all metrics to assembly_evaluation_metrics.csv and accept the HiFiCCL-Hifiasm pipeline only when improved BUSCO values accompany the expected misassembled-contig-length reduction (average 21.19%, up to 38.58%, across the 45 ~5x human datasets).

## Related tools

- **BUSCO** (Runs in genome mode with the vertebrate conserved single-copy ortholog database (v5.4.7) to score complete/fragmented/missing orthologs for each assembly)
- **hifiasm** (Produces the primary contig assemblies (both HiFiCCL-Hifiasm and baseline) that BUSCO evaluates; contigs are extracted from the bp.p_ctg.gfa output) — https://github.com/chhylp123/hifiasm
- **HiFiCCL** (Reference-guided chromosome-by-chromosome framework that regenerates hifiasm-based assemblies expected to show improved BUSCO values at ~5x coverage) — https://github.com/zjjbuqi/HiFiCCL.git
- **seqtk** (Downsamples the HiFi read sets to ~5x coverage (v1.3-r106) before assembly so BUSCO comparisons are made on matched inputs) — https://github.com/lh3/seqtk
- **QUAST** (Provides the paired misassembled contig length, NG50 and NGA50 metrics (v5.2.0) against which BUSCO completeness is jointly interpreted)

## Examples

```
busco -i test.p_ctg.fa -l vertebrata_odb10 -m genome -o busco_hificcl_hifiasm --offline
```

## Evaluation signals

- BUSCO is run identically (v5.4.7, vertebrate single-copy ortholog database, genome mode) on both the HiFiCCL-Hifiasm and hifiasm assemblies of the same seqtk-downsampled ~5x dataset
- HiFiCCL-Hifiasm shows improved BUSCO values relative to the hifiasm baseline together with reduced misassembled contig length (average 21.19%, up to 38.58%)
- All BUSCO metrics are written to assembly_evaluation_metrics.csv alongside QUAST MCL/NG50/NGA50 and Merqury completeness/QV entries
- BUSCO completeness deltas are consistent with Merqury completeness and QUAST NGA50 improvements rather than contradicting them
- Contig FASTA inputs are valid primary contigs (e.g. extracted from hifiasm's bp.p_ctg.gfa with the documented awk one-liner)

## Limitations

- At ~5x coverage many genomic regions remain missing, so BUSCO completeness scores will be depressed relative to high-coverage assemblies and gaps are not resolved by this skill
- BUSCO completeness alone does not detect misassemblies; the paper validates HiFiCCL only when improved BUSCO co-occurs with reduced misassembled contig length (QUAST) and better Merqury/NG50 values
- The study validated the BUSCO-based evaluation on 45 ~5x human (vertebrate) datasets; plant genomes were tested with only two samples, so completeness baselines for higher-diversity taxa are less established

## Evidence

- [methods] Assess gene completeness with BUSCO 5.4.7 using the vertebrate conserved single-copy ortholog database, and completeness/QV with Merqury.: [evidence span withheld — anchor status: not_found]
- [methods] with improved BUSCO, Merqury and NG50 values, and write all metrics to assembly_evaluation_metrics.csv.: [evidence span withheld — anchor status: not_found]
- [readme] awk '/^S/{print ">"$2;print $3}' test.bp.p_ctg.gfa > test.p_ctg.fa  # get primary contigs in FASTA: [evidence span withheld — anchor status: external_doc]
- [discussion] due to low coverage and other limitations, many genomic regions remain missing: "due to low coverage and other limitations, many genomic regions remain missing"
