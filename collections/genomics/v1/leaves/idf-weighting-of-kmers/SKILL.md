---
name: idf-weighting-of-kmers
description: Use when building a sparse k-mer feature matrix from long reads (FASTQ/FASTA)
  and repeated/repetitive k-mers would dominate similarity search; weighting by inverse
  collection frequency emphasizes informative k-mers prior to DR/ANN.
when_to_use_negative:
- Input is already a dense, weighted feature/embedding matrix — re-weighting would
  distort downstream ANN distances
- Data are short reads or small-scale similarity search where BLAST/DALIGN-style direct
  alignment is feasible and DR-based weighting adds unnecessary complexity
- No k-mer vocabulary has been counted/filtered yet — apply k-mer counting and multiplicity
  filtering first
edam_operation: http://edamontology.org/operation_3891
edam_topics:
- http://edamontology.org/topic_0102
- http://edamontology.org/topic_0622
tools:
- name: Fedrann
  role: Implements ICF/IDF weighting and its fusion with the SRP projection matrix
    during feature extraction/dimensionality reduction
  repo: https://github.com/jzhang-dev/FEDRANN
- name: Jellyfish
  role: Counts k-mer multiplicities upstream, providing the collection frequencies
    used for IDF weighting
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/gigascience/giag048
    title: 'FEDRANN: effective long-read overlap detection based on dimensionality
      reduction and approximate nearest neighbors'
  source_content_hash: sha256:388ae50e5aa0f8ec909db0d256fe7215b6f3d2c8d5fa4ea39e6d9501ea454473
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/jzhang-dev/FEDRANN
  tools:
  - Fedrann
  - Jellyfish
  - Python
  - minimap2
  edam_operation: http://edamontology.org/operation_3891
  edam_topics:
  - http://edamontology.org/topic_0102
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 9
---

# idf-weighting-of-kmers

## Summary

Weight k-mer features by inverse document frequency (IDF), approximated as inverse collection frequency (ICF), to down-weight repetitive high-frequency k-mers before dimensionality reduction in long-read overlap detection. This weighting is the key transform that makes the subsequent sparse random projection discriminative.

## When to use

Use when building a sparse k-mer feature matrix from long reads (FASTQ/FASTA) and repeated/repetitive k-mers would dominate similarity search; weighting by inverse collection frequency emphasizes informative k-mers prior to DR/ANN.

## When NOT to use

- Input is already a dense, weighted feature/embedding matrix — re-weighting would distort downstream ANN distances
- Data are short reads or small-scale similarity search where BLAST/DALIGN-style direct alignment is feasible and DR-based weighting adds unnecessary complexity
- No k-mer vocabulary has been counted/filtered yet — apply k-mer counting and multiplicity filtering first

## Inputs

- Sparse k-mer feature matrix built from FASTQ/FASTA long reads (e.g. PacBio, ONT, BGI CycloneSEQ reads)
- K-mer collection frequency table from Jellyfish counting with multiplicity filtering
- SRP projection matrix (n_components x n_features) with fixed random seed

## Outputs

- IDF/ICF-weighted sparse k-mer feature matrix (optionally saved as feature_matrix.npz)
- Fused pre-multiplied projection matrix (SRP_transpose x ICF) used to embed reads

## How to apply

After Jellyfish k-mer counting with --kmer-min-multiplicity filtering (ignore k-mers appearing fewer than 2 times to remove sequencing-error k-mers) and optional --kmer-sample-fraction sampling (e.g. 5% of k-mers), compute each retained k-mer's collection frequency (CF) across the dataset as a computationally efficient approximation of document frequency (DF). Apply ICF/IDF weighting so k-mers shared by many reads receive low weight. For efficiency, Fedrann fuses the IDF weighting with sparse random projection by pre-multiplying the SRP projection matrix with the IDF weight vector (SRP_transpose x ICF), so weighting costs one matrix product rather than a separate pass over the sparse feature matrix. Fixed seed (e.g. --seed 602) keeps the projection and sampled k-mer library reproducible.

## Related tools

- **Fedrann** (Implements ICF/IDF weighting and its fusion with the SRP projection matrix during feature extraction/dimensionality reduction) — https://github.com/jzhang-dev/FEDRANN
- **Jellyfish** (Counts k-mer multiplicities upstream, providing the collection frequencies used for IDF weighting)

## Examples

```
fedrann -i reads.fasta.gz -o output/ -k 15 --kmer-sample-fraction 0.05 --seed 602 --kmer-min-multiplicity 2 --save-feature-matrix --threads 32
```

## Evaluation signals

- Weighted feature matrix remains sparse and its dimensionality equals the number of sampled/retained k-mers (n_features matches the sampled k-mer library)
- Fused projection matrix has shape (n_components, n_features), confirming ICF pre-multiplication rather than a separate weighting pass
- Downstream NNDescent (cosine distance) on SRP+ICF embeddings produces overlaps.tsv rows with plausible distance values (e.g. ~0.66-0.78 in the README example) and valid neighbor ranks
- Resulting overlap graph accuracy is comparable to or better than MECAT2/minimap2/wtdbg2 baselines when benchmarked against minimap2 reference alignments
- fedrann.log shows IDF weighting completed within the batched pipeline and, with SRP, stays within the ~6-hour / 1-TB budget on the largest H3-class datasets

## Limitations

- IDF is approximated by total collection frequency (CF) per k-mer rather than exact document frequency, which is a computationally motivated approximation
- SRP/ICF embeddings feed memory-hungry NNDescent; Fedrann's memory consumption is higher than most existing overlap detection tools, motivating future memory-efficient representations
- Weighting quality depends on k-mer size and sampling choices (e.g. k=30 for H4C, k=50 for H4O, --kmer-sample-fraction 0.05); unsuitable k can degrade overlap detection

## Evidence

- [discussion] Fedrann therefore adopts the total count (i.e. collection frequency, CF) of a k-mer within the dataset as a computationally efficient approximation of its DF: "Fedrann therefore adopts the total count (i.e. collection frequency, CF) of a k-mer within the dataset as a computationally efficient approximation of its DF"
- [discussion] the implementation fuses these steps by pre-multiplying the SRP projection matrix with the IDF weight vector: "the implementation fuses these steps by pre-multiplying the SRP projection matrix with the IDF weight vector"
- [readme] Ignore k-mers that appear less than 2 times to filter out sequencing errors: "Ignore k-mers that appear less than 2 times to filter out sequencing errors"
- [methods] Applies inverse collection frequency (ICF) as a practical approximation of IDF: "Applies inverse collection frequency (ICF) as a practical approximation of IDF"
- [abstract] the pipeline combining inverse document frequency (IDF) transformation, sparse random projection (SRP), and NNDescent enables accurate detection of overlaps across diverse datasets: "the pipeline combining inverse document frequency (IDF) transformation, sparse random projection (SRP), and NNDescent enables accurate detection of overlaps"
