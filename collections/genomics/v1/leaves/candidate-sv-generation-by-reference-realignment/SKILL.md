---
name: candidate-sv-generation-by-reference-realignment
description: Use when calling germline structural variants from mapped PacBio HiFi
  reads by assembling local haplotype contigs at breakpoint clusters and realigning
  them to a reference to create SV candidates.
when_to_use_negative:
- Input reads are short reads or unaligned FASTQ — sawfish requires mapped high-quality
  HiFi reads as its input
- You need somatic or single-cell SV calling — sawfish discovers germline structural
  variants and jointly genotypes them across samples
- Only CNV (copy number) calls are needed without breakpoint-level SV resolution —
  the local-assembly pipeline is built for breakpoint-based SVs
edam_operation: http://edamontology.org/operation_3227
edam_topics:
- http://edamontology.org/topic_0622
tools:
- name: sawfish
  role: 'Runs the discover step: scans alignments, clusters breakpoint evidence, assembles
    SV haplotype contigs, and generates candidate SVs'
  repo: https://github.com/PacificBiosciences/sawfish
- name: pbmm2
  role: Maps HG002 HiFi reads (--preset CCS --sample HG002 --sort --unmapped) to the
    GRCh38 no-alt reference to produce the input alignments scanned for breakpoint
    evidence
- name: spoa
  role: Partial order alignment library used to assign reads to POA graphs and generate
    consensus SV haplotype contigs
  repo: https://github.com/rvaser/spoa
- name: WFA2-lib
  role: Wavefront alignment library used to realign assembled SV haplotype contigs
    back to the genome for candidate breakpoint refinement
  repo: https://github.com/smarco/WFA2-lib
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/bioinformatics/btaf136
    title: 'Sawfish: improving long-read structural variant discovery and genotyping
      with local haplotype modeling'
  source_content_hash: sha256:e0e4598911c2c07fb95b8767ed8d3c4531f6a302e7696ec7385ad51dbdb13945
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
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5454545454545454
n_claims: 11
---

# Candidate SV generation by reference realignment

## Summary

Sawfish's core discovery mechanism: clustered breakpoint evidence from mapped HiFi reads is assembled into SV haplotype contigs (POA via spoa, wavefront alignment via WFA2-lib), then realigned back to the reference genome to generate basepair-resolution candidate SVs that are later jointly genotyped by read support.

## When to use

Use when calling germline structural variants from mapped PacBio HiFi reads by assembling local haplotype contigs at breakpoint clusters and realigning them to a reference to create SV candidates.

## When NOT to use

- Input reads are short reads or unaligned FASTQ — sawfish requires mapped high-quality HiFi reads as its input
- You need somatic or single-cell SV calling — sawfish discovers germline structural variants and jointly genotypes them across samples
- Only CNV (copy number) calls are needed without breakpoint-level SV resolution — the local-assembly pipeline is built for breakpoint-based SVs

## Inputs

- BAM of HiFi reads mapped to human_GRCh38_no_alt_analysis_set.fasta (e.g. m84011_220902_175841_s1.pbmm2-1.13.1.GRCh38.bam)
- Reference FASTA: human_GRCh38_no_alt_analysis_set/human_GRCh38_no_alt_analysis_set.fasta
- Optional copy-number inputs: expected_cn.hg38.XY.bed and annotation_and_common_cnv.hg38.bed.gz for downstream depth-track segmentation

## Outputs

- sawfish_discover_output directory containing clustered breakpoint evidence, assembled SV haplotype contigs, candidate SVs, and a depth track with draft copy-number segmentation
- Candidate SVs assembled to basepair resolution with breakpoint homology and insertion sequence details

## How to apply

