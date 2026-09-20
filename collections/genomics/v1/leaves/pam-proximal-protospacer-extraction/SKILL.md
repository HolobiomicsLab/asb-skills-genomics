---
name: pam-proximal-protospacer-extraction
description: Use when designing strand-specific CRISPR-Cas9 crRNAs (e.g., FUDGE fusion-gene
  enrichment) against known driver exon loci such as BRAF Exon 10, EWSR1 Exon 7, or
  SS18 Exon 9, where guides must orient Cas9 cutting toward an unknown fusion partner.
when_to_use_negative:
- The input DNA is fragmented FFPE material — the FUDGE crRNA protocol applied to
  degraded DNA will most likely fail to comprehensively identify fusion genes
- The fusion partner's identity and breakpoint are already known and the goal is re-sequencing
  a known junction rather than partner-independent discovery
- Only one candidate PAM exists per target and no guide on the strand that orients
  sequencing toward the unknown partner can be found — no valid strand-directed guide
  can be produced
edam_operation: http://edamontology.org/operation_0431
edam_topics:
- http://edamontology.org/topic_0622
tools:
- name: FUDGE
  role: Assay whose strand-directed crRNA design rule this extraction implements;
    guides orient Cas9 cutting toward the unknown fusion partner
- name: IDT custom gRNA design tool
  role: Used to rank candidate protospacers by on-target and off-target scores before
    selecting the final crRNA per locus
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-020-16641-7
    title: Partner independent fusion gene detection by multiplexed CRISPR-Cas9 enrichment
      and long read nanopore sequencing
  source_content_hash: sha256:0ee58b27aacd31805ba172d9b416f17fd05e6835149cbba97f44a0deafbbb123
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - FUDGE
  - IDT custom gRNA design tool
  edam_operation: http://edamontology.org/operation_0431
  edam_topics:
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# pam-proximal-protospacer-extraction

## Summary

Scans target-locus reference sequences for NGG PAM motifs and extracts the 20-nt protospacer immediately 5' of each PAM to generate candidate FUDGE crRNAs for strand-directed fusion-gene enrichment prior to nanopore sequencing.

## When to use

Use when designing strand-specific CRISPR-Cas9 crRNAs (e.g., FUDGE fusion-gene enrichment) against known driver exon loci such as BRAF Exon 10, EWSR1 Exon 7, or SS18 Exon 9, where guides must orient Cas9 cutting toward an unknown fusion partner.

## When NOT to use

- The input DNA is fragmented FFPE material — the FUDGE crRNA protocol applied to degraded DNA will most likely fail to comprehensively identify fusion genes
- The fusion partner's identity and breakpoint are already known and the goal is re-sequencing a known junction rather than partner-independent discovery
- Only one candidate PAM exists per target and no guide on the strand that orients sequencing toward the unknown partner can be found — no valid strand-directed guide can be produced

## Inputs

- Target list of known fusion-gene driver exon loci (e.g., BRAF Exon 10, EWSR1 Exon 7, SS18 Exon 9) with 5'/3' partner designations
- Exon reference sequences fetched from the GRCh37 human reference genome (both strands)

## Outputs

- crRNAs.fasta containing one selected 20-nt DNA guide sequence per target locus, with one header per target locus
- Ranked list of candidate guides with on-target/off-target quality scores

## How to apply

Fetch the exon reference sequence for each target locus from the GRCh37 human reference genome, retrieving both strands. Scan each fetched sequence for NGG PAM motifs and extract the 20-nt protospacer immediately 5' of every PAM candidate. Apply the FUDGE strand-specificity rule: retain only guides whose Cas9 cut orients sequencing toward the unknown 5' or 3' fusion partner, per the known gene's partner designation. Rank retained guides using the IDT custom gRNA design tool criterion — maximum on-target and lowest off-target scores — and write exactly one selected 20-nt guide per target locus to a FASTA file (e.g., crRNAs.fasta). These crRNAs are then cloned/ordered as Custom Alt-R crRNAs and complexed with Cas9 into RNPs for target-selected, strand-specific enrichment.

## Related tools

- **FUDGE** (Assay whose strand-directed crRNA design rule this extraction implements; guides orient Cas9 cutting toward the unknown fusion partner)
- **IDT custom gRNA design tool** (Used to rank candidate protospacers by on-target and off-target scores before selecting the final crRNA per locus)

## Evaluation signals

- Output parses as a valid FASTA file (crRNAs.fasta)
- Every FASTA record is exactly a 20-nt DNA guide sequence
- Exactly one selected guide is present per target locus (one header per target)
- Each retained guide's protospacer sits immediately 5' of an NGG PAM and lies on the strand whose Cas9 cut orients sequencing toward the designated 5' or 3' fusion partner
- Selected guides correspond to maximum on-target and lowest off-target scores from the IDT custom gRNA design tool

## Limitations

- Assay requires non-fragmented DNA; FFPE material will most likely fail to comprehensively identify fusion genes
- Decreased on-target coverage is observed for low-throughput sequencing runs and/or more distal breakpoint events
- Guide quality is constrained by PAM availability in the driver exon; only NGG-proximal 20-mers on the partner-directed strand are eligible

## Evidence

- [discussion] FUDGE achieves directionality by designing crRNAs in a strand-directed manner, so reads extend upstream or downstream of the crRNA sequence into the suspected 5' or 3' fusion partner: [evidence span withheld — anchor status: not_found]
- [discussion] Custom Alt-R crRNAs were designed with the IDT custom gRNA design tool and chosen with maximum on-target and lowest off-target scores: "Custom Alt-R® crRNAs were designed with the Integrated DNA Technologies (IDT) custom gRNA design tool and chosen with maximum on-target and lowest off-target scores"
- [abstract] FUDGE couples target-selected and strand-specific CRISPR-Cas9 activity for fusion gene driver enrichment: "FUDGE couples target-selected and strand-specific CRISPR-Cas9 activity for fusion gene driver enrichment"
- [discussion] A limitation of this approach is the requirement of non-fragmented DNA.: "A limitation of this approach is the requirement of non-fragmented DNA. Applying the FUDGE crRNA protocol to FFPE material"
- [discussion] Reads were mapped to the human reference genome version GRCHh37 by using minimap2 (v. 2.6): "Reads were mapped to the human reference genome version GRCHh37 by using minimap2 (v. 2.6)"
