---
name: cumulative-contribution-abundance-computation
description: Use when a MAF (or VCF) somatic mutation dataset is paired with known
  signature matrices P (e.g., COSMIC v3.2 SBS) and S, and you need per-gene signature-gene
  association (CCA) tables for downstream filtering or visualization.
when_to_use_negative:
- You only need de novo signature extraction without gene-level association — use
  denovoNMF instead of cumulativeCA
- Input lacks a P matrix and S matrix (or COSMIC signature choice) for the signatures
  of interest, so per-signature CCA cannot be attributed
- Only a single small tumor sample without a reliable S matrix — InverseNMF is the
  intended route for small datasets/single samples
edam_operation: http://edamontology.org/operation_0362
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_2640
tools:
- name: RNMF
  role: R package providing cumulativeCA, genePerMutSigs, samPerMutSigs, samFisherSigs,
    and eachMutationCA to compute per-gene CCA from P and S matrices and a MAF file
  repo: https://github.com/zhenzhang-li/RNMF
- name: BSgenome.Hsapiens.UCSC.hg19
  role: Bioconductor genome-build dependency used by cumulativeCA to classify mutation
    contexts (genome.build='Ch37')
- name: BSgenome.Hsapiens.UCSC.hg38
  role: Bioconductor genome-build dependency (genome.build='Ch38') alternative for
    mutation context annotation
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1002/cam4.4717
    title: A practical framework <scp>RNMF</scp> for exploring the association between
      mutational signatures and genes using gene cumulative contribution abundance
  source_content_hash: sha256:1930efbbd6f6cb72374ecd6455d4b6dfb66e501d9065ab5138c842556fe26540
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/zhenzhang-li/RNMF
  tools:
  - BSgenome.Hsapiens.UCSC.hg19
  - BSgenome.Hsapiens.UCSC.hg38
  - R
  - RNMF
  edam_operation: http://edamontology.org/operation_0362
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_2640
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9333333333333333
n_claims: 15
---

# cumulative-contribution-abundance-computation

## Summary

Computes per-gene cumulative contribution abundance (CCA) matrices with the RNMF R package to quantify the association between COSMIC or de novo mutational signatures and individual genes, using signature matrix P, abundance matrix S, and a MAF mutation file.

## When to use

Use when a MAF (or VCF) somatic mutation dataset is paired with known signature matrices P (e.g., COSMIC v3.2 SBS) and S, and you need per-gene signature-gene association (CCA) tables for downstream filtering or visualization.

## When NOT to use

- You only need de novo signature extraction without gene-level association — use denovoNMF instead of cumulativeCA
- Input lacks a P matrix and S matrix (or COSMIC signature choice) for the signatures of interest, so per-signature CCA cannot be attributed
- Only a single small tumor sample without a reliable S matrix — InverseNMF is the intended route for small datasets/single samples

## Inputs

- MAF somatic mutation file (columns: Tumor_Sample_Barcode, Chromosome, Start_position, Reference_Allele, Tumor_Seq_Allele2, Hugo_Symbol)
- Signature probability matrix P (COSMIC v3.2/v2.0 SBS or denovoNMF-derived, e.g., SBS.Pmatrix.txt)
- Sample abundance fraction matrix S (e.g., SBS.Smatrix.txt)
- Group file for tumor grouping (Groupfile.txt)
- Optional gene list sorting file (geneListSortFile)
- Genome build annotation via BSgenome.Hsapiens.UCSC.hg19 or hg38

## Outputs

- Per-signature geneCumulativeContributionAbundance table (e.g., samplesResults/SBS3.SBS.geneCumulativeContributionAbundance.txt) with cumulative abundance α_s,g and relative abundance η_s,g per gene
- CCA matrix visualization (heatmap image of gene-by-signature cumulative contribution abundance)
- Filtered gene-signature association pairs passing Mutfreq=0.04, threshold=0.06, Qvalue=0.1

## How to apply

