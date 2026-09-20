---
name: crrna-guide-sequence-design
description: Use when the fusion-driver exon of a known gene (e.g., BRAF Exon 10,
  EWSR1 Exon 7, SS18 Exon 9) is known but the fusion partner gene and breakpoint are
  unknown and Cas9-enriched nanopore sequencing is planned.
when_to_use_negative:
- Both fusion partners and the breakpoint are already known — then targeted breakpoint
  PCR or a two-partner design is sufficient, and the article notes WGA on low-input
  DNA only works with prior knowledge of both partners
- Input DNA is fragmented, e.g. FFPE material — the FUDGE crRNA protocol is reported
  to most likely fail on such degraded DNA
- No NGG PAM exists on the strand that would orient Cas9 cutting toward the unknown
  fusion partner at the driver exon — a strand-directed guide cannot be designed and
  the enrichment directionality requirement fails
tools:
- name: FUDGE
  role: Defines the strand-directed crRNA design rule that gives Cas9 enrichment its
    directionality toward the unknown fusion partner
- name: IDT custom gRNA design tool
  role: Ranks candidate guides by on-target and off-target scores; guides with maximum
    on-target and lowest off-target scores are selected
- name: NanoFG
  role: 'Downstream consumer: the designed crRNAs produce enriched nanopore reads
    that NanoFG analyzes to detect the fusion'
  repo: https://github.com/SdeBlank/NanoFG
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-020-16641-7
    title: Partner independent fusion gene detection by multiplexed CRISPR-Cas9 enrichment
      and long read nanopore sequencing
  source_content_hash: sha256:7c97be2117e107ccdc1fbc52f2143685f1dc75dc825c21c0f6e0babd74d1dfec
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/SdeBlank/NanoFG
  tools:
  - FUDGE
  - IDT custom gRNA design tool
  - NanoFG
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# crrna-guide-sequence-design

## Summary

Design strand-directed CRISPR-Cas9 crRNAs against known fusion-driver exon loci so that nanopore sequencing reads extend from the cut site into the unknown fusion partner, as implemented in FUDGE (FUsion Detection from Gene Enrichment).

## When to use

Use when the fusion-driver exon of a known gene (e.g., BRAF Exon 10, EWSR1 Exon 7, SS18 Exon 9) is known but the fusion partner gene and breakpoint are unknown and Cas9-enriched nanopore sequencing is planned.

## When NOT to use

- Both fusion partners and the breakpoint are already known — then targeted breakpoint PCR or a two-partner design is sufficient, and the article notes WGA on low-input DNA only works with prior knowledge of both partners
- Input DNA is fragmented, e.g. FFPE material — the FUDGE crRNA protocol is reported to most likely fail on such degraded DNA
- No NGG PAM exists on the strand that would orient Cas9 cutting toward the unknown fusion partner at the driver exon — a strand-directed guide cannot be designed and the enrichment directionality requirement fails

## Inputs

- List of known fusion-gene driver exon loci (e.g., BRAF Exon 10, EWSR1 Exon 7, SS18 Exon 9) with each gene's 5'/3' fusion-partner designation
- GRCh37 human reference genome exon sequences for each target locus, retrieved for both strands

## Outputs

- crRNAs.fasta containing one selected 20-nt DNA guide sequence per target locus, with one FASTA header per locus
- Ranked list of candidate protospacers with on-target and off-target scores from the IDT custom gRNA design tool

## How to apply

For each known driver exon locus, fetch the reference exon sequence (both strands) from the GRCh37 human reference genome, then scan every fetched sequence for NGG PAM motifs and extract the 20-nt protospacer immediately 5' of each PAM candidate. Apply the FUDGE strand-specificity rule: keep only guides whose Cas9 cut orients nanopore reads toward the suspected fusion partner, dictated by whether the known gene contributes its 5' or 3' end to the fusion (reads extend upstream or downstream of the crRNA accordingly). Rank retained guides with the IDT custom gRNA design tool criteria — maximum on-target score and lowest off-target score — and select one guide per target locus. Write the selected 20-nt guide sequences, one FASTA header per target locus, to crRNAs.fasta for use as Alt-R crRNAs in subsequent Cas9 RNP cutting and adapter-ligation nanopore library prep.

## Related tools

- **FUDGE** (Defines the strand-directed crRNA design rule that gives Cas9 enrichment its directionality toward the unknown fusion partner)
- **IDT custom gRNA design tool** (Ranks candidate guides by on-target and off-target scores; guides with maximum on-target and lowest off-target scores are selected)
- **NanoFG** (Downstream consumer: the designed crRNAs produce enriched nanopore reads that NanoFG analyzes to detect the fusion) — https://github.com/SdeBlank/NanoFG

## Evaluation signals

- crRNAs.fasta parses as a valid FASTA in which every record is a 20-nt DNA guide
- Exactly one selected guide is present per target locus header
- Every retained guide lies on the strand whose Cas9 cut orients sequencing toward the unknown fusion partner, per the given 5'/3' designation of the known gene
- Each selected guide corresponds to a 20-nt protospacer immediately 5' of an NGG PAM candidate in the GRCh37 exon reference
- Selected guides match the ranking criterion of maximum on-target and lowest off-target scores from the IDT custom gRNA design tool

## Limitations

- The assay requires non-fragmented DNA; applying the FUDGE crRNA protocol to FFPE material will most likely fail to comprehensively identify fusion genes
- Demonstrated design scope covers ten recurrent fusion genes; the article states expansion to any gene of interest is possible but was not broadly shown
- A decrease in on-target coverage was observed for low-throughput sequencing runs and/or more distal breakpoint events, reducing effective enrichment for guides whose partner breakpoints are far from the cut site

## Evidence

- [abstract] FUDGE couples target-selected and strand-specific CRISPR-Cas9 activity for fusion gene driver enrichment: "FUDGE couples target-selected and strand-specific CRISPR-Cas9 activity for fusion gene driver enrichment"
- [abstract] identify fusion breakpoints without prior knowledge of partner: "without prior knowledge of fusion partner or breakpoint-location"
- [discussion] IDT guide ranking criterion: "Custom Alt-R® crRNAs were designed with the Integrated DNA Technologies (IDT) custom gRNA design tool and chosen with maximum on-target and lowest off-target scores"
- [discussion] GRCh37 reference used for sequence retrieval context: "Reads were mapped to the human reference genome version GRCHh37 by using minimap2 (v. 2.6)"
