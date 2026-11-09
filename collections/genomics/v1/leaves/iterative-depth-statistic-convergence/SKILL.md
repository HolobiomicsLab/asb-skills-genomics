---
name: iterative-depth-statistic-convergence
description: Use when a hybrid scaffolder needs to label contigs in a short-read draft
  assembly (e.g., SPAdes contigs.fasta) as unique vs repetitive before scaffolding
  with nanopore long reads, using coverage ratios.
when_to_use_negative:
- The draft contigs come from long-read-only assemblers (e.g., Celera Assembler) whose
  contigs lack SPAdes-style coverage values in headers
- The organism is not bacterial, so the assumption that the 20 largest >20 Kb contigs
  are unique may not hold (e.g., eukaryotic genomes with repeated large regions)
- Contigs are already annotated as unique/repetitive by a prior copy-number step and
  only scaffolding is needed
edam_operation: http://edamontology.org/operation_0525
edam_topics:
- http://edamontology.org/topic_0196
- http://edamontology.org/topic_0622
tools:
- name: npScarf
  role: Implements the iterative depthg estimation and θ=1.5 multiplicity classification
    of contigs prior to scaffolding (jsa.np.npscarf)
  repo: https://github.com/mdcao/npScarf
- name: SPAdes
  role: Produces the Illumina draft assembly whose contig headers supply per-contig
    coverage values used by the depth statistic
- name: jsa.seq.sort
  role: Sorts the SPAdes contigs by length in reverse so the largest contigs can be
    selected for the depth estimate
  repo: https://github.com/mdcao/japsa
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/ncomms14515
    title: Scaffolding and completing genome assemblies in real-time with nanopore
      sequencing
  source_content_hash: sha256:a233b777687d2fec6f8f997c9437fd075cef5cb5386ca935d5d45996cd8a8aa8
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/mdcao/npScarf
  tools:
  - SPAdes
  - jsa.seq.sort
  - npScarf
  edam_operation: http://edamontology.org/operation_0525
  edam_topics:
  - http://edamontology.org/topic_0196
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8181818181818182
n_claims: 11
---

# Iterative depth-statistic convergence for contig multiplicity estimation

## Summary

npScarf's procedure for classifying draft assembly contigs as unique or repetitive by iteratively converging a whole-genome depth estimate (depthg) from the largest contigs' short-read coverage and computing each contig's multiplicity as the coverage ratio.

## When to use

Use when a hybrid scaffolder needs to label contigs in a short-read draft assembly (e.g., SPAdes contigs.fasta) as unique vs repetitive before scaffolding with nanopore long reads, using coverage ratios.

## When NOT to use

- The draft contigs come from long-read-only assemblers (e.g., Celera Assembler) whose contigs lack SPAdes-style coverage values in headers
- The organism is not bacterial, so the assumption that the 20 largest >20 Kb contigs are unique may not hold (e.g., eukaryotic genomes with repeated large regions)
- Contigs are already annotated as unique/repetitive by a prior copy-number step and only scaffolding is needed

## Inputs

- SPAdes draft assembly FASTA (e.g., contigs.fasta) with per-contig length and short-read coverage encoded in the headers
- Short-read (Illumina MiSeq paired-end) sequencing coverage of contigs from the draft assembly

## Outputs

- Per-contig multiplicity table (sequence ID, coverage, multiplicity, unique/repetitive label), e.g., contig_multiplicity.tsv
- Set of unique contigs used as the scaffolding backbone and set of repetitive contigs reserved for gap filling

## How to apply

Parse the SPAdes draft assembly FASTA, extracting per-contig length and short-read coverage from the contig headers, focusing on up to the 20 largest contigs longer than 20 Kb since these are most likely unique in a bacterial genome. Initialize the genome depth estimate depthg to the coverage of the largest contig, then iteratively update the estimate until the termination threshold on stability is reached. Compute each contig's multiplicity as the ratio of its short-read coverage to the converged depthg, and classify the contig as unique (non-repetitive) if the multiplicity is below the threshold θ = 1.5, otherwise as repetitive. Unique contigs form the genome backbone for npScarf scaffolding, while repetitive contigs are reserved for gap filling.

## Related tools

- **npScarf** (Implements the iterative depthg estimation and θ=1.5 multiplicity classification of contigs prior to scaffolding (jsa.np.npscarf)) — https://github.com/mdcao/npScarf
- **SPAdes** (Produces the Illumina draft assembly whose contig headers supply per-contig coverage values used by the depth statistic)
- **jsa.seq.sort** (Sorts the SPAdes contigs by length in reverse so the largest contigs can be selected for the depth estimate) — https://github.com/mdcao/japsa

## Examples

```
jsa.seq.sort -r -n --input spades/contigs.fasta --output Kp2146_spades.fasta && jsa.np.npscarf -bwaThread=10 -input Kp2146_ONT.fastq -format fastq -seq Kp2146_spades.fasta -prefix Kp2146-batch
```

## Evaluation signals

- The converged depthg statistic is stable when computed on up to 20 of the largest contigs longer than 20 Kb
- Each contig's multiplicity is a positive ratio near 1 for unique contigs and near 2+ for two-copy repeats
- Classification output table contains sequence ID, coverage, multiplicity, and unique/repetitive label for every contig
- Unique contigs (multiplicity < 1.5) should dominate the npScarf genome backbone, with repetitive contigs appearing only as gap-filling insertions

## Limitations

- The θ = 1.5 threshold was set from the authors' experience ('In our experience, the statistic is stable...') rather than a principled derivation
- The depth statistic's stability assumes the 20 largest >20 Kb contigs are unique, which is grounded in bacterial genomes; it may fail for eukaryotic assemblies
- Coverage-based multiplicity cannot distinguish very-high-copy repeats from low-coverage unique regions under uneven sequencing depth

## Evidence

- [discussion] npScarf determines the multiplicity of each contig in the assembly by comparing short read sequencing coverage of the contig to that of the whole genome.: "npScarf determines the multiplicity of each contig in the assembly by comparing short read sequencing coverage of the contig to that of the whole genome."
- [discussion] npScarf set the threshold θ to 1.5.: "npScarf set the threshold θ to 1.5."
- [discussion] In our experience, the statistic is stable with up to 20 of the largest contigs longer than 20 Kb, which are most likely unique contigs in bacterial genomes: "In our experience, the statistic is stable with up to 20 of the largest contigs longer than 20 Kb, which are most likely unique contigs in bacterial genomes38."
- [discussion] npScarf first initializes depthg to that of the largest contig: "npScarf first initializes depthg to that of the largest contig: It then iteratively updates the estimate"
- [readme] The result contigs file of interest is spades/contigs.fasta. The contig list is then sorted with: "The result contigs file of interest is spades/contigs.fasta. The contig list is then sorted with"