Load the signature matrix P (COSMIC v3.2/v2.0 or denovoNMF-derived) and sample abundance fraction matrix S in R, import somatic mutations from a MAF file mapping sample.id='Tumor_Sample_Barcode' and Hugo='Hugo_Symbol', and call cumulativeCA with Filetype='MAF', AnalCOSMICSigType='SBS', genome.build='Ch37' (or Ch38), and a groupFile. The model computes the per-gene impact factor ρ_mgn (fraction of category-m mutations in tumor n in gene g), the signature-specific probability ρ_s,mn = P_ms·S_sn / Σ_k P_mk·S_kn, aggregates θ_sg,mn over mutation categories M and tumors N into cumulative abundance α_s,g and relative abundance η_s,g, and applies gene-length normalization ρ'_nmg = (N_mgn/l_g)/Σ_i(N_min/l_i) to correct gene-length bias. Then, for each chosen signature (e.g., SBS3, SBS17b, New), run samFisherSigs, samPerMutSigs, and genePerMutSigs with filter thresholds Mutfreq=0.04, threshold=0.06, Qvalue=0.1, plot=TRUE to discard low-confidence gene-signature pairs and produce the geneCumulativeContributionAbundance table and CCA visualization.

## Related tools

- **RNMF** (R package providing cumulativeCA, genePerMutSigs, samPerMutSigs, samFisherSigs, and eachMutationCA to compute per-gene CCA from P and S matrices and a MAF file) — https://github.com/zhenzhang-li/RNMF
- **BSgenome.Hsapiens.UCSC.hg19** (Bioconductor genome-build dependency used by cumulativeCA to classify mutation contexts (genome.build='Ch37'))
- **BSgenome.Hsapiens.UCSC.hg38** (Bioconductor genome-build dependency (genome.build='Ch38') alternative for mutation context annotation)

## Examples

```
cumulativeCA(file=maffile, Filetype='MAF', Pfile=Pfile, Sfile=Sfile, sample.id='Tumor_Sample_Barcode', Hugo='Hugo_Symbol', AnalCOSMICSigType='SBS', genome.build="Ch37", groupFile=Groupfile, plot=TRUE)
```

## Evaluation signals

- Output CSV/table exists and contains cumulative abundance α_s,g and relative abundance η_s,g values for every gene in the MAF
- genePerMutSigs/samPerMutSigs/samFisherSigs runs use the exact filter thresholds Mutfreq=0.04, threshold=0.06, Qvalue=0.1 as specified
- Per-signature output files follow the naming pattern samplesResults/<sig>.SBS.geneCumulativeContributionAbundance.txt for every entry of Sigstype
- Relative abundance metric shows gene-length normalization applied (ρ'_nmg computed with gene length l_g denominator)
- plot=TRUE produces the CCA matrix image for each signature; sanity-check known associations (e.g., SBS2/SBS13 with PIK3CA) appear

## Limitations

- Framework is slower than previously published de novo signature extraction software due to the 'Re-updated the initial value' correction and 'build a solution space' processes; needs future optimization
- Analysis is limited by inconsistent sequencing background of the pooled data; only coding regions were uniformly analyzed
- Filtering removes gene-signature pairs below Mutfreq=0.04 / Qvalue=0.1, so low-frequency but potentially real associations are excluded

## Evidence

- [methods] genePerMutSigs(Sfile = Sfile, choose.Sigs = isig, Gfile = Gfile, file = maffile: "genePerMutSigs(Sfile = Sfile, choose.Sigs = isig, 
                 Gfile = Gfile, file = maffile"
- [methods] Mutfreq = 0.04, threshold = 0.06, Qvalue = 0.1: "Mutfreq = 0.04, threshold = 0.06, Qvalue = 0.1"
- [readme] Gfile = paste('samplesResults/',isig,'.',AnalCOSMICSigType,'.geneCumulativeContributionAbundance.txt',sep=""): "Gfile = paste('samplesResults/',isig,'.',AnalCOSMICSigType,'.geneCumulativeContributionAbundance.txt',sep="")"
- [readme] cumulativeCA(file = maffile, Filetype = 'MAF', Pfile = Pfile, Sfile = Sfile,: "cumulativeCA(file = maffile, Filetype = 'MAF', 
             Pfile = Pfile, Sfile = Sfile, "
- [methods] somatic variants can be imported from a Variant Call Format (VCF) file or a Mutation Annotation Format (MAF) file: "somatic variants can be imported from a Variant Call Format (VCF) file or a Mutation Annotation Format (MAF) file"
- [abstract] including a key model of cumulative contribution abundance (CCA), which was designed to highlight the association between mutational signatures and genes: "including a key model of cumulative contribution abundance (CCA), which was designed to highlight the association between mutational signatures and genes"
