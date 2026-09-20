---
name: clipped-alignment-extraction
description: Use when BAM alignments of Illumina short reads and SMS long reads mapped
  to a draft assembly must be mined for clipped reads (CIGAR S/H) to build per-position
  clipping tables as the first step of CRAQ assembly-error detection.
when_to_use_negative:
- Evaluating assemblies of species with higher ploidy levels — the paper states such
  evaluation 'may not be as accurate as the benchmarked cases presented here'
- Input BAMs still contain unmapped, secondary, QC-failed, PCR-duplicated, or MAPQ<20
  records — extraction counts would be inflated and downstream clipping ratios invalid
- The task is reference-based comparison against a finished genome (e.g. BUSCO/QUAST-style
  assessment); CRAQ clipping extraction is explicitly reference-free and only needs
  reads mapped to the draft assembly itself
edam_operation: http://edamontology.org/operation_0292
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_0102
tools:
- name: CRAQ
  role: 'Implements the clipping extraction: parses all clipped reads coded ''S''
    or ''H'' in the CIGAR string from the two filtered BAMs and emits the SR_clip.coverRate
    / LR_clip.coverRate tables'
  repo: https://github.com/JiaoLaboratory/CRAQ
- name: SAMtools
  role: Pre-filters the Minimap2 BAMs with 'samtools view -F 1796 -q 20' before clipped-read
    extraction; also used via its library to access SAM/BAM files
- name: Minimap2
  role: Produces the NGS ('-ax sr') and SMS ('-ax map-hifi'/'map-pb'/'map-ont') alignments
    against the draft assembly from which clipped reads are extracted
  repo: https://github.com/lh3/minimap2
- name: Perl
  role: Scripting language (version >= 5) in which CRAQ's clipping extraction and
    table generation are implemented
provenance:
  source_task_ids:
  - task_007
  - task_006
  - task_002
  source_papers:
  - doi: 10.1038/s41467-023-42336-w
    title: Identification of errors in draft genome assemblies at single-nucleotide
      resolution for quality assessment and improvement
  source_content_hash: sha256:2f473038f36a1c91258fa931470e8226cc98987749e1048f19bcf10d03dd526a
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/JiaoLaboratory/CRAQ
  tools:
  - CRAQ
  - Integrative Genomics Viewer (IGV)
  - Minimap2
  - Perl
  - SAMtools
  edam_operation: http://edamontology.org/operation_0292
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_0102
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# clipped-alignment-extraction

## Summary

Extracts soft/hard-clipped alignment records (CIGAR 'S' or 'H') from filtered NGS and SMS BAMs mapped to a draft assembly and tallies per-position clipped-read counts and coverage — the raw clipping-ratio signal CRAQ converts into CRE/CSE error breakpoints and CRH/CSH heterozygous loci.

## When to use

Use when BAM alignments of Illumina short reads and SMS long reads mapped to a draft assembly must be mined for clipped reads (CIGAR S/H) to build per-position clipping tables as the first step of CRAQ assembly-error detection.

## When NOT to use

- Evaluating assemblies of species with higher ploidy levels — the paper states such evaluation 'may not be as accurate as the benchmarked cases presented here'
- Input BAMs still contain unmapped, secondary, QC-failed, PCR-duplicated, or MAPQ<20 records — extraction counts would be inflated and downstream clipping ratios invalid
- The task is reference-based comparison against a finished genome (e.g. BUSCO/QUAST-style assessment); CRAQ clipping extraction is explicitly reference-free and only needs reads mapped to the draft assembly itself

## Inputs

- Draft genome assembly FASTA to which reads were mapped
- NGS short-read alignment BAM (SR_sort.bam) filtered with 'samtools view -F 1796 -q 20'
- SMS long-read alignment BAM (LR_sort.bam) filtered with 'samtools view -F 1796 -q 20'
- Minimap2 alignment parameters: '-ax sr' for short reads, '-ax map-hifi'/'map-pb'/'map-ont' for long reads

## Outputs

- SR_clip.coverRate: tab-delimited NGS clipping table (chr, position, strand, number of clipped-reads, total coverage at that position)
- LR_clip.coverRate: tab-delimited SMS clipping table (chr, position, strand, number of clipped-reads, total coverage at that position)

