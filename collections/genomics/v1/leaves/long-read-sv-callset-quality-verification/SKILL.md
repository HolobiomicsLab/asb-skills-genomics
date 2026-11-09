---
name: long-read-sv-callset-quality-verification
description: Use when a Truvari-based SV caller ranking (e.g. sawfish vs Sniffles2
  vs pbsv on HG002 32-fold HiFi WGS) needs confirmation with an independent benchmarking
  engine before trusting the conclusion.
when_to_use_negative:
- Assessing the GIAB CMRG SV benchmark (HG002_GRCh38_CMRG_SV_v1.00), which the paper
  evaluates with the truvari bench command alone rather than hap-eval
- 'Evaluating CNV-only or small-variant callsets: the benchmark is pre-filtered to
  structural-variant records with INFO/SVTYPE!="."'
- Assessing SVs on sex chromosomes, since the confident-region BED is deliberately
  restricted to autosomes
edam_operation: http://edamontology.org/operation_0336
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_0102
tools:
- name: Sentieon/hap-eval
  role: Independent VCF comparison engine used as the second benchmarking method to
    compute precision/recall/F1 per SV caller
  repo: https://github.com/Sentieon/hap-eval
- name: bcftools
  role: 'Preprocessing: selecting SVTYPE records from the T2T benchmark VCF (''bcftools
    view -i INFO/SVTYPE!="."'') and tabix indexing'
- name: sawfish
  role: Caller under evaluation; the 32x HG002 genotyped.sv.vcf.gz callset being verified
    for cross-method accuracy superiority
  repo: https://github.com/PacificBiosciences/sawfish
- name: pbsv
  role: Comparison SV caller whose 32x HG002 callset is benchmarked to reproduce the
    4.5% F1 deficit vs sawfish
  repo: https://github.com/PacificBiosciences/pbsv
- name: truvari 4.2.2
  role: Primary (first-method) SV benchmarking engine whose rankings this skill independently
    verifies; used with MAFFT for refinement
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/bioinformatics/btaf136
    title: 'Sawfish: improving long-read structural variant discovery and genotyping
      with local haplotype modeling'
  source_content_hash: sha256:50ade9926f4443b3fe649944c0c382eaa26111bc16b22e9cb76413fdab6fc9c5
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Sentieon/hap-eval
  tools:
  - PacificBiosciences/pbsv
  - Sentieon/hap-eval
  - bcftools
  - pbsv
  - sawfish
  - truvari 4.2.2
  edam_operation: http://edamontology.org/operation_0336
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_0102
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9166666666666666
n_claims: 12
---

# long-read-sv-callset-quality-verification

## Summary

Cross-method verification of long-read SV caller accuracy rankings by re-scoring 32x HG002 callsets (sawfish, Sniffles2, pbsv) against the GIAB draft T2T SV benchmark (V0.019-20241113, GRCh38_HG2-T2TQ100-V1.1_stvar) with Sentieon hap-eval instead of relying solely on Truvari.

## When to use

Use when a Truvari-based SV caller ranking (e.g. sawfish vs Sniffles2 vs pbsv on HG002 32-fold HiFi WGS) needs confirmation with an independent benchmarking engine before trusting the conclusion.

## When NOT to use

- Assessing the GIAB CMRG SV benchmark (HG002_GRCh38_CMRG_SV_v1.00), which the paper evaluates with the truvari bench command alone rather than hap-eval
- Evaluating CNV-only or small-variant callsets: the benchmark is pre-filtered to structural-variant records with INFO/SVTYPE!="."
- Assessing SVs on sex chromosomes, since the confident-region BED is deliberately restricted to autosomes

## Inputs

- Three 32x HG002 SV callset VCFs: sawfish genotyped.sv.vcf.gz, Sniffles2 output VCF, pbsv output VCF (bgzipped/tabix-indexed)
- GIAB draft T2T SV benchmark VCF, version V0.019-20241113 (GRCh38_HG2-T2TQ100-V1.1_stvar)
- GIAB T2T benchmark confident-regions BED file
- human_GRCh38_no_alt_analysis_set.fasta reference
- Sentieon hap-eval (pip install from github.com/Sentieon/hap-eval with vcflib submodule)

## Outputs

