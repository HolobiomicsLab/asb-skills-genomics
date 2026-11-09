---
name: nanopore-read-alignment-and-preprocessing
description: Use when raw ONT reads must be converted into methylation-annotated primary-alignment
  BAMs (MM/ML tags) plus SNV-phased, haplotagged BAM/VCF inputs for methylation-based
  phasing of ~30X human autosomes.
when_to_use_negative:
- The target is sex chromosomes (X/Y) — MethPhaser cannot improve phasing there because
  random X inactivation in females yields an inconsistent haplotype pattern
- Coverage is well below ~30X — HapCUT2 and WhatsHap only work reasonably above that
  coverage, so the haplotagged input would be unreliable
- The reads carry no base-modification information (no MM/ML tags), so methylation-based
  read phasing cannot be performed
edam_operation: http://edamontology.org/operation_0292
edam_topics:
- http://edamontology.org/topic_3168
- http://edamontology.org/topic_3295
tools:
- name: Dorado/Bonito/Guppy
  role: Basecall raw ONT reads with methylation information included to produce MM/ML
    tags
- name: minimap2
  role: Align ONT reads to the reference genome while keeping MM and ML base-modification
    tags
- name: SAMtools
  role: Filter alignments to primary-only records (view -bF 2304) before MethPhaser
    input
- name: Clair3
  role: Variant calling (v0.1-r1263) with consideration of different read types
- name: HapCUT2
  role: SNV phasing of filtered variants using the ONT error model
- name: WhatsHap
  role: haplotag module tags reads based on HapCUT2's SNV phasing results; whatshap
    phase is an alternative SNV phaser
- name: MethPhaser
  role: Downstream consumer of the methylation-annotated, haplotagged BAM and phased
    VCF
  repo: https://github.com/treangenlab/methphaser
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-024-49588-0
    title: 'MethPhaser: methylation-based long-read haplotype phasing of human genomes'
  source_content_hash: sha256:d35fae5491d5e47245469529d691ee71798058fbd6655ee0e1416683bb188134
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/treangenlab/methphaser
  tools:
  - Clair3
  - Dorado/Bonito/Guppy
  - HapCUT2
  - MethPhaser
  - SAMtools
  - WhatsHap
  - WhatsHap compare
  - meth_phaser_parallel
  - minimap2
  edam_operation: http://edamontology.org/operation_0292
  edam_topics:
  - http://edamontology.org/topic_3168
  - http://edamontology.org/topic_3295
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8823529411764706
n_claims: 17
---

# nanopore-read-alignment-and-preprocessing

## Summary

Turns raw ~30X Oxford Nanopore reads into the methylation-annotated, primary-only, haplotagged inputs MethPhaser requires: Dorado/Bonito/Guppy basecalling with methylation, minimap2 v2.24 alignment preserving MM/ML tags, primary-alignment filtering, Clair3 variant calling with AF > 0.25 filtering in GIAB high-confidence regions, HapCUT2 SNV phasing, and WhatsHap haplotagging.

## When to use

Use when raw ONT reads must be converted into methylation-annotated primary-alignment BAMs (MM/ML tags) plus SNV-phased, haplotagged BAM/VCF inputs for methylation-based phasing of ~30X human autosomes.

## When NOT to use

- The target is sex chromosomes (X/Y) — MethPhaser cannot improve phasing there because random X inactivation in females yields an inconsistent haplotype pattern
- Coverage is well below ~30X — HapCUT2 and WhatsHap only work reasonably above that coverage, so the haplotagged input would be unreliable
- The reads carry no base-modification information (no MM/ML tags), so methylation-based read phasing cannot be performed

## Inputs

- Raw Oxford Nanopore FASTQ/POD5 reads (~30X coverage) requiring basecalling with methylation calling
- Reference genome FASTA (e.g. GCA_000001405.15_GRCh38_no_alt_analysis_set)
- GIAB high-confidence region BED for variant filtering
- HapCUT2 phased SNV VCF for haplotagging

## Outputs

