---
name: genome-scaffolding-with-reference-alignment
description: Use when existing contig assemblies (e.g., hifiasm output from ultra-low-coverage
  HG002/NA19240 HiFi reads) must be placed onto chromosomes against CHM13 (v2.0) and
  inter-chromosomal mis-scaffolding must be counted and compared between assemblers.
when_to_use_negative:
- 'When the goal is de novo structural variant discovery from the scaffolds: the article
  notes reference-guided scaffolding may overlook true SVs (only 62.5% of inversion
  SVs detected), and RagTag''s ''correct'' function is deliberately avoided here for
  that reason'
- When input is raw reads rather than an existing contig-level assembly — run the
  HiFiCCL chromosome-by-chromosome assembly workflow first
- When a reference-biased, corrected chromosome placement is acceptable — this skill
  intentionally skips RagTag's correct step, so scaffold positions preserve true SVs
  instead of matching the reference
edam_operation: http://edamontology.org/operation_0310
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_0102
tools:
- name: RagTag
  role: Scaffolds HiFiCCL-Hifiasm and Hifiasm contigs against CHM13 (v2.0); its 'correct'
    function is deliberately not used so true SVs are not overwritten
- name: NGenomeSyn
  role: Synteny analysis/visualization used to count inter-chromosomal links >100,000
    bp as mis-scaffolding events
- name: minimap2
  role: Assembly-to-assembly alignment of scaffolded assemblies against the reference
    to derive chromosome-level scaffold placement
  repo: https://github.com/lh3/minimap2
- name: hifiasm
  role: Produces the primary contig assemblies (HiFiCCL-Hifiasm vs Hifiasm) that are
    scaffolded in this comparison
  repo: https://github.com/chhylp123/hifiasm
- name: Quast
  role: Continuity/misassembly metrics compiled alongside link counts into scaffolding_mislinks_comparison.tsv
- name: BUSCO
  role: Completeness metric compiled with the mis-scaffolding comparison
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1002/advs.202515308
    title: Reference‐Guided Chromosome‐by‐Chromosome <i>de novo</i> Assembly at Scale
      Using Low‐Coverage High‐Fidelity Long‐Reads with HiFiCCL
  source_content_hash: sha256:21ad5a6550f60c6b6f7811231a759c38daef2d6191482f8284af09ede37e11e2
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/lh3/minimap2
  tools:
  - BUSCO
  - HiFiCCL
  - NGenomeSyn
  - Quast
  - RagTag
  - hifiasm
  - minimap2
  edam_operation: http://edamontology.org/operation_0310
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_0102
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8125
n_claims: 16
---

# genome-scaffolding-with-reference-alignment

## Summary

Scaffold contig-level assemblies (e.g., HiFiCCL-Hifiasm vs Hifiasm primary contigs from low-coverage HiFi data) to chromosome level with RagTag against CHM13 (v2.0), deliberately skipping RagTag's 'correct' step, then quantify inter-chromosomal mis-scaffolding as inter-chromosomal synteny links >100,000 bp with NGenomeSyn.

## When to use

Use when existing contig assemblies (e.g., hifiasm output from ultra-low-coverage HG002/NA19240 HiFi reads) must be placed onto chromosomes against CHM13 (v2.0) and inter-chromosomal mis-scaffolding must be counted and compared between assemblers.

## When NOT to use

- When the goal is de novo structural variant discovery from the scaffolds: the article notes reference-guided scaffolding may overlook true SVs (only 62.5% of inversion SVs detected), and RagTag's 'correct' function is deliberately avoided here for that reason
- When input is raw reads rather than an existing contig-level assembly — run the HiFiCCL chromosome-by-chromosome assembly workflow first
- When a reference-biased, corrected chromosome placement is acceptable — this skill intentionally skips RagTag's correct step, so scaffold positions preserve true SVs instead of matching the reference

## Inputs

- Contig-level assembly FASTA files (HiFiCCL-Hifiasm and Hifiasm primary contigs from ultra-low-coverage HG002/NA19240 HiFi datasets)
- Reference genome FASTA: CHM13 (v2.0) (GRCh38 also used as a reference in the article)
- Assemblies retrieved from Zenodo deposit DOI: 10.5281/zenodo.17204429

