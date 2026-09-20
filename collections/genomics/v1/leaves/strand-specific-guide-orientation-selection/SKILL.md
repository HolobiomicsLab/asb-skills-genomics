---
name: strand-specific-guide-orientation-selection
description: Use when designing enrichment guides for a known fusion-gene driver locus
  whose fusion partner gene and breakpoint are unknown, and long reads must be directed
  outward from the crRNA cut site into the uncharacterized partner.
when_to_use_negative:
- The fusion partner gene and breakpoint location are already known (no strand-directional
  enrichment is needed; standard two-sided guide design suffices)
- Input DNA is degraded/fragmented FFPE material, where the FUDGE crRNA protocol 'will
  most likely fail to comprehensively identify fusion genes'
- Only a single guide per locus is needed regardless of orientation, e.g., for non-directional
  target enrichment rather than partner discovery
edam_topics:
- http://edamontology.org/topic_3168
- http://edamontology.org/topic_0622
tools:
- name: IDT custom gRNA design tool
  role: Scores candidate crRNAs for on-target efficiency and off-target risk; guides
    are chosen with maximum on-target and lowest off-target scores after strand filtering
- name: FUDGE
  role: The fusion-detection assay whose strand-directed CRISPR-Cas9 enrichment strategy
    this guide-selection rule implements
- name: NanoFG
  role: Downstream pipeline that detects fusion genes from the strand-directed nanopore
    reads enriched by the selected crRNAs
  repo: https://github.com/SdeBlank/NanoFG
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-020-16641-7
    title: Partner independent fusion gene detection by multiplexed CRISPR-Cas9 enrichment
      and long read nanopore sequencing
  source_content_hash: sha256:25fb70269ba690ab893d80628e79f95ac4fb3e76972514637e0c2b9ee27e5d51
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/SdeBlank/NanoFG
  tools:
  - FUDGE
  - IDT custom gRNA design tool
  - NanoFG
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3168
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 7
---

# Strand-specific guide orientation selection for fusion-directed sequencing

## Summary

A crRNA design procedure used in the FUDGE assay in which only CRISPR-Cas9 guides on the strand that orients nanopore reads from the known fusion-gene driver (e.g., BRAF exon 10, EWSR1 exon 7, SS18 exon 9) toward the unknown 5' or 3' fusion partner are retained, enabling partner- and breakpoint-independent fusion detection.

## When to use

Use when designing enrichment guides for a known fusion-gene driver locus whose fusion partner gene and breakpoint are unknown, and long reads must be directed outward from the crRNA cut site into the uncharacterized partner.

## When NOT to use

- The fusion partner gene and breakpoint location are already known (no strand-directional enrichment is needed; standard two-sided guide design suffices)
- Input DNA is degraded/fragmented FFPE material, where the FUDGE crRNA protocol 'will most likely fail to comprehensively identify fusion genes'
- Only a single guide per locus is needed regardless of orientation, e.g., for non-directional target enrichment rather than partner discovery

## Inputs

- List of known fusion-gene driver exon loci with 5'/3' partner designation (e.g., BRAF exon 10, EWSR1 exon 7, SS18 exon 9)
- Exon reference sequences from the GRCh37 human reference genome (both strands)
- Guide candidate scores from the IDT custom gRNA design tool (on-target and off-target)

## Outputs

- crRNAs.fasta containing exactly one selected 20-nt strand-directed DNA guide per target locus, each with a header naming its target locus

## How to apply

First fetch the exon reference sequence for each target driver locus (e.g., BRAF exon 10, EWSR1 exon 7, SS18 exon 9) from the GRCh37 human reference genome, retrieving both strands. Scan for NGG PAM motifs and extract the 20-nt protospacer immediately 5' of each PAM candidate. Apply the FUDGE strand-specificity rule: keep only guides whose Cas9 cut orientation sends sequencing reads upstream or downstream toward the suspected 5' or 3' fusion partner as designated for that known driver gene. Rank the retained guides with the IDT custom gRNA design tool, choosing those with maximum on-target and lowest off-target scores, then write one selected 20-nt guide per target locus to crRNAs.fasta.

## Related tools

- **IDT custom gRNA design tool** (Scores candidate crRNAs for on-target efficiency and off-target risk; guides are chosen with maximum on-target and lowest off-target scores after strand filtering)
- **FUDGE** (The fusion-detection assay whose strand-directed CRISPR-Cas9 enrichment strategy this guide-selection rule implements)
- **NanoFG** (Downstream pipeline that detects fusion genes from the strand-directed nanopore reads enriched by the selected crRNAs) — https://github.com/SdeBlank/NanoFG

## Evaluation signals

- crRNAs.fasta parses as valid FASTA with one record per target driver locus
- Every record is a 20-nt DNA guide sequence with no ambiguous bases
- Each retained guide lies on the strand whose Cas9 cut direction points reads toward the designated 5' or 3' fusion partner of its driver gene
- Each guide corresponds to a real NGG PAM candidate with its protospacer immediately 5' of the PAM in GRCh37
- Selected guides show maximum on-target and lowest off-target scores among strand-compatible candidates per the IDT design tool ranking

## Limitations

- Requires non-fragmented, high-quality DNA; application to FFPE material will most likely fail to comprehensively identify fusion genes
- Only filters guide orientation — low sequencing throughput or distal breakpoints still decrease on-target coverage
- Guide design is tied to the small panel of recurrent driver loci (ten genes demonstrated); expansion to arbitrary genes is possible but not yet demonstrated

## Evidence

- [abstract] FUDGE couples target-selected and strand-specific CRISPR-Cas9 activity for fusion gene driver enrichment: "FUDGE couples target-selected and strand-specific CRISPR-Cas9 activity for fusion gene driver enrichment"
- [discussion] Custom Alt-R crRNAs designed with IDT tool, maximum on-target and lowest off-target scores: "Custom Alt-R® crRNAs were designed with the Integrated DNA Technologies (IDT) custom gRNA design tool and chosen with maximum on-target and lowest off-target scores"
- [discussion] FUDGE enables rapid partner-location and breakpoint-location independent fusion gene detection: "We here developed FUDGE, a fusion detection assay from gene enrichment coupled to nanopore sequencing, which enables rapid partner-location and breakpoint-location independent fusion gene detection"
- [discussion] Limitation: requirement of non-fragmented DNA; FFPE material likely fails: "A limitation of this approach is the requirement of non-fragmented DNA. Applying the FUDGE crRNA protocol to FFPE material"
- [discussion] Reads mapped to GRCh37 with minimap2 as reference context for guide design and analysis: "Reads were mapped to the human reference genome version GRCHh37 by using minimap2 (v. 2.6)"
