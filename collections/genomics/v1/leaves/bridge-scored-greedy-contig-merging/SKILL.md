---
name: bridge-scored-greedy-contig-merging
description: Use when scaffolding a short-read (Illumina MiSeq/SPAdes) draft assembly
  with streaming Oxford Nanopore long reads, and long reads span pairs of unique contigs
  whose relative order and orientation must be resolved greedily.
when_to_use_negative:
- Input is already a finished scaffold or a closed genome — no bridging or gap-filling
  is needed.
- No long reads spanning two unique contigs are available (npScarf builds the backbone
  from reads aligned to two unique contigs; without them no bridges form).
- Millisecond-level real-time updating is required — npScarf operates at minute-level
  temporal resolution and still depends on a base-caller and pre-generated short-read
  data.
edam_operation: http://edamontology.org/operation_0525
edam_topics:
- http://edamontology.org/topic_0196
- http://edamontology.org/topic_0622
tools:
- name: npScarf
  role: jsa.np.npscarf implements the multiplicity classification, bridge scoring,
    Kruskal-based greedy merging, gap filling and repetitive-contig insertion
  repo: https://github.com/mdcao/npScarf
- name: BWA-MEM
  role: aligns nanopore reads to the draft assembly with fixed parameters (-k11 -W20
    -r10 -A1 -B1 -O1 -E1 -L0 -a -Y) supporting streaming SAM output
- name: SPAdes
  role: produces the short-read draft contigs (--careful, 16 threads) that form the
    input assembly
- name: Japsa (jsa.seq.sort)
  role: sorts SPAdes contigs by length (-r -n) before indexing and scaffolding; npScarf
    module is bundled within Japsa
  repo: https://github.com/mdcao/japsa
- name: npReader
  role: streams base-called nanopore data in real-time mode (jsa.np.npreader --realtime
    --folder Downloads) into the BWA-MEM/npScarf pipeline
  repo: https://github.com/mdcao/npReader
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/ncomms14515
    title: Scaffolding and completing genome assemblies in real-time with nanopore
      sequencing
  source_content_hash: sha256:3db072945a6da0abdc0d813506737533611b812d7dc83fd02f0479421c97e64f
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/mdcao/npScarf
  tools:
  - BWA-MEM
  - Japsa (jsa.seq.sort)
  - SPAdes
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
yes_rate: 0.6666666666666666
n_claims: 9
---

# bridge-scored-greedy-contig-merging

## Summary

Kruskal-style greedy contig merging from long-read bridges: npScarf classifies SPAdes contigs by copy number, groups nanopore reads consistently supporting a relative position into scored bridges, merges unique contigs greedily once a bridge scoring threshold is reached, fills gaps with bridge-read consensus, and inserts repetitive contigs into those gaps.

## When to use

Use when scaffolding a short-read (Illumina MiSeq/SPAdes) draft assembly with streaming Oxford Nanopore long reads, and long reads span pairs of unique contigs whose relative order and orientation must be resolved greedily.

## When NOT to use

- Input is already a finished scaffold or a closed genome — no bridging or gap-filling is needed.
- No long reads spanning two unique contigs are available (npScarf builds the backbone from reads aligned to two unique contigs; without them no bridges form).
- Millisecond-level real-time updating is required — npScarf operates at minute-level temporal resolution and still depends on a base-caller and pre-generated short-read data.

## Inputs

- Draft assembly FASTA from SPAdes on Illumina MiSeq paired-end reads (e.g. spades/contigs.fasta, sorted to Kp2146_spades.fasta)
- Nanopore long reads in FASTQ/FASTA (e.g. Kp2146_ONT.fastq) or SAM/BAM alignments from BWA-MEM streamed on stdin (-input - -format sam)
- BWA index built on the draft assembly (bwa index Kp2146_spades.fasta)

## Outputs

- prefix.fin.fasta — final scaffolded genome in FASTA (e.g. Kp2146-batch.fin.fasta)
- prefix.fin.japsa — final scaffold in JAPSA format
- prefix.anno.japsa — in real-time mode when annotation analysis is enabled, features detected after scaffolding

## How to apply

