---
name: inter-chromosomal-misscaffolding-quantification
description: Use when comparing chromosome-level scaffolded assemblies for inter-chromosomal
  mis-scaffolding, e.g. after RagTag scaffolding of low-coverage HiFi assemblies against
  CHM13 (v2.0), when you need counts of inter-chromosomal links >100 kb.
when_to_use_negative:
- When the goal is to produce a final corrected chromosome-level assembly rather than
  to quantify mis-scaffolding — RagTag's correct function must then be used, but it
  overwrites true SVs and biases the comparison
- When assemblies have not been scaffolded to chromosome level (e.g. raw contigs),
  since inter-chromosomal links cannot be defined
- For plant genomes with higher genetic diversity beyond the two samples tested, where
  the article notes broader evaluation has not yet been performed
edam_operation: http://edamontology.org/operation_0362
edam_topics:
- http://edamontology.org/topic_0196
tools:
- name: RagTag
  role: Scaffolds contig assemblies to chromosome level against CHM13 (v2.0); the
    correct function is deliberately not used so true SVs are not overwritten by the
    reference
- name: NGenomeSyn
  role: Synteny analysis/visualization used to count inter-chromosomal links larger
    than 100 kb (mis-scaffolding events)
- name: minimap2
  role: Aligns scaffolded assemblies to the corresponding reference genome to derive
    chromosome-level scaffold placement (asm-to-asm alignment)
  repo: https://github.com/lh3/minimap2
- name: hifiasm
  role: Produces the contig-level assemblies (Hifiasm vs HiFiCCL-Hifiasm) that are
    scaffolded and compared
  repo: https://github.com/chhylp123/hifiasm
- name: Quast
  role: Provides continuity metrics compiled alongside mis-link counts into the comparison
    table
- name: BUSCO
  role: Provides completeness/continuity metrics compiled alongside mis-link counts
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1002/advs.202515308
    title: Reference‐Guided Chromosome‐by‐Chromosome <i>de novo</i> Assembly at Scale
      Using Low‐Coverage High‐Fidelity Long‐Reads with HiFiCCL
  source_content_hash: sha256:b0212a3cf661b936934f63bc930b739236e073dd50d5bf0850c2d354878acc08
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
  edam_operation: http://edamontology.org/operation_0362
  edam_topics:
  - http://edamontology.org/topic_0196
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# inter-chromosomal-misscaffolding-quantification

## Summary

Quantifies inter-chromosomal mis-scaffolding in chromosome-level scaffolded assemblies by scaffolding with RagTag (omitting the correct step), aligning scaffolds to the reference, and counting inter-chromosomal synteny links >100,000 bp with NGenomeSyn. Used here to show HiFiCCL-Hifiasm reduces mis-scaffolding versus Hifiasm (HG002: 14 vs 189 links, 92.59% reduction).

## When to use

Use when comparing chromosome-level scaffolded assemblies for inter-chromosomal mis-scaffolding, e.g. after RagTag scaffolding of low-coverage HiFi assemblies against CHM13 (v2.0), when you need counts of inter-chromosomal links >100 kb.

## When NOT to use

- When the goal is to produce a final corrected chromosome-level assembly rather than to quantify mis-scaffolding — RagTag's correct function must then be used, but it overwrites true SVs and biases the comparison
- When assemblies have not been scaffolded to chromosome level (e.g. raw contigs), since inter-chromosomal links cannot be defined
- For plant genomes with higher genetic diversity beyond the two samples tested, where the article notes broader evaluation has not yet been performed

## Inputs

- HiFiCCL-Hifiasm and Hifiasm contig-level assemblies (FASTA) for low-coverage datasets, e.g. HG002 (SRR10382244) and NA19240 (SRR14611231)
- Reference genome for scaffolding and alignment, CHM13 (v2.0)
- RagTag-scaffolded assemblies per sample

## Outputs

- Chromosome-level placement of scaffolds from assembly-to-reference alignment
- NGenomeSyn synteny plots with inter-chromosomal links annotated
- Count of inter-chromosomal links >100,000 bp per assembly (mis-scaffolding events)
- scaffolding_mislinks_comparison.tsv with link counts, percent reductions, and continuity metrics

## How to apply

Obtain the assemblies to compare (e.g. HiFiCCL-Hifiasm and Hifiasm results for HG002 and NA19240 ultra-low-coverage datasets from Zenodo DOI 10.5281/zenodo.17204429) and scaffold each with the Ragtag tool using CHM13 (v2.0) as the reference genome, deliberately omitting RagTag's correct function so true structural variants are not overwritten by the reference. Align the scaffolded assemblies to the corresponding reference genome (e.g. with minimap2 asm-to-asm alignment) to derive chromosome-level placement of scaffolds. Perform synteny analysis with NGenomeSyn and count inter-chromosomal links larger than 100 kb, which represent inter-chromosomal mis-scaffolding events. Compare link counts per sample between the two assemblies (HG002: 14 vs 189 links = 92.59% reduction; HG00438: 95.51%/96.77% reduction) and compile counts, reductions, and continuity metrics into a table such as scaffolding_mislinks_comparison.tsv.

## Related tools

- **RagTag** (Scaffolds contig assemblies to chromosome level against CHM13 (v2.0); the correct function is deliberately not used so true SVs are not overwritten by the reference)
- **NGenomeSyn** (Synteny analysis/visualization used to count inter-chromosomal links larger than 100 kb (mis-scaffolding events))
- **minimap2** (Aligns scaffolded assemblies to the corresponding reference genome to derive chromosome-level scaffold placement (asm-to-asm alignment)) — https://github.com/lh3/minimap2
- **hifiasm** (Produces the contig-level assemblies (Hifiasm vs HiFiCCL-Hifiasm) that are scaffolded and compared) — https://github.com/chhylp123/hifiasm
- **Quast** (Provides continuity metrics compiled alongside mis-link counts into the comparison table)
- **BUSCO** (Provides completeness/continuity metrics compiled alongside mis-link counts)

## Examples

```
minimap2 -cx asm5 CHM13v2.fa scaffolded_hificcl_hifiasm.fa > scaffold_placement.paf
```

## Evaluation signals

- HiFiCCL-Hifiasm shows fewer inter-chromosomal links (>100 kb) than Hifiasm for every sample analyzed
- HG002 reduction matches the reported 92.59% (14 vs 189 links) within rounding tolerance
- HG00438 reductions match the reported 95.51%/96.77% range
- RagTag was run without the correct step — verify the workflow explicitly omitted it so true SVs were not overwritten by the reference
- All counts, reductions, and continuity metrics are present in scaffolding_mislinks_comparison.tsv

## Limitations

- RagTag scaffolding relies on reference genome alignments, which may overlook true SVs; only 62.5% of inversion SVs were detected
- Omitting RagTag's correct step is required to preserve true SVs, but leaves scaffolding errors uncorrected by the reference
- Version numbers for HiFiCCL, Hifiasm, and RagTag are not stated, complicating exact reproduction

## Evidence

- [discussion] RagTag correct step omitted: "To mitigate this effect, we chose not to use RagTag's correct function"
- [methods] CHM13 reference for RagTag scaffolding: "using the Ragtag tool, with CHM13 (v2.0) selected as the reference genome"
- [discussion] assemblies compared for HG002: "When using RagTag to scaffold assemblies, such as those generated by HiFiCCL‐Hifiasm and Hifiasm for the HG002 dataset"
- [readme] minimap2 asm-to-asm alignment for scaffold placement: [evidence span withheld — anchor status: external_doc]
