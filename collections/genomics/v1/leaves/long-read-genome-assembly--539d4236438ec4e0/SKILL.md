---
name: long-read-genome-assembly
description: Use when assembling a diploid genome from long noisy reads (PacBio CLR
  or Nanopore R9/R10 FASTQ/FASTA) and haplotype-resolved, haplotype-specific contigs
  are needed without parental/trio data.
when_to_use_negative:
- Input reads are high-accuracy PacBio HiFi or Nanopore duplex — the article reports
  PECAT achieves 'mediocre performance' there and tools like Hifiasm are better suited
- The genome is haploid or non-diploid — the haplotype-aware correction and two-round
  phasing pipeline is designed for diploid genomes
- Data coverage is low and no additional coverage can be obtained — PECAT 'needs higher
  coverage of data' for high-quality phased assembly
edam_operation: http://edamontology.org/operation_0525
edam_topics:
- http://edamontology.org/topic_0196
- http://edamontology.org/topic_0622
tools:
- name: PECAT
  role: 'Core pipeline: haplotype-aware error correction plus two-round string graph
    diploid assembly'
  repo: https://github.com/lemene/PECAT
- name: minimap2
  role: Candidate overlap detection ('-x ava-pb' for PacBio CLR) and read-to-contig
    mapping for phasing ('-x map-pb -c -p 0.5 -r 1000')
  repo: https://github.com/lh3/minimap2
- name: Clair3
  role: SNP re-calling on raw Nanopore reads to improve inconsistent overlap identification
  repo: https://github.com/HKU-BAL/Clair3
- name: Medaka
  role: Polishing of Nanopore assemblies to improve assembly quality
  repo: https://github.com/nanoporetech/medaka
- name: racon
  role: Consensus/polishing module within the PECAT pipeline (dependency v1.4.21+)
  repo: https://github.com/lbcb-sci/racon
- name: Pomoxis
  role: Reference-based evaluation of assembly base-level accuracy
  repo: https://github.com/nanoporetech/pomoxis
- name: hapdiff
  role: SV calling between the two haplotype assemblies against a curated SV set
  repo: https://github.com/KolmogorovLab/hapdiff
provenance:
  source_task_ids:
  - task_006
  - task_007
  source_papers:
  - doi: 10.1038/s41467-024-47349-7
    title: De novo diploid genome assembly using long noisy reads
  source_content_hash: sha256:7c1bcf4b336e974f35a37424550d60a5f4d361362dfb7c65b4aafd9462846f3d
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/lemene/PECAT
  tools:
  - Clair3
  - FALCON-Unzip
  - Medaka
  - NECAT
  - PECAT
  - PacBio CLR
  - Pomoxis
  - hapdiff
  - minimap2
  - racon
  edam_operation: http://edamontology.org/operation_0525
  edam_topics:
  - http://edamontology.org/topic_0196
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 17
---

# long-read-genome-assembly

## Summary

De novo diploid genome assembly from long noisy reads (PacBio CLR, Nanopore R9/R10) using PECAT, which combines haplotype-aware error correction that retains heterozygote alleles with a two-round string-graph assembly that removes inconsistent overlaps.

## When to use

Use when assembling a diploid genome from long noisy reads (PacBio CLR or Nanopore R9/R10 FASTQ/FASTA) and haplotype-resolved, haplotype-specific contigs are needed without parental/trio data.

## When NOT to use

- Input reads are high-accuracy PacBio HiFi or Nanopore duplex — the article reports PECAT achieves 'mediocre performance' there and tools like Hifiasm are better suited
- The genome is haploid or non-diploid — the haplotype-aware correction and two-round phasing pipeline is designed for diploid genomes
- Data coverage is low and no additional coverage can be obtained — PECAT 'needs higher coverage of data' for high-quality phased assembly

## Inputs

- Long noisy reads in FASTQ/FASTA (PacBio CLR, Nanopore R9, or Nanopore R10)
- PECAT config file generated from a dataset-matched template in demo/configs
- Optional reference genome for evaluation (BUSCO, Pomoxis, QUAST-style NG50)

## Outputs

- Two haplotype-resolved contig sets (hap1/hap2 FASTA) from 'pecat.pl unzip cfgfile'
- First-round haplotype-collapsed primary contigs ($PROJECT/3-assemble/primary.fasta)
- Contiguity metrics: NG50 and Merqury phase block NG50 (e.g., 79.6/86.1 Mb for B. taurus Bison×Simmental; 59.4/58.0 Mb for HG002 R10)
- Quality metrics: Merqury Hamming error, BUSCO completeness, Pomoxis base-level accuracy

## How to apply

