---
name: nanopore-direct-rna-basecalling
description: Use when raw nanopore fast5 dRNA-Seq reads (HEK293 WT/KO, Arabidopsis
  col0/vir1, or synthetic RL/SL oligos) must be converted to basecalled sequences
  and hidden features as the first step of an mAFiA m6A workflow.
when_to_use_negative:
- Input is already basecalled FASTQ/FASTA and only alignment or modification calling
  is needed
- The goal is DNA nanopore basecalling — RODAN is trained for direct RNA sequencing
  data
- Training a new basecaller from scratch rather than applying the deposited pretrained
  model
edam_operation: http://edamontology.org/operation_3182
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_0080
tools:
- name: RODAN
  role: Fully convolutional basecaller whose adapted basecall.py converts fast5 signals
    to rodan.fasta; also supplies hidden features for mAFiA
  repo: https://github.com/biodlab/RODAN
- name: mAFiA
  role: Consumes RODAN basecalls and extracts 768-d features from backbone.torch layer
    convlayers.conv21 for logistic-regression m6A prediction
  repo: https://github.com/dieterich-lab/mAFiA
provenance:
  source_task_ids:
  - task_001
  - task_003
  - task_007
  - task_008
  - task_004
  - task_005
  source_papers:
  - doi: 10.1038/s41467-024-47661-2
    title: Detecting m6A at single-molecular resolution via direct RNA sequencing
      and realistic training data
  source_content_hash: sha256:0e608da5d733cc4fc9c446f2718805cac3e2a154d3cfff3410e4254ecfa8a831
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/biodlab/RODAN
  tools:
  - CHEUI
  - Oxford Nanopore
  - RODAN
  - m6Anet
  - mAFiA
  - minimap2
  - minimap2.22
  - samtools
  - sklearn
  edam_operation: http://edamontology.org/operation_3182
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_0080
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6923076923076923
n_claims: 13
---

# nanopore-direct-rna-basecalling

## Summary

Basecall Oxford Nanopore dRNA-Seq raw fast5 files with the RODAN-based backbone.torch model (adapted basecall.py, batchsize 4096), producing FASTA reads suitable for downstream mAFiA m6A detection and minimap2 alignment.

## When to use

Use when raw nanopore fast5 dRNA-Seq reads (HEK293 WT/KO, Arabidopsis col0/vir1, or synthetic RL/SL oligos) must be converted to basecalled sequences and hidden features as the first step of an mAFiA m6A workflow.

## When NOT to use

- Input is already basecalled FASTQ/FASTA and only alignment or modification calling is needed
- The goal is DNA nanopore basecalling — RODAN is trained for direct RNA sequencing data
- Training a new basecaller from scratch rather than applying the deposited pretrained model

## Inputs

- Raw Oxford Nanopore dRNA-Seq fast5 files (single- or multi-fast5), e.g. fast5_chrX from HEK293 WT mRNA
- backbone.torch RODAN-based model weights and backbone.config from Zenodo record 8321727

## Outputs

- Basecalled FASTA file (rodan.fasta) in the output directory, ready for minimap2 alignment
- Per-nucleotide 768-dimensional hidden features from RODAN layer 'convlayers.conv21' when used inside mAFiA

## How to apply

Run the RODAN-adapted basecall.py script from the mAFiA repository against the fast5 directory, passing --model backbone.torch (the RODAN-based network downloaded from Zenodo record 8321727), --batchsize 4096, and an output directory; the script writes rodan.fasta into the output directory. On a modern GPU this takes under 30 minutes for a chromosome-X subset. The same backbone also serves mAFiA's feature extraction: a 768-dimensional vector per target nucleotide is taken from the RODAN layer 'convlayers.conv21' and fed into pickled logistic-regression classifiers, so keeping the identical backbone.torch for basecalling and prediction is essential. If GPU memory runs out, reduce the batch size (the original RODAN default is 200). Recursive search covers both single- and multi-fast5 files.

## Related tools

- **RODAN** (Fully convolutional basecaller whose adapted basecall.py converts fast5 signals to rodan.fasta; also supplies hidden features for mAFiA) — https://github.com/biodlab/RODAN
- **mAFiA** (Consumes RODAN basecalls and extracts 768-d features from backbone.torch layer convlayers.conv21 for logistic-regression m6A prediction) — https://github.com/dieterich-lab/mAFiA

## Examples

```
python3 ${mafia}/RODAN/basecall.py --fast5dir ${fast5dir} --model ${backbone} --batchsize 4096 --outdir ${output}
```

## Evaluation signals

- rodan.fasta is produced in the output directory and contains one entry per fast5 read
- Read sequences align to the expected reference (e.g. GRCh38_96.X or TAIR10) with minimap2 at mapping quality ≥ 50 for a substantial fraction of reads
- Basecalling completes within the expected runtime (under ~30 min for the chrX subset on a modern GPU); CUDA out-of-memory is resolved by lowering batch size
- Downstream mAFiA predictions yield mAFiA.reads.bam with MM/ML tags and mAFiA.sites.bed with coverage ≥ --min_coverage 50, confirming feature extraction used the same backbone

## Limitations

- Requires torch >= 1.4.0 <= 1.8.0 per the RODAN requirements; batchsize 4096 can exhaust GPU memory on smaller cards
- The mAFiA backbone is a RODAN-based network with its own backbone.config, not the stock RODAN release weights; models must be downloaded from Zenodo record 8321727
- mAFiA (Jan 2025) is superseded by psi-co-mAFiA, which extends the motif set to all 18 m6A DRACH motifs and 16 Ψ motifs

## Evidence

- [readme] The basecalling script is adapted from the [RODAN](https://github.com/biodlab/RODAN) repository.: "The basecalling script is adapted from the [RODAN](https://github.com/biodlab/RODAN) repository."
- [methods] backbone.torch: [RODAN](https://github.com/biodlab/RODAN)-based neural network for basecalling and feature extraction: "backbone.torch: [RODAN](https://github.com/biodlab/RODAN)-based neural network for basecalling and feature extraction"
- [results] Our m6A Finding Algorithm (mAFiA) makes use of hidden features generated by a fully-convolutional basecaller RODAN: "Our m6A Finding Algorithm (mAFiA) makes use of hidden features generated by a fully-convolutional basecaller RODAN"
- [readme] If you run out of memory, reduce the batch size with the basecaller with "-b 100" or lower. The default is 200.: [evidence span withheld — anchor status: external_doc]
- [results] a 768-dimensional feature vector is extracted from the last convolution layer ('convlayers.conv21') of RODAN: [evidence span withheld — anchor status: not_found]
