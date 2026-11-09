---
name: hamming-error-quantification
description: Use when you need to measure how accurately a consensus sequence was
  reconstructed from reads, by comparing called consensus FASTAs against their source
  reference via normalized Hamming error.
when_to_use_negative:
- 'The reference is an ultra-long genome (e.g., a human chromosome): ViralConsensus
  memory consumption scales linearly with reference genome length and may be prohibitively
  large'
- There is no single reference sequence for reconstruction — the comparison requires
  aligning each consensus to a defined source reference genome
- You need variant-level accuracy (SNP/indel calls) rather than whole-genome consensus-level
  error rates
edam_operation: http://edamontology.org/operation_0292
edam_topics:
- http://edamontology.org/topic_0080
- http://edamontology.org/topic_0102
tools:
- name: MAFFT
  role: Aligns each reconstructed consensus to its source reference genome so the
    normalized Hamming error can be computed
- name: ViralConsensus
  role: Calls the consensus FASTA from the read alignment that is then scored by normalized
    Hamming error
  repo: https://github.com/niemasd/ViralConsensus
- name: iVar
  role: Baseline consensus caller (v1.3.1, default settings) whose normalized Hamming
    errors are compared against ViralConsensus
- name: minimap2
  role: Maps simulated reads (-a -x sr for short reads) to the lineage reference to
    produce the alignments from which consensus sequences are called
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/bioinformatics/btad317
    title: 'ViralConsensus: a fast and memory-efficient tool for calling viral consensus
      genome sequences directly from read alignment data'
  source_content_hash: sha256:6d25a968773e181ca7144bdc6e92d274aa06e2e2c631e7b3ccb4bec69d16603d
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/niemasd/ViralConsensus
  tools:
  - MAFFT
  - ViralConsensus
  - iVar
  - minimap2
  edam_operation: http://edamontology.org/operation_0292
  edam_topics:
  - http://edamontology.org/topic_0080
  - http://edamontology.org/topic_0102
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 5
---

# hamming-error-quantification

## Summary

Quantifies viral consensus genome reconstruction accuracy by aligning each called consensus (e.g., from ViralConsensus or iVar) back to its source reference genome with MAFFT and computing the normalized Hamming error, then aggregating errors across lineages, sequencing technologies, and coverage levels.

## When to use

Use when you need to measure how accurately a consensus sequence was reconstructed from reads, by comparing called consensus FASTAs against their source reference via normalized Hamming error.

## When NOT to use

- The reference is an ultra-long genome (e.g., a human chromosome): ViralConsensus memory consumption scales linearly with reference genome length and may be prohibitively large
- There is no single reference sequence for reconstruction — the comparison requires aligning each consensus to a defined source reference genome
- You need variant-level accuracy (SNP/indel calls) rather than whole-genome consensus-level error rates

## Inputs

- Called consensus genome FASTA files from each tool under test (e.g., ViralConsensus and iVar) per condition
- Source lineage reference genome FASTA (e.g., NC_045512.2, LC650844, OQ523614.1, OQ194009.1, OQ346068.1)
- Simulated read alignments in SAM/BAM from minimap2 -a -x sr (Illumina) mapping to the lineage reference

## Outputs

- MAFFT v7.505 pairwise alignment of each reconstructed consensus against its source reference
- Per-replicate normalized Hamming error values for each caller
- Single aggregated TSV of normalized Hamming errors across lineages, coverages, and replicates for ViralConsensus vs. iVar

## How to apply

For every simulated condition (SARS-CoV-2 lineages B/NC_045512.2, B.1.1.7/LC650844, BA.1/OQ523614.1, BA.2/OQ194009.1, XBB.1/OQ346068.1 at 30x, 40x, and 50x coverage with 10 replicates each, from both ART-simulated Illumina HiSeq 2000 single-end reads and NanoSim-H ONT reads), call a consensus with each tool under test (e.g., ViralConsensus v0.0.1 and iVar v1.3.1) from the minimap2 (-a -x sr) alignment to the lineage's own reference. Align each reconstructed consensus FASTA back to its source reference genome with MAFFT v7.505 and compute the normalized Hamming error (mismatched positions divided by genome length). Aggregate the normalized Hamming errors for all callers into a single TSV table across all lineages, coverages, and replicates (5 lineages x 3 coverages x 10 replicates x 2 simulators). Validation succeeds when callers reach parity, i.e., comparable normalized Hamming error across all simulated conditions, confirming that speed/memory gains (e.g., ViralConsensus) do not sacrifice consensus accuracy.

## Related tools

- **MAFFT** (Aligns each reconstructed consensus to its source reference genome so the normalized Hamming error can be computed)
- **ViralConsensus** (Calls the consensus FASTA from the read alignment that is then scored by normalized Hamming error) — https://github.com/niemasd/ViralConsensus
- **iVar** (Baseline consensus caller (v1.3.1, default settings) whose normalized Hamming errors are compared against ViralConsensus)
- **minimap2** (Maps simulated reads (-a -x sr for short reads) to the lineage reference to produce the alignments from which consensus sequences are called)

## Examples

```
minimap2 -t 1 -a -x sr reference.fas reads.fastq.gz | viral_consensus -i - -r reference.fas -o consensus.fas
```

## Evaluation signals

- Every reconstructed consensus has a MAFFT alignment against its own source reference before error computation (no cross-lineage references)
- Normalized Hamming error approaches 0 for correct reconstruction and is computed consistently (normalized by genome length) across all conditions
- ViralConsensus and iVar reach parity in normalized Hamming error across the full factorial design: 5 lineages x 3 coverages (30x/40x/50x) x 10 replicates x 2 simulators (ART, NanoSim-H)
- All per-replicate errors are aggregated into a single TSV covering every condition for both callers, with no missing condition cells
- Consensus sequences were called at default settings from the same minimap2 alignments for both tools, so errors are directly comparable

## Limitations

- ViralConsensus memory consumption scales linearly with reference genome length, so ultra-long references may require prohibitively large amounts of memory
- The skill is demonstrated on simulated SARS-CoV-2 data (single-end HiSeq 2000 via ART and ONT via NanoSim-H) at 30x-50x; accuracy conclusions may not transfer to other coverage regimes, read models, or genomes without re-validation
- Normalized Hamming error measures whole-genome consensus mismatch rate and does not localize error positions or distinguish SNP from indel error types

## Evidence

- [other] Align each reconstructed consensus to its source reference genome with MAFFT v7.505 and compute the normalized Hamming error.: [evidence span withheld — anchor status: not_found]
- [readme] ViralConsensus is a fast and memory-efficient tool for calling viral consensus genome sequences directly from read alignment data.: "ViralConsensus is a fast and memory-efficient tool for calling viral consensus genome sequences directly from read alignment data."
- [methods] minimap2 -t 1 -a -x sr reference.fas reads.fastq.gz | viral_consensus -i - -r reference.fas -o consensus.fas: "minimap2 -t 1 -a -x sr reference.fas reads.fastq.gz | viral_consensus -i - -r reference.fas -o consensus.fas"
- [readme] memory consumption scales linearly with reference genome length, meaning ultra-long reference genomes (e.g. human chromosomes) may require prohibitively large amounts of memory: "memory consumption scales linearly with reference genome length, meaning ultra-long reference genomes (e.g. human chromosomes) may require prohibitively large amounts of memory"
- [other] confirm ViralConsensus v0.0.1 and iVar v1.3.1 reach comparable consensus accuracy, i.e. parity in normalized Hamming error: [evidence span withheld — anchor status: not_found]