## How to apply

First pre-filter both alignment files with 'samtools view -F 1796 -q 20' to remove low MAPQ (<20), unmapped, secondary, QC-failed, and PCR-duplicate records; NGS reads are mapped with Minimap2 '-ax sr' and SMS reads with '-ax map-hifi' (or map-pb/map-ont). Then iterate through each filtered BAM (SR_sort.bam, LR_sort.bam) and extract every read whose CIGAR string contains an S (soft-clip) or H (hard-clip) operation, recording the precise mapped reference coordinate of the clip and encoding orientation as strand '+' for left-clipped and '-' for right-clipped. At each genomic position, tally the number of clipped reads and the total mapped-read coverage, and write two five-column tab-delimited tables — SR_clip.coverRate for NGS and LR_clip.coverRate for SMS — with columns chr, position, strand, number of clipped-reads, and total coverage. The resulting clipping ratios (clipped reads / total coverage) are what CRAQ's downstream stages compare against the heterozygous cutoff region h (default 0.4–0.6) and the mapping-breakpoint cutoff f (default 0.75).

## Related tools

- **CRAQ** (Implements the clipping extraction: parses all clipped reads coded 'S' or 'H' in the CIGAR string from the two filtered BAMs and emits the SR_clip.coverRate / LR_clip.coverRate tables) — https://github.com/JiaoLaboratory/CRAQ
- **SAMtools** (Pre-filters the Minimap2 BAMs with 'samtools view -F 1796 -q 20' before clipped-read extraction; also used via its library to access SAM/BAM files)
- **Minimap2** (Produces the NGS ('-ax sr') and SMS ('-ax map-hifi'/'map-pb'/'map-ont') alignments against the draft assembly from which clipped reads are extracted) — https://github.com/lh3/minimap2
- **Perl** (Scripting language (version >= 5) in which CRAQ's clipping extraction and table generation are implemented)

## Examples

```
samtools view -bF 1796 -q 20 SR_sort.bam > SR.filtered.bam; minimap2 -ax sr assembly.fa reads.fq | samtools view -bF 1796 -q 20 - > SR_sort.bam
```

## Evaluation signals

- Both output tables exist and contain exactly the five required columns: chr, position, strand, number of clipped-reads, total coverage
- At every position the reported clipped-read count never exceeds the total coverage
- Input BAMs can be shown to have been pre-filtered with 'samtools view -F 1796 -q 20' (no MAPQ<20, unmapped, secondary, QC-failed, or duplicate records remain)
- Each clipped-read entry carries a precise mapped reference coordinate and a strand tag ('+' for left-clipped, '-' for right-clipped)
- Downstream sanity check: computing clipped/total ratios on the tables yields a distribution compatible with CRAQ's h (0.4–0.6) and f (0.75) cutoff classes rather than degenerate all-0 or all-1 ratios

## Limitations

- Candidates with extremely low coverage (m, default = 2) or high coverage (M, default = 5 * average coverage) are excluded downstream, so extraction alone does not decide loci
- SMS clipping events shorter than 0.1 * the total read length are filtered out, and multi-mapped reads are filtered when identifying CSE breakpoints by default
- Read mapping is the most resource-intensive step of CRAQ; users must manually split query sequences into multiple fragments to parallelize
- Error breakpoint ratios can be lower than 100% in practice due to sequencing errors or inaccurate read mapping, so extracted clipping signals are noisy

## Evidence

- [discussion] Extract clipped alignments (CIGAR S/H) from filtered BAM files: "CRAQ first extracts all clipped reads, coded as “S” or “H” in the Compact Idiosyncratic Gapped Alignment Report (CIGAR) string"
- [discussion] Pre-filtering BAMs before extraction: [evidence span withheld — anchor status: not_found]
- [intro] Rationale for clipping extraction: "CRAQ uses clipping information to reveal assembly errors and low-quality regions by mapping the original sequencing reads back to the draft genome assembly."
- [readme] CRAQ's purpose per README: "CRAQ (Clipping Reveals Assembly Quality) is a reference-free genome assembly evaluator that can assess the accuracy of assembled genomic sequences"
