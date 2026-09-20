---
name: illumina-fastq-read-handling
description: Use when starting a DBG2OLC hybrid assembly from Illumina FASTQ data
  (e.g. S.cer w303 2x300 reads) and you need a 50x subset and raw DBG contigs (Contigs.txt)
  before overlap/layout with long reads.
when_to_use_negative:
- The task requires polished, scaffolded, or repeat-resolved short-read contigs —
  DBG2OLC explicitly requires raw DBG contigs without gap closing or scaffolding
- The input is long-read (PacBio/ONT) data rather than Illumina FASTQ — those reads
  belong to Step2 overlap/layout and Step3 consensus, not SparseAssembler preassembly
- You want a purely short-read finished assembly — this skill only produces intermediate
  contigs meant as anchors for long erroneous 3GS reads
edam_operation: http://edamontology.org/operation_0524
edam_topics:
- http://edamontology.org/topic_0196
- http://edamontology.org/topic_3693
tools:
- name: SparseAssembler
  role: Preassembles ~50x Illumina FASTQ reads into raw de Bruijn graph contigs (Contigs.txt);
    supports LD 1 reloading to retune NodeCovTh/EdgeCovTh
  repo: https://github.com/yechengxi/SparseAssembler
- name: AssemblyUtility
  role: SelectLongestReads extracts a fixed-coverage subset (e.g. sum 600000000 longest
    0) from the Illumina FASTQ before assembly
  repo: https://github.com/yechengxi/AssemblyUtility
- name: QUAST 3.0
  role: Evaluates the SparseAssembler contigs in default setting to report the N50
    (29 kbp default run, 32 kbp tuned run on the w303 dataset)
- name: DBG2OLC
  role: Consumes the resulting Contigs.txt as short accurate contig anchors in the
    Step2 overlap and layout stage
  repo: https://github.com/yechengxi/DBG2OLC
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/srep31900
    title: 'DBG2OLC: Efficient Assembly of Large Genomes Using Long Erroneous Reads
      of the Third Generation Sequencing Technologies'
  source_content_hash: sha256:3dfacd4fadd327a66fa37046d940215178b4eedf0fac0cd9c53d952264b0f5c5
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/yechengxi/SparseAssembler
  tools:
  - AssemblyUtility
  - DBG2OLC
  - QUAST 3.0
  - SparseAssembler
  edam_operation: http://edamontology.org/operation_0524
  edam_topics:
  - http://edamontology.org/topic_0196
  - http://edamontology.org/topic_3693
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# illumina-fastq-read-handling

## Summary

Handling of highly accurate Illumina FASTQ reads in the DBG2OLC hybrid-assembly workflow: selecting a 50x subset with SelectLongestReads and preassembling them into raw, unscaffolded de Bruijn graph contigs with SparseAssembler. This is the short-read foundation that anchors long erroneous 3GS reads in later steps.

## When to use

Use when starting a DBG2OLC hybrid assembly from Illumina FASTQ data (e.g. S.cer w303 2x300 reads) and you need a 50x subset and raw DBG contigs (Contigs.txt) before overlap/layout with long reads.

## When NOT to use

- The task requires polished, scaffolded, or repeat-resolved short-read contigs — DBG2OLC explicitly requires raw DBG contigs without gap closing or scaffolding
- The input is long-read (PacBio/ONT) data rather than Illumina FASTQ — those reads belong to Step2 overlap/layout and Step3 consensus, not SparseAssembler preassembly
- You want a purely short-read finished assembly — this skill only produces intermediate contigs meant as anchors for long erroneous 3GS reads

## Inputs

- Illumina short-read FASTQ file(s) (e.g. Illumina_500bp_2x300_R1.fastq.gz from the S.cer w303 dataset)
- Genome size estimate in bp (GS, e.g. 12000000 for the 12 Mbp yeast genome)
- Optional: previously saved SparseAssembler k-mer graph for LD 1 reloading

## Outputs

- Contigs.txt — raw, unscaffolded de Bruijn graph contigs in fasta format for use by DBG2OLC
- Saved SparseAssembler k-mer graph enabling LD 1 reload and parameter retuning
- QUAST-reported assembly statistics (e.g. N50 of the contig set)

