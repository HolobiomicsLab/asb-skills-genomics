---
name: containerized-pipeline-execution
description: Use when a multi-tool haplotype-aware methylation workflow on PacBio
  HiFi BAMs must be run reproducibly end-to-end (methylation calling through DMR detection)
  via Nextflow with Singularity isolation.
when_to_use_negative:
- The goal is detection of non-CpG 5mC, hemimethylated CpGs, or other modifications
  such as 6mA — ccsmeth only considers symmetric CpG methylation and combines features
  from both strands.
- Input reads are Nanopore or short-read data — the workflow is built for PacBio CCS/HiFi
  reads (>=10 Kb insert recommended) and Clair3's hifi model.
- A single tool step suffices (e.g. only alignment with pbmm2) — containerized pipeline
  orchestration adds overhead without benefit.
tools:
- name: ccsmethphase
  role: Nextflow pipeline wrapper orchestrating all steps in Singularity containers
  repo: https://github.com/PengNi/ccsmethphase
- name: Nextflow
  role: Workflow engine executing the process graph with container profiles
- name: ccsmeth
  role: Read-level and site-level 5mCpG deep-learning calling producing modbam files
  repo: https://github.com/PengNi/ccsmeth
- name: pbmm2
  role: Aligns and merges ccsmeth modbam files
  repo: https://github.com/PacificBiosciences/pbmm2
- name: Clair3
  role: SNV calling from aligned modbams using the hifi model
- name: WhatsHap
  role: Phases PASS SNVs and reads to yield haplotype-tagged modbams
- name: DSS
  role: Differential methylation (DML/DMR) analysis between haplotypes, referenced
    from NanoMethPhase
  repo: https://github.com/vahidAK/NanoMethPhase
- name: Singularity
  role: Container runtime isolating each tool's dependencies
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-023-39784-9
    title: DNA 5-methylcytosine detection and methylation phasing using PacBio circular
      consensus sequencing
  source_content_hash: sha256:945031e41d6a816b0e925026e3a7353cca72ef4d6782edd8cc90aefc29544f14
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/PengNi/ccsmethphase
  tools:
  - Clair3
  - DSS
  - Nextflow
  - Singularity
  - WhatsHap
  - ccsmeth
  - ccsmethphase
  - pbmm2
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5384615384615384
n_claims: 13
---

# containerized-pipeline-execution

## Summary

Run the multi-tool ccsmethphase methylation-phasing workflow as a reproducible Nextflow pipeline executed in Singularity containers, chaining ccsmeth, pbmm2, Clair3, WhatsHap, and DSS on PacBio CCS/HiFi reads.

## When to use

Use when a multi-tool haplotype-aware methylation workflow on PacBio HiFi BAMs must be run reproducibly end-to-end (methylation calling through DMR detection) via Nextflow with Singularity isolation.

## When NOT to use

- The goal is detection of non-CpG 5mC, hemimethylated CpGs, or other modifications such as 6mA — ccsmeth only considers symmetric CpG methylation and combines features from both strands.
- Input reads are Nanopore or short-read data — the workflow is built for PacBio CCS/HiFi reads (>=10 Kb insert recommended) and Clair3's hifi model.
- A single tool step suffices (e.g. only alignment with pbmm2) — containerized pipeline orchestration adds overhead without benefit.

## Inputs

- PacBio HiFi BAM (e.g. hg002.chr20_demo.hifi.bam) or subreads.bam
- input_sheet.tsv with Group_ID, Sample_ID, Type, Path columns
- Reference genome FASTA (e.g. chr20_demo.fa) and its index
- ccsmeth 5mCpG pre-trained models
- Conda/Nextflow environment with Singularity

## Outputs

- Haplotype-tagged modbam BAM (e.g. G1.HG002_demo.hifi.ccsmeth.modbam.pbmm2.merged_size1.SNV_PASS_whatshap.bam)
- clair3_called directory of PASS SNV VCFs
- whatshap_phased directory of phased SNV VCFs and phased BAMs
- mods_freq bedmethyl files (hp1.bed, hp2.bed, all.bed)
- diff_methyl DML/DMR results, e.g. ...aggregate.hp_callDMR.autosomes_cf0.2.bed

## How to apply

