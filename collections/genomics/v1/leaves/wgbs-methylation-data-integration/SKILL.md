---
name: wgbs-methylation-data-integration
description: Use when benchmarking nanopore methylation calls (nanoHiMe, nanopolish,
  Megalodon) against ENCODE WGBS data or across nanopore coverage depths in HepG2,
  requiring per-CpG correlation under matched coverage filters.
when_to_use_negative:
- Input methylation data lacks per-CpG coverage estimates, since the ≥20 depth filter
  in both WGBS and nanoHiMe-seq cannot be applied
- The reference bisulfite dataset is not WGBS/CUT&Tag-BS per-CpG methylation but aggregated
  peak-level signal, which cannot be correlated site-by-site
- The goal is 6mA region calling from nanoHiMe-seq reads rather than CpG methylation
  comparison with bisulfite data
edam_operation: http://edamontology.org/operation_3207
edam_topics:
- http://edamontology.org/topic_3674
- http://edamontology.org/topic_3168
tools:
- name: nanoHiMe
  role: HMM-based CpG methylation caller run in mCG mode on nanopolish eventalign
    files with LLR thresholds +1.5/-1.5
  repo: github.com/YinLabTJ/nanoHiMe
- name: nanopolish
  role: Aligns nanopore electrical events to reference k-mers (--scale-events) producing
    the eventalign input for methylation calling
- name: Megalodon
  role: v2.5.0 with res_dna_r941_min_modbases-all-context_v001.cfg model, used as
    a comparator 5mC caller in CpG context
- name: minimap2
  role: v2.17, aligns basecalled ONT reads to hg19 with the map-ont preset before
    methylation calling
- name: samtools
  role: v1.9, sorts/indexes BAMs and filters mapped reads with MAPQ>=20 via 'samtools
    view -b -q 20 -F 4'
provenance:
  source_task_ids:
  - task_006
  - task_005
  source_papers:
  - doi: 10.1038/s41467-022-35650-2
    title: Simultaneous profiling of histone modifications and DNA methylation via
      nanopore sequencing
  source_content_hash: sha256:c38234721d3a1a8ea3feda113451d5a37ea08c81289f2ec87dfab2fc382fbd42
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Guppy
  - Megalodon
  - Minimap2
  - minimap2
  - nanoHiMe
  - nanopolish
  - samtools
  edam_operation: http://edamontology.org/operation_3207
  edam_topics:
  - http://edamontology.org/topic_3674
  - http://edamontology.org/topic_3168
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7272727272727273
n_claims: 11
---

# wgbs-methylation-data-integration

## Summary

Integrates nanopore-based CpG methylation calls (nanoHiMe mCG mode, nanopolish, Megalodon) with ENCODE WGBS reference data by restricting correlation analyses to CpG sites with coverage depth ≥20 in both assays and computing pairwise Pearson correlations of methylation levels.

## When to use

Use when benchmarking nanopore methylation calls (nanoHiMe, nanopolish, Megalodon) against ENCODE WGBS data or across nanopore coverage depths in HepG2, requiring per-CpG correlation under matched coverage filters.

## When NOT to use

- Input methylation data lacks per-CpG coverage estimates, since the ≥20 depth filter in both WGBS and nanoHiMe-seq cannot be applied
- The reference bisulfite dataset is not WGBS/CUT&Tag-BS per-CpG methylation but aggregated peak-level signal, which cannot be correlated site-by-site
- The goal is 6mA region calling from nanoHiMe-seq reads rather than CpG methylation comparison with bisulfite data

## Inputs

- nanoHiMe mCG output (output.mCG.eventalign.txt with per-read Log(LIKELIHOOD_RATIO)_mCG values)
- nanopolish and Megalodon v2.5.0 per-CpG methylation calls on the same reads
- ENCODE HepG2 WGBS methylation data (ENCFF847OWL, ENCFF390OZB, ENCFF064GJQ, ENCFF369YQW)
- H3K27me3 ChIP-seq peak BED files (ENCFF546TJV, ENCFF950VUB) for region-restricted aggregation
- Subsampled nanoHiMe-seq FASTQ/BAM at 5x, 10x, 30x, 50x coverage

## Outputs

- methylation_correlation_matrix.csv (pairwise Pearson r values: nanoHiMe vs WGBS r=0.931, nanoHiMe vs nanopolish r=0.953, etc.)
- coverage_correlation.csv (Pearson r of nanoHiMe vs WGBS at each subsampling depth: 0.82 at 10x, ~0.92 at 30x; 10x vs 50x r=0.914)
- Per-CpG scatterplot coordinates (nanoHiMe vs WGBS at 10x and 30x; 10x vs 50x)

