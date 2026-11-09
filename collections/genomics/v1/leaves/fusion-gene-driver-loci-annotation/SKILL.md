---
name: fusion-gene-driver-loci-annotation
description: Use when a recurrent fusion-gene driver locus and its 5'/3' partner designation
  are known but the fusion partner gene and breakpoint are unknown, and crRNAs must
  orient Cas9 cutting so ONT reads extend toward the unsequenced partner.
when_to_use_negative:
- Input DNA is degraded/fragmented, e.g. FFPE material — the article states applying
  FUDGE crRNAs to FFPE material will most likely fail due to short read lengths
- Both fusion gene partners are already known and only breakpoint resolution is needed
  with low-input WGA-DNA — the article notes WGA in low-DNA situations only works
  with prior knowledge of both partners
- The fusion spans genes on opposite strands that cannot produce a continuous transcript
  — NanoFG filtering retains only same-strand continuous-transcript fusions
edam_topics:
- http://edamontology.org/topic_0622
tools:
- name: FUDGE
  role: Assay providing strand-directed CRISPR-Cas9 enrichment whose directionality
    is set by the crRNA strand choice at each driver locus
- name: IDT custom gRNA design tool
  role: Scores candidate crRNAs; guides chosen with maximum on-target and lowest off-target
    scores
- name: NanoFG
  role: Bioinformatic pipeline detecting fusion genes from the resulting long-read
    nanopore data
  repo: https://github.com/SdeBlank/NanoFG
- name: minimap2
  role: Maps nanopore reads to GRCh37 (v2.6) within NanoFG
  repo: https://github.com/lh3/minimap2
- name: NanoSV
  role: Optional SV caller (v1.2.4, default 'min_mapq=12, depth_support=False, mapq_flag=48')
    used by NanoFG to call the fusion-spanning SV
  repo: https://github.com/mroosmalen/nanosv
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-020-16641-7
    title: Partner independent fusion gene detection by multiplexed CRISPR-Cas9 enrichment
      and long read nanopore sequencing
  source_content_hash: sha256:106292d072ea5f20c88cbda79c93f2d2e005b33ee526ddd7fd1e9057de7f8d6a
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/SdeBlank/NanoFG
  tools:
  - FUDGE
  - IDT custom gRNA design tool
  - NanoFG
  - NanoSV
  - minimap2
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 10
---

# fusion-gene-driver-loci-annotation

## Summary

Designs strand-directed FUDGE crRNAs against known fusion-gene driver exon loci (e.g. BRAF Exon 10, EWSR1 Exon 7, SS18 Exon 9) so that Cas9-directed nanopore reads extend from the known driver into the unknown fusion partner, enabling partner- and breakpoint-independent fusion detection.

## When to use

Use when a recurrent fusion-gene driver locus and its 5'/3' partner designation are known but the fusion partner gene and breakpoint are unknown, and crRNAs must orient Cas9 cutting so ONT reads extend toward the unsequenced partner.

## When NOT to use

- Input DNA is degraded/fragmented, e.g. FFPE material — the article states applying FUDGE crRNAs to FFPE material will most likely fail due to short read lengths
- Both fusion gene partners are already known and only breakpoint resolution is needed with low-input WGA-DNA — the article notes WGA in low-DNA situations only works with prior knowledge of both partners
- The fusion spans genes on opposite strands that cannot produce a continuous transcript — NanoFG filtering retains only same-strand continuous-transcript fusions

## Inputs

- Target list of known fusion-gene driver exon loci with 5'/3' partner designation (e.g. BRAF Exon 10, EWSR1 Exon 7, SS18 Exon 9)
- GRCh37 human reference genome sequences for each driver exon (both strands)
- Genomic DNA or WGA-DNA (~1 μg, non-fragmented; WGA from 10 ng input supported)

## Outputs

- crRNAs.fasta containing one 20-nt strand-directed DNA guide per target driver locus
- Cas9 RNP-enriched nanopore read set (FASTQ) with ~68x average breakpoint-spanning coverage (~665-fold breakpoint-site enrichment)
- NanoFG fusion call output: candidate fusions requiring both SV ends in different genes and at least two fusion-spanning reads

