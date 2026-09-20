---
name: peak-restricted-methylation-quantification
description: Use when comparing CpG methylation estimates from nanopore methylation
  callers against WGBS or CUT&Tag-BS truth data, requiring per-CpG aggregation restricted
  to defined peak regions with a minimum coverage depth.
when_to_use_negative:
- When profiling 6mA-containing regions rather than endogenous CpG methylation — use
  the 6mA-calling module with input.peak.bed instead
- When CpG coverage depth is below 20x in either WGBS or nanoHiMe-seq data, since
  the correlation analysis explicitly requires dual coverage >=20
- When no ChIP-seq peak definitions (e.g. ENCFF546TJV, ENCFF950VUB) are available
  and genome-wide aggregation is desired instead
edam_operation: http://edamontology.org/operation_3208
edam_topics:
- http://edamontology.org/topic_3673
- http://edamontology.org/topic_3295
tools:
- name: nanoHiMe
  role: HMM-based mCG calling from nanopolish eventalign files, classifying CpGs by
    log likelihood ratio thresholds (+1.5 / -1.5)
  repo: github.com/YinLabTJ/nanoHiMe
- name: Megalodon
  role: ONT methylation caller used as a comparator, 5mC in CpG context with model
    res_dna_r941_min_modbases-all-context_v001.cfg (v2.5.0)
- name: nanopolish
  role: v0.13.2; produces event-level alignments via 'nanopolish eventalign -n -t
    16 --scale-events' that feed nanoHiMe mCG calling, and provides comparator 5mC
    calls
- name: Minimap2
  role: v2.17; aligns basecalled nanopore reads to hg19 in map-ont mode with 8 threads
    before methylation calling
- name: Samtools
  role: v1.9; sorts, indexes, and filters mapped reads with mapping quality >=20 via
    'samtools view -b -q 20 -F 4'
- name: Guppy
  role: v4.4.2; basecalls FAST5 signal data with the dna_r9.4.1_450bps_hac high-accuracy
    model prior to alignment
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41467-022-35650-2
    title: Simultaneous profiling of histone modifications and DNA methylation via
      nanopore sequencing
  source_content_hash: sha256:1a242becf8e19e99865d1f052ab5e97d943c1fd8504c094823a04f874667f937
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Guppy
  - Megalodon
  - Minimap2
  - Samtools
  - nanoHiMe
  - nanopolish
  edam_operation: http://edamontology.org/operation_3208
  edam_topics:
  - http://edamontology.org/topic_3673
  - http://edamontology.org/topic_3295
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5384615384615384
n_claims: 13
---

# peak-restricted-methylation-quantification

## Summary

Aggregates per-CpG methylation calls from nanopore (nanoHiMe, nanopolish, Megalodon), WGBS, and CUT&Tag-BS within H3K27me3 ChIP-seq peak regions, restricting to CpGs covered >=20x, to benchmark methylation callers against gold-standard WGBS via Pearson correlation.

## When to use

Use when comparing CpG methylation estimates from nanopore methylation callers against WGBS or CUT&Tag-BS truth data, requiring per-CpG aggregation restricted to defined peak regions with a minimum coverage depth.

## When NOT to use

- When profiling 6mA-containing regions rather than endogenous CpG methylation — use the 6mA-calling module with input.peak.bed instead
- When CpG coverage depth is below 20x in either WGBS or nanoHiMe-seq data, since the correlation analysis explicitly requires dual coverage >=20
- When no ChIP-seq peak definitions (e.g. ENCFF546TJV, ENCFF950VUB) are available and genome-wide aggregation is desired instead

## Inputs

- nanoHiMe mCG mode output (output.mCG / output.mCG.eventalign.txt) with per-read Log(LIKELIHOOD_RATIO)_mCG values
- Megalodon v2.5.0 5mC calls (CpG context, res_dna_r941_min_modbases-all-context_v001.cfg)
- nanopolish v0.13.2 methylation calls on the same aligned reads
- ENCODE WGBS methylation data (ENCFF847OWL, ENCFF390OZB, ENCFF064GJQ, ENCFF369YQW)
- HepG2 H3K27me3 CUT&Tag-BS methylation calls
- ENCODE H3K27me3 ChIP-seq peak BED files (ENCFF546TJV, ENCFF950VUB)

## Outputs

