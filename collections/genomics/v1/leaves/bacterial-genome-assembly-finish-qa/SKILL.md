---
name: bacterial-genome-assembly-finish-qa
description: Use when finishing a bacterial draft assembly from Illumina MiSeq paired-end
  reads plus Oxford Nanopore long reads, and you must scaffold contigs, fill gaps
  with repetitive contigs, and verify the npScarf output artifacts.
when_to_use_negative:
- Input is already a finished, closed genome — no bridging or gap-filling is needed
- 'You only have long reads and no short-read assembly: npScarf scaffolds a short-read
  draft (SPAdes output) and cannot de novo assemble the genome'
- You need de novo long-read assembly of raw nanopore data rather than contig-ordering/gap-filling
  of an existing draft (the successor npGraph, not npScarf, is under development for
  streaming assembly)
edam_operation: http://edamontology.org/operation_0526
edam_topics: []
tools:
- name: npScarf
  role: 'Scaffolding engine (jsa.np.npscarf): classifies contig multiplicity, builds
    scored bridges, greedily merges contigs, fills gaps, and writes fin.fasta/fin.japsa'
  repo: https://github.com/mdcao/npScarf
- name: BWA-MEM
  role: Aligns nanopore long reads to the draft assembly with the fixed parameter
    set -k11 -W20 -r10 -A1 -B1 -O1 -E1 -L0 -a -Y, streaming SAM into npScarf
- name: SPAdes
  role: Assembles trimmed Illumina MiSeq paired-end reads (--careful, -t 16) into
    the draft contigs.fasta used as the scaffolding backbone
- name: jsa.seq.sort
  role: Sorts SPAdes contigs by length (-r -n) to produce the length-ordered draft
    FASTA
  repo: https://github.com/mdcao/japsa
