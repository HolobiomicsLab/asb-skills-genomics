---
name: batch-mode-hybrid-genome-scaffolding
description: Use when all nanopore reads have already been sequenced and base-called
  (a complete FASTQ/FASTA exists) and the goal is to scaffold and gap-fill an Illumina/SPAdes
  draft assembly offline, e.g. reducing contig count and raising N50.
when_to_use_negative:
- The nanopore data are still being generated and you need assembly metrics during
  the run — use npScarf's real-time streaming mode (jsa.np.npscarf -realtime) with
  npReader/jsa.np.timeEmulate instead.
- You have only long reads and no short-read draft — npScarf scaffolds an existing
  short-read assembly and requires an Illumina-based draft as input.
- npScarf is unmaintained for your use case; the developers point to npGraph (github.com/hsnguyen/assembly)
  as the replacement for streaming assembly.
edam_operation: http://edamontology.org/operation_0525
edam_topics:
- http://edamontology.org/topic_0196
- http://edamontology.org/topic_0622
tools:
- name: npScarf (jsa.np.npscarf)
  role: 'Core scaffolder: estimates contig multiplicity, builds scored bridges from
    long reads, and merges contigs greedily via Kruskal''s algorithm'
  repo: https://github.com/mdcao/npScarf
- name: SPAdes
  role: Assembles Illumina paired-end reads into the draft contigs (>= 3.5, --careful
    optional) that npScarf scaffolds
- name: BWA-MEM
  role: Aligns nanopore long reads to the draft with the fixed parameter set -k11
    -W20 -r10 -A1 -B1 -O1 -E1 -L0 -a -Y, piped as SAM into npScarf
- name: japsa
  role: Package providing jsa.seq.sort (contig length sorting) and jsa.np.npscarf;
    installed from source with make install
  repo: https://github.com/mdcao/japsa
- name: QUAST
  role: Evaluates final scaffolds (contig count, N50, completeness) against the pre-scaffolding
    draft
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/ncomms14515
    title: Scaffolding and completing genome assemblies in real-time with nanopore
      sequencing
  source_content_hash: sha256:77353ecc62842403ebbb7629fa5adbbeb8e0b5d8910f1adc3ed17aa40a99bb0c
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/mdcao/npScarf
  tools:
  - BWA-MEM
  - QUAST
  - SPAdes
  - japsa
  - npReader
  - npScarf
  - npScarf (jsa.np.npscarf)
  edam_operation: http://edamontology.org/operation_0525
  edam_topics:
  - http://edamontology.org/topic_0196
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8823529411764706
n_claims: 17
---

# batch-mode-hybrid-genome-scaffolding

## Summary

Batch-mode hybrid scaffolding with npScarf (jsa.np.npscarf): align all available nanopore long reads against a SPAdes short-read draft with BWA-MEM and merge contigs via Kruskal-based greedy bridging to produce a more complete assembly in a single post-sequencing pass.

## When to use

Use when all nanopore reads have already been sequenced and base-called (a complete FASTQ/FASTA exists) and the goal is to scaffold and gap-fill an Illumina/SPAdes draft assembly offline, e.g. reducing contig count and raising N50.

## When NOT to use

- The nanopore data are still being generated and you need assembly metrics during the run — use npScarf's real-time streaming mode (jsa.np.npscarf -realtime) with npReader/jsa.np.timeEmulate instead.
- You have only long reads and no short-read draft — npScarf scaffolds an existing short-read assembly and requires an Illumina-based draft as input.
- npScarf is unmaintained for your use case; the developers point to npGraph (github.com/hsnguyen/assembly) as the replacement for streaming assembly.

## Inputs

- Sorted SPAdes draft assembly FASTA (e.g. from Illumina MiSeq paired-end reads, trimmed with trimmomatic V0.32)
- Nanopore long-read FASTQ/FASTA file (complete, base-called, e.g. ENA ERP008615 for S. Typhi H58)
- BWA index built on the draft assembly

## Outputs

- <prefix>.fin.fasta — final scaffolded assembly in FASTA format
- <prefix>.fin.japsa — scaffold structure in JAPSA format
- QUAST report of contig count, N50, and completeness

## How to apply

