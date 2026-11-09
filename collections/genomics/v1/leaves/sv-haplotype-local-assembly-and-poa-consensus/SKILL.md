---
name: sv-haplotype-local-assembly-and-poa-consensus
description: Use when calling germline SVs from mapped HiFi reads where breakpoint
  evidence must be resolved to basepair resolution via local haplotype assembly rather
  than read-pair heuristics.
when_to_use_negative:
- Input reads are short reads or noisy long reads below HiFi quality — the method
  is built for mapped HiFi sequencing reads with high gap-compressed identity (>=
  0.97 retained).
- The goal is benchmarking an existing SV callset rather than discovery — use truvari
  bench/refine or hap-eval instead of re-running local haplotype assembly.
- Only copy number variation (depth-based) is of interest without breakpoint-level
  SV resolution — depth segmentation alone may suffice.
edam_operation: http://edamontology.org/operation_3227
edam_topics:
- http://edamontology.org/topic_0199
- http://edamontology.org/topic_0622
tools:
- name: sawfish
  role: 'Implements the discover step: breakpoint-evidence scanning, clustering, SV
    haplotype contig assembly, and realignment'
  repo: https://github.com/PacificBiosciences/sawfish
- name: spoa
  role: Partial order alignment (POA) library used to assign each trimmed read to
    a POA graph and generate the consensus haplotype contig
  repo: https://github.com/rvaser/spoa
- name: WFA2-lib
  role: Wavefront alignment library used to align assembled SV haplotype contigs back
    to the genome
  repo: https://github.com/smarco/WFA2-lib
- name: pbmm2
  role: Maps HiFi reads (--preset CCS --sort --unmapped) to the GRCh38 no-alt reference
    to produce the input alignments
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/bioinformatics/btaf136
    title: 'Sawfish: improving long-read structural variant discovery and genotyping
      with local haplotype modeling'
  source_content_hash: sha256:841cb4ce6b0b081c042e0479d6a9ba6555b4450b1eb4279489a0b61a8facbd72
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/PacificBiosciences/sawfish
  tools:
  - WFA2-lib
  - pbmm2
  - sawfish
  - spoa
  edam_operation: http://edamontology.org/operation_3227
  edam_topics:
  - http://edamontology.org/topic_0199
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 12
---

# sv-haplotype-local-assembly-and-poa-consensus

## Summary

Assembles breakpoint-based structural variants as local haplotype contigs from clustered long-read breakpoint evidence using partial order alignment (spoa) and wavefront alignment (WFA2-lib), then realigns the contigs to the reference genome to resolve SVs to basepair resolution with breakpoint homology and insertion detail.

## When to use

Use when calling germline SVs from mapped HiFi reads where breakpoint evidence must be resolved to basepair resolution via local haplotype assembly rather than read-pair heuristics.

## When NOT to use

- Input reads are short reads or noisy long reads below HiFi quality — the method is built for mapped HiFi sequencing reads with high gap-compressed identity (>= 0.97 retained).
- The goal is benchmarking an existing SV callset rather than discovery — use truvari bench/refine or hap-eval instead of re-running local haplotype assembly.
- Only copy number variation (depth-based) is of interest without breakpoint-level SV resolution — depth segmentation alone may suffice.

## Inputs

- Mapped HiFi (CCS) BAM, e.g. pbmm2-produced m84011_220902_175841_s1.pbmm2-1.13.1.GRCh38.bam
- Reference FASTA: human_GRCh38_no_alt_analysis_set/human_GRCh38_no_alt_analysis_set.fasta
- Read alignment flag/MQ/identity filters applied during the breakpoint-evidence scan

## Outputs

- SV haplotype contigs assembled per candidate breakpoint cluster (spoa consensus)
- Contig-to-reference alignments produced via WFA2-lib wavefront alignment
- Candidate SV set and depth track written to the sawfish_discover_output directory

## How to apply