Run 'sawfish discover' once per sample on the pbmm2-aligned BAM (e.g. HG002 HiFi reads mapped with pbmm2 1.13.1 --preset CCS to human_GRCh38_no_alt_analysis_set), scanning alignments and clustering breakpoint evidence into candidate breakpoint clusters. Filter reads from the scan if unmapped/secondary/QC-failed/duplicate, gap-compressed identity < 0.97, or MAPQ < 5; merge clusters whose breakpoint distance is 500 bases or less and discard single-read clusters as noise. For each surviving breakpoint candidate, select trimmed reads carrying breakpoint evidence (indel >= 25 bases or soft-clip in the trimmed region, soft-clipped ends requiring >=500 bases clipped on one end and no clipping on the other), deterministically subsampling to 100 reads per candidate. Assign each read to a partial order alignment (POA) graph with the spoa library — requiring aligned read length >= 100 and a normalized alignment score >= 0.96 — to assemble SV haplotype contigs, then align contigs back to the genome with wavefront alignment (WFA2-lib) to resolve candidate breakpoints at basepair resolution with homology and insertion detail. The rationale is that modeling SVs as local haplotypes yields substantial accuracy gains over signature-based callers on modern truth sets such as the GIAB HG002 T2T SVs.

## Related tools

- **sawfish** (Runs the discover step: scans alignments, clusters breakpoint evidence, assembles SV haplotype contigs, and generates candidate SVs) — https://github.com/PacificBiosciences/sawfish
- **pbmm2** (Maps HG002 HiFi reads (--preset CCS --sample HG002 --sort --unmapped) to the GRCh38 no-alt reference to produce the input alignments scanned for breakpoint evidence)
- **spoa** (Partial order alignment library used to assign reads to POA graphs and generate consensus SV haplotype contigs) — https://github.com/rvaser/spoa
- **WFA2-lib** (Wavefront alignment library used to realign assembled SV haplotype contigs back to the genome for candidate breakpoint refinement) — https://github.com/smarco/WFA2-lib

## Examples

```
sawfish discover --threads 16 --ref human_GRCh38_no_alt_analysis_set/human_GRCh38_no_alt_analysis_set.fasta --bam m84011_220902_175841_s1.pbmm2-1.13.1.GRCh38.bam --output sawfish_discover_output
```

## Evaluation signals

- The discover output directory (sawfish_discover_output) is produced with candidate SVs and a depth track from draft segmentation
- Downstream 'sawfish joint-call' completes and produces a valid gzipped VCF (sawfish_joint-call_output/genotyped.sv.vcf.gz) with joint-genotyped SV records
- Single-read breakpoint clusters and clusters not passing the 500-base merge distance were removed, indicating noise filtering ran
- Assembled contigs carry basepair-resolution breakpoints with reported homology and insertion details, verifiable against benchmark sets (e.g. GIAB HG002 T2T SVs via truvari bench/refine with MAFFT)

## Limitations

- Read filtering excludes gap-compressed identity < 0.97 and MAPQ < 5, so SV evidence in poorly aligned or repetitive regions may be missed
- Soft-clipped read ends only contribute when at least 500 bases are clipped on one end with no clipping on the other; smaller clips are ignored
- Trimmed reads are deterministically subsampled to 100 per breakpoint candidate, which may cap sensitivity in very high-depth regions
- Other SV callers required at least 30-fold coverage to match sawfish accuracy at 15-fold, but very low depths (< 10x) were not validated

## Evidence

- [methods] The first ‘discover’ step of sawfish is run once on each input sample. In this step each sample is scanned for SV and CNV evidence.: "The first ‘discover’ step of sawfish is run once on each input sample. In this step each sample is scanned for SV and CNV evidence."
- [methods] candidate breakpoints are assembled into SV haplotype contigs, which are then aligned back to the genome: "candidate breakpoints are assembled into SV haplotype contigs, which are then aligned back to the genome"
- [methods] using a simple iterative procedure to assign each read to a partial order alignment (POA) graph using the spoa library: "using a simple iterative procedure to assign each read to a partial order alignment (POA) graph using the spoa library"
- [methods] Reads with gap-compressed identity less than 0.97 are filtered out: "Reads with gap-compressed identity less than 0.97 are filtered out"
- [intro] All breakpoint-based structural variants are modeled and genotyped as local haplotypes, yielding substantial accuracy gains on modern SV truth sets such as the GIAB HG002 T2T SVs.: "All breakpoint-based structural variants are modeled and genotyped as local haplotypes, yielding substantial accuracy gains on modern SV truth sets such as the GIAB HG002 T2T SVs."
- [readme] Sawfish is a joint structural variant (SV) and copy number variant (CNV) caller for mapped HiFi sequencing reads.: "Sawfish is a joint structural variant (SV) and copy number variant (CNV) caller for mapped HiFi sequencing reads."
