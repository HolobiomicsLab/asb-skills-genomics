---
name: vcf-variant-record-parsing
description: Use when a NanoFG (or NanoSV/Sniffles) breakpoint VCF must be read programmatically
  to extract fusion-gene breakpoint coordinates and flanking sequence for amplicon/primer
  design.
when_to_use_negative:
- The VCF records are SNVs/small indels or insertions — NanoFG explicitly removes
  insertions and targets only intergene SV ends flagged as possible fusions
- Input reads come from fragmented FFPE DNA, which the authors state will most likely
  fail the upstream FUDGE/NanoFG workflow that produces these VCFs
- A fusion call is supported by only one fusion-spanning read and a different validation
  strategy is required — such calls sit below NanoFG's two-read reliability cut-off
edam_topics:
- http://edamontology.org/topic_0199
tools:
- name: NanoFG
  role: Produces the fusion breakpoint VCF whose records are parsed; integrates minimap2
    mapping and NanoSV/Sniffles SV calling
  repo: https://github.com/SdeBlank/NanoFG
- name: NanoSV
  role: Optional SV caller (v1.2.4) generating the VCF records with default parameters
    'min_mapq=12, depth_support=False, mapq_flag=48'
  repo: https://github.com/mroosmalen/nanosv
- name: Sniffles
  role: Alternative SV caller (v1.0.9) generating VCF records with '-s 2 -n -1 --genotype'
- name: Primer3
  role: Designs forward/reverse primers on the breakpoint-flanking sequence extracted
    after VCF parsing, producing junction-spanning amplicons
  repo: https://github.com/primer3-org/primer3
- name: minimap2
  role: Maps ONT long reads to GRCh37 upstream of SV calling (v2.6), providing the
    alignments from which VCF breakpoint support is derived
  repo: https://github.com/lh3/minimap2
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-020-16641-7
    title: Partner independent fusion gene detection by multiplexed CRISPR-Cas9 enrichment
      and long read nanopore sequencing
  source_content_hash: sha256:b06c02febac75dd2aeb68001ee55a480d9ad0c0ad5fa7e1ee190e3742d228a21
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
  - minimap2
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0199
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 12
---

# vcf-variant-record-parsing

## Summary

Parses structural-variant VCF records (here, fusion breakpoint calls produced by the NanoFG pipeline via NanoSV/Sniffles) to obtain breakpoint coordinates and sequences, enabling downstream PCR-primer design with Primer3 for fusion validation.

## When to use

Use when a NanoFG (or NanoSV/Sniffles) breakpoint VCF must be read programmatically to extract fusion-gene breakpoint coordinates and flanking sequence for amplicon/primer design.

## When NOT to use

- The VCF records are SNVs/small indels or insertions — NanoFG explicitly removes insertions and targets only intergene SV ends flagged as possible fusions
- Input reads come from fragmented FFPE DNA, which the authors state will most likely fail the upstream FUDGE/NanoFG workflow that produces these VCFs
- A fusion call is supported by only one fusion-spanning read and a different validation strategy is required — such calls sit below NanoFG's two-read reliability cut-off

## Inputs

- Structural-variant VCF file with fusion breakpoint records produced by NanoFG (via NanoSV 1.2.4 or Sniffles 1.0.9)
- GRCh37 human reference genome (FASTA) for extracting breakpoint-flanking sequence
- Optional BAM alignment (minimap2 2.6, map-ont) if re-running from alignments rather than the fastq entry point

## Outputs

- List of parsed fusion breakpoint records (gene partners, breakpoint coordinates, breakpoint sequence)
- Breakpoint-flanking reference sequence (200-400 bp amplicon window)
- primer_pairs.tsv containing primer pair IDs, forward/reverse sequences, coordinates, and amplicon size

## How to apply

Parse the VCF emitted by NanoFG's SV-calling step (NanoSV v1.2.4 with 'min_mapq=12, depth_support=False, mapq_flag=48' or Sniffles v1.0.9 with '-s 2 -n -1 --genotype'), applying NanoFG's default filtering: drop SVs failing built-in filters, drop insertions, and keep only candidates with both SV ends in different genes that can produce a continuous same-strand transcript, supported by at least two fusion-spanning reads. For each retained fusion record, read the breakpoint coordinates and breakpoint sequence, then extract 200-400 bp of flanking reference sequence from GRCh37 as the amplicon target window. Feed that flanking sequence to Primer3 (primer3_core) to design forward and reverse primers whose amplicon spans the fusion junction, and write primer IDs, sequences, coordinates and amplicon size to primer_pairs.tsv. Rationale: the article reports that fusions detected with only one spanning read require manual breakpoint-PCR validation, so VCF parsing is the bridge between NanoFG's variant calling and wet-lab validation.

## Related tools

- **NanoFG** (Produces the fusion breakpoint VCF whose records are parsed; integrates minimap2 mapping and NanoSV/Sniffles SV calling) — https://github.com/SdeBlank/NanoFG
- **NanoSV** (Optional SV caller (v1.2.4) generating the VCF records with default parameters 'min_mapq=12, depth_support=False, mapq_flag=48') — https://github.com/mroosmalen/nanosv
- **Sniffles** (Alternative SV caller (v1.0.9) generating VCF records with '-s 2 -n -1 --genotype')
- **Primer3** (Designs forward/reverse primers on the breakpoint-flanking sequence extracted after VCF parsing, producing junction-spanning amplicons) — https://github.com/primer3-org/primer3
- **minimap2** (Maps ONT long reads to GRCh37 upstream of SV calling (v2.6), providing the alignments from which VCF breakpoint support is derived) — https://github.com/lh3/minimap2

## Examples

```
git clone https://github.com/primer3-org/primer3.git primer3; cd primer3/src && make; ./primer3_core breakpoint_flank.seq > primer3.out
```

## Evaluation signals

- Parsed records retain only SVs that passed NanoSV/Sniffles built-in filters; insertions are absent from the parsed set
- Every parsed fusion has both SV ends in different genes, same-strand continuous-transcript configuration, and at least two fusion-spanning reads
- Extracted flanking windows are 200-400 bp and match the GRCh37 reference at the VCF breakpoint coordinates
- primer_pairs.tsv is present, well-formed, and each row carries primer IDs, sequences, coordinates and amplicon size
- Designed primer sequences reproduce published Table 2 validation primers (e.g., SS18 forward CTGCTTGCCTCAACTAGAAAC) for the corresponding fusion

## Limitations

- Fusions detected with only a single fusion-spanning read fall below the two-read reliability cut-off and require manual breakpoint PCR validation
- The upstream assay requires non-fragmented DNA; FFPE-derived material will most likely fail, so no usable VCF may be produced for parsing
- Decreased on-target coverage is observed for low-throughput sequencing runs and more distal breakpoint events, which can leave fewer supported records to parse

## Evidence

- [discussion] single-read fusion calls require manual PCR validation: "two fusions were only detected with one fusion-spanning read each, requiring the manual validation of the fusion gene by breakpoint PCR."
- [readme] NanoFG as integrated fusion-detection pipeline: "NanoFG integrates existing genomic tools and additonal code to provide an easy-to-use pipeline for the detection of fusion genes from Nanopore sequencing data"
- [methods] Primer3 repository referenced in methods: "Primer3 - https://github.com/primer3-org/primer3"
- [discussion] GRCh37 mapping with minimap2 upstream of SV calling: "Reads were mapped to the human reference genome version GRCHh37 by using minimap2 (v. 2.6)"
