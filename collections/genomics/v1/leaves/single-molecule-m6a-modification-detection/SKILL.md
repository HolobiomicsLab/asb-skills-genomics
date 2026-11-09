---
name: single-molecule-m6a-modification-detection
description: Use when raw Oxford Nanopore direct RNA fast5 reads must be converted
  into read-level m6A modification calls (MM/ML tags) and site-level stoichiometry
  (modRatio), especially on DRACH motifs such as GGACU in human or synthetic RNA benchmarks.
when_to_use_negative:
- The task requires calling modification types beyond the trained m6A DRACH motifs
  (e.g. pseudouridine or all 18 DRACH motifs) — use the successor psi-co-mAFiA instead
- Site coverage is below the required minimum (fewer than 50 reads per site), so stoichiometry
  estimates would not be reliable
- Input is already basecalled/aligned data from a non-RODAN basecaller, since the
  classifiers are calibrated to RODAN's 768-dimensional hidden features and may not
  transfer
edam_operation: http://edamontology.org/operation_3900
edam_topics:
- http://edamontology.org/topic_0080
- http://edamontology.org/topic_3524
tools:
- name: mAFiA
  role: 'Main algorithm: extracts RODAN hidden features per nucleotide, applies logistic
    regression classifiers for read-level m6A probabilities, and piles them up into
    site-level stoichiometry'
  repo: https://github.com/dieterich-lab/mAFiA
- name: RODAN
  role: Fully-convolutional basecaller whose backbone.torch model provides basecalls
    and the 768-dimensional feature vector from layer convlayers.conv21
  repo: https://github.com/biodlab/RODAN
- name: minimap2
  role: Aligns RODAN-basecalled reads to the reference (splice/uf preset, --secondary=no,
    -k14, --cs)
- name: samtools
  role: Filters alignments to MAPQ >= 50 (samtools view -q50), sorts and indexes the
    BAM
- name: CHEUI
  role: Single-molecule m6A comparison method run on the same TEST1/TEST2 reads for
    PRC AUC benchmarking
- name: m6Anet
  role: Single-molecule m6A comparison method run on the same TEST1/TEST2 reads for
    PRC AUC benchmarking
provenance:
  source_task_ids:
  - task_001
  - task_003
  - task_004
  source_papers:
  - doi: 10.1038/s41467-024-47661-2
    title: Detecting m6A at single-molecular resolution via direct RNA sequencing
      and realistic training data
  source_content_hash: sha256:89b9b3eac32b1297019d47e1371bd39a624c3af06423f804bbe5a3d4a6b5b4a7
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/dieterich-lab/mAFiA
  tools:
  - CHEUI
  - RODAN
  - m6Anet
  - mAFiA
  - minimap2
  - minimap2.22
  - samtools
  - sklearn
  edam_operation: http://edamontology.org/operation_3900
  edam_topics:
  - http://edamontology.org/topic_0080
  - http://edamontology.org/topic_3524
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9583333333333334
n_claims: 24
---

# single-molecule-m6a-modification-detection

## Summary

Detect m6A nucleotides on individual dRNA-Seq reads and aggregate per-site stoichiometry using mAFiA, a RODAN-backbone feature extractor with per-motif logistic regression classifiers trained on synthetic RNAs with precisely controlled modification labels.

## When to use

Use when raw Oxford Nanopore direct RNA fast5 reads must be converted into read-level m6A modification calls (MM/ML tags) and site-level stoichiometry (modRatio), especially on DRACH motifs such as GGACU in human or synthetic RNA benchmarks.

## When NOT to use

- The task requires calling modification types beyond the trained m6A DRACH motifs (e.g. pseudouridine or all 18 DRACH motifs) — use the successor psi-co-mAFiA instead
- Site coverage is below the required minimum (fewer than 50 reads per site), so stoichiometry estimates would not be reliable
- Input is already basecalled/aligned data from a non-RODAN basecaller, since the classifiers are calibrated to RODAN's 768-dimensional hidden features and may not transfer

## Inputs

- Raw nanopore dRNA-Seq fast5 files (single- or multi-read), e.g. fast5_chrX from HEK293 WT mRNA or TEST1/TEST2 synthetic heteropolymer reads
- Genome/transcriptome reference FASTA (e.g. GRCh38_96.X.fa)
- BED file of target m6A sites in DRACH motifs (e.g. GLORI_chrX.bed)
- mAFiA models from Zenodo record 8321727: backbone.torch, backbone.config, and classifiers directory of pickled logistic regression models

## Outputs

- rodan.fasta: RODAN basecalled read sequences
- minimap.q50.bam: coordinate-sorted, mapping-quality >= 50 filtered alignment BAM with index
- mAFiA.reads.bam: modBAM with MM and ML tags marking per-read modification locations and probabilities
- mAFiA.sites.bed: per-site table with coverage, modRatio (fraction of nucleotides with P(m6A) >= 0.5), and confidence (fraction with P >= 0.75 or < 0.25)