- name: npReader
  role: Streams real-time base-called nanopore reads ('jsa.np.npreader --realtime
    --folder Downloads --fail --stat --number --output -') into the BWA-MEM/npScarf
    pipeline
  repo: https://github.com/mdcao/npReader
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/ncomms14515
    title: Scaffolding and completing genome assemblies in real-time with nanopore
      sequencing
  source_content_hash: sha256:d8e8f2ea1529db5f36700385ee17baeea2ecf2251b9109ac1df5da769239b479
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/mdcao/npScarf
  tools:
  - BWA-MEM
  - Japsa (jsa.seq.sort)
  - SPAdes
  - jsa.seq.sort
  - npReader
  - npScarf
  - npScarf (jsa.np.npscarf)
  edam_operation: http://edamontology.org/operation_0526
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 12
---

# bacterial-genome-assembly-finish-qa

## Summary

Reconstruct and quality-check the npScarf hybrid scaffolding workflow that joins SPAdes short-read contigs into genome scaffolds using streaming nanopore reads, ending in the canonical prefix.fin.fasta / prefix.fin.japsa outputs.

## When to use

Use when finishing a bacterial draft assembly from Illumina MiSeq paired-end reads plus Oxford Nanopore long reads, and you must scaffold contigs, fill gaps with repetitive contigs, and verify the npScarf output artifacts.

## When NOT to use

- Input is already a finished, closed genome — no bridging or gap-filling is needed
- You only have long reads and no short-read assembly: npScarf scaffolds a short-read draft (SPAdes output) and cannot de novo assemble the genome
- You need de novo long-read assembly of raw nanopore data rather than contig-ordering/gap-filling of an existing draft (the successor npGraph, not npScarf, is under development for streaming assembly)

## Inputs

- Trimmed Illumina MiSeq paired-end FASTQ files (e.g. Kp2146_paired_1.fastq.gz / Kp2146_paired_2.fastq.gz, trimmed with trimmomatic V0.32)
- Nanopore long-read FASTQ/FASTA (raw Kpn2146 data via ENA run accession ERR868296)
- SPAdes draft assembly FASTA (spades/contigs.fasta), length-sorted to Kp2146_spades.fasta with a bwa index
- SAM/BAM alignments between draft assembly and nanopore reads (or raw long reads when BWA-MEM is integrated into jsa.np.npscarf)

## Outputs

- prefix.fin.fasta — final genome scaffold in FASTA format
- prefix.fin.japsa — scaffold in JAPSA format
- prefix.anno.japsa — features detected after scaffolding (real-time mode with annotation enabled)
- Real-time assembly metrics used to decide sequencing-run termination

## How to apply

Assemble the trimmed Illumina MiSeq paired-end reads with SPAdes (--careful, -t 16) to obtain spades/contigs.fasta, then sort contigs by length with 'jsa.seq.sort -r -n' to produce the draft FASTA and build a 'bwa index' on it. Align the nanopore reads to the draft with BWA-MEM using the fixed parameter set -k11 -W20 -r10 -A1 -B1 -O1 -E1 -L0 -a -Y and pipe the SAM stream into 'jsa.np.npscarf -input - -format sam -seq <draft> -prefix <prefix>'; npScarf classifies contigs by copy number against whole-genome coverage (multiplicity threshold theta = 1.5), groups consistently supporting long reads into scored bridges, merges contigs greedily Kruskal-style, temporarily fills gaps with bridge-read consensus, and inserts repetitive contigs into gaps. In real-time mode add '-realtime' and stream base-called reads via 'jsa.np.npreader --realtime --folder Downloads --fail --stat --number --output -' with BWA-MEM '-K 3000', terminating the run once assembly metrics report sufficient quality. Success is judged by npScarf terminating cleanly and emitting the two canonical files prefix.fin.fasta (scaffold in FASTA) and prefix.fin.japsa (scaffold in JAPSA format); annotation mode instead produces prefix.anno.japsa.

## Related tools

- **npScarf** (Scaffolding engine (jsa.np.npscarf): classifies contig multiplicity, builds scored bridges, greedily merges contigs, fills gaps, and writes fin.fasta/fin.japsa) — https://github.com/mdcao/npScarf
- **BWA-MEM** (Aligns nanopore long reads to the draft assembly with the fixed parameter set -k11 -W20 -r10 -A1 -B1 -O1 -E1 -L0 -a -Y, streaming SAM into npScarf)
- **SPAdes** (Assembles trimmed Illumina MiSeq paired-end reads (--careful, -t 16) into the draft contigs.fasta used as the scaffolding backbone)
- **jsa.seq.sort** (Sorts SPAdes contigs by length (-r -n) to produce the length-ordered draft FASTA) — https://github.com/mdcao/japsa
- **npReader** (Streams real-time base-called nanopore reads ('jsa.np.npreader --realtime --folder Downloads --fail --stat --number --output -') into the BWA-MEM/npScarf pipeline) — https://github.com/mdcao/npReader

## Examples

```
bwa mem -t 10 -k11 -W20 -r10 -A1 -B1 -O1 -E1 -L0 -a -Y Kp2146_spades.fasta Kp2146_ONT.fastq | jsa.np.npscarf -input - -format sam -seq Kp2146_spades.fasta -prefix Kp2146-batch
```

## Evaluation signals

- npScarf terminates successfully and both canonical output files exist: prefix.fin.fasta (FASTA scaffold) and prefix.fin.japsa (JAPSA-format scaffold)
- Contig classification is sane: multiplicity statistic is stable when computed on up to 20 of the largest contigs longer than 20 Kb (theta = 1.5 uniqueness threshold)
- Every merged pair of contigs corresponds to a bridge that reached the pre-defined scoring threshold; contradicting bridges should have the lowest-scoring one broken in favor of the current bridge
- Scaffold completeness/accuracy improves over the SPAdes draft while using less long-read data than competitors (npScarf claims more complete and accurate assemblies with much less data)
- In real-time mode, the SAM stream is fed block-wise (-K 3000) and assembly metrics update at minute-level temporal resolution during the run

## Limitations

- npScarf is real-time at the temporal resolution of minutes, not at the millisecond level required to update with the addition of each base
- The pipeline still requires short-read data, which cannot be generated in real-time with current technology, and still relies on a base-caller (latency reduced by Nanocall/DeepNano)
- The gap is only temporarily filled with the consensus sequence of bridging reads; the bridge scoring threshold is described only as 'pre-defined' with no value given in the text
- npScarf is no longer maintained; npGraph in hsnguyen/assembly is the successor under development
- Always check 'jsa.np.npscarf --help' before running, since the parameter list can change significantly between versions

## Evidence

- [readme] Normally the tool generate two files: *prefix*.fin.fasta and *prefix*.fin.japsa which indicate the result scaffolders in FASTA and JAPSA format.: "Normally the tool generate two files: *prefix*.fin.fasta and *prefix*.fin.japsa which 
indicate the result scaffolders in FASTA and JAPSA format."
- [discussion] npScarf groups consistently supporting reads into scored bridges and merges contigs greedily: "npScarf groups reads that consistently support a particular relative position into a bridge and assigns the bridge a score"
- [discussion] Real-time streaming alignment with BWA-MEM: "npScarf employs a mapping method that supports streaming processing such as BWA-MEM40 to align a small number of long reads to the existing assembly as they arrive."
- [abstract] Real-time assembly metrics enable early termination of the sequencing run: "It reports assembly metrics in real-time so the sequencing run can be terminated once an assembly of sufficient quality is obtained."
- [readme] Fixed BWA-MEM parameters piped into jsa.np.npscarf in batch mode: "bwa mem -t 10 -k11 -W20 -r10 -A1 -B1 -O1 -E1 -L0 -a -Y Kp2146_spades.fasta Kp2146_ONT.fastq"
