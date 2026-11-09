---
name: complex-sv-simulation
description: Use when benchmarking complex SV (TRA/DUP/INV) detection on PacBio CLR
  or ONT long reads and no truth callset exists; simulated reads with known inserted
  variants are needed to compute recall/precision/F1.
when_to_use_negative:
- Evaluating germline indel SVs against the complete GIAB HG002 gold standard — real
  benchmarking callsets are available and preferable
- The target caller cannot detect complex SVs (e.g. Dipcall, which was not designed
  to detect complex SVs)
- Single-chromosome analysis — translocation detection requires a WGS BAM, so simulation
  and calling must be whole-genome
edam_operation: http://edamontology.org/operation_3557
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_0196
tools:
- name: VISOR
  role: Inserts SNPs and complex SVs (translocations, duplications, inversions) into
    the hg19 reference to create the truth set
- name: PBSIM
  role: Simulates long reads (v3.0.0) for each data type from the mutated reference
- name: VolcanoSV
  role: 'Caller under test: run volcanosv-asm.py + volcanosv-vc-large-indel.py (WGS
    mode, -cl/-px) then volcanosv-vc-complex-sv.py'
  repo: https://github.com/maiziezhoulab/VolcanoSV
- name: SVIM-asm
  role: Assembly-based comparison caller whose complex SV recall is benchmarked against
    VolcanoSV
  repo: https://github.com/eldariont/svim-asm
- name: Truvari
  role: Benchmarks SV calls against the simulated truth with p=0.5, P=0.5, r=500,
    S=30, O=0.01
- name: SURVIVOR
  role: Calls somatic variants from paired normal-tumor VCFs when extending the workflow
    to real cancer data
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-024-51282-0
    title: VolcanoSV enables accurate and robust structural variant calling in diploid
      genomes from single-molecule long read sequencing
  source_content_hash: sha256:fa1db9947c496f68c073302e9ce205f000dd1558f65de9737e4fcf0b91fdcd50
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/maiziezhoulab/VolcanoSV
  tools:
  - DeBreak
  - MAMnet
  - NanoVar
  - PBSIM
  - SURVIVOR
  - SVDSS
  - SVIM-asm
  - Sniffles2
  - Truvari
  - VISOR
  - VolcanoSV
  - cuteSV
  - pbsv
  edam_operation: http://edamontology.org/operation_3557
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_0196
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8125
n_claims: 16
---

# complex-sv-simulation

## Summary

Simulate ground-truth complex structural variants (translocations, duplications, inversions) plus SNPs into a human reference genome and generate matching long reads, enabling controlled benchmarking of complex SV callers such as VolcanoSV against SVIM-asm.

## When to use

Use when benchmarking complex SV (TRA/DUP/INV) detection on PacBio CLR or ONT long reads and no truth callset exists; simulated reads with known inserted variants are needed to compute recall/precision/F1.

## When NOT to use

- Evaluating germline indel SVs against the complete GIAB HG002 gold standard — real benchmarking callsets are available and preferable
- The target caller cannot detect complex SVs (e.g. Dipcall, which was not designed to detect complex SVs)
- Single-chromosome analysis — translocation detection requires a WGS BAM, so simulation and calling must be whole-genome

## Inputs

- hg19 human reference genome FASTA
- VISOR v1.1.2 complex SV insertion config (translocations, duplications, inversions, SNPs)
- PBSIM v3.0.0 simulation settings per data type (PacBio CLR, ONT, Hifi)

## Outputs

- Mutated reference FASTA with known inserted complex SVs and SNPs
- Simulated long reads (FASTQ/BAM) per sequencing platform
- Truvari benchmark metrics (recall, precision, F1, GT_F1) per SV type per caller, e.g. complex_sv_performance.csv

## How to apply

