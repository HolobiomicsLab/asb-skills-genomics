---
name: gene-length-normalization-mutation-bias-correction
description: Use when computing per-gene mutational signature contribution (CCA) matrices
  from a MAF/VCF plus P and S matrices, and relative abundance η_s,g must be corrected
  for gene length before filtering gene–signature pairs.
when_to_use_negative:
- The input is already a gene-level abundance or normalized feature table rather than
  raw per-sample, per-category mutation counts (N_mgn) from a MAF/VCF — the correction
  formula requires raw counts and gene lengths l_g
- You are only extracting de novo signatures with denovoNMF or estimating S with InverseNMF
  and never produce gene-level CCA outputs
- No reference genome build can be supplied (genome.build/BSgenome), since gene lengths
  l_g cannot be resolved
edam_operation: http://edamontology.org/operation_3435
edam_topics:
- http://edamontology.org/topic_0622
tools:
- name: RNMF
  role: R package providing cumulativeCA and genePerMutSigs, which implement the gene-length-normalized
    relative abundance η_s,g and filter gene–signature pairs
  repo: https://github.com/zhenzhang-li/RNMF
- name: BSgenome.Hsapiens.UCSC.hg19
  role: Bioconductor genome build supplying gene lengths (l_g) for the normalization
    ρ'_nmg = (N_mgn/l_g)/Σ_i (N_min/l_i)
- name: BSgenome.Hsapiens.UCSC.hg38
  role: Alternative Bioconductor genome build supplying gene lengths for hg38-aligned
    MAF inputs
- name: R
  role: Language in which the RNMF CCA and normalization routines are implemented
    and run
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1002/cam4.4717
    title: A practical framework <scp>RNMF</scp> for exploring the association between
      mutational signatures and genes using gene cumulative contribution abundance
  source_content_hash: sha256:034c4bbd4a41a02654bb8ddcd0bfb18ac8bfb714f95be906e5a08e4afc558905
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/zhenzhang-li/RNMF
  tools:
  - BSgenome.Hsapiens.UCSC.hg19
  - BSgenome.Hsapiens.UCSC.hg38
  - R
  - RNMF
  edam_operation: http://edamontology.org/operation_3435
  edam_topics:
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 7
---

# gene-length-normalization-mutation-bias-correction

## Summary

Corrects gene-length bias in RNMF's gene-level cumulative contribution abundance (CCA) analysis by applying the length-normalized mutation fraction ρ'_nmg = (N_mgn/l_g) / Σ_i (N_min/l_i), so that long genes do not dominate signature–gene association results.

## When to use

Use when computing per-gene mutational signature contribution (CCA) matrices from a MAF/VCF plus P and S matrices, and relative abundance η_s,g must be corrected for gene length before filtering gene–signature pairs.

## When NOT to use

- The input is already a gene-level abundance or normalized feature table rather than raw per-sample, per-category mutation counts (N_mgn) from a MAF/VCF — the correction formula requires raw counts and gene lengths l_g
- You are only extracting de novo signatures with denovoNMF or estimating S with InverseNMF and never produce gene-level CCA outputs
- No reference genome build can be supplied (genome.build/BSgenome), since gene lengths l_g cannot be resolved

## Inputs

- MAF (or VCF) somatic mutation file with Tumor_Sample_Barcode, Chromosome, Start_position, Reference_Allele, Tumor_Seq_Allele2, Hugo_Symbol columns
- Mutational signature matrix P (COSMIC v3.2/v2.0 or denovoNMF-derived, e.g. SBS.Pmatrix.txt)
- Sample abundance fraction matrix S (e.g. SBS.Smatrix.txt)
- COSMIC signature type (AnalCOSMICSigType, e.g. 'SBS') and genome build (BSgenome.Hsapiens.UCSC.hg19/hg38, e.g. 'Ch37')
- Optional grouping file and gene-list sort file

## Outputs

- Per-gene geneCumulativeContributionAbundance table (e.g. samplesResults/SBS3.SBS.geneCumulativeContributionAbundance.txt) with cumulative abundance α_s,g and gene-length-corrected relative abundance η_s,g
- CCA matrix/heat-map visualization of gene–signature associations
- Filtered gene–signature pairs passing Mutfreq=0.04, threshold=0.06, Qvalue=0.1

## How to apply

