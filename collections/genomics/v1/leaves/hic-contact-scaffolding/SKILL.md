---
name: hic-contact-scaffolding
description: Use when draft contigs (original or split at CSE misjoin breakpoints)
  must be assembled into chromosome-scale scaffolds with Hi-C contact maps, and when
  comparing scaffold quality before vs. after chimeric-error correction via N50, R-AQI,
  and S-AQI.
when_to_use_negative:
- The assembly already has Hi-C reads or contact evidence showing the contig joins
  are correct, so CRAQ CSE splitting and rescaffolding would remove true joins rather
  than misjoins
- 'Only regional/base-level accuracy is at stake: Hi-C rescaffolding leaves R-AQI
  nearly unchanged (78.7 vs 79.1 in A. oxysepala), so CRE-driven improvement needs
  sequence-level correction instead'
- No Hi-C sequencing data are available for the sample — the scaffolding step cannot
  be run de novo from contact maps
edam_topics:
- http://edamontology.org/topic_0196
- http://edamontology.org/topic_0622
tools:
- name: 3D-DNA
  role: De novo Hi-C scaffolding of original and CRAQ-corrected contigs with '-m haploid
    -r 0' (no internal error correction)
- name: Juicer
  role: Processing raw MboI Hi-C reads into valid contact maps (.hic) that drive 3D-DNA
    scaffolding
- name: CRAQ
  role: Detects CSE misjoin breakpoints and splits contigs (with --break|-b T) so
    that corrected, not chimeric, contigs are scaffolded
  repo: https://github.com/JiaoLaboratory/CRAQ
- name: Bionano RefAligner / Solve
  role: Aligns split contigs to DLE-1 optical consensus maps to confirm split assignments
    (ctg8_1→CMAP-1, ctg8_2→CMAP-10) and count conflicts
- name: Minimap2
  role: Maps NGS short reads ('-ax sr') and SMS long reads ('map-pb'/'map-hifi') back
    to contigs so CRAQ can call the CSE breakpoints used for splitting
  repo: https://github.com/lh3/minimap2
- name: SAMtools
  role: Filters alignments with '-F 1796 -q 20' before CRAQ error calling
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41467-023-42336-w
    title: Identification of errors in draft genome assemblies at single-nucleotide
      resolution for quality assessment and improvement
  source_content_hash: sha256:b3002416eb6b5416e954c5c9e908531fe3846077830e69d05f6a845c7e7dd5d9
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/JiaoLaboratory/CRAQ
  tools:
  - 3D-DNA
  - Bionano RefAligner / Solve
  - CRAQ
  - Integrative Genomics Viewer (IGV)
  - Juicer
  - Minimap2
  - SAMtools
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0196
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9090909090909091
n_claims: 11
---

# hic-contact-scaffolding

## Summary

Order, orient, and join draft or CRAQ-corrected genome contigs into chromosome-scale scaffolds using Hi-C contact maps (Juicer + 3D-DNA de novo scaffolding), then quantify the structural quality gain with N50 and CRAQ's R-AQI/S-AQI scores.

## When to use

Use when draft contigs (original or split at CSE misjoin breakpoints) must be assembled into chromosome-scale scaffolds with Hi-C contact maps, and when comparing scaffold quality before vs. after chimeric-error correction via N50, R-AQI, and S-AQI.

## When NOT to use

- The assembly already has Hi-C reads or contact evidence showing the contig joins are correct, so CRAQ CSE splitting and rescaffolding would remove true joins rather than misjoins
- Only regional/base-level accuracy is at stake: Hi-C rescaffolding leaves R-AQI nearly unchanged (78.7 vs 79.1 in A. oxysepala), so CRE-driven improvement needs sequence-level correction instead
- No Hi-C sequencing data are available for the sample — the scaffolding step cannot be run de novo from contact maps

## Inputs

- Draft contig assembly in FASTA (e.g., A.oxysepala_contig_draft.fa)
- CRAQ-corrected contigs split at CSE breakpoints in FASTA (out_correct.fa, e.g., A.oxysepala_contig_CRAQcorrect.fa)
- Raw paired-end Hi-C reads (MboI-digested)
- chrom.sizes and MboI cut-sites file for Juicer
- Bionano DLE-1 optical consensus maps (CMAPs) for post-hoc structural validation
- CRAQ report (out_final.Report) with CSE coordinates and AQI metrics

## Outputs

- Chromosome-scale scaffold FASTAs for original and corrected contigs (e.g., A.oxysepala_original.scaffold.hic.fas and A.oxysepala_corrected.scaffold.hic.fas)
- Hi-C interaction (.hic) file and contig order/orientation (.assembly) file loadable into Juicebox
- Comparison table craq_correction_metrics.tsv with N50, R-AQI, S-AQI per scaffold version
- ctg8/CMAP-1/CMAP-10 optical-map confirmation record and conflict counts before vs. after correction

