---
name: inverse-collection-frequency-kmer-weighting
description: Use when building k-mer count feature vectors for long-read overlap detection
  where frequent/repetitive k-mers would dominate similarity search, and you need
  cheap TF-IDF-style weighting at genome scale.
when_to_use_negative:
- The feature representation is not k-mer count based (e.g. minimizer or alignment-based
  overlap detection), where IDF-style k-mer weighting does not apply
- Document-frequency values are cheap to compute exactly and a strict IDF formulation
  is required; ICF is only an approximation of DF-based IDF
- Memory is severely constrained — the article notes Fedrann's memory consumption
  is higher than most existing overlap detection tools
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_0199
- http://edamontology.org/topic_0622
tools:
- name: Fedrann
  role: Implements ICF weighting and fuses it with the SRP projection matrix by pre-multiplication,
    producing weighted low-dimensional read embeddings
  repo: https://github.com/jzhang-dev/FEDRANN
- name: kmer_searcher
  role: C++ component that locates k-mer positions in reads to build the sparse count
    vectors that ICF weights are applied to
  repo: https://github.com/jzhang-dev/FEDRANN
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/gigascience/giag048
    title: 'FEDRANN: effective long-read overlap detection based on dimensionality
      reduction and approximate nearest neighbors'
  source_content_hash: sha256:197037e2ce69cea23b5ed1c3c38a4c1fe2f721b29c4fcdbd75d7feafd12dbcda
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/jzhang-dev/FEDRANN
  tools:
  - Fedrann
  - Python
  - kmer_searcher
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_0199
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 7
---

# inverse-collection-frequency-kmer-weighting

## Summary

Down-weights repetitive k-mers and up-weights informative ones before low-dimensional projection in long-read overlap detection, using total collection frequency (CF) as a computationally efficient approximation of inverse document frequency (IDF), as implemented in Fedrann.

## When to use

Use when building k-mer count feature vectors for long-read overlap detection where frequent/repetitive k-mers would dominate similarity search, and you need cheap TF-IDF-style weighting at genome scale.

## When NOT to use

- The feature representation is not k-mer count based (e.g. minimizer or alignment-based overlap detection), where IDF-style k-mer weighting does not apply
- Document-frequency values are cheap to compute exactly and a strict IDF formulation is required; ICF is only an approximation of DF-based IDF
- Memory is severely constrained — the article notes Fedrann's memory consumption is higher than most existing overlap detection tools

## Inputs

- Sparse k-mer count vectors per read (built by kmer_searcher over FASTQ/FASTA long reads)
- Collection frequency (total count) of each sampled k-mer across the read set
- Sparse Random Projection (SRP) matrix of shape n_components × n_features

## Outputs

- ICF-weighted SRP projection matrix (SRP_transpose × ICF) used for fused transformation
- Normalized low-dimensional embedding vectors for each read (optionally saved as feature_matrix.npz)

## How to apply

After building sparse k-mer count vectors for each read (via Jellyfish counting with k=15, --kmer-min-multiplicity 2, and 5% k-mer sampling with --seed 602), compute the total collection frequency (CF) of each k-mer across the dataset and derive an inverse-collection-frequency (ICF) weight as a practical approximation of IDF. Rather than applying weights to the count matrix and then projecting separately, pre-multiply the sparse random projection (SRP) matrix with the ICF weight vector (forming SRP_transpose × ICF) so the weighted, dimensionality-reduced embedding is computed in a single fused transformation, parallelized with --threads 32. The rationale is that DF-based IDF is computationally prohibitive at genome scale, so Fedrann adopts total k-mer count within the dataset as a cheap surrogate that still suppresses repetitive k-mers and sequencing-error noise while emphasizing informative ones. The resulting normalized low-dimensional embedding vectors feed NNDescent cosine-distance search to rank candidate overlaps.

## Related tools

- **Fedrann** (Implements ICF weighting and fuses it with the SRP projection matrix by pre-multiplication, producing weighted low-dimensional read embeddings) — https://github.com/jzhang-dev/FEDRANN
- **kmer_searcher** (C++ component that locates k-mer positions in reads to build the sparse count vectors that ICF weights are applied to) — https://github.com/jzhang-dev/FEDRANN

## Examples

```
fedrann -i test/data/reads.fasta.gz -o test/output/ -k 15 --kmer-sample-fraction 0.05 --seed 602 --kmer-min-multiplicity 2 --save-feature-matrix --threads 32
```

## Evaluation signals

- The weighted embeddings are sparse and normalized; feature_matrix.npz (if --save-feature-matrix is set) loads as a valid sparse matrix of n_reads × n_components
- Downstream NNDescent cosine-distance search yields overlaps.tsv where distance values for putative overlaps are smaller than for random pairs (smaller distance = higher similarity)
- k-mer library contains only k-mers with multiplicity ≥ 2 and only the sampled fraction (default 5%) of total k-mers, confirming filtering preceded weighting
- fedrann.log records progression through the dimensionality-reduction matrix precomputation step (SRP_transpose × ICF) without errors
- Assembled overlap graphs using the weighted embeddings are comparable to or better than those from MECAT2, minimap2, and wtdbg2 in benchmarking workflows

## Limitations

- ICF uses total collection frequency as an approximation of document frequency (DF), which is not an exact IDF computation
- Memory consumption of Fedrann is higher than most existing overlap detection tools; more memory-efficient data representations are noted as future work
- Scaling more standard DR methods (PCA, UMAP) to human-genome-scale feature matrices fails, which is why the SRP+ICF fusion is used; no empirical validation yet for metagenomic binning or taxonomic classification applications

## Evidence

- [discussion] Fedrann therefore adopts the total count (i.e. collection frequency, CF) of a k-mer within the dataset as a computationally efficient approximation of its DF: "Fedrann therefore adopts the total count (i.e. collection frequency, CF) of a k-mer within the dataset as a computationally efficient approximation of its DF"
- [discussion] the implementation fuses these steps by pre-multiplying the SRP projection matrix with the IDF weight vector: "the implementation fuses these steps by pre-multiplying the SRP projection matrix with the IDF weight vector"
- [methods] Applies inverse collection frequency (ICF) as a practical approximation of IDF: "Applies inverse collection frequency (ICF) as a practical approximation of IDF"
- [abstract] the pipeline combining inverse document frequency (IDF) transformation, sparse random projection (SRP), and NNDescent enables accurate detection of overlaps across diverse datasets: "the pipeline combining inverse document frequency (IDF) transformation, sparse random projection (SRP), and NNDescent enables accurate detection of overlaps across diverse datasets"
- [discussion] The pipeline utilizes Jellyfish for efficient k-mer counting and filtering, retaining only k-mers whose multiplicities exceed a predefined threshold: "The pipeline utilizes Jellyfish for efficient k-mer counting and filtering, retaining only k-mers whose multiplicities exceed a predefined threshold"