- Per-CpG methylation fractions per caller, restricted to H3K27me3 peak regions with coverage >=20 by WGBS and nanoHiMe-seq
- Pairwise Pearson correlation matrix of methylation levels across nanoHiMe, nanopolish, Megalodon, WGBS, and CUT&Tag-BS (methylation_correlation_matrix.csv)

## How to apply

Call CpG methylation genome-wide in nanoHiMe mCG mode ('nanoHiMe mCG ref_upper.fa test.eventalign.txt output.mCG'), which classifies CpGs by log likelihood ratio with a threshold of 1.5 for methylated and -1.5 for unmethylated CpGs; also call 5mC with Megalodon v2.5.0 (res_dna_r941_min_modbases-all-context_v001.cfg) and nanopolish v0.13.2 on the same mapped reads. Aggregate per-CpG methylation fractions only within H3K27me3 ChIP-seq peak regions (ENCODE accessions ENCFF546TJV and ENCFF950VUB for HepG2), keeping only CpG sites with coverage depth >=20 by both WGBS and nanoHiMe-seq; WGBS data come from ENCODE accessions ENCFF847OWL, ENCFF390OZB, ENCFF064GJQ, and ENCFF369YQW, alongside HepG2 H3K27me3 CUT&Tag-BS methylation calls. Compute pairwise Pearson correlations among nanoHiMe, nanopolish, Megalodon, WGBS, and CUT&Tag-BS methylation levels and write the matrix to methylation_correlation_matrix.csv. The rationale is that restricting to ChIP-seq peak regions and >=20x dual coverage removes noisy low-coverage CpGs so caller concordance reflects true accuracy rather than sampling variance.

## Related tools

- **nanoHiMe** (HMM-based mCG calling from nanopolish eventalign files, classifying CpGs by log likelihood ratio thresholds (+1.5 / -1.5)) — github.com/YinLabTJ/nanoHiMe
- **Megalodon** (ONT methylation caller used as a comparator, 5mC in CpG context with model res_dna_r941_min_modbases-all-context_v001.cfg (v2.5.0))
- **nanopolish** (v0.13.2; produces event-level alignments via 'nanopolish eventalign -n -t 16 --scale-events' that feed nanoHiMe mCG calling, and provides comparator 5mC calls)
- **Minimap2** (v2.17; aligns basecalled nanopore reads to hg19 in map-ont mode with 8 threads before methylation calling)
- **Samtools** (v1.9; sorts, indexes, and filters mapped reads with mapping quality >=20 via 'samtools view -b -q 20 -F 4')
- **Guppy** (v4.4.2; basecalls FAST5 signal data with the dna_r9.4.1_450bps_hac high-accuracy model prior to alignment)

## Examples

```
./nanoHiMe mCG ref_upper.fa test.eventalign.txt output.mCG
```

## Evaluation signals

- nanoHiMe vs WGBS Pearson r = 0.931, exceeding Megalodon and nanopolish consistency with WGBS
- nanoHiMe vs nanopolish r = 0.953; Megalodon vs nanopolish r = 0.885
- CUT&Tag-BS vs WGBS r = 0.922; nanoHiMe vs CUT&Tag-BS r = 0.871; Megalodon vs CUT&Tag-BS r = 0.77
- Every retained CpG must have coverage depth >=20 by both WGBS and nanoHiMe-seq
- Positive Log(LIKELIHOOD_RATIO)_mCG values support methylation; negative values support unmethylation, with +1.5 / -1.5 classification thresholds

## Limitations

- CpG sites with coverage depth <20 by either WGBS or nanoHiMe-seq must be excluded, limiting analysis to well-covered peak-region CpGs
- nanoHiMe classification relies on fixed log likelihood ratio thresholds (1.5 / -1.5); borderline CpGs fall into an ambiguous zone
- Restricting to H3K27me3 ChIP-seq peaks (ENCFF546TJV, ENCFF950VUB) means results are only representative of those repressed domains, not the whole genome

## Evidence

- [discussion] CpG coverage filter: "calculated for the CpG sites with coverage depth ≥20 by both WGBS and nanoHiMe-seq data"
- [discussion] LLR thresholds: "we set a threshold of 1.5 to call methylated CpGs and −1.5 to call unmethylated CpGs"
- [readme] mCG mode invocation: "./nanoHiMe mCG ref_upper.fa test.eventalign.txt(.gz) output.mCG"
- [results] correlation finding: [evidence span withheld — anchor status: not_found]
- [abstract] key finding: "using a hidden Markov model, which is implemented in the nanoHiMe software package"
