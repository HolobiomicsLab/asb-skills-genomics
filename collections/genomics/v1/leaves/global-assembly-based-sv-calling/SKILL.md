---
name: global-assembly-based-sv-calling
description: Use when benchmarking long-read SV caller accuracy on HiFi data downsampled
  to low coverage (e.g. 10-fold HG002 Revio reads) and an assembly-derived SV callset
  (hifiasm + PAV) is needed as a comparison against mapping-based callers.
when_to_use_negative:
- The analysis goal is production SV calling on a single sample rather than benchmarking
  caller accuracy across coverage levels — sawfish alone (sawfish discover + joint-call)
  is the appropriate tool.
- Input reads are not HiFi long reads (e.g. short-read or ONT data), since hifiasm
  global assembly here is specifically applied to subsampled HiFi Revio reads.
- A caller comparison including the assembly-based arm is unnecessary, e.g. when evaluating
  only read-mapping callers such as Sniffles2 and pbsv against sawfish.
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_0199
- http://edamontology.org/topic_0622
tools:
- name: hifiasm
  role: Globally assembles each subsampled HiFi read set into a diploid assembly from
    which SVs are called
- name: PAV
  role: Calls structural variants from the hifiasm assembly to produce the assembly-based
    SV callset for benchmarking
- name: pbmm2 1.13.1
  role: Maps each subsampled BAM to human_GRCh38_no_alt_analysis_set.fasta with --preset
    CCS --sort for the caller arms
- name: truvari 4.2.2
  role: Assesses each caller VCF (bench --passonly --pick ac --dup-to-ins, MAFFT refine,
    ga4gh consolidation) against the GIAB T2T SV benchmark
  repo: library://ctsa/truvari/truvari:4.2.2
- name: sawfish
  role: Mapping-based SV caller arm that the assembly-based PAV arm is compared against;
    source and binaries on GitHub
  repo: https://github.com/PacificBiosciences/sawfish
- name: pbsv
  role: Mapping-based SV caller comparison arm for PacBio HiFi data
  repo: https://github.com/PacificBiosciences/pbsv
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/bioinformatics/btaf136
    title: 'Sawfish: improving long-read structural variant discovery and genotyping
      with local haplotype modeling'
  source_content_hash: sha256:a461c2beb917f2681e3400c9a1c37be9976fabb95321d170a1007cf5b5160539
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - MAFFT
  - PAV
  - Sniffles2
  - bcftools
  - hifiasm
  - pbmm2
  - pbmm2 1.13.1
  - pbsv
  - sawfish
  - singularity
  - truvari 4.2.2
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_0199
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7333333333333333
n_claims: 15
---

# global-assembly-based-sv-calling

## Summary

Benchmark arm of the sawfish coverage-downscaling study in which each subsampled HG002 HiFi set (10-, 15-, 30-fold) is globally assembled with hifiasm and structural variants are called from the resulting diploid assembly with PAV, providing an assembly-based comparison point for read-mapping SV callers such as sawfish, Sniffles2, and pbsv.

## When to use

Use when benchmarking long-read SV caller accuracy on HiFi data downsampled to low coverage (e.g. 10-fold HG002 Revio reads) and an assembly-derived SV callset (hifiasm + PAV) is needed as a comparison against mapping-based callers.

## When NOT to use

- The analysis goal is production SV calling on a single sample rather than benchmarking caller accuracy across coverage levels — sawfish alone (sawfish discover + joint-call) is the appropriate tool.
- Input reads are not HiFi long reads (e.g. short-read or ONT data), since hifiasm global assembly here is specifically applied to subsampled HiFi Revio reads.
- A caller comparison including the assembly-based arm is unnecessary, e.g. when evaluating only read-mapping callers such as Sniffles2 and pbsv against sawfish.

## Inputs

- Subsampled HG002 Revio HiFi read sets at 10-, 15-, and 30-fold mean coverage (from m84011_220902_175841_s1.hifi_reads.bam)
- human_GRCh38_no_alt_analysis_set.fasta reference
- GIAB T2T SV benchmark VCF GRCh38_HG2-T2TQ100-V1.1_stvar.vcf.gz
- GRCh38_HG2-T2TQ100-V1.1_stvar.benchmark.bed confident regions

## Outputs

- Per-coverage-level hifiasm assemblies of the subsampled HG002 read sets
- PAV-derived SV VCFs from each assembly
- truvari bench/refine/ga4gh assessment outputs against the GIAB T2T benchmark
- F1-score table per caller per coverage level (comparable to Supplementary Table S3 / Supplementary Fig. S2)

