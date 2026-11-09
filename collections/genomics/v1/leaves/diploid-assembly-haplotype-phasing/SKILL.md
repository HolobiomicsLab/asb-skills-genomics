---
name: diploid-assembly-haplotype-phasing
description: Use when assembling a diploid genome from long noisy reads (PacBio CLR,
  Nanopore R9/R10) and you need haplotype-specific contigs plus phasing metrics such
  as phase block NG50.
when_to_use_negative:
- Input reads are already high-fidelity (PacBio HiFi) or Nanopore duplex — the article
  reports PECAT achieves mediocre performance on these, where Hifiasm-style approaches
  are more appropriate
- The organism is haploid or you only need a collapsed reference assembly without
  haplotype phasing
- Data coverage is low — the article notes PECAT needs higher data coverage to obtain
  a high-quality phased assembly
edam_operation: http://edamontology.org/operation_0525
edam_topics:
- http://edamontology.org/topic_0196
- http://edamontology.org/topic_0622
tools:
- name: PECAT
  role: 'Main pipeline: haplotype-aware error correction, SNP-based read phasing,
    inconsistent-overlap removal, two-round string graph assembly'
  repo: https://github.com/lemene/PECAT
- name: minimap2
  role: Read-to-read overlap detection ('-x ava-pb' for PacBio CLR) and corrected-read-to-contig
    mapping for SNP calling
  repo: https://github.com/lh3/minimap2
- name: Clair3
  role: Recalls SNPs on raw Nanopore reads to improve inconsistent-overlap identification
  repo: https://github.com/HKU-BAL/Clair3
- name: Medaka
  role: Consensus polishing to further improve Nanopore assembly quality
  repo: https://github.com/nanoporetech/medaka
- name: racon
  role: Dependency of the PECAT pipeline (v1.4.21+)
  repo: https://github.com/lbcb-sci/racon
- name: Pomoxis
  role: Reference-based evaluation of assembly base-level accuracy
  repo: https://github.com/nanoporetech/pomoxis
- name: hapdiff
  role: Structural-variant comparison between phased haplotype assemblies and curated
    SV sets
  repo: https://github.com/KolmogorovLab/hapdiff
provenance:
  source_task_ids:
  - task_007
  - task_006
  source_papers:
  - doi: 10.1038/s41467-024-47349-7
    title: De novo diploid genome assembly using long noisy reads
  source_content_hash: sha256:b18f45afbd63c0baec603cda483a500e1f535f199c1a07fcd6497b80dac14373
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
n_claims: 18
---

# diploid-assembly-haplotype-phasing

## Summary

Reconstructs diploid genomes from long noisy reads (PacBio CLR, Nanopore R9/R10) using PECAT, which combines haplotype-aware error correction with a two-round string-graph assembly that phases reads by heterozygous SNPs and removes inconsistent overlaps.

## When to use

Use when assembling a diploid genome from long noisy reads (PacBio CLR, Nanopore R9/R10) and you need haplotype-specific contigs plus phasing metrics such as phase block NG50.

## When NOT to use

- Input reads are already high-fidelity (PacBio HiFi) or Nanopore duplex — the article reports PECAT achieves mediocre performance on these, where Hifiasm-style approaches are more appropriate
- The organism is haploid or you only need a collapsed reference assembly without haplotype phasing
- Data coverage is low — the article notes PECAT needs higher data coverage to obtain a high-quality phased assembly

## Inputs

- Long raw reads in FASTA/FASTQ from PacBio CLR, Nanopore R9, or Nanopore R10 (e.g. ultra-long) sequenced diploid samples
- Seven reference diploid datasets: S. cerevisiae SK1×Y12, A. thaliana Col-0×Cvi-0, A. thaliana Col-0×C24, D. melanogaster ISO1×A4, B. taurus Angus×Brahman, B. taurus Bison×Simmental, HG002 (deposited at Zenodo DOI 10.5281/zenodo.8380113)
- PECAT config file template matched to platform/genome size (demo/configs: cfg_yeast_clr, cfg_arab_clr, cfg_dro_clr, cfg_cattle_clr, cfg_arab_ont, cfg_cattle_ont, cfg_hg002_ont)

## Outputs

- First-round haplotype-collapsed contigs at $PROJECT/3-assemble/primary.fasta
- Two haplotype-specific phased assemblies with NG50, phase block NG50, Hamming error, BUSCO and Pomoxis accuracy metrics
- Inconsistent-overlap fraction table (e.g. inconsistent_overlap_fractions.tsv) comparing round-1 vs round-2 string graphs
- Medaka-polished Nanopore assemblies; hapdiff SV comparison against curated SV sets

## How to apply

