---
name: corrected-read-alignment-error-rate-computation
description: Use when you have VeChat-corrected long reads (FASTA) and a reference
  genome, and need to quantify post-correction error rate to validate or benchmark
  correction performance.
when_to_use_negative:
- The reads have not been error-corrected — raw PacBio CLR reads carry ~15% error
  and alignment-based error-rate computation would measure sequencing noise, not correction
  performance
- You want to benchmark computational resources (runtime, peak memory) — the article
  notes VeChat experiences longer runtimes and higher peak memory usage on large datasets,
  which is out of scope for this accuracy metric
edam_operation: http://edamontology.org/operation_0292
edam_topics:
- http://edamontology.org/topic_0102
- http://edamontology.org/topic_0196
tools:
- name: VeChat
  role: Corrects the simulated PacBio CLR read sets (--platform pb, -t) before error-rate
    assessment; output written with -o reads.corrected.fa
  repo: https://github.com/HaploKit/vechat
- name: Minimap2
  role: Aligns each corrected read set back to the triploid reference genome so overall
    corrected-read error rates can be computed
  repo: https://github.com/lh3/minimap2
- name: PBSIM2
  role: Simulates PacBio CLR long reads from the triploid E. coli reference at 15%
    and 5% sequencing error rates to create the benchmark conditions
- name: Racon
  role: Mentioned alongside minimap2 in the VeChat methods environment as supporting
    tooling for long-read correction workflows
  repo: https://github.com/lbcb-sci/racon
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-022-34381-8
    title: 'VeChat: correcting errors in long reads using variation graphs'
  source_content_hash: sha256:540b9670ce0c2f1bbbe2941e827f6bbbc91d14163a8d4a08f0208346b6cf8ded
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/HaploKit/vechat
  tools:
  - Minimap2
  - PBSIM2
  - Racon
  - VeChat
  edam_operation: http://edamontology.org/operation_0292
  edam_topics:
  - http://edamontology.org/topic_0102
  - http://edamontology.org/topic_0196
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 6
---

# corrected-read-alignment-error-rate-computation

## Summary

Benchmarking method that quantifies VeChat error-correction quality by aligning corrected long reads back to the reference genome with minimap2 and computing the overall corrected-read error rate. It enables side-by-side comparison of corrected error rates across input-error-rate conditions (e.g. 15% vs. 5% simulated PacBio CLR).

## When to use

Use when you have VeChat-corrected long reads (FASTA) and a reference genome, and need to quantify post-correction error rate to validate or benchmark correction performance.

## When NOT to use

- The reads have not been error-corrected — raw PacBio CLR reads carry ~15% error and alignment-based error-rate computation would measure sequencing noise, not correction performance
- You want to benchmark computational resources (runtime, peak memory) — the article notes VeChat experiences longer runtimes and higher peak memory usage on large datasets, which is out of scope for this accuracy metric

## Inputs

- Corrected long reads in FASTA format (e.g. reads.corrected.fa from VeChat)
- Reference genome FASTA (e.g. triploid E. coli reference of three strains)
- Raw simulated PacBio CLR reads in FASTQ format (PBSIM2 output at 15% and 5% error rates)
- VeChat invocation parameters (--platform pb, -t threads)

## Outputs

- Per-condition corrected-read error rates tabulated side by side (e.g. vechat_triploid_error_rates.csv)
- minimap2 alignments (SAM/PAF) of corrected reads against the triploid reference
- Overall corrected error rate per input-error condition (e.g. 0.091% at 15%, 0.009% at 5%)

## How to apply

Simulate PacBio CLR long reads (e.g. from a triploid E. coli genome of three strains at 30x coverage per haplotype) with PBSIM2 at the desired sequencing error rates (15% and 5%). Correct each read set with VeChat using the --platform pb flag and multi-threading (-t), producing corrected FASTA output via -o. Align each corrected read set back to the triploid reference with minimap2 using the PacBio CLR preset, and compute the overall error rate of the aligned corrected reads. Tabulate corrected error rates per condition side by side (e.g. in vechat_triploid_error_rates.csv) to quantify how correction performance scales with input error rate.

## Related tools

- **VeChat** (Corrects the simulated PacBio CLR read sets (--platform pb, -t) before error-rate assessment; output written with -o reads.corrected.fa) — https://github.com/HaploKit/vechat
- **Minimap2** (Aligns each corrected read set back to the triploid reference genome so overall corrected-read error rates can be computed) — https://github.com/lh3/minimap2
- **PBSIM2** (Simulates PacBio CLR long reads from the triploid E. coli reference at 15% and 5% sequencing error rates to create the benchmark conditions)
- **Racon** (Mentioned alongside minimap2 in the VeChat methods environment as supporting tooling for long-read correction workflows) — https://github.com/lbcb-sci/racon

## Examples

```
vechat reads.fq.gz --platform pb -t 8 -o reads.corrected.fa; minimap2 -ax map-pb ref.fa reads.corrected.fa > corrected.sam
```

## Evaluation signals

- Corrected error rate at 5% input error should reproduce the reported 0.009% (with 0.091% at 15% input error) on the simulated triploid PacBio CLR data
- Corrected error rate at the lower input error rate (5%) must be strictly lower than at 15%, confirming error-rate-scaling behavior
- Corrected error rates should reflect the paper's reported substantial reductions on triploid genomes (approximately 9-26 times lower error rates than state-of-the-art correction)
- Each corrected read set should align back to the triploid reference with minimap2 producing a complete alignment file (SAM/PAF) before error-rate tabulation
- The CSV (e.g. vechat_triploid_error_rates.csv) must contain both conditions tabulated side by side for direct comparison

## Limitations

- VeChat does not win the competition in terms of computational resources; on large datasets it experiences longer runtimes and higher peak memory usage
- VeChat uses off-the-shelf approaches in some places without exploiting all their features, causing unnecessary overheads
- Optimal pipeline combinations for polyploid, cancer, or metagenomic settings are beyond the scope of the study and require further efforts

## Evidence

- [results] VeChat achieves approximately 14–30, 9–26 and 4–11 times lower error rates on diploid, triploid and tetraploid genomes, respectively.: "VeChat achieves approximately 14–30, 9–26 and 4–11 times lower error rates on diploid, triploid and tetraploid genomes, respectively."
- [readme] VeChat distinguishes errors from haplotype-specific true variants based on variation graphs: "We present VeChat, as a novel approach to implement this idea: VeChat distinguishes errors from haplotype-specific true variants based on variation graphs"
- [readme] Minimap2 is a versatile sequence alignment program that aligns DNA or mRNA sequences against a large reference database.: [evidence span withheld — anchor status: external_doc]
- [results] step 1 computes minimizer based all-vs-all overlaps, for which we employ Minimap2: "step 1 computes minimizer based all-vs-all overlaps, for which we employ Minimap229"