## Outputs

- Chromosome-level scaffolded assemblies (RagTag scaffold output FASTA, produced without the 'correct' step)
- Scaffold-to-reference alignments (PAF/SAM) giving chromosome-level placement of scaffolds
- NGenomeSyn synteny plots with inter-chromosomal links annotated
- Count of inter-chromosomal links >100,000 bp per sample/assembler, plus Quast/BUSCO continuity metrics, compiled in scaffolding_mislinks_comparison.tsv

## How to apply

Obtain the HiFiCCL-Hifiasm and Hifiasm assemblies from the Zenodo deposit (DOI: 10.5281/zenodo.17204429) for each ultra-low-coverage sample, then scaffold each with the RagTag tool against CHM13 (v2.0) as the reference genome while deliberately omitting RagTag's 'correct' step so that true structural variants are not overwritten by the reference. Align the scaffolded assemblies to the corresponding reference (e.g., minimap2 asm-to-asm alignment) to derive chromosome-level placement of scaffolds. Perform synteny analysis with NGenomeSyn and count inter-chromosomal links larger than 100 kb, which represent inter-chromosomal mis-scaffolding events. Compare link counts per sample — for HG002, HiFiCCL-Hifiasm had 14 links versus 189 for Hifiasm (92.59% reduction; HG00438 showed 95.51%/96.77% reductions) — and compile counts, reductions, and continuity metrics (Quast, BUSCO) into scaffolding_mislinks_comparison.tsv.

## Related tools

- **RagTag** (Scaffolds HiFiCCL-Hifiasm and Hifiasm contigs against CHM13 (v2.0); its 'correct' function is deliberately not used so true SVs are not overwritten)
- **NGenomeSyn** (Synteny analysis/visualization used to count inter-chromosomal links >100,000 bp as mis-scaffolding events)
- **minimap2** (Assembly-to-assembly alignment of scaffolded assemblies against the reference to derive chromosome-level scaffold placement) — https://github.com/lh3/minimap2
- **hifiasm** (Produces the primary contig assemblies (HiFiCCL-Hifiasm vs Hifiasm) that are scaffolded in this comparison) — https://github.com/chhylp123/hifiasm
- **Quast** (Continuity/misassembly metrics compiled alongside link counts into scaffolding_mislinks_comparison.tsv)
- **BUSCO** (Completeness metric compiled with the mis-scaffolding comparison)

## Examples

```
minimap2 -cx asm5 CHM13v2.0.fa ragtag.scaffold.fasta > scaffolds_to_ref.paf
```

## Evaluation signals

- HiFiCCL-Hifiasm scaffolds show fewer inter-chromosomal links (>100,000 bp) than Hifiasm for every sample
- HG002 link counts reproduce 14 vs 189 (92.59% reduction) within rounding tolerance
- HG00438 reproduces the reported ~95.51%/96.77% reductions
- RagTag run logs confirm the 'correct' step was skipped (only the scaffold module executed), per the article's rationale for preserving true SVs
- scaffolding_mislinks_comparison.tsv contains per-sample link counts, percent reductions, and Quast/BUSCO continuity metrics

## Limitations

- RagTag scaffolding relies on reference genome alignments, which may overlook true SVs — only 62.5% of inversion SVs were detected in the article's discussion of this effect
- No version numbers stated for RagTag (or HiFiCCL/Hifiasm), complicating exact reproduction
- At ~5x coverage many genomic regions remain missing, so completeness gaps coexist with the scaffolding comparison

## Evidence

- [methods] using the Ragtag tool, with CHM13 (v2.0) selected as the reference genome: "using the Ragtag tool, with CHM13 (v2.0) selected as the reference genome"
- [discussion] To mitigate this effect, we chose not to use RagTag's correct function: "To mitigate this effect, we chose not to use RagTag's correct function"
- [abstract] These improved assemblies excel in detecting large germline structural variants, minimize inter-chromosome mis-scaffolding: "These improved assemblies excel in detecting large germline structural variants, minimize inter-chromosome mis-scaffolding"
- [discussion] the scaffolding process relies on reference genome alignments, which may overlook true SVs: "the scaffolding process relies on reference genome alignments, which may overlook true SVs"
