---
name: assembly-misassembly-quantification
description: Use when comparing two or more assemblies (e.g., HiFiCCL-Hifiasm vs hifiasm
  on ~5x HiFi human reads) and you must quantify misassembled contig length reduction,
  NG50/NGA50, BUSCO and Merqury deltas against a reference genome.
when_to_use_negative:
- You are evaluating scaffolding (chromosome-level placement) rather than contig-level
  misassembly — RagTag scaffolding relies on reference alignments and may overlook
  true SVs (only 62.5% of inversion SVs detected), so do not conflate scaffolding
  quality with MCL.
- The input is raw reads rather than assembled contigs — this skill quantifies misassemblies
  of assembly outputs, not read quality or coverage.
- The genome is a high-diversity plant genome with only limited validated benchmarks
  — the article tested only two plant samples, so MCL reduction expectations (average
  21.19%) are calibrated on 45 ~5x human datasets.
edam_operation: http://edamontology.org/operation_0525
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_0080
tools:
- name: QUAST
  role: Reference-based assembly evaluation producing misassembled contig length,
    NG50, NGA50, contig count and size (v5.2.0)
- name: HiFiCCL
  role: Reference-guided chromosome-by-chromosome assembly framework whose output
    assemblies are quantified for MCL reduction
  repo: https://github.com/zjjbuqi/HiFiCCL.git
- name: hifiasm
  role: Baseline (-t20 --primary) and embedded (--primary) assembler producing .bp.p_ctg.gfa
    primary contigs evaluated by QUAST
  repo: https://github.com/chhylp123/hifiasm
- name: seqtk
  role: Downsamples HiFi read sets to ~5x coverage before assembly (v1.3-r106)
  repo: https://github.com/lh3/seqtk
- name: minimap2
  role: Assembly-to-assembly alignment (asm5 preset) as an auxiliary check of contig
    agreement between pipelines
  repo: https://github.com/lh3/minimap2
- name: BUSCO
  role: Gene completeness assessment (v5.4.7, vertebrate conserved single-copy ortholog
    database) complementing MCL metrics
- name: Merqury
  role: k-mer-based completeness and QV estimation complementing MCL metrics
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1002/advs.202515308
    title: Reference‐Guided Chromosome‐by‐Chromosome <i>de novo</i> Assembly at Scale
      Using Low‐Coverage High‐Fidelity Long‐Reads with HiFiCCL
  source_content_hash: sha256:5df0c6146828a248cdf3a153fcf84cef9879505cfdd1569f4fb22ba9357db882
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/zjjbuqi/HiFiCCL.git
  tools:
  - BUSCO
  - HiFiCCL
  - Hifiasm
  - Merqury
  - QUAST
  - hifiasm
  - minimap2
  - seqtk
  edam_operation: http://edamontology.org/operation_0525
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_0080
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9090909090909091
n_claims: 11
---

# assembly-misassembly-quantification

## Summary

Quantifies misassembly burden of genome assemblies (specifically misassembled contig length, MCL) by evaluating contigs with QUAST v5.2.0 against a high-quality reference, then computing percentage reductions (e.g., HiFiCCL-Hifiasm vs hifiasm alone at ~5x HiFi coverage).

## When to use

Use when comparing two or more assemblies (e.g., HiFiCCL-Hifiasm vs hifiasm on ~5x HiFi human reads) and you must quantify misassembled contig length reduction, NG50/NGA50, BUSCO and Merqury deltas against a reference genome.

## When NOT to use

- You are evaluating scaffolding (chromosome-level placement) rather than contig-level misassembly — RagTag scaffolding relies on reference alignments and may overlook true SVs (only 62.5% of inversion SVs detected), so do not conflate scaffolding quality with MCL.
- The input is raw reads rather than assembled contigs — this skill quantifies misassemblies of assembly outputs, not read quality or coverage.
- The genome is a high-diversity plant genome with only limited validated benchmarks — the article tested only two plant samples, so MCL reduction expectations (average 21.19%) are calibrated on 45 ~5x human datasets.

## Inputs

- Primary contig assemblies in FASTA or GFA format (e.g., .bp.p_ctg.gfa / .p_ctg.fa from hifiasm or HiFiCCL-hifiasm)
- High-quality reference genome FASTA (e.g., CHM13 v2.0, CHM13 v1.0, or GRCh38)
- Low-coverage (~5x) PacBio HiFi read sets (FASTQ/FASTA, downsampled with seqtk 1.3-r106)
- Vertebrate conserved single-copy ortholog database for BUSCO 5.4.7

## Outputs

- QUAST v5.2.0 report with contig size, number of contigs, misassembled contig length, NG50 and NGA50 per assembly
- Misassembled-contig-length (MCL) reduction percentages between baseline and treatment assemblies
- BUSCO and Merqury (completeness/QV) metrics per assembly
- assembly_evaluation_metrics.csv containing all assembly evaluation metrics plus runtime (wall time) and maximum resident set size

