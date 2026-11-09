---
name: merqury-kmer-quality-evaluation
description: Use when evaluating contig-level assemblies from low-coverage (~5x) PacBio
  HiFi reads — e.g., HiFiCCL-Hifiasm vs Hifiasm outputs — and you need reference-free
  completeness and QV estimates alongside QUAST/BUSCO metrics.
when_to_use_negative:
- Input is a high-coverage (>~25x) HiFi dataset where the base assembler alone (e.g.,
  Hifiasm 0.19.5-r592 --primary) already produces high-quality contigs and the reference-guided
  chromosome-binning pipeline offers no benefit
- The reads are noisy long reads (ONT) rather than high-fidelity HiFi reads — HiFiCCL
  and its evaluation workflow are specifically designed for low-coverage high-fidelity
  reads
- You only need SV detection on a pangenome graph rather than assembly completeness/QV
  evaluation
edam_operation: http://edamontology.org/operation_0525
edam_topics:
- http://edamontology.org/topic_0196
tools:
- name: Merqury
  role: k-mer-based assessment of assembly completeness and consensus QV for HiFiCCL-Hifiasm
    and Hifiasm contigs
- name: BUSCO
  role: complementary gene-completeness check (v5.4.7, vertebrate conserved single-copy
    ortholog database) reported alongside Merqury QV
- name: QUAST
  role: reference-guided contig evaluation (v5.2.0) supplying misassembled contig
    length, NG50 and NGA50 that complement Merqury's reference-free metrics
- name: seqtk
  role: downsampling HiFi read sets to ~5x coverage (1.3-r106) before assembly and
    evaluation
  repo: https://github.com/lh3/seqtk
- name: HiFiCCL
  role: produces the candidate assemblies whose Merqury completeness/QV are evaluated
    against the Hifiasm baseline
  repo: https://github.com/zjjbuqi/HiFiCCL
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1002/advs.202515308
    title: Reference‐Guided Chromosome‐by‐Chromosome <i>de novo</i> Assembly at Scale
      Using Low‐Coverage High‐Fidelity Long‐Reads with HiFiCCL
  source_content_hash: sha256:7c2b9a3e415e69f6338480430d63b6412e42456144441a2bb287143fee269b6e
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/lh3/seqtk
  tools:
  - BUSCO
  - HiFiCCL
  - Hifiasm
  - Merqury
  - QUAST
  - minimap2
  - seqtk
  edam_operation: http://edamontology.org/operation_0525
  edam_topics:
  - http://edamontology.org/topic_0196
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 10
---

# merqury-kmer-quality-evaluation

## Summary

Use Merqury k-mer-based analysis to assess assembly completeness and consensus quality value (QV) for low-coverage HiFi assemblies produced by HiFiCCL-Hifiasm versus Hifiasm alone, as part of the HiFiCCL evaluation pipeline on human (~5x) and plant datasets.

## When to use

Use when evaluating contig-level assemblies from low-coverage (~5x) PacBio HiFi reads — e.g., HiFiCCL-Hifiasm vs Hifiasm outputs — and you need reference-free completeness and QV estimates alongside QUAST/BUSCO metrics.

## When NOT to use

- Input is a high-coverage (>~25x) HiFi dataset where the base assembler alone (e.g., Hifiasm 0.19.5-r592 --primary) already produces high-quality contigs and the reference-guided chromosome-binning pipeline offers no benefit
- The reads are noisy long reads (ONT) rather than high-fidelity HiFi reads — HiFiCCL and its evaluation workflow are specifically designed for low-coverage high-fidelity reads
- You only need SV detection on a pangenome graph rather than assembly completeness/QV evaluation

## Inputs

- Primary contig FASTA assemblies from HiFiCCL-Hifiasm and Hifiasm (--primary) on ~5x HiFi datasets (e.g., HG002 SRR10382244, NA19240 SRR14611231)
- k-mer count sets derived from the HiFi read sets (Merqury meryl databases)
- Reference assemblies for complementary QUAST evaluation (CHM13 v2.0/v1.0, GRCh38)