## How to apply

Call CpG methylation from nanopolish eventalign files using nanoHiMe in mCG-calling mode with log-likelihood-ratio thresholds of +1.5 (methylated) and -1.5 (unmethylated), and in parallel call 5mC with Megalodon v2.5.0 (res_dna_r941_min_modbases-all-context_v001.cfg) and nanopolish on the same MAPQ≥20 aligned BAMs. Download the ENCODE HepG2 WGBS datasets (ENCFF847OWL, ENCFF390OZB, ENCFF064GJQ, ENCFF369YQW) as bisulfite-based reference methylation levels. Before correlating, restrict to CpG sites with coverage depth ≥20 by both WGBS and nanoHiMe-seq to avoid noise from poorly covered cytosines. Compute pairwise Pearson correlations between nanoHiMe, nanopolish, Megalodon, WGBS, and optionally CUT&Tag-BS methylation levels, and write the correlation matrix to CSV. For depth sensitivity, subsample reads to 5x-50x with seqkit sample and correlate each depth against WGBS to find the minimum depth at which agreement plateaus (~0.92 at 30x).

## Related tools

- **nanoHiMe** (HMM-based CpG methylation caller run in mCG mode on nanopolish eventalign files with LLR thresholds +1.5/-1.5) — github.com/YinLabTJ/nanoHiMe
- **nanopolish** (Aligns nanopore electrical events to reference k-mers (--scale-events) producing the eventalign input for methylation calling)
- **Megalodon** (v2.5.0 with res_dna_r941_min_modbases-all-context_v001.cfg model, used as a comparator 5mC caller in CpG context)
- **minimap2** (v2.17, aligns basecalled ONT reads to hg19 with the map-ont preset before methylation calling)
- **samtools** (v1.9, sorts/indexes BAMs and filters mapped reads with MAPQ>=20 via 'samtools view -b -q 20 -F 4')

## Examples

```
./nanoHiMe mCG ref_upper.fa test.eventalign.txt(.gz) output.mCG
```

## Evaluation signals

- nanoHiMe vs WGBS Pearson r should reproduce ~0.931 when aggregating CpGs in H3K27me3 ChIP-seq peak regions with coverage ≥20 by both assays
- Depth-sensitivity correlation table should show r≈0.82 for 10x vs WGBS and r≈0.92 (near plateau) for 30x vs WGBS, with 10x vs 50x r=0.914
- nanoHiMe vs nanopolish consistency (r≈0.953) should exceed Megalodon vs nanopolish (r≈0.885)
- CUT&Tag-BS vs WGBS r≈0.922 should serve as a sanity check of the bisulfite reference data
- All correlated CpG sites must have coverage depth ≥20 in both WGBS and nanoHiMe-seq; verify no sites below this threshold entered the correlation

## Limitations

- Correlation is restricted to CpG sites with coverage ≥20 by both WGBS and nanoHiMe-seq, so sparse low-coverage CpGs are excluded and results may not generalize genome-wide
- Bisulfite-based references (WGBS, CUT&Tag-BS) suffer DNA damage, complexity reduction and biases, and CUT&Tag-BS can miss weak peaks due to bisulfite degradation
- Below ~10x coverage nanoHiMe-seq CpG accuracy degrades; near-plateau agreement with WGBS is only reached around 30x (one or two MinION flowcells)

## Evidence

- [discussion] Coverage depth filter for correlations: "calculated for the CpG sites with coverage depth ≥20 by both WGBS and nanoHiMe-seq data"
- [discussion] LLR thresholds for CpG calls: "we set a threshold of 1.5 to call methylated CpGs and −1.5 to call unmethylated CpGs"
- [discussion] WGBS dataset accessions: "Whole-genome Bisulfite Sequencing (WGBS) data was from ENCODE with accession numbers ENCFF847OWL, ENCFF390OZB, ENCFF064GJQ and ENCFF369YQW"
- [methods] MAPQ filter command: "samtools view -b -q 20 -F 4 test.sorted.bam > test.sorted.q20.mapped.bam"
- [readme] mCG calling usage from README: "A positive value in Log(LIKELIHOOD_RATIO)_mCG shows the support for mCG methylation, and a negative value indicates the support for unmethylation."