## How to apply

Optionally select a read subset with SelectLongestReads (e.g. 'sum 600000000 longest 0' on Illumina_500bp_2x300_R1.fastq.gz to produce Illumina_50x.fastq). Run SparseAssembler in a first pass with 'LD 0 k 51 g 15 NodeCovTh 1 EdgeCovTh 0 GS 12000000' on the FASTQ file to build raw de Bruijn graph contigs; at ~50x coverage NodeCovTh 1 EdgeCovTh 0 normally produces good results. If the first run is suboptimal (e.g. the file head is lower quality), relaunch with 'LD 1 NodeCovTh 2 EdgeCovTh 1' to reload the saved k-mer graph and tune only the node/edge coverage thresholds for a cleaner graph. Critically, the contigs must be raw DBG contigs — do not apply repeat resolving, gap closing, or scaffolding, since those heuristics introduce errors that degrade the final hybrid assembly. The output Contigs.txt is then fed to DBG2OLC in Step2 as anchors compressed into the long reads.

## Related tools

- **SparseAssembler** (Preassembles ~50x Illumina FASTQ reads into raw de Bruijn graph contigs (Contigs.txt); supports LD 1 reloading to retune NodeCovTh/EdgeCovTh) — https://github.com/yechengxi/SparseAssembler
- **AssemblyUtility** (SelectLongestReads extracts a fixed-coverage subset (e.g. sum 600000000 longest 0) from the Illumina FASTQ before assembly) — https://github.com/yechengxi/AssemblyUtility
- **QUAST 3.0** (Evaluates the SparseAssembler contigs in default setting to report the N50 (29 kbp default run, 32 kbp tuned run on the w303 dataset))
- **DBG2OLC** (Consumes the resulting Contigs.txt as short accurate contig anchors in the Step2 overlap and layout stage) — https://github.com/yechengxi/DBG2OLC

## Examples

```
./SparseAssembler LD 0 k 51 g 15 NodeCovTh 1 EdgeCovTh 0 GS 12000000 f ../Illumina_data/Illumina_50x.fastq
```

## Evaluation signals

- Contigs.txt exists and is in fasta format, suitable to pass to DBG2OLC via the Contigs parameter
- QUAST 3.0 (default setting) N50 is approximately 29 kbp for the default run (LD 0, NodeCovTh 1, EdgeCovTh 0) on the S.cer w303 dataset
- After the LD 1 tuned run (NodeCovTh 2, EdgeCovTh 1), the QUAST N50 improves to approximately 32 kbp
- Contigs are raw DBG contigs with no scaffolding/gap-closing artifacts (verify no scaffold joining or filled gaps were applied)
- Read subset selection consumed the expected total bases (e.g. ~600 Mbp ≈ 50x of the 12 Mbp genome)

## Limitations

- The beginning of the sequencing file may be of lower quality, degrading the first-pass assembly; retuning with LD 1 mitigates but does not guarantee the best result
- For other genomes or coverages the first run may not generate the best result and NodeCovTh/EdgeCovTh must be fine-tuned iteratively
- Combining DBG2OLC with other assembly/error-correction tools may produce worse results because many such tools introduce errors

## Evidence

- [readme] Raw DBG contigs required, no scaffolding: "Please make sure they are the raw DBG contigs without using repeat resolving techniques such as gap closing or scaffolding."
- [readme] Default SparseAssembler command and N50: "./SparseAssembler LD 0 k 51 g 15 NodeCovTh 1 EdgeCovTh 0 GS 12000000 f ../Illumina_data/Illumina_50x.fastq"
- [readme] Tuned reload command and N50 improvement: "./SparseAssembler LD 1 NodeCovTh 2 EdgeCovTh 1 k 51 g 15 GS 12000000 f ../Illumina_data/Illumina_50x.fastq"
- [results] Article preassembly step: "we used SparseAssembler (Ye et al.5) to preassemble 50x Illumina short reads into contigs and then to compress the 3GS reads"
- [methods] Step1 description from the article: "## Step1. Use an accurate DBG-assembler to construct short but accurate contigs."