## How to apply

Subsample the HG002 Revio HiFi BAM (m84011_220902_175841_s1.hifi_reads.bam) to 10-, 15-, and 30-fold mean coverage, then for each level assemble the reads globally with hifiasm and call SVs from the assembly with PAV, running this arm in parallel to sawfish (discover then joint-call), Sniffles2, and pbsv on the pbmm2 1.13.1 --preset CCS alignments to human_GRCh38_no_alt_analysis_set.fasta. Assess each caller VCF against the GIAB draft T2T SV benchmark (GRCh38_HG2-T2TQ100-V1.1_stvar.vcf.gz, filtered with bcftools view -i 'INFO/SVTYPE!="."' and confident regions restricted to autosomes) using truvari 4.2.2 bench (--passonly --pick ac --dup-to-ins) followed by MAFFT refinement, truvari ga4gh consolidation, and process_truvari_ga4gh_vcfs.py summarization. Tabulate F1 per caller per coverage level to compare the assembly-based approach against the mapping-based callers.

## Related tools

- **hifiasm** (Globally assembles each subsampled HiFi read set into a diploid assembly from which SVs are called)
- **PAV** (Calls structural variants from the hifiasm assembly to produce the assembly-based SV callset for benchmarking)
- **pbmm2 1.13.1** (Maps each subsampled BAM to human_GRCh38_no_alt_analysis_set.fasta with --preset CCS --sort for the caller arms)
- **truvari 4.2.2** (Assesses each caller VCF (bench --passonly --pick ac --dup-to-ins, MAFFT refine, ga4gh consolidation) against the GIAB T2T SV benchmark) — library://ctsa/truvari/truvari:4.2.2
- **sawfish** (Mapping-based SV caller arm that the assembly-based PAV arm is compared against; source and binaries on GitHub) — https://github.com/PacificBiosciences/sawfish
- **pbsv** (Mapping-based SV caller comparison arm for PacBio HiFi data) — https://github.com/PacificBiosciences/pbsv

## Examples

```
hifiasm -o hg002_10x.asm -t 16 subsampled_10x.hifi_reads.fastq.gz && PAV call -c pav.cfg -s HG002 -o pav_10x
```

## Evaluation signals

- The generated F1 table reproduces the qualitative pattern that sawfish has the highest F1 at 10x coverage and that other callers (including the hifiasm+PAV arm) need ~30x coverage to match sawfish's 15x F1.
- Benchmark VCF preprocessing is correct: bcftools view -i 'INFO/SVTYPE!="."' retains only structural variant records and the confident-region BED retains autosomes only (grep -v chr[XY]).
- Each caller VCF passes through the full truvari 4.2.2 assessment chain: bench (--passonly --pick ac --dup-to-ins), MAFFT refine (--recount --use-region-coords --use-original-vcfs --align mafft), ga4gh consolidation, and process_truvari_ga4gh_vcfs.py summarization.
- Per-coverage F1 values are consistent with the published Supplementary Table S3 / Supplementary Fig. S2 for the assembly-based arm.
- Assessment is anchored to the GIAB draft T2T SV benchmark (V0.019-20241113) derived from the T2T-HG002-Q100v1.1 diploid assembly.

## Limitations

- The assembly-based (hifiasm+PAV) arm requires substantial coverage — other callers needed at least 30-fold coverage to match sawfish's accuracy at 15-fold — so this approach gains accuracy only at higher depths.
- Benchmarking is restricted to autosomes of the GIAB T2T benchmark; chrX/chrY regions are excluded, so accuracy on sex chromosomes is not assessed.
- Assessment depends on the specific truvari 4.2.2 bench/refine/ga4gh chain with MAFFT; results are not comparable to CMRG assessments that use truvari bench alone.

## Evidence

- [methods] Call SVs at each coverage level with sawfish: [evidence span withheld — anchor status: not_found]
- [methods] call SVs from the assembly with PAV: [evidence span withheld — anchor status: not_found]
- [abstract] other callers required at least 30-fold coverage: "other callers required at least 30-fold coverage to match sawfish accuracy at 15-fold"
- [abstract] GIAB T2T benchmark assessment: "Assessment against the draft Genome in a Bottle (GIAB) SV benchmark from the T2T-HG002-Q100 diploid assembly shows that sawfish has the highest accuracy"
- [readme] pbsv scope: [evidence span withheld — anchor status: not_found]