Generate a PECAT config file (pecat.pl config cfgfile) matched to your data type from demo/configs templates (e.g. cfg_cattle_ont, cfg_arab_clr) and run 'pecat.pl unzip cfgfile' to produce first-round haplotype-collapsed contigs in $PROJECT/3-assemble/primary.fasta. PECAT first corrects raw reads with a haplotype-aware POA-based method that retains heterozygote alleles, then maps corrected reads back to the first-round assembly with minimap2 (e.g. '-x map-pb -c -p 0.5 -r 1000' for PacBio CLR) to call heterozygous SNPs. SNPs are accepted only when coverage is in the range [10, 1000] and the second-most common base count passes r_l/r_h/c_l/c_h thresholds (defaults 0.4, 0.2, 10, 100); for Nanopore data Clair3 (v0.1-r12+) recalls SNPs on raw reads. Inconsistent overlaps are identified with a strict threshold of 8 different SNP alleles for corrected reads and a loose threshold of 6 for raw reads, removed per phase_filter_options, and the genome is reassembled (asm2) — inconsistent-overlap percentages should drop sharply from round 1 to round 2. For Nanopore datasets enable Clair3 SNP calling and Medaka (v1.7.2+) polishing via phase_clair3_command and polish_medaka_command, invoking them through singularity/docker if conda conflicts occur; large genomes additionally need '-f 0.005' or '-f 0.002' added to corr_rd2rd_options and align_rd2rd_options with cleanup=1.

## Related tools

- **PECAT** (Main pipeline: haplotype-aware error correction, SNP-based read phasing, inconsistent-overlap removal, two-round string graph assembly) — https://github.com/lemene/PECAT
- **minimap2** (Read-to-read overlap detection ('-x ava-pb' for PacBio CLR) and corrected-read-to-contig mapping for SNP calling) — https://github.com/lh3/minimap2
- **Clair3** (Recalls SNPs on raw Nanopore reads to improve inconsistent-overlap identification) — https://github.com/HKU-BAL/Clair3
- **Medaka** (Consensus polishing to further improve Nanopore assembly quality) — https://github.com/nanoporetech/medaka
- **racon** (Dependency of the PECAT pipeline (v1.4.21+)) — https://github.com/lbcb-sci/racon
- **Pomoxis** (Reference-based evaluation of assembly base-level accuracy) — https://github.com/nanoporetech/pomoxis
- **hapdiff** (Structural-variant comparison between phased haplotype assemblies and curated SV sets) — https://github.com/KolmogorovLab/hapdiff

## Examples

```
git clone --recursive https://github.com/lemene/PECAT.git && pecat.pl config cfgfile && pecat.pl unzip cfgfile
```

## Evaluation signals

- Round-2 inconsistent-overlap percentage must be lower than round-1 for every dataset; reported round-1 values 36.98%, 16.00%, 15.94%, 36.55%, 16.62%, 18.08%, 37.45% drop to round-2 values 0.32%, 0.57%, 0.90%, 0.63%, 0.79%, 0.03%, 1.09% across the seven diploid datasets
- Phase block NG50 of the two haplotype assemblies should reach published values, e.g. 79.6/86.1 Mb for B. taurus Bison×Simmental (Nanopore R9) and 59.4/58.0 Mb for HG002 Nanopore R10
- Every accepted SNP site must satisfy the coverage filter range [10, 1000]; corrected-read inconsistent-overlap calls require ≥8 different SNP alleles, raw-read calls ≥6
- BUSCO completeness and Pomoxis reference-based accuracy metrics should be tabulated per assembly and compared against Tables 1–2 within tolerance for the fixed parameter settings
- First-round output file $PROJECT/3-assemble/primary.fasta must exist after 'pecat.pl unzip cfgfile' completes

## Limitations

- SNP calling accuracy is low in low heterozygosity rate regions
- Error correction may not distinguish small sequencing errors from heterozygote alleles in very low heterozygosity regions
- PECAT achieves mediocre performance on Nanopore duplex and PacBio HiFi reads
- PECAT needs higher data coverage to obtain a high-quality phased assembly
- Assembly parameters are only described in Supplementary Notes 4 and 5, not in the main text

## Evidence

- [intro] PECAT follows the correct-then-assemble strategy, including a haplotype-aware error correction module: "PECAT follows the correct-then-assemble strategy, including a haplotype-aware error correction module, which can retain heterozygote alleles while correcting sequencing errors"
- [abstract] Two-round string graph assembly minimizes local alignments and removes inconsistent overlaps: "We combine a corrected read SNP caller and a raw read SNP caller to further improve the identification of inconsistent overlaps in the string graph."
- [discussion] SNP coverage filter range: "(1) Its coverage is in the range [10, 1000]."
- [discussion] Strict vs loose SNP-allele thresholds for inconsistent overlaps: "using corrected reads with the strict threshold that the pair of reads should contain 8 different SNP alleles"
- [discussion] Clair3 for Nanopore SNP calling: "Therefore, we use Clair3 to call SNPs again for Nanopore reads."
- [readme] Clair3 and medaka are optional dependencies of the PECAT pipeline: "+ [clair3](https://github.com/HKU-BAL/Clair3)  (v0.1-r12+) (optional)"
- [discussion] minimap2 overlap detection parameters: "we find candidate overlaps between raw reads using minimap2 with parameters “-x ava-pb” for PacBio CLR reads"