- hapeval_t2t_32x_f1_scores.csv with columns caller, precision, recall, F1, margin vs sawfish
- Filtered SV-only benchmark VCF GRCh38_HG2-T2TQ100-V1.1_stvar.svtype.vcf.gz with tabix index
- Autosome-restricted confident-region BED (chrX/chrY lines removed)

## How to apply

Collect the three 32x HG002 SV callset VCFs (sawfish genotyped.sv.vcf.gz, Sniffles2 output, pbsv output), the GIAB draft T2T SV benchmark VCF (V0.019-20241113, GRCh38_HG2-T2TQ100-V1.1_stvar), its confident-region BED, and the human_GRCh38_no_alt_analysis_set FASTA. Preprocess the benchmark with bcftools: select structural-variant records only via 'bcftools view -i INFO/SVTYPE!="."' and index with 'bcftools index -t', then restrict the confident-region BED to autosomes with 'grep -v chr[XY]'. Run Sentieon hap-eval genome-wide on each callset against the filtered benchmark VCF and autosome-restricted BED using the GRCh38 reference. Parse per-caller precision, recall, and F1 and compute the sawfish-vs-Sniffles2 and sawfish-vs-pbsv F1 margins; the rationale is that an F1 of 0.977 for sawfish, exceeding Sniffles2 by 3.7% and pbsv by 4.5%, confirms the Truvari-based ranking with a second, independent method.

## Related tools

- **Sentieon/hap-eval** (Independent VCF comparison engine used as the second benchmarking method to compute precision/recall/F1 per SV caller) — https://github.com/Sentieon/hap-eval
- **bcftools** (Preprocessing: selecting SVTYPE records from the T2T benchmark VCF ('bcftools view -i INFO/SVTYPE!="."') and tabix indexing)
- **sawfish** (Caller under evaluation; the 32x HG002 genotyped.sv.vcf.gz callset being verified for cross-method accuracy superiority) — https://github.com/PacificBiosciences/sawfish
- **pbsv** (Comparison SV caller whose 32x HG002 callset is benchmarked to reproduce the 4.5% F1 deficit vs sawfish) — https://github.com/PacificBiosciences/pbsv
- **truvari 4.2.2** (Primary (first-method) SV benchmarking engine whose rankings this skill independently verifies; used with MAFFT for refinement)

## Examples

```
bcftools view -i 'INFO/SVTYPE!="."' -Oz -o GRCh38_HG2-T2TQ100-V1.1_stvar.svtype.vcf.gz && bcftools index -t GRCh38_HG2-T2TQ100-V1.1_stvar.svtype.vcf.gz
```

## Evaluation signals

- sawfish F1 from hap-eval must match the reported 0.977 within tolerance
- Sniffles2 and pbsv F1 values must reproduce the reported deficits of 3.7% and 4.5% vs sawfish within tolerance
- Filtered benchmark VCF contains only structural-variant records (INFO/SVTYPE!=".") and is indexed with 'bcftools index -t'
- Confident-region BED contains no chrX or chrY lines after 'grep -v chr[XY]'
- Output table hapeval_t2t_32x_f1_scores.csv contains caller, precision, recall, F1, and margin-vs-sawfish columns for all three callers

## Limitations

- hap-eval is pre-release software under active development and may fail on some VCFs (per its README)
- The T2T benchmark V0.019-20241113 is a draft GIAB release; results reflect that draft, and sex chromosomes are excluded by the autosome-only confident-region restriction
- Truvari 4.2.2 with MAFFT remains the primary assessment method; hap-eval only confirms cross-method consistency, so divergent results do not by themselves identify which method is wrong

## Evidence

- [readme] An open-source VCF comparison engine for structural variant benchmarking: [evidence span withheld — anchor status: external_doc]
- [methods] bcftools benchmark SVTYPE filter: "bcftools view -i 'INFO/SVTYPE!="."' -Oz -o GRCh38_HG2-T2TQ100-V1.1_stvar.svtype.vcf.gz"
- [methods] confident regions restricted to autosomes: "The confident regions are restricted to autosomes as follows: grep -v chr[XY]"
- [methods] Truvari as primary assessment method: "All assessment below uses truvari 4.2.2 with MAFFT for refinement."
- [abstract] sawfish highest accuracy: "sawfish has the highest accuracy among state-of-the-art long-read SV callers across every tested SV size group."