Run 'pecat.pl config cfgfile' from a dataset-matched template in demo/configs (cfg_yeast_clr, cfg_arab_clr, cfg_dro_clr, cfg_cattle_clr, cfg_arab_ont, cfg_cattle_ont, cfg_hg002_ont) and execute 'pecat.pl unzip cfgfile'. PECAT first performs haplotype-aware error correction on the longest 80X raw reads (a POA-graph method that retains heterozygote alleles), assembles a haplotype-collapsed primary set, then in the second round maps reads to primary.fasta with minimap2 ('-x map-pb -c -p 0.5 -r 1000' for PacBio CLR), calls heterozygous SNPs, and removes inconsistent overlaps before reassembling. For corrected reads use the strict threshold of 8 different SNP alleles per read pair; for raw reads use the loose 6-allele threshold; SNP site coverage must lie in [10, 1000]. For Nanopore datasets enable Clair3-based SNP calling (v0.1-r12) and Medaka polishing (v1.7.2) via phase_clair3_command and polish_medaka_command (singularity/docker if conda conflicts occur); for large genomes add '-f 0.005' or '-f 0.002' to corr_rd2rd_options and align_rd2rd_options and set cleanup=1.

## Related tools

- **PECAT** (Core pipeline: haplotype-aware error correction plus two-round string graph diploid assembly) — https://github.com/lemene/PECAT
- **minimap2** (Candidate overlap detection ('-x ava-pb' for PacBio CLR) and read-to-contig mapping for phasing ('-x map-pb -c -p 0.5 -r 1000')) — https://github.com/lh3/minimap2
- **Clair3** (SNP re-calling on raw Nanopore reads to improve inconsistent overlap identification) — https://github.com/HKU-BAL/Clair3
- **Medaka** (Polishing of Nanopore assemblies to improve assembly quality) — https://github.com/nanoporetech/medaka
- **racon** (Consensus/polishing module within the PECAT pipeline (dependency v1.4.21+)) — https://github.com/lbcb-sci/racon
- **Pomoxis** (Reference-based evaluation of assembly base-level accuracy) — https://github.com/nanoporetech/pomoxis
- **hapdiff** (SV calling between the two haplotype assemblies against a curated SV set) — https://github.com/KolmogorovLab/hapdiff

## Examples

```
git clone --recursive https://github.com/lemene/PECAT.git; cd PECAT/demo/configs; pecat.pl config cfg_hg002_ont; pecat.pl unzip cfg_hg002_ont
```

## Evaluation signals

- Round-2 inconsistent-overlap percentage drops sharply below round-1 (reported: 36.98%→0.32%, 16.00%→0.57%, 15.94%→0.90%, 36.55%→0.63%, 16.62%→0.79%, 18.08%→0.03%, 37.45%→1.09% across the seven datasets)
- Phase block NG50 matches published Tables 1–2 values, e.g. 79.6/86.1 Mb for B. taurus Bison×Simmental and 59.4/58.0 Mb for HG002 Nanopore R10
- Merqury Hamming error and BUSCO scores are consistent with Table 1–2 headline values for the fixed parameter settings
- Both hap1 and hap2 contig sets pass structural-variant sanity checks with hapdiff against curated SVs
- The pipeline completes both rounds (primary.fasta from round 1 plus phased hap1/hap2 FASTA from round 2) without cleanup removing intermediates needed for verification (set cleanup=1 only after validation)

## Limitations

- SNP calling accuracy is low in low heterozygosity rate regions, even after error correction
- Error correction may not distinguish small sequencing errors from heterozygote alleles in very low heterozygosity regions
- PECAT achieves mediocre performance on Nanopore duplex and PacBio HiFi reads (Supplementary Table 12, Supplementary Figs. 21–24)
- PECAT needs higher data coverage to obtain a high-quality phased assembly
- Assembly parameters are described only in Supplementary Notes 4 and 5, not the main text; Clair3/Medaka may need singularity or docker if conda installations conflict

## Evidence

- [abstract] Here, we present PECAT, a Phased Error Correction and Assembly Tool, for reconstructing diploid genomes from long noisy reads.: "Here, we present PECAT, a Phased Error Correction and Assembly Tool, for reconstructing diploid genomes from long noisy reads."
- [results] inconsistent overlaps decreased sharply from first-round to second-round assemblies: [evidence span withheld — anchor status: not_found]
- [methods] SNP coverage range filter: "(1) Its coverage is in the range [10, 1000]."
- [discussion] strict vs loose SNP allele thresholds: "using corrected reads with the strict threshold that the pair of reads should contain 8 different SNP alleles"
- [discussion] map corrected reads to first-round assembly: "We map corrected reads to the first round of assembly using minimap2 with parameters “-x map-pb -c -p 0.5 -r 1000” for PacBio CLR reads"
- [methods] PECAT docker image availability: "git clone --recursive https://github.com/lemene/PECAT.git"
- [discussion] Clair3 re-calling for Nanopore reads: "Therefore, we use Clair3 to call SNPs again for Nanopore reads."
- [readme] PECAT dependency list from README: "PECAT is a phased error correction and assembly tool for long reads. It includes a haplotype-aware correction method and an efficient diploid assembly method."
- [discussion] limitations in low heterozygosity regions: "the accuracy of this SNP call is still low for low heterozygosity rate regions"