Use VISOR (v1.1.2) to insert SNPs and complex SVs (translocations, duplications, inversions) into the hg19 human reference, then simulate long reads per data type (Hifi/CLR/ONT) with PBSIM (v3.0.0). Align the simulated reads to produce BAM inputs, and run the SV callers on the same simulated BAMs; exclude assembly callers like Dipcall that were not designed to detect complex SVs. Benchmark calls with Truvari (v4.0.0) using p=0.5, P=0.5, r=500, S=30, O=0.01, computing F1 and GT_F1 for inversions and duplications and recall only for translocations (genotyping reciprocal TRAs is ambiguous). Rationale: simulation gives exact breakpoints and genotypes, so recall/precision differences reflect caller capability rather than truth-set incompleteness, unlike incomplete real high-confidence callsets such as HCC1395's.

## Related tools

- **VISOR** (Inserts SNPs and complex SVs (translocations, duplications, inversions) into the hg19 reference to create the truth set)
- **PBSIM** (Simulates long reads (v3.0.0) for each data type from the mutated reference)
- **VolcanoSV** (Caller under test: run volcanosv-asm.py + volcanosv-vc-large-indel.py (WGS mode, -cl/-px) then volcanosv-vc-complex-sv.py) — https://github.com/maiziezhoulab/VolcanoSV
- **SVIM-asm** (Assembly-based comparison caller whose complex SV recall is benchmarked against VolcanoSV) — https://github.com/eldariont/svim-asm
- **Truvari** (Benchmarks SV calls against the simulated truth with p=0.5, P=0.5, r=500, S=30, O=0.01)
- **SURVIVOR** (Calls somatic variants from paired normal-tumor VCFs when extending the workflow to real cancer data)

## Examples

```
VISOR -i hg19.fa -o mutated_ref.fa -c sv_config.yaml; PBSIM --sim-mode QSHMM-RS --accuracy mean --depth 50 mutated_ref.fa; Truvari bench -b truth.vcf -c calls.vcf --pctsize 0.5 --pctsim 0.5 --refdist 500 --sizeseq 30 --pick both
```

## Evaluation signals

- Reproduced metrics confirm VolcanoSV outperforms SVIM-asm in recall across all libraries and complex SV types, with the largest advantage on duplications (12% higher on PacBio CLR, 17% higher on ONT)
- Truvari run completed with the exact parameter set p=0.5, P=0.5, r=500, S=30, O=0.01
- F1 and GT_F1 are reported for INVs and DUPs only; only recall is reported for TRAs (genotyping reciprocal TRAs is ambiguous)
- Per-tool recall/precision/F1/GT_F1 values aggregate into the final performance table (e.g. complex_sv_performance.csv) without missing library/SV-type entries

## Limitations

- Simulated-data recall is optimistic: VolcanoSV's recall for real complex somatic SVs (HCC1395) was relatively lower than on simulated data
- The HCC1395 high-confidence benchmark callset included SV calls only from alignment-based tools, potentially biasing real-data evaluation
- Since the real high-confidence callset is incomplete, only recall can be plotted for HCC1395 somatic SVs
- Assembly algorithms prefer simplest paths through overlapping reads and may miss complex SVs

## Evidence

- [results] VISOR inserts complex SVs into hg19: "We used VISOR (v1.1.2)33 to insert SNPs and complex SVs into the hg19 human reference, and then used PBSIM (v3.0.0)34 to simulate reads"
- [results] Truvari parameters: "The following metric/parameter setting was used in Truvari: p = 0.5, P = 0.5, r = 500, O = 0.01"
- [results] F1 only for INVs/DUPs: "We thus only calculated F1 and GT_F1 for INVs and DUPs"
- [results] Dipcall exclusion: "Dipcall was not involved in this analysis since it was not designed to detect complex SVs"
- [readme] WGS mode for complex SV: "since translocation detection requires WGS BAM file as support, it does not make sense to run it on single chromosome level. Therefore, we only provide the complex SV pipeline in WGS mode."
