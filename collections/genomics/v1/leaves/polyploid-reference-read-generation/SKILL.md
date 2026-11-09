---
name: polyploid-reference-read-generation
description: Use when benchmarking haplotype-aware long-read error correction on polyploid/poly-ploid
  mixtures (ploidy > 2) and you need simulated noisy PacBio CLR reads at controlled
  input error rates and per-haplotype coverage.
when_to_use_negative:
- The reference is haploid or diploid — standard single/diploid simulation suffices
  and polyploid-aware ground truth is unnecessary
- Real sequencing data are already available; simulation is only needed to establish
  ground truth and controlled error rates
- The data are high-accuracy PacBio HiFi or ONT Q20 reads, where the noisy CLR simulation
  model (up to ~15% error) does not apply
edam_topics:
- http://edamontology.org/topic_3168
tools:
- name: PBSIM2
  role: Simulates PacBio CLR long reads from the triploid E. coli reference at controlled
    sequencing error rates (15% and 5%) with 30x coverage per haplotype
- name: VeChat
  role: Corrects the simulated polyploid read sets (vechat reads.fq.gz --platform
    pb -t -o reads.corrected.fa) to measure error-rate reduction per input-error condition
  repo: https://github.com/HaploKit/vechat
- name: Minimap2
  role: Aligns each corrected read set back to the triploid reference to compute the
    overall corrected-read error rate
  repo: https://github.com/lh3/minimap2
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-022-34381-8
    title: 'VeChat: correcting errors in long reads using variation graphs'
  source_content_hash: sha256:82ff63d1e3a64b4cd94f6595a41c1d7f15dab184a1cc73302a2ea824b15ae87b
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/HaploKit/vechat
  tools:
  - Minimap2
  - PBSIM2
  - Racon
  - VeChat
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3168
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# polyploid-reference-read-generation

## Summary

Generates simulated PacBio CLR long reads from a polyploid reference (e.g. a triploid genome built from three E. coli strains) at controlled sequencing error rates (15% and 5%) and ~30x coverage per haplotype, using PBSIM2, to benchmark haplotype-aware error correction such as VeChat.

## When to use

Use when benchmarking haplotype-aware long-read error correction on polyploid/poly-ploid mixtures (ploidy > 2) and you need simulated noisy PacBio CLR reads at controlled input error rates and per-haplotype coverage.

## When NOT to use

- The reference is haploid or diploid — standard single/diploid simulation suffices and polyploid-aware ground truth is unnecessary
- Real sequencing data are already available; simulation is only needed to establish ground truth and controlled error rates
- The data are high-accuracy PacBio HiFi or ONT Q20 reads, where the noisy CLR simulation model (up to ~15% error) does not apply

## Inputs

- Triploid reference FASTA (three E. coli strain genomes concatenated as haplotypes)
- PBSIM2 simulation configuration specifying PacBio CLR model, sequencing error rate (15% and 5%), and coverage (30x per haplotype)
- Raw simulated PacBio CLR reads (FASTQ/FASTQ.gz) per error-rate condition

## Outputs

- Simulated PacBio CLR read sets (FASTQ/FASTQ.gz) at 15% and 5% input error rates
- VeChat-corrected reads (FASTA, e.g. reads.corrected.fa) per condition
- Minimap2 alignments of corrected reads against the triploid reference (SAM/PAF)
- Corrected error-rate table per input-error condition (e.g. vechat_triploid_error_rates.csv)

## How to apply

Assemble the polyploid reference by combining the component haplotypes (for the VeChat benchmark, a triploid genome consisting of three E. coli strains). Run PBSIM2 twice against this reference, once at a 15% sequencing error rate and once at 5%, using PacBio CLR read simulation with 30x coverage per haplotype. Correct each simulated read set with VeChat using --platform pb and multi-threading (-t), producing reads.corrected.fa. Align each corrected set back to the triploid reference with minimap2 (PacBio CLR preset) and compute the overall corrected-read error rate per condition. The rationale is that simulated reads with known ground truth and titrated error rates expose how correction quality scales with input noise, which VeChat's authors reported as dropping from 0.091% (15% input error) to 0.009% (5% input error) on the triploid E. coli data.

## Related tools

- **PBSIM2** (Simulates PacBio CLR long reads from the triploid E. coli reference at controlled sequencing error rates (15% and 5%) with 30x coverage per haplotype)
- **VeChat** (Corrects the simulated polyploid read sets (vechat reads.fq.gz --platform pb -t -o reads.corrected.fa) to measure error-rate reduction per input-error condition) — https://github.com/HaploKit/vechat
- **Minimap2** (Aligns each corrected read set back to the triploid reference to compute the overall corrected-read error rate) — https://github.com/lh3/minimap2

## Examples

```
vechat reads.fq.gz --platform pb -t -o reads.corrected.fa
```

## Evaluation signals

- Corrected-read error rate at 15% input error reproduces the reported ~0.091%, and at 5% input error the reported ~0.009%, on the triploid PacBio CLR data
- Error-rate improvement falls in the reported range of approximately 9-26 times lower error rates than state-of-the-art correction on triploid genomes
- Both simulated read sets carry ~30x coverage per haplotype across the three E. coli strains, verified by read-depth inspection
- Minimap2 alignments of corrected reads map back to the correct haplotype in the triploid reference with high identity
- Results are tabulated side-by-side per condition (e.g. vechat_triploid_error_rates.csv with rows for 15% and 5% input error)

## Limitations

- VeChat has longer runtimes and higher peak memory usage on large datasets, so polyploid simulations of large genomes may be computationally expensive
- The reported 0.091%/0.009% error rates derive from simulated PacBio CLR data with known ground truth; absolute numbers may not transfer to real reads
- Optimal combinations of preprocessing, error correction and assembly for polyploid, cancer or metagenomic settings remain to be explored and are beyond the scope of the original study

## Evidence

- [other] Simulate PacBio CLR long reads from the triploid E. coli reference (three E. coli strains) using PBSIM2 twice, once with a 15% sequencing error rate and once with a 5% sequencing error rate.: [evidence span withheld — anchor status: not_found]
- [results] triploid genome of three E. coli strains: "we focused on the triploid genome, consisting of three E. coli strains as described before"
- [intro] ploidies larger than two: "This reflects a novelty for ploidies larger than two, because earlier approaches only deal with diploid scenarios21"
- [readme] minimap2 PacBio CLR preset: [evidence span withheld — anchor status: external_doc]