## How to apply

Run de novo Hi-C scaffolding separately on the original and the CRAQ-corrected contig sets: process raw Hi-C reads with Juicer ('-s MboI -d juicer -p chrom.sizes -y cut-sites.txt') and run 3D-DNA with '-m haploid -r 0' and no internal error correction, splitting the mega-scaffold only at large-scale diagonal discrepancies in the contact map. The rationale is that CRAQ splitting at CSE breakpoints (e.g., the misjoin at contig8:1,874,290) removes chimeric joins that Hi-C would otherwise be forced to honor, so corrected contigs scaffold into higher-quality chromosomes. Validate the correction structurally by aligning the split contigs (ctg8_1, ctg8_2) back to Bionano DLE-1 optical consensus maps with RefAligner in the Bionano Solve pipeline, confirming assignment to CMAP-1 and CMAP-10 respectively and counting optical-map conflicts before and after. Extract N50, R-AQI, and S-AQI from the CRAQ report (out_final.Report) and scaffold statistics for both scaffold versions, and tabulate them in craq_correction_metrics.tsv. Judge success by a large S-AQI and N50 gain with R-AQI essentially unchanged, since Hi-C rescaffolding fixes structural joins but not regional base-level accuracy.

## Related tools

- **3D-DNA** (De novo Hi-C scaffolding of original and CRAQ-corrected contigs with '-m haploid -r 0' (no internal error correction))
- **Juicer** (Processing raw MboI Hi-C reads into valid contact maps (.hic) that drive 3D-DNA scaffolding)
- **CRAQ** (Detects CSE misjoin breakpoints and splits contigs (with --break|-b T) so that corrected, not chimeric, contigs are scaffolded) — https://github.com/JiaoLaboratory/CRAQ
- **Bionano RefAligner / Solve** (Aligns split contigs to DLE-1 optical consensus maps to confirm split assignments (ctg8_1→CMAP-1, ctg8_2→CMAP-10) and count conflicts)
- **Minimap2** (Maps NGS short reads ('-ax sr') and SMS long reads ('map-pb'/'map-hifi') back to contigs so CRAQ can call the CSE breakpoints used for splitting) — https://github.com/lh3/minimap2
- **SAMtools** (Filters alignments with '-F 1796 -q 20' before CRAQ error calling)

## Examples

```
juicer.sh -s MboI -d juicer -p chrom.sizes -y cut-sites.txt A.oxysepala_corrected_contigs; run-3d-dna.sh -m haploid -r 0 A.oxysepala_corrected.scaffold.hic.fas
```

## Evaluation signals

- Corrected-scaffold N50 ≈ 28 Mb vs. ≈ 20 Mb for scaffolds built directly from the draft contigs
- S-AQI improves substantially (58.1 vs. 39.0) while R-AQI stays nearly unchanged (78.7 vs. 79.1)
- CSE count in the vicinity of the reported ~117, including the breakpoint at contig8:1,874,290
- RefAligner maps ctg8_1 to CMAP-1 and ctg8_2 to CMAP-10, with fewer optical-map conflicts after correction
- Both .hic and .assembly files for original and corrected scaffolds load in Juicebox and show anchoring of the split contigs

## Limitations

- Optical-mapping validation only infers the approximate genomic locations of misjoins, not exact breakpoints
- Hi-C rescaffolding improves structural quality (S-AQI, N50) but regional accuracy (R-AQI) is essentially unchanged, so base-level errors are not addressed
- Mega-scaffold splitting in 3D-DNA must be done manually at large-scale diagonal discrepancies, introducing a subjective judgment step
- CRAQ's underlying error classification is user-tunable (e.g., the '-d' coverage-discrepancy cutoff has no stated universal value) and accuracy for polyploid genomes is not established

## Evidence

- [intro] CRAQ offers the option to correct conflicting contigs by breaking them at relevant error breakpoints; optical maps or Hi-C can then be integrated to fix such errors and improve the assembly.: "CRAQ offers the option to correct conflicting contigs by breaking them at relevant error breakpoints; optical maps or Hi-C can then be integrated to fix such errors"
- [readme] 3D-DNA pipeline (without correction: -r 0) was performed for the De novo scaffolding process.: [evidence span withheld — anchor status: external_doc]
- [discussion] The optical mapping-based approach could only infer the approximate genomic locations of misjoins.: "The optical mapping-based approach could only infer the approximate genomic locations of misjoins."
- [discussion] The normalized CRE and CSE numbers were then transformed into the R-AQI and S-AQI scores, respectively.: "The normalized CRE and CSE numbers were then transformed into the R-AQI and S-AQI scores, respectively."
