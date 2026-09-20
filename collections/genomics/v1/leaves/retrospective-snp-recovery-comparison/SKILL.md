---
name: retrospective-snp-recovery-comparison
description: Use when a reference-free exon/splice-graph prediction method must be
  validated against a published set of validated SNPs (e.g. 96 tench SNPs from Kumar
  et al. 2019) to quantify recovery (92/96 genotyped, 100% precision over the known
  set).
when_to_use_negative:
- A reference genome and annotation exist for the organism — a splice-aware aligner
  such as GMAP against the reference is the appropriate route instead of reference-free
  exon prediction.
- The input transcriptome is of poor quality or from a single tissue — article notes
  that lowly expressed transcripts appear fragmented and other tissues' transcripts
  are missing, degrading exon prediction and hence SNP recovery.
- The goal is de novo SNP discovery over the whole genome rather than validating exon-proximal
  SNP recovery against a known SNP set.
edam_operation: http://edamontology.org/operation_3227
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_0199
tools:
- name: EXFI
  role: build_baited_bloom_filter and build_splice_graph to predict exons from the
    transcriptome and pooled WGS reads; gfa1_to_fasta exports predicted exon FASTA
  repo: https://github.com/jlanga/exfi
- name: abyss-bloom
  role: builds the cascading Bloom filter retaining error-free transcriptome-overlapping
    k-mers from the WGS reads
  repo: https://github.com/bcgsc/abyss/
- name: Bowtie2
  role: maps raw pooled genomic reads to the predicted exon sequences
- name: BCFTools
  role: calls SNPs from reads mapped to predicted exons; variants below quality 20
    are filtered out
- name: biobloomcategorizer
  role: 'read filtering: discards WGS reads that share no k-mer with the transcriptome,
    reducing BF FPR by 68%–75%'
  repo: https://github.com/bcgsc/biobloom
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1002/ece3.6587
    title: 'EXFI: Exon and splice graph prediction without a reference genome'
  source_content_hash: sha256:2cb1dc966cf5875eccc358bfc98b6a2ced0adcd5614ef491dc6cad548ceb01ca
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/jlanga/exfi
  tools:
  - BCFTools
  - Bowtie2
  - EXFI
  - abyss-bloom
  - biobloomcategorizer
  edam_operation: http://edamontology.org/operation_3227
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_0199
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# retrospective-snp-recovery-comparison

## Summary

Benchmark an exon-prediction pipeline (EXFI) by retrospective recovery of known transcriptomic SNPs: predict exons de novo, map pooled WGS reads to predicted exons, call SNPs with BCFTools, and measure how many of a previously validated SNP set fall in genotypable regions.

## When to use

Use when a reference-free exon/splice-graph prediction method must be validated against a published set of validated SNPs (e.g. 96 tench SNPs from Kumar et al. 2019) to quantify recovery (92/96 genotyped, 100% precision over the known set).

## When NOT to use

- A reference genome and annotation exist for the organism — a splice-aware aligner such as GMAP against the reference is the appropriate route instead of reference-free exon prediction.
- The input transcriptome is of poor quality or from a single tissue — article notes that lowly expressed transcripts appear fragmented and other tissues' transcripts are missing, degrading exon prediction and hence SNP recovery.
- The goal is de novo SNP discovery over the whole genome rather than validating exon-proximal SNP recovery against a known SNP set.

## Inputs

- assembled tench transcriptome (de novo assembly, Kumar et al. 2019 dataset)
- raw pooled whole-genome sequencing reads from two pools of five diploid individuals at ~52x coverage (FASTQ/FASTA)
- reference set of 96 previously validated transcriptomic SNPs (Kumar et al. 2019)

## Outputs

- predicted exon sequences in FASTA format (via gfa1_to_fasta from the GFA1 splice graph)
- BCFTools variant calls mapped to predicted exons
- filtered genotypable SNP set (quality >= 20, >= 35 bp from other variants and exon boundaries)
- recovery statistics against the 96-SNP reference (e.g. 92/96 genotyped; 100% precision, 99% recall)

## How to apply