Within RNMF, somatic mutations are imported from a MAF (or VCF) file and mapped to genes via Hugo_Symbol; the per-gene impact factor ρ_mgn = N_mgn / Σ_i N_min is first computed. For the relative abundance metric, apply the gene-length normalization ρ'_nmg = (N_mgn/l_g) / Σ_i (N_min/l_i), where l_g is the gene length drawn from the BSgenome.Hsapiens.UCSC.hg19 or hg38 build specified by genome.build (e.g. 'Ch37'). This normalized ρ' is combined with the signature probability ρ_s,mn = P_ms·S_sn / Σ_k P_mk·S_kn to yield θ_sg,mn and the aggregated relative abundance η_s,g = Σ_m Σ_n γ_sg,mn, while the unnormalized cumulative abundance α_s,g is kept separately. Gene–signature pairs are then filtered in genePerMutSigs with Mutfreq = 0.04, threshold = 0.06, Qvalue = 0.1 to remove low-confidence pairs before the geneCumulativeContributionAbundance table and CCA plots are produced.

## Related tools

- **RNMF** (R package providing cumulativeCA and genePerMutSigs, which implement the gene-length-normalized relative abundance η_s,g and filter gene–signature pairs) — https://github.com/zhenzhang-li/RNMF
- **BSgenome.Hsapiens.UCSC.hg19** (Bioconductor genome build supplying gene lengths (l_g) for the normalization ρ'_nmg = (N_mgn/l_g)/Σ_i (N_min/l_i))
- **BSgenome.Hsapiens.UCSC.hg38** (Alternative Bioconductor genome build supplying gene lengths for hg38-aligned MAF inputs)
- **R** (Language in which the RNMF CCA and normalization routines are implemented and run)

## Examples

```
genePerMutSigs(Sfile='SBS.Smatrix.txt', choose.Sigs='SBS3', Gfile='samplesResults/SBS3.SBS.geneCumulativeContributionAbundance.txt', file='Regular.ESCC.exon.WES.maf-input-nonsilent.txt', sample.id='Tumor_Sample_Barcode', Hugo='Hugo_Symbol', Mutfreq=0.04, threshold=0.06, Qvalue=0.1, plot=TRUE)
```

## Evaluation signals

- Output CSV/txt (e.g. cca_gene_abundance.csv or .geneCumulativeContributionAbundance.txt) exists and contains cumulative abundance values α_s,g and relative abundance values η_s,g for each gene
- Relative abundance η_s,g (length-corrected) differs from the raw cumulative abundance α_s,g for genes with extreme lengths, confirming l_g entered the calculation
- Filter thresholds Mutfreq=0.04 and Qvalue=0.1 were applied as specified: low-frequency gene–signature pairs are absent from the filtered output
- genome.build string (e.g. 'Ch37') and AnalCOSMICSigType (e.g. 'SBS') match the genome build of the MAF coordinates and the P-matrix signature type
- Signatures such as SBS3*, SBS17b*, and 'New' yield CCA gene matrices consistent with the paper's reported associations (e.g. APOBEC SBS2*/SBS13* with PIK3CA, age signature with TP53)

## Limitations

- Due to inconsistent sequencing background of the 1073-sample ESCC data, only coding regions were uniformly analyzed, which introduces limitations for gene-level abundance
- The framework spends more time on de novo signature extraction than previously published software; the 'Re-updated the initial value' correction and 'build a solution space' processes need future optimization
- No software version numbers, installation instructions, or code availability details beyond the GitHub repo are stated for the framework

## Evidence

- [methods] Apply gene-length normalization using ρ'_nmg = (N_mgn/l_g) / Σ_i (N_min/l_i) to correct for gene-length bias in the relative abundance metric.: [evidence span withheld — anchor status: not_found]
- [abstract] a key model of cumulative contribution abundance (CCA), which was designed to highlight the association between mutational signatures and genes: "a key model of cumulative contribution abundance (CCA), which was designed to highlight the association between mutational signatures and genes"
- [readme] genePerMutSigs(Sfile = Sfile, choose.Sigs = isig, Gfile = Gfile, file = maffile: "genePerMutSigs(Sfile = Sfile, choose.Sigs = isig, 
                 Gfile = Gfile, file = maffile"
- [methods] Then it relies on the Bioconductor library, such as BS.genome.Hsapiens.UCSC.hg19 or BS.genome.Hsapiens.UCSC.hg38: "Then it relies on the Bioconductor library, such as BS.genome.Hsapiens.UCSC.hg19 or BS.genome.Hsapiens.UCSC.hg38"
