---
name: mutational-signature-gene-association-analysis
description: Use when you have a MAF (or VCF) somatic mutation file together with
  signature matrix P and sample abundance matrix S (COSMIC v3.2/v2.0 or denovoNMF-derived)
  and need per-gene, per-signature association quantification or filtering.
when_to_use_negative:
- Input is already a gene-signature association table or CCA matrix — re-running cumulativeCA
  is redundant
- You only need de novo signature extraction without gene-level association — use
  denovoNMF directly instead
- You have a very small dataset or single tumor sample with user-defined signatures
  — use InverseNMF, which was designed for that setting
edam_operation: http://edamontology.org/operation_3222
edam_topics:
- http://edamontology.org/topic_3676
- http://edamontology.org/topic_0199
tools:
- name: RNMF
  role: R package providing cumulativeCA, genePerMutSigs, samPerMutSigs and samFisherSigs
    to compute and filter gene cumulative contribution abundance from P/S matrices
    and a MAF file
  repo: https://github.com/zhenzhang-li/RNMF
- name: BSgenome.Hsapiens.UCSC.hg19
  role: Bioconductor genome build dependency used by cumulativeCA for trinucleotide
    context reconstruction (genome.build='Ch37')
- name: BSgenome.Hsapiens.UCSC.hg38
  role: Alternative Bioconductor genome build dependency for mutation context assignment
    in cumulativeCA
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1002/cam4.4717
    title: A practical framework <scp>RNMF</scp> for exploring the association between
      mutational signatures and genes using gene cumulative contribution abundance
  source_content_hash: sha256:f594a83777495e5ef748e9441114cec77e1d8634dbe8b282850cc829042b351f
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/zhenzhang-li/RNMF
  tools:
  - BSgenome.Hsapiens.UCSC.hg19
  - BSgenome.Hsapiens.UCSC.hg38
  - COSMIC Mutational Signatures
  - R
  - RNMF
  edam_operation: http://edamontology.org/operation_3222
  edam_topics:
  - http://edamontology.org/topic_3676
  - http://edamontology.org/topic_0199
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8666666666666667
n_claims: 15
---

# mutational-signature-gene-association-analysis

## Summary

Computes gene-level cumulative contribution abundance (CCA) using the RNMF R package to quantify and visualize the association between mutational signatures (COSMIC SBS/DBS/ID or de novo NMF) and individual genes from a MAF file plus P and S signature matrices.

## When to use

Use when you have a MAF (or VCF) somatic mutation file together with signature matrix P and sample abundance matrix S (COSMIC v3.2/v2.0 or denovoNMF-derived) and need per-gene, per-signature association quantification or filtering.

## When NOT to use

- Input is already a gene-signature association table or CCA matrix — re-running cumulativeCA is redundant
- You only need de novo signature extraction without gene-level association — use denovoNMF directly instead
- You have a very small dataset or single tumor sample with user-defined signatures — use InverseNMF, which was designed for that setting

## Inputs

- MAF or VCF somatic mutation file (e.g. 'ESCC.exon.ALL.input.maf' with Tumor_Sample_Barcode, Hugo_Symbol, Chromosome, Start_position, Reference_Allele, Tumor_Seq_Allele2 columns)
- Signature matrix P (COSMIC v3.2 or v2.0, or denovoNMF-derived, e.g. SBS.Pmatrix.txt)
- Sample abundance fraction matrix S (e.g. SBS.Smatrix.txt)
- Genome build specification (Ch37/hg19 or hg38 via BSgenome libraries)
- Optional grouping file (Groupfile.txt) and gene-list sort file

## Outputs

- Per-signature geneCumulativeContributionAbundance table (e.g. samplesResults/SBS3.SBS.geneCumulativeContributionAbundance.txt) with α_s,g and η_s,g values
- Filtered gene-signature association results (Mutfreq=0.04, threshold=0.06, Qvalue=0.1)
- CCA matrix visualization plots linking genes to mutational signatures

## How to apply