Assemble the trimmed Illumina paired-end reads with SPAdes (--careful), sort contigs with jsa.seq.sort, and index the draft with bwa index. Align nanopore reads to the draft with BWA-MEM using the fixed parameter set -t 10 -k11 -W20 -r10 -A1 -B1 -O1 -E1 -L0 -a -Y, streaming SAM into jsa.np.npscarf. npScarf first determines contig multiplicity by comparing short-read coverage of each contig to the whole-genome coverage (initializing depthg to the largest contig coverage, threshold θ = 1.5 for uniqueness), then builds the genome backbone from unique contigs: long reads aligned to two unique contigs are grouped into bridges with scores, and a greedy Kruskal-based strategy merges contigs once a bridge reaches a pre-defined scoring threshold. On contradiction, npScarf revisits previous bridges, breaks the lowest-scoring contradicting bridge, and uses the current bridge instead. Gaps are temporarily filled with the consensus of bridging reads, then repetitive contigs aligned to that consensus are inserted into the gaps.

## Related tools

- **npScarf** (jsa.np.npscarf implements the multiplicity classification, bridge scoring, Kruskal-based greedy merging, gap filling and repetitive-contig insertion) — https://github.com/mdcao/npScarf
- **BWA-MEM** (aligns nanopore reads to the draft assembly with fixed parameters (-k11 -W20 -r10 -A1 -B1 -O1 -E1 -L0 -a -Y) supporting streaming SAM output)
- **SPAdes** (produces the short-read draft contigs (--careful, 16 threads) that form the input assembly)
- **Japsa (jsa.seq.sort)** (sorts SPAdes contigs by length (-r -n) before indexing and scaffolding; npScarf module is bundled within Japsa) — https://github.com/mdcao/japsa
- **npReader** (streams base-called nanopore data in real-time mode (jsa.np.npreader --realtime --folder Downloads) into the BWA-MEM/npScarf pipeline) — https://github.com/mdcao/npReader

## Examples

```
bwa mem -t 10 -k11 -W20 -r10 -A1 -B1 -O1 -E1 -L0 -a -Y Kp2146_spades.fasta Kp2146_ONT.fastq | jsa.np.npscarf -input - -format sam -seq Kp2146_spades.fasta -prefix Kp2146-batch
```

## Evaluation signals

- npScarf terminates successfully and produces exactly the two canonical output files prefix.fin.fasta and prefix.fin.japsa
- Contig multiplicity calls are stable: the depth statistic is initialized to the largest contig coverage and is stable with up to 20 of the largest contigs longer than 20 Kb treated as unique
- Scaffold completeness/accuracy improves over the SPAdes draft with less long-read data than competitor scaffolders (SSPACE-LongRead, Cerulean, LINKS)
- In real-time mode, assembly metrics update at minute-level resolution and the run can be terminated once sufficient quality is reached
- Contradicting bridges are resolved by breaking the lowest-scoring contradicting bridge and keeping the current bridge, so the final scaffold should contain no conflicting bridge pairs

## Limitations

- Pipeline is real-time at minute-level temporal resolution, not at the millisecond level required to update with each added base.
- Pipeline still requires short-read data, which cannot be generated in real-time with current technology, and relies on a base-caller (latency reduced by fast real-time base-callers such as Nanocall and DeepNano).
- npScarf is no longer maintained; npGraph is under development as the replacement.
- Bridge scoring threshold is only described as 'pre-defined' with no value given; θ = 1.5 was set from authors' experience rather than a principled derivation; gaps are 'temporarily' filled with bridge-read consensus without stated downstream validation.

## Evidence

- [discussion] npScarf groups reads that consistently support a particular relative position into a bridge and assigns the bridge a score: "npScarf groups reads that consistently support a particular relative position into a bridge and assigns the bridge a score"
- [discussion] greedy Kruskal-based merging: "npScarf uses a greedy strategy based on Kruskal's algorithm39"
- [discussion] gap filling and repetitive contig insertion: "the gap is temporarily filled with the consensus sequence of the reads forming the bridge. npScarf then identifies repetitive contigs that are aligned to this consensus sequence"
- [discussion] contig multiplicity via coverage ratio and θ = 1.5 threshold: "npScarf determines the multiplicity of each contig in the assembly by comparing short read sequencing coverage of the contig to that of the whole genome."
- [readme] batch-mode invocation from README tutorial: "bwa mem -t 10 -k11 -W20 -r10 -A1 -B1 -O1 -E1 -L0 -a -Y Kp2146_spades.fasta Kp2146_ONT.fastq  | jsa.np.npscarf -input - -format sam -seq Kp2146_spades.fasta -prefix Kp2146-batch"
- [readme] canonical output files: "Normally the tool generate two files: *prefix*.fin.fasta and *prefix*.fin.japsa which 
indicate the result scaffolders in FASTA and JAPSA format."