Run EXFI's build_baited_bloom_filter on the assembled transcriptome and raw pooled genomic reads (two pools of five diploid individuals, ~52x coverage), using a cascading Bloom filter built with ABySS to retain error-free transcriptome-overlapping k-mers; then run build_splice_graph to predict exon sequences and export them as FASTA with gfa1_to_fasta. Map the raw pooled genomic reads to the predicted exons with Bowtie2 and call variants with BCFTools. Apply two filters to derive genotypable regions: discard variants with quality below 20, then exclude variants within 35 bp of another variant or of a predicted exon boundary. Finally, intersect the surviving genotypable SNP positions with the reference SNP set (the 96 transcriptomic SNPs of Kumar et al.) and tabulate recovery — the tench benchmark reported 92 of 96 SNPs successfully genotyped, i.e. 100% precision over this SNP set with 99% recall.

## Related tools

- **EXFI** (build_baited_bloom_filter and build_splice_graph to predict exons from the transcriptome and pooled WGS reads; gfa1_to_fasta exports predicted exon FASTA) — https://github.com/jlanga/exfi
- **abyss-bloom** (builds the cascading Bloom filter retaining error-free transcriptome-overlapping k-mers from the WGS reads) — https://github.com/bcgsc/abyss/
- **Bowtie2** (maps raw pooled genomic reads to the predicted exon sequences)
- **BCFTools** (calls SNPs from reads mapped to predicted exons; variants below quality 20 are filtered out)
- **biobloomcategorizer** (read filtering: discards WGS reads that share no k-mer with the transcriptome, reducing BF FPR by 68%–75%) — https://github.com/bcgsc/biobloom

## Examples

```
docker run --rm --volume $PWD:$PWD --user "$UID:$GID" exfi:1.5.6 build_baited_bloom_filter --transcriptome tench.fa --reads pool1.fq pool2.fq
```

## Evaluation signals

- Recovery rate of the 96 known tench transcriptomic SNPs after quality (<20) and 35 bp proximity filtering — the paper benchmark reports 92/96 genotyped (99% recall).
- Precision over the known SNP set: all IEBs proximal to the 96 known SNPs were detected, giving 100% precision.
- Predicted splice graph is emitted in GFA1 format encoding exon sequences and their transcript connections.
- Read filtering should reduce the Bloom filter false-positive rate by roughly 68%–75% and slightly improve classification metrics.
- Exons shorter than k + q (q default 5) must be absent from the output (exon length filter applied).

## Limitations

- Only 92 of the 96 reference SNPs were recovered (99% recall); SNPs near predicted exon boundaries or other variants fall outside genotypable regions due to the 35 bp proximity filter.
- Performance depends on transcriptome quality: in the single-tissue herring case, lowly expressed transcripts were fragmented and other tissues' transcripts missing, lowering exon prediction accuracy (though mappability remained 99.5%).
- Full WGS read accession numbers for the tench data are deferred to Table S1, complicating provenance without the supplementary material.

## Evidence

- [other] EXFI detected all IEBs proximal to the 96 known tench SNPs, giving 100% precision over this SNP set in the retrospective SNP discovery evaluation.: [evidence span withheld — anchor status: not_found]
- [methods] Variant quality and proximity filters define genotypable regions.: "variants with a quality value below 20 were filtered out"
- [methods] Proximity filter definition.: "and then those that were within 35 bp to another variant or a predicted exon boundary"
- [results] Read filtering rationale and effect.: "Filtering the reads resulted in a 68%–75% reduction of the BF FPR while also slightly improving all the classification metrics"
- [intro] Tench dataset used for SNP-discovery evaluation.: "we applied EXFI to a recently published dataset on tench (Kumar et al., 2019) to evaluate its success in IEB detection for SNP discovery in exonic regions"
- [abstract] Exon output format.: "The results are returned in GFA1 format, which encodes both the predicted exon sequences and how they are connected to form transcripts"
- [methods] Exon length filter parameter.: "exons of length less than k + q (q by default five) are filtered out (Figure 2c)"
- [readme] EXFI commands per README/docker image.: [evidence span withheld — anchor status: not_found]