Scan mapped HiFi alignments (e.g., pbmm2 1.13.1 CCS alignments to human_GRCh38_no_alt_analysis_set) for breakpoint evidence, filtering reads flagged as unmapped, secondary, QC-failed or duplicate, with gap-compressed identity < 0.97 or mapping quality < 5 removed, and accepting soft-clipped read ends only when at least 500 bases are clipped on one end with no clipping on the other. Cluster breakpoint evidence from individual reads into candidate breakpoint clusters, merging clusters within 500 bases breakpoint distance and discarding single-read clusters as noise. For each candidate, trim reads, optionally filtering trimmed reads that lack breakpoint evidence (requiring an indel of at least 25 bases or a soft-clip in the trimmed region), and deterministically subsample to 100 reads per breakpoint candidate. Assign each read to a partial order alignment (POA) graph using the spoa library, accepting the consensus when at least one read alignment has aligned length >= 100 and a normalized alignment score >= 0.96, then align the resulting SV haplotype contigs back to the genome with wavefront alignment (WFA2-lib). The rationale is that modeling breakpoint-based SVs as local haplotypes yields basepair-resolution breakpoints with homology and insertion sequence detail, feeding downstream genotyping by relative read support per haplotype.

## Related tools

- **sawfish** (Implements the discover step: breakpoint-evidence scanning, clustering, SV haplotype contig assembly, and realignment) — https://github.com/PacificBiosciences/sawfish
- **spoa** (Partial order alignment (POA) library used to assign each trimmed read to a POA graph and generate the consensus haplotype contig) — https://github.com/rvaser/spoa
- **WFA2-lib** (Wavefront alignment library used to align assembled SV haplotype contigs back to the genome) — https://github.com/smarco/WFA2-lib
- **pbmm2** (Maps HiFi reads (--preset CCS --sort --unmapped) to the GRCh38 no-alt reference to produce the input alignments)

## Examples

```
sawfish discover --threads 16 --reference human_GRCh38_no_alt_analysis_set/human_GRCh38_no_alt_analysis_set.fasta m84011_220902_175841_s1.pbmm2-1.13.1.GRCh38.bam
```

## Evaluation signals

- POA consensus acceptance: at least one read alignment per candidate has aligned length >= 100 and normalized alignment score >= 0.96.
- Cluster sanity: after merging, no retained breakpoint cluster is supported by only a single read (single-read clusters discarded as noise).
- Assembled haplotype contigs align back to the genome at candidate breakpoint loci via WFA2-lib with basepair-resolved breakpoints and reported homology/insertion sequence.
- sawfish_discover_output directory is produced containing candidate SVs and the depth track, suitable for consumption by sawfish joint-call.
- Final genotyped.sv.vcf.gz from joint-call exists and is a valid gzipped VCF of joint-genotyped SV records.

## Limitations

- VACmap is still unsupported; the tooling is still working towards clear error messages for problematic alignments from any source.
- Soft-clipped read ends only contribute to breakend candidates when at least 500 bases are clipped on one end and none on the other, so heavily clipped reads with intermediate clipping are ignored.
- Trimmed reads are deterministically subsampled to 100 per breakpoint candidate, so candidates with very deep support use only a subset of reads for consensus generation.

## Evidence

- [methods] using a simple iterative procedure to assign each read to a partial order alignment (POA) graph using the spoa library: "using a simple iterative procedure to assign each read to a partial order alignment (POA) graph using the spoa library"
- [methods] WFA realignment of contigs: "using wavefront alignment (https://github.com/smarco/WFA2-lib)."
- [methods] Filtering thresholds: "Reads are filtered from this scan if they are flagged as unmapped, secondary, QC-failed or duplicate. Reads with gap-compressed identity less than 0.97 are filtered out"
- [methods] Subsampling policy: "If there are more than 100 trimmed reads for a given breakpoint candidate, the reads are deterministically subsampled to 100."
- [methods] POA acceptance criteria: "If at least one read alignment has an aligned read length of 100 or higher, and an alignment score normalized by the aligned read length of 0.96 or higher"
- [readme] Rationale from README: "It discovers germline structural variants from local sequence assembly and jointly genotypes these variants across multiple samples."