## How to apply

Basecall the fast5 files with the RODAN-adapted basecall.py using the mAFiA backbone.torch model (batchsize 4096), which also exposes a 768-dimensional feature vector per nucleotide from the last convolution layer 'convlayers.conv21'. Align the resulting rodan.fasta to the reference with minimap2 (--secondary=no -ax splice -uf -k14 --cs), then filter, sort, and index with samtools view -bST <ref> -q50 followed by samtools sort/index. Run test_mAFiA (or process_reads) on the q50 BAM with --min_coverage 50, --max_num_reads 1000, --mod_prob_thresh 0.5, the backbone.torch model, and the pickled logistic regression classifiers directory; the classifiers are per-DRACH-motif models trained on 15 UNM/MOD synthetic sequence pairs via 4-fold cross-validation. Aggregate read probabilities with the pileup step to obtain mAFiA.sites.bed, where coverage >= 50 qualifies a site for stoichiometry comparison against orthogonal assays like GLORI.

## Related tools

- **mAFiA** (Main algorithm: extracts RODAN hidden features per nucleotide, applies logistic regression classifiers for read-level m6A probabilities, and piles them up into site-level stoichiometry) — https://github.com/dieterich-lab/mAFiA
- **RODAN** (Fully-convolutional basecaller whose backbone.torch model provides basecalls and the 768-dimensional feature vector from layer convlayers.conv21) — https://github.com/biodlab/RODAN
- **minimap2** (Aligns RODAN-basecalled reads to the reference (splice/uf preset, --secondary=no, -k14, --cs))
- **samtools** (Filters alignments to MAPQ >= 50 (samtools view -q50), sorts and indexes the BAM)
- **CHEUI** (Single-molecule m6A comparison method run on the same TEST1/TEST2 reads for PRC AUC benchmarking)
- **m6Anet** (Single-molecule m6A comparison method run on the same TEST1/TEST2 reads for PRC AUC benchmarking)

## Examples

```
python3 ${mafia}/RODAN/basecall.py --fast5dir ${fast5dir} --model ${backbone} --batchsize 4096 --outdir ${output} && test_mAFiA --bam_file ${bam} --fast5_dir ${fast5dir} --sites ${mod} --backbone_model_path backbone.torch --classifier_model_dir classifiers --min_coverage 50 --max_num_reads 1000 --mod_prob_thresh 0.5 --out_dir ${output}
```

## Evaluation signals

- mAFiA.reads.bam exists, is indexed, and carries MM/ML tags conforming to the modBAM standard marking modification locations and probabilities
- mAFiA.sites.bed lists only sites with coverage above the --min_coverage 50 threshold and includes the modRatio and confidence columns
- Alignment BAM contains only reads with mapping quality >= 50 after the samtools view -q50 filter
- On synthetic benchmarks (TEST1/TEST2), thresholding read-level P(m6A) at 0.5 yields precision-recall AUCs comparable to Fig. 2c/2f, with mAFiA resolving even-cycle GGACU patterns where CHEUI and m6Anet deteriorate
- Site-level modRatio correlates with orthogonal GLORI measurements when coverage >= 50 is enforced

## Limitations

- Reads with mapping quality below 50 are discarded, and in case of discontinuous alignment only the section with the longest continuous mapping is used
- A minimum dRNA-Seq coverage of 50 is required for a site to be compared against GLORI; low-coverage sites are excluded from stoichiometry
- mAFiA is superseded by psi-co-mAFiA (Jan 2025), which extends coverage to all 18 m6A DRACH motifs plus 16 pseudouridine motifs
- Basecalling with batchsize 4096 assumes a GPU; CUDA out-of-memory errors require reducing batchsize or job counts

## Evidence

- [results] mAFiA extracts a 768-dimensional feature vector per nucleotide and applies logistic regression: [evidence span withheld — anchor status: not_found]
- [results] mapq 50 filter: "Reads with mapq below 50 are filtered out."
- [results] read-level threshold: [evidence span withheld — anchor status: not_found]
- [results] min coverage for GLORI comparison: "A minimum dRNA-Seq coverage of 50 is required for a site to be compared against GLORI measurements"
- [readme] README basecalling command: "python3 ${mafia}/RODAN/basecall.py --fast5dir ${fast5dir} --model ${backbone} --batchsize 4096 --outdir ${output}"
- [readme] README alignment command: "minimap2 --secondary=no -ax splice -uf -k14 -t 36 --cs ${ref} ${basecall} | samtools view -bST ${ref} -q50 - | samtools sort - > ${bam}"