- Methylation-annotated BAM aligned with minimap2, retaining MM and ML tags
- Primary-alignment-only BAM (samtools view -bF 2304)
- Clair3 variant VCF filtered to AF > 0.25 within GIAB high-confidence regions
- HapCUT2 phased SNV VCF and WhatsHap haplotagged BAM (HP tags) ready for meth_phaser_parallel

## How to apply

Basecall raw ONT reads with methylation information included (Dorado/Bonito/Guppy), then align to the reference (e.g. GCA_000001405.15_GRCh38_no_alt_analysis_set) with minimap2 v2.24 while keeping the MM and ML base-modification tags. Keep only primary alignments via `samtools view -bF 2304`, since secondary/supplementary records otherwise trigger a pysam MM tag reading error downstream in MethPhaser. Call variants with Clair3 v0.1-r1263, restrict to GIAB high-confidence regions to remove Clair3 false positives, and filter to allele frequency > 0.25. Phase SNVs with HapCUT2 (--ont 1) or WhatsHap, then apply the WhatsHap haplotag module to tag reads from HapCUT2's SNV phasing; keep the analysis on autosomes only and work with at least 30X coverage, as SNV phasers like HapCUT2 and WhatsHap only perform reasonably above that depth.

## Related tools

- **Dorado/Bonito/Guppy** (Basecall raw ONT reads with methylation information included to produce MM/ML tags)
- **minimap2** (Align ONT reads to the reference genome while keeping MM and ML base-modification tags)
- **SAMtools** (Filter alignments to primary-only records (view -bF 2304) before MethPhaser input)
- **Clair3** (Variant calling (v0.1-r1263) with consideration of different read types)
- **HapCUT2** (SNV phasing of filtered variants using the ONT error model)
- **WhatsHap** (haplotag module tags reads based on HapCUT2's SNV phasing results; whatshap phase is an alternative SNV phaser)
- **MethPhaser** (Downstream consumer of the methylation-annotated, haplotagged BAM and phased VCF) — https://github.com/treangenlab/methphaser

## Examples

```
samtools view -bF 2304 -o output.bam input.bam
```

## Evaluation signals

- Alignment BAM retains MM and ML tags on reads (pysam can read the MM tag without error)
- samtools flagstat shows zero secondary (0x100) and supplementary (0x800) alignments in the output BAM
- Variant VCF entries all satisfy AF > 0.25 and fall within GIAB high-confidence regions
- Reads carry HP tags from WhatsHap haplotag, and haplotagged read counts are non-trivial on autosomes
- Coverage is at least ~30X and phase blocks are restricted to autosomes (no X/Y)

## Limitations

- Secondary or supplementary alignments left in the BAM will trigger a pysam MM tag reading error in MethPhaser — they must be removed with samtools view -bF 2304
- SNV phasing methods (HapCUT2, WhatsHap) only work reasonably at ~30X coverage or above
- Results depend on the accuracy of methylation detection, overall read length, and a certain minimum coverage of the data set
- Sex chromosomes are excluded from phasing because random deactivation of chromosome X in females leads to an inconsistent haplotype pattern

## Evidence

- [methods] We then used minimap2 v2.2462 to align reads to the reference and keep the MM and ML tags.: "We then used minimap2 v2.2462 to align reads to the reference and keep the MM and ML tags."
- [readme] Primary-alignment requirement: "The BAM file should only contain primary alignment (no secondary alignment and supplementary alignment), otherwise it will trigger pysam MM tag reading error."
- [methods] Clair3 variant calling: "Clair3 v0.1-r1263 was used for variant calling with the consideration of different read types"
- [methods] GIAB high-confidence and AF filtering: "limited our interested region to “high confidence” regions provided by GIAB to remove potential Clair3’s false positive calls."
- [methods] WhatsHap haplotag: "The WhatsHap haplotag module was applied to tag reads based on HapCUT2’s SNV phasing results."
- [readme] Coverage recommendation: "We recommend using MethPhaser for at least 30X coverage ONT reads since most of the SNV phasing methods (HapCUT2, Whatshap) only work reasonably above that coverage"