## How to apply

Assemble the low-coverage (~5x, seqtk-downsampled) HiFi reads with each pipeline (e.g., HiFiCCL in default mode via hificcl.py -f <reads> -r CHM13v2.0.fasta -t 20, which invokes hifiasm --primary, versus hifiasm 0.19.5-r592 -t20 --primary alone), extracting primary contigs from the .bp.p_ctg.gfa outputs to FASTA. Run QUAST v5.2.0 with the reference genome (e.g., CHM13 v2.0 or GRCh38) to obtain contig size, number of contigs, misassembled contig length, NG50 and NGA50 for each assembly. Compute the MCL reduction percentage as (baseline MCL − treatment MCL)/baseline MCL × 100 — e.g., 17.38% on HG002 (331.84 to 274.16 Mb) and 27.16% on NA19240 (245.96 to 179.15 Mb). Complement the contiguity/misassembly metrics with BUSCO 5.4.7 (vertebrate conserved single-copy ortholog database) and Merqury completeness/QV, and record wall time and maximum resident set size per pipeline. Write all deltas and reduction percentages to assembly_evaluation_metrics.csv.

## Related tools

- **QUAST** (Reference-based assembly evaluation producing misassembled contig length, NG50, NGA50, contig count and size (v5.2.0))
- **HiFiCCL** (Reference-guided chromosome-by-chromosome assembly framework whose output assemblies are quantified for MCL reduction) — https://github.com/zjjbuqi/HiFiCCL.git
- **hifiasm** (Baseline (-t20 --primary) and embedded (--primary) assembler producing .bp.p_ctg.gfa primary contigs evaluated by QUAST) — https://github.com/chhylp123/hifiasm
- **seqtk** (Downsamples HiFi read sets to ~5x coverage before assembly (v1.3-r106)) — https://github.com/lh3/seqtk
- **minimap2** (Assembly-to-assembly alignment (asm5 preset) as an auxiliary check of contig agreement between pipelines) — https://github.com/lh3/minimap2
- **BUSCO** (Gene completeness assessment (v5.4.7, vertebrate conserved single-copy ortholog database) complementing MCL metrics)
- **Merqury** (k-mer-based completeness and QV estimation complementing MCL metrics)

## Examples

```
quast.py -r CHM13v2.0.fasta -t 20 -o quast_out HG002.hificcl_hifiasm.p_ctg.fa HG002.hifiasm.p_ctg.fa
```

## Evaluation signals

- QUAST report.tsv contains a misassembled contig length value (in Mb) for every assembly compared; the MCL reduction (baseline − HiFiCCL)/baseline × 100 must be positive, averaging ~21.19% and up to 38.58% across the 45 ~5x human datasets.
- Spot-check expected magnitudes: HG002 MCL drops 331.84 → 274.16 Mb (17.38%) and NA19240 drops 245.96 → 179.15 Mb (27.16%).
- NG50 and NGA50 values in the QUAST report must be reported alongside MCL, and HiFiCCL-Hifiasm should show improved NG50 versus hifiasm alone.
- BUSCO (vertebrate orthologs) and Merqury completeness/QV deltas should be recorded per assembly in assembly_evaluation_metrics.csv; improvements should be directionally consistent with the MCL reduction.
- Wall time and maximum resident set size must be present for both pipelines so the runtime ratio can be computed.

## Limitations

- At ~5x coverage many genomic regions remain missing, so low MCL does not imply completeness — completeness gaps are unresolved at low coverage.
- Reference-based evaluation (QUAST against CHM13/GRCh38) and reference-guided assembly itself can bias against true structural variants; scaffolding with RagTag overlooks true SVs (only 62.5% of inversion SVs detected).
- MCL reduction benchmarks were established on 45 ~5x human datasets and only two plant samples; plant-genome expectations are less validated.
- No version numbers were stated for HiFiCCL, hifiasm (in some contexts), or RagTag, so exact reproduction may vary.

## Evidence

- [abstract] reduces the length of misassembled contigs relative to hifiasm by an average of 21.19% and up to 38.58%: "reduces the length of misassembled contigs relative to hifiasm by an average of 21.19% and up to 38.58%"
- [methods] Evaluate contigs with QUAST v5.2.0 against high-quality published assemblies to obtain contig size, number of contigs, misassembled contig length, NG50 and NGA50.: [evidence span withheld — anchor status: not_found]
- [readme] awk '/^S/{print ">"$2;print $3}' test.bp.p_ctg.gfa > test.p_ctg.fa  # get primary contigs in FASTA: [evidence span withheld — anchor status: external_doc]
- [readme] ./minimap2 -cx asm5 asm1.fa asm2.fa > aln.paf             # intra-species asm-to-asm alignment: [evidence span withheld — anchor status: external_doc]
- [discussion] due to low coverage and other limitations, many genomic regions remain missing: "due to low coverage and other limitations, many genomic regions remain missing"