## How to apply

Fetch exon reference sequences for each target driver locus from the GRCh37 human reference genome, retrieving both strands. Scan the fetched sequences for NGG PAM motifs and extract the 20-nt protospacer immediately 5' of every PAM candidate. Apply the FUDGE strand-specificity rule: keep only guides on the strand whose Cas9 cut orients sequencing toward the suspected 5' or 3' fusion partner. Rank retained guides with the IDT custom gRNA design tool criterion (maximum on-target score, lowest off-target score) and write exactly one selected guide per target locus to crRNAs.fasta. These crRNAs are then used in Cas9 RNPs on dephosphorylated genomic DNA (~1 μg), followed by dA-tailing, adapter ligation with Long Fragment Buffer cleanup (fragments <3 kb washed away), and GridION X5 nanopore sequencing analyzed by NanoFG (minimap2 v2.6 to GRCh37, NanoSV v1.2.4 or Sniffles v1.0.9).

## Related tools

- **FUDGE** (Assay providing strand-directed CRISPR-Cas9 enrichment whose directionality is set by the crRNA strand choice at each driver locus)
- **IDT custom gRNA design tool** (Scores candidate crRNAs; guides chosen with maximum on-target and lowest off-target scores)
- **NanoFG** (Bioinformatic pipeline detecting fusion genes from the resulting long-read nanopore data) — https://github.com/SdeBlank/NanoFG
- **minimap2** (Maps nanopore reads to GRCh37 (v2.6) within NanoFG) — https://github.com/lh3/minimap2
- **NanoSV** (Optional SV caller (v1.2.4, default 'min_mapq=12, depth_support=False, mapq_flag=48') used by NanoFG to call the fusion-spanning SV) — https://github.com/mroosmalen/nanosv

## Examples

```
bash NanoFG.sh -f /path/to/fastqdir -n SAMPLE_NAME -s SELECTION
```

## Evaluation signals

- crRNAs.fasta parses as valid FASTA with exactly one 20-nt DNA guide record per target driver locus
- Each retained guide sits immediately 5' of an NGG PAM and lies on the strand whose Cas9 cut orients sequencing toward the designated 5'/3' fusion partner
- Resulting sequencing achieves breakpoint-spanning coverage near the reported ~68x average (~665-fold breakpoint-site enrichment)
- NanoFG fusion calls pass the built-in NanoSV/Sniffles filters (insertions removed), flag both SV ends in different genes, and are supported by at least two fusion-spanning reads
- Identified fusions produce a continuous transcript on the same strand

## Limitations

- Requires non-fragmented (non-FFPE) genomic DNA; on-target coverage decreases for low-throughput runs and more distal breakpoint events
- WGA of low-input DNA (10 ng) is only useful for breakpoint identification when both fusion partners are already known
- Fusions detected with only one supporting read require manual validation by breakpoint PCR; current multiplexing via pooled demultiplex-PCR prolongs assay duration and increases processing complexity

## Evidence

- [abstract] FUDGE couples target-selected and strand-specific CRISPR-Cas9 activity for fusion gene driver enrichment: "FUDGE couples target-selected and strand-specific CRISPR-Cas9 activity for fusion gene driver enrichment"
- [abstract] We observe on-average 665 fold breakpoint-site enrichment: "We observe on-average 665 fold breakpoint-site enrichment"
- [discussion] Custom Alt-R crRNAs designed with IDT tool, maximum on-target and lowest off-target scores: "Custom Alt-R® crRNAs were designed with the Integrated DNA Technologies (IDT) custom gRNA design tool and chosen with maximum on-target and lowest off-target scores"
- [discussion] Limitation: requirement of non-fragmented DNA; FFPE likely fails: "A limitation of this approach is the requirement of non-fragmented DNA."
- [readme] NanoFG README: pipeline for detection of fusion genes from Nanopore sequencing data: "NanoFG integrates existing genomic tools and additonal code to provide an easy-to-use pipeline for the detection of fusion genes from Nanopore sequencing data"
