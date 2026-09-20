---
name: ploidy-aware-haplotype-evaluation
description: Use when evaluating error-correction or assembly tools on long reads
  (PacBio CLR/ONT) from polyploid genomes or metagenomes, where per-haplotype accuracy
  (QV, switch error, haplotype coverage) must be quantified against reference haplotypes.
when_to_use_negative:
- The input is short-read (Illumina) data or already-high-accuracy reads (HiFi/CCS),
  where haplotype-aware long-read self-correction benchmarks do not apply
- The sample is a single-haploid/clonal genome with no low-frequency haplotypes, where
  consensus-template correction biases are not at issue
- You need only a speed/resource benchmark and do not have reference haplotypes —
  ploidy-aware QV/HC computation requires aligning corrected reads or assemblies back
  to known references
edam_topics:
- http://edamontology.org/topic_0622
tools:
- name: VeChat
  role: Variation-graph-based haplotype-aware error correction whose output (reads.corrected.fa)
    is the subject of the ploidy-aware evaluation
  repo: https://github.com/HaploKit/vechat
- name: Racon
  role: Consensus-based baseline corrector compared against VeChat per ploidy condition
  repo: https://github.com/lbcb-sci/racon
- name: Minimap2
  role: Minimizer-based all-vs-all overlap computation (cycle 1) and aligner for mapping
    corrected reads back to reference haplotypes for QV/switch-error computation
  repo: https://github.com/lh3/minimap2
- name: Canu
  role: Error-correction module baseline; HiCanu used to assemble raw vs. VeChat-corrected
    reads for haplotype-coverage evaluation
- name: QUAST
  role: Computes haplotype coverage, misassembly counts, and error rates of each assembly
    against the reference genome
- name: fpa
  role: Filters all-against-all read overlaps (internal match, containment, dovetail)
    feeding the variation-graph construction
  repo: https://github.com/natir/fpa
- name: yacrd
  role: Chimeric-read detection from all-vs-all read mappings, supporting read scrubbing
    before correction
  repo: https://github.com/natir/yacrd
provenance:
  source_task_ids:
  - task_001
  - task_005
  - task_002
  source_papers:
  - doi: 10.1038/s41467-022-34381-8
    title: 'VeChat: correcting errors in long reads using variation graphs'
  source_content_hash: sha256:39041aadc522b9363cc5c4042959e8cccb91f61e3731beefd7d35b09cdd3d39e
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/HaploKit/vechat
  tools:
  - CONSENT
  - Canu
  - Daccord
  - Minimap2
  - QUAST
  - Racon
  - VeChat
  - fpa
  - yacrd
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 12
---

# ploidy-aware-haplotype-evaluation

## Summary

Benchmark long-read error correction and assembly quality in mixed samples by evaluating haplotype-aware metrics — residual error rate (QV), switch error, and haplotype coverage (HC) — across diploid, triploid, tetraploid, and metagenomic datasets, rather than consensus-only accuracy, because consensus-template methods mask low-frequency haplotype variants.

## When to use

Use when evaluating error-correction or assembly tools on long reads (PacBio CLR/ONT) from polyploid genomes or metagenomes, where per-haplotype accuracy (QV, switch error, haplotype coverage) must be quantified against reference haplotypes.

## When NOT to use

- The input is short-read (Illumina) data or already-high-accuracy reads (HiFi/CCS), where haplotype-aware long-read self-correction benchmarks do not apply
- The sample is a single-haploid/clonal genome with no low-frequency haplotypes, where consensus-template correction biases are not at issue
- You need only a speed/resource benchmark and do not have reference haplotypes — ploidy-aware QV/HC computation requires aligning corrected reads or assemblies back to known references

## Inputs

- Raw long-read files (FASTQ/FASTQ.gz or FASTA), PacBio CLR or ONT, from diploid/triploid/tetraploid or metagenomic samples
- Simulated reference haplotype sequences (e.g., PBSIM2-simulated E. coli mixtures at 30x coverage per haplotype, 10% simulated error)
- Real PacBio benchmark read sets (Microbial 10-plex, NWC metagenome, yeast pseudo-diploid ENA PRJEB7245)
- VeChat-corrected reads (reads.corrected.fa) and matching raw read sets for assembly comparison

## Outputs

- error_rate_table.tsv — residual error rate (QV) per tool, platform, and ploidy/metagenome condition
- pacbio_error_rates.tsv — per-dataset error rates (0.089% Microbial 10-plex, 0.101% NWC, 0.236% yeast pseudo-diploid expected)
- quast_report.tsv — QUAST haplotype coverage, misassembly counts, and error rates per assembler and read type
- reads.corrected.fa — corrected reads per tool and condition

## How to apply

