---
name: long-read-amplicon-otu-clustering
description: Use when evaluating whether direct de novo OTU clustering of full-length
  nanopore 16S amplicon reads at 97% similarity yields accurate richness estimates
  for mock communities, or when benchmarking a partition-based clustering method like
  nanoClust against a direct-clustering baseline.
when_to_use_negative:
- Input reads are raw INC-Seq consensus reads that have not yet been fragmented and
  orientation-corrected with chopSeq
- The goal is accurate OTU richness estimation for the mock community — this skill
  is explicitly a negative-control/baseline demonstration; use the partition-based
  nanoClust approach instead
edam_operation: http://edamontology.org/operation_0291
edam_topics:
- http://edamontology.org/topic_0637
tools:
- name: VSEARCH
  role: De novo OTU clustering (cluster_fast) of full-length amplicon reads at 97%
    sequence similarity; output parsed to count OTUs and singletons
- name: chopSeq
  role: Upstream fragmentation and read orientation correction of INC-Seq consensus
    reads producing the full-length read set clustered here
  repo: github.com/umerijaz/nanopore
- name: nanoClust
  role: The partitioning-based de novo clustering and within-cluster consensus calling
    alternative that this direct-clustering baseline motivates
  repo: github.com/umerijaz/nanopore
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/gigascience/giy140
    title: 'NanoAmpli-Seq: a workflow for amplicon sequencing for mixed microbial
      communities on the nanopore sequencing platform'
  source_content_hash: sha256:177fb8093c51beb2a06d69556cbc9ab86b60c3c2e6fd2239d9feed521f446552
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - VSEARCH
  - chopSeq
  - nanoClust
  edam_operation: http://edamontology.org/operation_0291
  edam_topics:
  - http://edamontology.org/topic_0637
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# long-read-amplicon-otu-clustering

## Summary

A diagnostic skill for de novo OTU clustering of full-length nanopore amplicon reads (post-chopSeq, size-selected) directly with VSEARCH at 97% identity, demonstrating that direct clustering of noisy long reads grossly inflates OTU counts and thus motivates the partition-based nanoClust approach of NanoAmpli-Seq.

## When to use

Use when evaluating whether direct de novo OTU clustering of full-length nanopore 16S amplicon reads at 97% similarity yields accurate richness estimates for mock communities, or when benchmarking a partition-based clustering method like nanoClust against a direct-clustering baseline.

## When NOT to use

- Input reads are raw INC-Seq consensus reads that have not yet been fragmented and orientation-corrected with chopSeq
- The goal is accurate OTU richness estimation for the mock community — this skill is explicitly a negative-control/baseline demonstration; use the partition-based nanoClust approach instead

## Inputs

- Post-chopSeq, size-selected full-length 16S amplicon reads in FASTA format (from INC-Seq consensus processing of 2D and 1D2 nanopore runs)
- Mock community read sets (single-organism and 10-organism) sequenced on the ONT MinION platform

## Outputs

- VSEARCH OTU clustering output (.uc / .clstr) per dataset
- Summary table otu_counts.csv with columns: dataset, total OTUs, singleton OTU count

## How to apply

Load the post-chopSeq, size-selected full-length amplicon reads (FASTA) for each mock community dataset (2D and 1D2 nanopore sequencing, single-organism and 10-organism mocks). Run de novo clustering with VSEARCH cluster_fast at a 97% identity threshold directly on each read set, without reference mapping or partitioning. Parse the VSEARCH uc/clstr output to count total OTUs and singleton OTUs (clusters with exactly one read), and tabulate dataset, total OTUs, and singleton counts in otu_counts.csv. The rationale, per the article, is that high raw nanopore error rates cause full-length direct clustering to fragment single organisms into many spurious clusters, so the expected outcome is massive OTU inflation (817/1301/2122/2725 OTUs) that demonstrates direct clustering fails and justifies read partitioning-based de novo clustering with within-cluster consensus calling (nanoClust).

## Related tools

- **VSEARCH** (De novo OTU clustering (cluster_fast) of full-length amplicon reads at 97% sequence similarity; output parsed to count OTUs and singletons)
- **chopSeq** (Upstream fragmentation and read orientation correction of INC-Seq consensus reads producing the full-length read set clustered here) — github.com/umerijaz/nanopore
- **nanoClust** (The partitioning-based de novo clustering and within-cluster consensus calling alternative that this direct-clustering baseline motivates) — github.com/umerijaz/nanopore

## Examples

```
vsearch --cluster_fast 2D_mock_fulllength.fasta --id 0.97 --uc otus_2D_mock.uc --sizeout
```

## Evaluation signals

- otu_counts.csv exists and is well-formed with dataset, total OTUs, and singleton columns
- 2D single-organism dataset reproduces 817 total OTUs with 777 singletons
- 1D2 single-organism dataset reproduces 1,301 total OTUs with 1,238 singletons
- 2D and 1D2 10-organism datasets reproduce 2,122 (1,742 singletons) and 2,725 OTUs (2,447 singletons), allowing minor divergence from read-set versioning
- All four total OTU counts far exceed theoretical mock community richness, confirming the inflation phenomenon

## Limitations

- Direct full-length VSEARCH clustering at 97% grossly inflates OTU counts (817 with 777 singletons for 2D single-organism; 2,725 with 2,447 for 1D2 10-organism), so results must not be interpreted as true community richness
- Reference-based clustering of noisy long reads is limited by incorrect mapping and discarding of reads from organisms absent from the reference database, which is why only a de novo baseline is run
- Counts may diverge slightly with different read-set versions

## Evidence

- [results] Direct VSEARCH clustering of full-length post-chopSeq, size-selected reads at 97% similarity grossly inflated OTU counts far above the theoretical mock community richness: [evidence span withheld — anchor status: not_found]
- [abstract] nanoClust algorithm was designed for read partitioning-based de novo clustering and within cluster consensus calling to obtain accurate full-length 16S rRNA gene sequences: "nanoClust algorithm was designed for read partitioning-based de novo clustering and within cluster consensus calling to obtain accurate full-length 16S rRNA gene sequences"
- [intro] incorrect mapping to a reference database is likely due to high error rates of raw nanopore reads: "incorrect mapping to a reference database is likely due to high error rates of raw nanopore reads"
- [readme] Scripts for processing short amplicon reads from Oxford Nanopore.: "Scripts for processing short amplicon reads from Oxford Nanopore. For details, please see:"