## Outputs

- Merqury completeness estimates per assembly
- Merqury consensus QV (quality value) per assembly
- Delta completeness/QV values between HiFiCCL-Hifiasm and Hifiasm baselines recorded in assembly_evaluation_metrics.csv

## How to apply

After assembling the ~5x downsampled HiFi read sets (produced with seqtk 1.3-r106) with either HiFiCCL (hificcl.py -f <reads> -r CHM13v2.0.fasta -t 20) or Hifiasm 0.19.5-r592 (-t20 --primary), run Merqury to estimate assembly completeness and consensus QV from k-mer sets, complementing QUAST v5.2.0 contig metrics (contig size, number of contigs, misassembled contig length, NG50, NGA50) and BUSCO 5.4.7 (vertebrate conserved single-copy ortholog database). Record the Merqury completeness and QV values for both pipelines into assembly_evaluation_metrics.csv and compute deltas between HiFiCCL-Hifiasm and Hifiasm baselines. Judging correct application: HiFiCCL-Hifiasm assemblies should show improved Merqury completeness/QV together with an average misassembled-contig-length reduction of 21.19% (up to 38.58%) across the 45 ~5x human datasets, and improved BUSCO and NG50 values.

## Related tools

- **Merqury** (k-mer-based assessment of assembly completeness and consensus QV for HiFiCCL-Hifiasm and Hifiasm contigs)
- **BUSCO** (complementary gene-completeness check (v5.4.7, vertebrate conserved single-copy ortholog database) reported alongside Merqury QV)
- **QUAST** (reference-guided contig evaluation (v5.2.0) supplying misassembled contig length, NG50 and NGA50 that complement Merqury's reference-free metrics)
- **seqtk** (downsampling HiFi read sets to ~5x coverage (1.3-r106) before assembly and evaluation) — https://github.com/lh3/seqtk
- **HiFiCCL** (produces the candidate assemblies whose Merqury completeness/QV are evaluated against the Hifiasm baseline) — https://github.com/zjjbuqi/HiFiCCL

## Evaluation signals

- HiFiCCL-Hifiasm assemblies show Merqury completeness/QV values at least as good as, and improved relative to, the Hifiasm-alone baseline across the 45 ~5x human datasets
- MCL (misassembled contig length, from QUAST) is reduced by an average of 21.19% and up to 38.58% versus Hifiasm; per-dataset deltas (e.g., HG002: 331.84 -> 274.16 Mb, 17.38%; NA19240: 245.96 -> 179.15 Mb, 27.16%) match expectations
- BUSCO 5.4.7 (vertebrate conserved single-copy ortholog database) completeness deltas are consistent with the Merqury completeness direction
- All metrics (QUAST contig metrics, BUSCO, Merqury completeness/QV, wall time, max RSS) are recorded in assembly_evaluation_metrics.csv
- Same downsampled ~5x read sets (seqtk 1.3-r106) and assembler threads (-t 20) are used for both pipelines so QV/completeness deltas are attributable to the assembly method

## Limitations

- Due to low coverage and other limitations, many genomic regions remain missing at ~5x, so completeness metrics cannot reach near-complete values even for correct assemblies
- Merqury completeness/QV must be interpreted jointly with reference-guided QUAST metrics; reference-based scaffolding (RagTag) may overlook true SVs (only 62.5% of inversion SVs detected), so QV alone does not certify structural correctness
- Plant genomes were evaluated on only two samples, so Merqury-based conclusions for high-diversity plant assemblies are less well supported than for human datasets

## Evidence

- [methods] Merqury completeness/QV used in the evaluation workflow: [evidence span withheld — anchor status: not_found]
- [abstract] Expected improvement pattern: "HiFiCCL improves low-coverage assembly performance of existing assemblers and outperforms the state-of-the-art assemblers on human and plant datasets"
- [discussion] Missing regions at low coverage: "due to low coverage and other limitations, many genomic regions remain missing"
- [readme] Extracting primary contigs for evaluation from hifiasm output: [evidence span withheld — anchor status: external_doc]
