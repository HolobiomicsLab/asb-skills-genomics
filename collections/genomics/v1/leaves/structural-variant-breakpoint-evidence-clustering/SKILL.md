---
name: structural-variant-breakpoint-evidence-clustering
description: Use when mapped long-read (HiFi) alignments must be scanned for SV breakpoint
  evidence and clustered into candidates as the discovery stage of sawfish SV/CNV
  calling on GRCh38.
when_to_use_negative:
- The input is raw unaligned reads — evidence scanning requires a coordinate-sorted
  mapped BAM, so alignment with pbmm2 must precede this step
- The goal is short-read SV calling — the article scopes sawfish to mapped HiFi sequencing
  reads
- Per-sample discover output already exists — discover is run once per sample, and
  multi-sample merging/genotyping is handled by the sawfish joint-call step instead
edam_operation: http://edamontology.org/operation_3227
edam_topics:
- http://edamontology.org/topic_3168
- http://edamontology.org/topic_0102
tools:
- name: sawfish
  role: Runs the discover step that scans alignments, clusters breakpoint evidence,
    and assembles SV haplotype contigs
  repo: https://github.com/PacificBiosciences/sawfish
- name: pbmm2
  role: Maps HG002 HiFi reads (--preset CCS --sample HG002 --sort --unmapped) to GRCh38
    to produce the alignment input for discovery
- name: spoa
  role: Partial order alignment library used to build the POA graph and generate SV
    haplotype consensus contigs from trimmed reads
  repo: https://github.com/rvaser/spoa
- name: WFA2-lib
  role: Wavefront alignment library used to align assembled SV haplotype contigs back
    to the reference genome
  repo: https://github.com/smarco/WFA2-lib
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/bioinformatics/btaf136
    title: 'Sawfish: improving long-read structural variant discovery and genotyping
      with local haplotype modeling'
  source_content_hash: sha256:5c8cb0d210ac3ec09ac1937e5760c629967e7e1a86d495b6f5fbaa36eae1d442
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
  - http://edamontology.org/topic_3168
  - http://edamontology.org/topic_0102
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# structural-variant-breakpoint-evidence-clustering

## Summary

Clusters per-read breakpoint evidence from mapped HiFi alignments into candidate breakpoint clusters, refines each cluster into SV haplotype contigs via POA assembly (spoa) and wavefront realignment (WFA2-lib), and discards noise so sawfish discover produces basepair-resolution candidate SVs ready for joint genotyping.

## When to use

Use when mapped long-read (HiFi) alignments must be scanned for SV breakpoint evidence and clustered into candidates as the discovery stage of sawfish SV/CNV calling on GRCh38.

## When NOT to use

- The input is raw unaligned reads — evidence scanning requires a coordinate-sorted mapped BAM, so alignment with pbmm2 must precede this step
- The goal is short-read SV calling — the article scopes sawfish to mapped HiFi sequencing reads
- Per-sample discover output already exists — discover is run once per sample, and multi-sample merging/genotyping is handled by the sawfish joint-call step instead

## Inputs

- pbmm2-mapped HiFi read BAM (e.g. m84011_220902_175841_s1.pbmm2-1.13.1.GRCh38.bam from m84011_220902_175841_s1.hifi_reads.bam)
- Reference FASTA (human_GRCh38_no_alt_analysis_set/human_GRCh38_no_alt_analysis_set.fasta)
- Expected copy number track bed (expected_cn.hg38.XY.bed) and CNV excluded regions track (annotation_and_common_cnv.hg38.bed.gz)

## Outputs

- sawfish_discover_output directory containing candidate breakpoint clusters, assembled SV haplotype contigs, depth track, and draft copy number segmentation
- Candidate SV set ready to enter sawfish joint-call for merging, per-sample read-support evaluation, and genotyping

## How to apply

