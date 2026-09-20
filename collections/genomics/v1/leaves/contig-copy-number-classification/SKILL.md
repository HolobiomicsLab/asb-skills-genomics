---
name: contig-copy-number-classification
description: Use when a draft assembly (e.g. SPAdes contigs from Illumina MiSeq paired-end
  reads) must be partitioned into unique vs repetitive contigs before scaffolding,
  gap-filling, or deciding which contigs can anchor long-read bridges.
when_to_use_negative:
- The assembly has no associated short-read data — multiplicity estimation relies
  on comparing contig short-read coverage to whole-genome coverage and cannot be computed
  from long reads alone.
- The assembly is largely fragmented with few contigs over 20 Kb — the depth statistic
  is only stated to be stable with up to 20 of the largest contigs longer than 20
  Kb, so bacterial-genome assumptions may not transfer.
- The goal is de novo long-read assembly rather than scaffolding a short-read draft
  (e.g. use Celera Assembler on corrected long reads instead).
edam_operation: http://edamontology.org/operation_0222
edam_topics:
- http://edamontology.org/topic_3174
- http://edamontology.org/topic_0622
tools:
- name: npScarf (jsa.np.npscarf)
  role: 'Performs the copy-number estimation: initializes depthg to the largest-contig
    coverage, iteratively updates it, and applies threshold theta = 1.5 to label contigs
    unique vs repetitive.'
  repo: https://github.com/mdcao/npScarf
- name: SPAdes
  role: Produces the draft Illumina contigs whose per-contig coverage is the basis
    for multiplicity classification (spades/contigs.fasta).
- name: Japsa (jsa.seq.sort)
  role: Length-sorts the draft contigs (-r -n) so the largest contigs used to seed
    the genome-depth estimate are available.
  repo: https://github.com/mdcao/japsa
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/ncomms14515
    title: Scaffolding and completing genome assemblies in real-time with nanopore
      sequencing
  source_content_hash: sha256:abba7859ff5a191d30c504953fd0ab2004b851fe2d314c88e3680866d1c2b98a
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/mdcao/npScarf
  tools:
  - BWA-MEM
  - Japsa (jsa.seq.sort)
  - SPAdes
  - npScarf
  - npScarf (jsa.np.npscarf)
  edam_operation: http://edamontology.org/operation_0222
  edam_topics:
  - http://edamontology.org/topic_3174
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9090909090909091
n_claims: 11
---

# contig-copy-number-classification

## Summary

Classifies draft-assembly contigs as unique (copy number 1) or repetitive by comparing each contig's short-read coverage against an iteratively estimated whole-genome depth, a prerequisite for npScarf's scaffolding from unique contigs and gap-filling with repetitive ones.

## When to use

Use when a draft assembly (e.g. SPAdes contigs from Illumina MiSeq paired-end reads) must be partitioned into unique vs repetitive contigs before scaffolding, gap-filling, or deciding which contigs can anchor long-read bridges.

## When NOT to use

- The assembly has no associated short-read data — multiplicity estimation relies on comparing contig short-read coverage to whole-genome coverage and cannot be computed from long reads alone.
- The assembly is largely fragmented with few contigs over 20 Kb — the depth statistic is only stated to be stable with up to 20 of the largest contigs longer than 20 Kb, so bacterial-genome assumptions may not transfer.
- The goal is de novo long-read assembly rather than scaffolding a short-read draft (e.g. use Celera Assembler on corrected long reads instead).

## Inputs

- Draft assembly FASTA (e.g. spades/contigs.fasta from SPAdes --careful on trimmed Illumina MiSeq paired-end reads, length-sorted via jsa.seq.sort -r -n)
- Per-contig short-read coverage derived from the Illumina paired-end reads used for the assembly

## Outputs

- Contig copy-number/multiplicity classification (unique vs repetitive) used internally by jsa.np.npscarf
- Estimated whole-genome depth (depthg) at termination of the iterative update

## How to apply

First produce the draft assembly (npScarf expects a SPAdes output like spades/contigs.fasta from Illumina MiSeq paired-end reads, trimmed with trimmomatic) and sort contigs by length (jsa.seq.sort -r -n) so the largest contigs are identifiable. Estimate the genome depth by initializing depthg to the coverage of the largest contig, then iteratively update the estimate until termination. Compute each contig's multiplicity as the ratio of its short-read coverage to this genome-wide depth, and declare a contig unique when the ratio falls below the threshold theta = 1.5. The depth statistic is empirically stable when based on up to the 20 largest contigs longer than 20 Kb, which are most likely unique contigs in bacterial genomes. The resulting classification drives downstream npScarf behavior: bridges are built only between unique contigs, while repetitive contigs are inserted into scaffold gaps via alignment to bridge-read consensus sequences.

## Related tools

- **npScarf (jsa.np.npscarf)** (Performs the copy-number estimation: initializes depthg to the largest-contig coverage, iteratively updates it, and applies threshold theta = 1.5 to label contigs unique vs repetitive.) — https://github.com/mdcao/npScarf
- **SPAdes** (Produces the draft Illumina contigs whose per-contig coverage is the basis for multiplicity classification (spades/contigs.fasta).)
- **Japsa (jsa.seq.sort)** (Length-sorts the draft contigs (-r -n) so the largest contigs used to seed the genome-depth estimate are available.) — https://github.com/mdcao/japsa

## Examples

```
jsa.seq.sort -r -n --input spades/contigs.fasta --output Kp2146_spades.fasta && bwa index Kp2146_spades.fasta && bwa mem -t 10 -k11 -W20 -r10 -A1 -B1 -O1 -E1 -L0 -a -Y Kp2146_spades.fasta Kp2146_ONT.fastq | jsa.np.npscarf -input - -format sam -seq Kp2146_spades.fasta -prefix Kp2146-batch
```

## Evaluation signals

- For bacterial genomes, verify the depth estimate converges using up to the 20 largest contigs longer than 20 Kb — larger deviations suggest mis-seeding.
- Check that declared-unique contigs have contig coverage / genome depth below the theta = 1.5 threshold; contigs at or above 1.5 must be treated as repetitive.
- Downstream invariant: npScarf bridges should only join contigs classified as unique; repetitive contigs should appear only inserted into gaps aligned to bridge-read consensus.
- Confirm the classification is computed from the sorted draft (Kp2146_spades.fasta) whose coverage derives from the trimmed Illumina MiSeq data, not from nanopore alignments.

## Limitations

- The theta = 1.5 threshold was set from the authors' experience ('In our experience, the statistic is stable with up to 20 of the largest contigs longer than 20 Kb') rather than a principled derivation, so it may need re-tuning for other genomes.
- The pipeline still requires short-read data, which cannot be generated in real-time with current technology; the coverage-based classification inherits that dependency.
- npScarf is no longer maintained — npGraph (hsnguyen/assembly) is the designated replacement.

## Evidence

- [discussion] npScarf determines the multiplicity of each contig in the assembly by comparing short read sequencing coverage of the contig to that of the whole genome.: "npScarf determines the multiplicity of each contig in the assembly by comparing short read sequencing coverage of the contig to that of the whole genome."
- [discussion] npScarf first initializes depthg to that of the largest contig: It then iteratively updates the estimate: "npScarf first initializes depthg to that of the largest contig: It then iteratively updates the estimate"
- [discussion] npScarf set the threshold θ to 1.5.: "npScarf set the threshold θ to 1.5."
- [readme] Normally this is the output from running SPAdes on Illumina MiSeq paired end reads.: "Normally this is the output from running SPAdes on Illumina MiSeq paired end reads."