Load the signature matrix P and sample abundance fraction matrix S, then import the MAF with column mappings sample.id='Tumor_Sample_Barcode', Hugo='Hugo_Symbol', chromosome/position/ref/alt columns and a genome build (BSgenome.Hsapiens.UCSC.hg19 or hg38) via AnalCOSMICSigType='SBS'. Run cumulativeCA to produce per-gene geneCumulativeContributionAbundance tables (α_s,g cumulative and η_s,g relative abundance, with gene-length normalization ρ'_nmg to correct for gene-length bias). For each chosen signature (e.g. SBS3, SBS16, SBS18, New, SBS1, SBS2, SBS13, SBS17b, SBS22), run samFisherSigs, samPerMutSigs and genePerMutSigs with Mutfreq=0.04, threshold=0.06, Qvalue=0.1 to filter low-confidence gene-signature pairs and plot the CCA matrix. The rationale is that CCA aggregates the signature-specific contribution θ_sg,mn over all mutation categories M and tumors N, exposing which genes are driven by which signatures (e.g. APOBEC SBS2*/SBS13* with PIK3CA).

## Related tools

- **RNMF** (R package providing cumulativeCA, genePerMutSigs, samPerMutSigs and samFisherSigs to compute and filter gene cumulative contribution abundance from P/S matrices and a MAF file) — https://github.com/zhenzhang-li/RNMF
- **BSgenome.Hsapiens.UCSC.hg19** (Bioconductor genome build dependency used by cumulativeCA for trinucleotide context reconstruction (genome.build='Ch37'))
- **BSgenome.Hsapiens.UCSC.hg38** (Alternative Bioconductor genome build dependency for mutation context assignment in cumulativeCA)

## Examples

```
cumulativeCA(file=maffile, Filetype='MAF', Pfile='SBS.Pmatrix.txt', Sfile='SBS.Smatrix.txt', sample.id='Tumor_Sample_Barcode', Hugo='Hugo_Symbol', AnalCOSMICSigType='SBS', genome.build='Ch37', plot=TRUE)
```

## Evaluation signals

- Output geneCumulativeContributionAbundance table exists for every signature in the Sigstype loop and contains cumulative (α_s,g) and relative (η_s,g) abundance values per gene
- Filter thresholds Mutfreq=0.04, threshold=0.06 and Qvalue=0.1 are applied in samFisherSigs/samPerMutSigs/genePerMutSigs calls as specified
- Gene-length normalization ρ'_nmg = (N_mgn/l_g)/Σ_i(N_min/l_i) is applied so relative abundance is not biased toward long genes
- MAF column mappings (Tumor_Sample_Barcode, Hugo_Symbol) match the actual input file headers and the genome.build matches the sequencing reference
- Biologically expected associations reproduce, e.g. APOBEC signatures SBS2*/SBS13* linked to PIK3CA (E545k) and age signature linked to TP53 R342*

## Limitations

- Framework is slower than previously published de novo signature extraction software due to the initial-value re-update and solution-space building processes
- Analysis limited by inconsistent sequencing background of pooled data; only coding regions were uniformly analyzed, introducing bias
- Mechanistic links (e.g. APOBEC–PIK3CA) are hypothesized and require verification by subsequent experiments

## Evidence

- [methods] cumulativeCA takes a MAF file, the P and S matrices, a chosen COSMIC signature type, genome build, and grouping information as inputs and produces per-gene cumulative contribution abundance outputs: [evidence span withheld — anchor status: not_found]
- [methods] Mutfreq = 0.04, threshold = 0.06, Qvalue = 0.1: "Mutfreq = 0.04, threshold = 0.06, Qvalue = 0.1"
- [methods] Then it relies on the Bioconductor library, such as BS.genome.Hsapiens.UCSC.hg19 or BS.genome.Hsapiens.UCSC.hg38: "Then it relies on the Bioconductor library, such as BS.genome.Hsapiens.UCSC.hg19 or BS.genome.Hsapiens.UCSC.hg38"
- [readme] genePerMutSigs(Sfile = Sfile, choose.Sigs = isig, Gfile = Gfile, file = maffile: "genePerMutSigs(Sfile = Sfile, choose.Sigs = isig, Gfile = Gfile, file = maffile"
- [abstract] APOBEC signatures (SBS2* and SBS13*) were closely associated with PIK3CA mutation, especially the E545k mutation: "APOBEC signatures (SBS2* and SBS13*) were closely associated with PIK3CA mutation, especially the E545k mutation"