Assemble Illumina paired-end reads (e.g. MiSeq 300 bp PE, trimmed with trimmomatic) with SPAdes, sort contigs by descending length using jsa.seq.sort -r -n, and build a bwa index on the draft FASTA. Align the full nanopore read set with bwa mem using npScarf's fixed parameter set -k11 -W20 -r10 -A1 -B1 -O1 -E1 -L0 -a -Y and pipe the SAM stream into jsa.np.npscarf -input - -format sam -seq <draft> -prefix <prefix>. npScarf determines each contig's multiplicity by comparing its short-read coverage to whole-genome coverage (uniqueness threshold theta = 1.5), builds the genome backbone from reads bridging two unique contigs, scores bridges and merges contigs greedily with Kruskal's algorithm, fills gaps with the consensus of bridging reads, and inserts repetitive contigs aligned to the consensus. Evaluate the resulting <prefix>.fin.fasta with QUAST, comparing contig count and N50 against the pre-scaffolding draft (e.g. the S. Typhi H58 batch run collapsed the draft to 8 contigs with N50 ~864 kb).

## Related tools

- **npScarf (jsa.np.npscarf)** (Core scaffolder: estimates contig multiplicity, builds scored bridges from long reads, and merges contigs greedily via Kruskal's algorithm) — https://github.com/mdcao/npScarf
- **SPAdes** (Assembles Illumina paired-end reads into the draft contigs (>= 3.5, --careful optional) that npScarf scaffolds)
- **BWA-MEM** (Aligns nanopore long reads to the draft with the fixed parameter set -k11 -W20 -r10 -A1 -B1 -O1 -E1 -L0 -a -Y, piped as SAM into npScarf)
- **japsa** (Package providing jsa.seq.sort (contig length sorting) and jsa.np.npscarf; installed from source with make install) — https://github.com/mdcao/japsa
- **QUAST** (Evaluates final scaffolds (contig count, N50, completeness) against the pre-scaffolding draft)

## Examples

```
bwa mem -t 10 -k11 -W20 -r10 -A1 -B1 -O1 -E1 -L0 -a -Y draft.fasta nanopore.fastq | jsa.np.npscarf -input - -format sam -seq draft.fasta -prefix sample-batch
```

## Evaluation signals

- QUAST on <prefix>.fin.fasta shows reduced contig count and increased N50 versus the input SPAdes draft (e.g. S. Typhi H58: 34 contigs/N50 319 kb pre-scaffolding vs 8 contigs/N50 ~864 kb in batch mode).
- Both expected output files exist: <prefix>.fin.fasta and <prefix>.fin.japsa.
- Total scaffolded assembly length remains consistent with genome size and no contigs were dropped or duplicated by the merge step.
- Coverage-based multiplicity statistic is stable, with up to 20 of the largest contigs longer than 20 Kb treated as the most likely unique contigs in bacterial genomes.
- Re-running with the fixed BWA-MEM parameter set reproduces the same scaffold structure (deterministic merge of scored bridges).

## Limitations

- npScarf is no longer maintained; npGraph (hsnguyen/assembly) is the replacement under development.
- The uniqueness threshold theta = 1.5 was set from the authors' experience and the bridge scoring threshold is only described as 'pre-defined' with no value given in the paper — reproducibility of these choices depends on the released code.
- The pipeline still requires short-read data, which cannot be generated in real-time, and it relies on a base-caller whose latency affects any streaming variant.
- Gaps are temporarily filled with the consensus of bridging reads; no downstream validation of gap accuracy is stated.

## Evidence

- [discussion] npScarf groups reads that consistently support a particular relative position into a bridge and assigns the bridge a score: [evidence span withheld — anchor status: not_found]
- [discussion] Multiplicity threshold: "npScarf determines the multiplicity of each contig in the assembly by comparing short read sequencing coverage of the contig to that of the whole genome."
- [readme] Fixed aligner parameters: "We use BWA-MEM as the recommended aligner 
with the fixed parameter set as follow:

	bwa mem -k11 -W20 -r10 -A1 -B1 -O1 -E1 -L0 -a -Y"
- [readme] Batch-mode command: "$ bwa mem -t 10 -k11 -W20 -r10 -A1 -B1 -O1 -E1 -L0 -a -Y Kp2146_spades.fasta Kp2146_ONT.fastq  | jsa.np.npscarf -input - -format sam -seq Kp2146_spades.fasta -prefix Kp2146-batch"
- [readme] Output files: "Normally the tool generate two files: *prefix*.fin.fasta and *prefix*.fin.japsa which 
indicate the result scaffolders in FASTA and JAPSA format."