First correct each read set with VeChat using the platform-appropriate flag (--platform pb or --platform ont) and its default graph-pruning thresholds (--min-confidence 0.2, --min-support 0.2, --min-identity-cns 0.99), applying --split for FASTQ inputs >20G or FASTA inputs >10G to bound memory. Run the comparison baselines — Racon, the Canu error-correction module, Daccord (local DBG), and CONSENT (MSA+DBG) — with their recommended settings on the identical raw reads. Align each tool's corrected reads (reads.corrected.fa) back to the reference haplotypes (e.g., simulated E. coli diploid/triploid/tetraploid sets generated at 30x per haplotype with PBSIM2, or real yeast pseudo-diploid, NWC, and Microbial 10-plex PacBio data) and compute residual error rates (QV) per tool, platform, and ploidy condition; on real diploid benchmarks also compute switch error (as on HG002, where VeChat shows ~1.2x lower switch error than CONSENT). For assembly-level evaluation, assemble raw vs. VeChat-corrected reads with HiCanu/Canu and Flye, run QUAST against the reference, and tabulate haplotype coverage, misassembly counts, and error rates per assembler and read type. Judge success by whether VeChat-corrected reads/assemblies reproduce the reported gains: 14–30x, 9–26x, and 4–11x lower error rates on diploid, triploid, and tetraploid simulated data, haplotype coverage +7.6% (92% → 99.6%) on diploid and +14% on tetraploid, and the lowest error rate among tools on every simulated condition.

## Related tools

- **VeChat** (Variation-graph-based haplotype-aware error correction whose output (reads.corrected.fa) is the subject of the ploidy-aware evaluation) — https://github.com/HaploKit/vechat
- **Racon** (Consensus-based baseline corrector compared against VeChat per ploidy condition) — https://github.com/lbcb-sci/racon
- **Minimap2** (Minimizer-based all-vs-all overlap computation (cycle 1) and aligner for mapping corrected reads back to reference haplotypes for QV/switch-error computation) — https://github.com/lh3/minimap2
- **Canu** (Error-correction module baseline; HiCanu used to assemble raw vs. VeChat-corrected reads for haplotype-coverage evaluation)
- **QUAST** (Computes haplotype coverage, misassembly counts, and error rates of each assembly against the reference genome)
- **fpa** (Filters all-against-all read overlaps (internal match, containment, dovetail) feeding the variation-graph construction) — https://github.com/natir/fpa
- **yacrd** (Chimeric-read detection from all-vs-all read mappings, supporting read scrubbing before correction) — https://github.com/natir/yacrd

## Examples

```
vechat reads.fq.gz -t 8 --platform pb -o reads.corrected.fa
```

## Evaluation signals

- On simulated E. coli PacBio CLR data, VeChat must show the lowest error rate of all tools, with baselines 14–30x (diploid), 9–26x (triploid), and 4–11x (tetraploid) higher; e.g., VeChat ≈ 0.014% vs Racon 0.276% on PacBio diploid
- On real PacBio datasets, reproduced error rates must match 0.089% (Microbial 10-plex), 0.101% (NWC metagenome), and 0.236% (yeast pseudo-diploid), and reads.corrected.fa must exist per dataset
- VeChat+HiCanu haplotype coverage must exceed Canu on raw reads by ~7.6% (92% → 99.6%) on diploid and ~14% on tetraploid, with fewer misassemblies in the QUAST report
- Switch error on real diploid benchmarks should be ~1.2x lower for VeChat than CONSENT (HG002 reference signal)
- Deviation of any per-tool error rate beyond expected stochastic simulation variance from the paper's benchmark values flags the reproduction as failed

## Limitations

- VeChat has higher runtimes and peak memory usage than competitors, particularly on large datasets, and does not win in computational resources
- VeChat uses off-the-shelf tools (Minimap2, edit-distance alignments, sequence-to-graph alignments) without exploiting all their features, causing avoidable overheads
- Optimal combinations of preprocessing, error correction, and assembly tools for polyploid, cancer, or metagenomic settings remain unexplored and beyond the study's scope

## Evidence

- [results] On simulated PacBio CLR reads, VeChat achieves approximately 14–30, 9–26 and 4–11 times lower error rates on diploid, triploid and tetraploid genomes, respectively: "VeChat achieves approximately 14–30, 9–26 and 4–11 times lower error rates on diploid, triploid and tetraploid genomes, respectively."
- [abstract] Mixed-sample bias motivating ploidy-aware evaluation: "Current self-correction methods, however, are affected by consensus sequence induced biases that mask true variants in haplotypes of lower frequency showing in mixed samples"
- [results] Switch error and QV benchmark on real diploid data: "VeChat achieves approximately 1.5 times lower error rate (QV) and 1.2 times lower switch error on HG002 compared with CONSENT"
- [readme] README rationale for higher-ploidy and metagenome evaluation: "in mixed samples, such as metagenomes or organisms of higher ploidy, consensus induced biases can mask true variants affecting haplotypes of lower frequencies"
- [discussion] Runtime/memory caveat for large datasets: "VeChat does not win the competition in terms of computational resources. In particular on large datasets, VeChat experiences longer runtimes and higher peak memory usage"
- [abstract] Assembly-level haplotype-awareness improvement: "using VeChat prior to long-read assembly significantly improves the haplotype awareness of the assemblies"