Run sawfish discover once per sample on a pbmm2-mapped BAM (e.g. pbmm2 1.13.1 --preset CCS --sample HG002 --sort --unmapped with 16 threads against the human_GRCh38_no_alt_analysis_set reference). During the scan, reads flagged as unmapped, secondary, QC-failed or duplicate, or with gap-compressed identity below 0.97 or mapping quality below 5, are filtered out; soft-clipped read ends only contribute breakend candidates when at least 500 bases are clipped on one end and none on the other. Breakpoint evidence from individual reads is clustered, clusters within 500 bases breakpoint distance are merged, and single-read clusters are discarded as noise. Refined clusters are assembled into SV haplotype contigs: trimmed reads are subsampled deterministically to at most 100 per breakpoint candidate and assigned to a POA graph (spoa) when a read has aligned length >= 100 and normalized alignment score >= 0.96, then haplotype contigs are aligned back to the genome with wavefront alignment (WFA2-lib). The rationale is that modeling breakpoint-based SVs as local haplotypes yields basepair-resolution breakpoints with homology and insertion detail, improving discovery accuracy on truth sets such as the GIAB HG002 T2T SVs.

## Related tools

- **sawfish** (Runs the discover step that scans alignments, clusters breakpoint evidence, and assembles SV haplotype contigs) — https://github.com/PacificBiosciences/sawfish
- **pbmm2** (Maps HG002 HiFi reads (--preset CCS --sample HG002 --sort --unmapped) to GRCh38 to produce the alignment input for discovery)
- **spoa** (Partial order alignment library used to build the POA graph and generate SV haplotype consensus contigs from trimmed reads) — https://github.com/rvaser/spoa
- **WFA2-lib** (Wavefront alignment library used to align assembled SV haplotype contigs back to the reference genome) — https://github.com/smarco/WFA2-lib

## Examples

```
sawfish discover --threads 16 --ref human_GRCh38_no_alt_analysis_set/human_GRCh38_no_alt_analysis_set.fasta --bam m84011_220902_175841_s1.pbmm2-1.13.1.GRCh38.bam --output-prefix sawfish_discover_output
```

## Evaluation signals

- sawfish_discover_output is produced and can be passed to sawfish joint-call via --sample without errors on 16 threads
- Read counts entering clusters are consistent with the documented filters: unmapped/secondary/QC-failed/duplicate flags, gap-compressed identity >= 0.97, mapping quality >= 5
- No single-read breakpoint clusters survive merging (they must be discarded as noise); retained clusters are >= 2 supporting reads
- Haplotype contigs were assembled only from trimmed reads meeting the POA thresholds (aligned length >= 100, normalized score >= 0.96) and are realignable to the GRCh38 no-alt reference
- Downstream joint-call output genotyped.sv.vcf.gz exists and contains joint-genotyped SV records at basepair-resolution breakpoints with homology and insertion detail

## Limitations

- VACmap is still unsupported in sawfish, and clear error messages for problematic alignments from any source are not yet available
- Soft-clipped read ends only contribute breakend candidates under strict conditions (>= 500 bases clipped on one end, no clipping on the other), so some clipped-evidence breakpoints may be missed
- Trimmed reads are deterministically subsampled to 100 per breakpoint candidate, which caps per-candidate assembly evidence at high depth

## Evidence

- [methods] read-filtering-before-clustering: "Reads are filtered from this scan if they are flagged as unmapped, secondary, QC-failed or duplicate."
- [methods] cluster-merging-rule: "If the breakpoint distance between the clusters is 500 bases or less, the breakpoint clusters are merged"
- [methods] single-read-noise-discard: "After merging is completed, clusters with only a single supporting read are discarded as noise."
- [methods] poa-assembly: "using a simple iterative procedure to assign each read to a partial order alignment (POA) graph using the spoa library"
- [methods] contig-realignment: "candidate breakpoints are assembled into SV haplotype contigs, which are then aligned back to the genome"
- [readme] tool-scope: "Sawfish is a joint structural variant (SV) and copy number variant (CNV) caller for mapped HiFi sequencing reads."
- [abstract] accuracy-rationale: "Assessment against the draft Genome in a Bottle (GIAB) SV benchmark from the T2T-HG002-Q100 diploid assembly shows that sawfish has the highest accuracy"