Install Nextflow (version >= 21.10.6) and prepare a containerized environment; the README recommends 128 GB RAM, 40 CPUs, 4 TB storage, and a >= 8 GB GPU. Encode the sample setup in an input_sheet.tsv with Group_ID, Sample_ID, Type='hifi', and Path columns, and pass the reference FASTA (e.g. chr20_demo.fa) plus --include_all_ctgs true to keep all contigs rather than only [chr][1-22,X,Y]. The pipeline then executes in order: ccsmeth read-level 5mCpG calling to modbam files, pbmm2 alignment/merging (merged_size1), Clair3 SNV calling with its hifi model filtered to PASS variants (SNV_PASS), WhatsHap SNV and read phasing, haplotype-specific bedmethyl aggregation (hp1.bed, hp2.bed, all.bed under mods_freq), and DSS-based DML/DMR detection (referenced from NanoMethPhase) at p<0.001 with |diff|>0.2, filtering to autosomes with cf 0.2. Run with the Singularity profile so each tool resolves in a pinned container, then verify outputs against the documented naming pattern.

## Related tools

- **ccsmethphase** (Nextflow pipeline wrapper orchestrating all steps in Singularity containers) — https://github.com/PengNi/ccsmethphase
- **Nextflow** (Workflow engine executing the process graph with container profiles)
- **ccsmeth** (Read-level and site-level 5mCpG deep-learning calling producing modbam files) — https://github.com/PengNi/ccsmeth
- **pbmm2** (Aligns and merges ccsmeth modbam files) — https://github.com/PacificBiosciences/pbmm2
- **Clair3** (SNV calling from aligned modbams using the hifi model)
- **WhatsHap** (Phases PASS SNVs and reads to yield haplotype-tagged modbams)
- **DSS** (Differential methylation (DML/DMR) analysis between haplotypes, referenced from NanoMethPhase) — https://github.com/vahidAK/NanoMethPhase
- **Singularity** (Container runtime isolating each tool's dependencies)

## Examples

```
git clone https://github.com/PengNi/ccsmethphase.git; nextflow run ccsmethphase -with-singularity --input_sheet input_sheet.tsv --genome chr20_demo.fa --include_all_ctgs true
```

## Evaluation signals

- ccsmethphase_results/<dsname> contains the expected subdirectories: bam, vcf (clair3_called and whatshap_phased), mods_freq, and diff_methyl files.
- Output filenames match the documented pattern, e.g. G1.HG002_demo.hifi.ccsmeth.modbam.pbmm2.merged_size1.SNV_PASS_whatshap.freq.aggregate.hp_callDMR.autosomes_cf0.2.bed.
- Only PASS SNVs proceed to phasing (SNV_PASS tag present in intermediate and final filenames).
- mods_freq contains three haplotype-specific bedmethyl outputs: hp1.bed, hp2.bed, and all.bed.
- DMR output reflects the configured thresholds: DMLs/DMRs at p<0.001 with |diff|>0.2, autosomes only, cf 0.2 — verify via the 'autosomes_cf0.2' suffix and file contents.

## Limitations

- By default only [chr][1-22,X,Y] contigs are processed; non-standard contigs require --include_all_ctgs true.
- The pipeline is specific to symmetric CpG methylation — it cannot detect hemimethylated CpGs, non-CpG 5mC, or other modifications such as 6mA.
- Recommended hardware is substantial: 128 GB RAM, 40 CPUs, 4 TB disk, >=8 GB GPU; the strand-specific-methylation model suffers reduced performance due to limited subread depth.
- DSS-based ASM/DMR code was referenced from NanoMethPhase (designed for Nanopore data), so its thresholds (p<0.001, |diff|>0.2, cf 0.2) are inherited defaults.

## Evidence

- [abstract] we develop a Nextflow pipeline, ccsmethphase, to detect haplotype-aware methylation using CCS reads: "we develop a Nextflow pipeline, ccsmethphase, to detect haplotype-aware methylation using CCS reads"
- [readme] ccsmethphase install requirements: "Recommended Hardware requirements: 128 GB RAM, 40 CPU processors, 4 TB disk storage, >=8 GB GPU"
- [readme] nextflow version: "Install [nextflow](https://www.nextflow.io/) (version>=21.10.6)"
- [methods] include_all_ctgs flag: "--include_all_ctgs: "true" or "false". default false, means only [chr][1-22,X,Y] included."
- [methods] DSS code source: "Code for ASM detection using DSS were referenced from NanoMethPhase"
- [methods] validation naming pattern: "G1.HG002_demo.hifi.ccsmeth.modbam.pbmm2.merged_size1.SNV_PASS_whatshap.bam"
