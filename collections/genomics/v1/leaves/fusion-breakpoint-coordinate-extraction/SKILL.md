---
name: fusion-breakpoint-coordinate-extraction
description: Use when a NanoFG fusion-detection run has produced a breakpoint VCF
  (NanoSV/Sniffles output) and you need exact breakpoint coordinates and flanking
  reference sequence to design fusion-junction-spanning validation primers.
when_to_use_negative:
- Fusions detected with only one fusion-spanning read — the paper requires manual
  breakpoint PCR validation for these, and the ≥2-read cutoff means they are not reliably
  detected.
- Input SV calls include insertions or SVs failing NanoSV/Sniffles built-in filters
  — NanoFG removes these before fusion calling, so they should not be treated as fusable
  breakpoints.
edam_operation: http://edamontology.org/operation_2403
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3175
tools:
- name: NanoFG
  role: Pipeline whose reporting step produces the fusion breakpoint VCF and per-fusion
    breakpoint sequences used for primer extraction
  repo: https://github.com/SdeBlank/NanoFG
- name: NanoSV
  role: Optional SV caller (v1.2.4, min_mapq=12, mapq_flag=48) whose filtered VCF
    supplies breakpoint coordinates
  repo: https://github.com/mroosmalen/nanosv
- name: Sniffles
  role: Alternative SV caller (v1.0.9, '-s 2 -n -1 --genotype') whose filtered VCF
    supplies breakpoint coordinates
- name: Primer3
  role: Designs forward/reverse primer pairs on breakpoint-flanking sequence so the
    amplicon spans the fusion junction
  repo: https://github.com/primer3-org/primer3
- name: FUDGE
  role: Upstream CRISPR-Cas9 enrichment + nanopore sequencing assay that generated
    the long reads input to NanoFG
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-020-16641-7
    title: Partner independent fusion gene detection by multiplexed CRISPR-Cas9 enrichment
      and long read nanopore sequencing
  source_content_hash: sha256:65e32c8cfac2813f1d981392a4f2ab9de5d6562f1a69c2ae99d7c8b0c15cb82b
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/SdeBlank/NanoFG
  tools:
  - FUDGE
  - NanoFG
  - NanoSV
  - Primer3
  - Sniffles
  edam_operation: http://edamontology.org/operation_2403
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3175
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# fusion-breakpoint-coordinate-extraction

## Summary

Parses the structural-variant VCF produced by the NanoFG pipeline (NanoSV or Sniffles calling) to extract exact genomic breakpoint coordinates and flanking sequence context for called fusion genes, enabling downstream junction-spanning PCR primer design.

## When to use

Use when a NanoFG fusion-detection run has produced a breakpoint VCF (NanoSV/Sniffles output) and you need exact breakpoint coordinates and flanking reference sequence to design fusion-junction-spanning validation primers.

## When NOT to use

- Fusions detected with only one fusion-spanning read — the paper requires manual breakpoint PCR validation for these, and the ≥2-read cutoff means they are not reliably detected.
- Input SV calls include insertions or SVs failing NanoSV/Sniffles built-in filters — NanoFG removes these before fusion calling, so they should not be treated as fusable breakpoints.

## Inputs

- NanoFG fusion breakpoint VCF (NanoSV v1.2.4 or Sniffles v1.0.9 output)
- GRCh37 human reference genome FASTA
- ENSEMBL gene annotation (used by NanoFG to assign SV ends to genes)

## Outputs

- primer_pairs.tsv with primer IDs, sequences, genomic coordinates, and amplicon size for each fusion junction
- Breakpoint coordinates and flanking reference sequences per called fusion gene

## How to apply

Parse the fusion breakpoint VCF emitted by NanoFG's reporting step, which flags an SV as a possible fusion only when both SV ends fall in different genes and retains only fusions able to produce a continuous same-strand transcript. For each called fusion, extract the breakpoint coordinates from the VCF POS/END fields, then pull the reference genomic sequence flanking each breakpoint (a 200–400 bp amplicon target window) from the GRCh37 human reference used for mapping. Pass this breakpoint-flanking sequence to Primer3 to design forward and reverse primer pairs whose amplicon spans the fusion junction, and write the designed pairs (IDs, sequences, coordinates, amplicon size) to primer_pairs.tsv. The rationale is that NanoFG reports, for each called fusion, the exact fusion gene configuration, breakpoint-location, breakpoint-sequence and fusion-spanning primer sequences intended for validation, so reproducing these from the VCF reproduces the published validation primers.

## Related tools

- **NanoFG** (Pipeline whose reporting step produces the fusion breakpoint VCF and per-fusion breakpoint sequences used for primer extraction) — https://github.com/SdeBlank/NanoFG
- **NanoSV** (Optional SV caller (v1.2.4, min_mapq=12, mapq_flag=48) whose filtered VCF supplies breakpoint coordinates) — https://github.com/mroosmalen/nanosv
- **Sniffles** (Alternative SV caller (v1.0.9, '-s 2 -n -1 --genotype') whose filtered VCF supplies breakpoint coordinates)
- **Primer3** (Designs forward/reverse primer pairs on breakpoint-flanking sequence so the amplicon spans the fusion junction) — https://github.com/primer3-org/primer3
- **FUDGE** (Upstream CRISPR-Cas9 enrichment + nanopore sequencing assay that generated the long reads input to NanoFG)

## Examples

```
bash NanoFG.sh -f /path/to/fastqdir -n sample1; primer3_core breakpoint_flanking.txt > primers.out
```

## Evaluation signals

- primer_pairs.tsv exists, is well-formed (columns: primer IDs, sequences, coordinates, amplicon size).
- Designed forward/reverse primer sequences reproduce the published Table 2 primers for the corresponding fusion (e.g., SS18 forward CTGCTTGCCTCAACTAGAAAC).
- Each extracted breakpoint satisfies NanoFG retention filters: both SV ends in different genes, same-strand continuous-transcript fusion, ≥2 fusion-spanning reads.
- Insertions and SVs failing NanoSV/Sniffles built-in filters are absent from the extracted breakpoint set.

## Limitations

- Two fusions detected with only one supporting read required manual breakpoint PCR validation — automated primer extraction is unreliable at that coverage.
- Decreased on-target coverage for low-throughput sequencing runs and more distal breakpoint events reduces confidence in breakpoint coordinates.
- Breakpoint coordinates are relative to GRCh37 (the reference used by minimap2 v2.6 in the FUDGE/NanoFG workflow); using a different assembly build will shift coordinates.

## Evidence

- [other] NanoFG's reporting step provides, for each called fusion gene, the exact fusion gene configuration, breakpoint-location, breakpoint-sequence and fusion-spanning primer sequences intended for validation purposes.: [evidence span withheld — anchor status: not_found]
- [discussion] If both ends of the SV are positioned in different genes it was flagged as a possible fusion.: "If both ends of the SV are positioned in different genes it was flagged as a possible fusion."
- [discussion] We set a cut-off of at least two fusion-spanning reads to reliably detect a fusion gene without further validation.: "We set a cut-off of at least two fusion-spanning reads to reliably detect a fusion gene without further validation."
- [methods] Primer3 repository reference from methods section.: "Primer3 - https://github.com/primer3-org/primer3"
- [discussion] Reads were mapped to the human reference genome version GRCh37.: "Reads were mapped to the human reference genome version GRCHh37 by using minimap2 (v. 2.6)"
