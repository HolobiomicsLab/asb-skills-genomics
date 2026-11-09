---
name: telobait-capture-read-parsing
description: Use when input is long-read data (fasta/fastq/bam/pacbio-bam) from the
  telobait telomere-capture wet-lab workflow and per-read telomere lengths or telomeric-read
  tables are needed.
when_to_use_negative:
- Input is vanilla whole-genome sequencing long-read data with no telobait capture
  — use telomap's wgs mode instead, since telobait mode 'requires data generated from
  a wet lab telomere capturing method'.
- Measuring telomere length of ALT cells, whose very long telomeres exceed PacBio
  HiFi DNA polymerase processivity.
- Barcode or capture-oligo sequences are unavailable — the required -c/-b FASTA files
  cannot be constructed.
edam_operation: http://edamontology.org/operation_3198
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3512
tools:
- name: telomap
  role: 'Primary tool: identifies telomeric reads in telobait-captured long-read data
    by capture-oligo/barcode alignment and telomeric-repeat detection, emitting per-read
    telomere length tables'
  repo: https://github.com/cytham/telomap
- name: PacBio high fidelity (HiFi) sequencing platform
  role: Sequencing platform producing the HiFi long reads on which the telobait-capture
    read parsing operates
provenance:
  source_task_ids:
  - task_001
  - task_005
  source_papers:
  - doi: 10.1038/s41467-023-35823-7
    title: High-throughput telomere length measurement at nucleotide resolution using
      the PacBio high fidelity sequencing platform
  source_content_hash: sha256:db173058382677e361f9f5ed6178622636dbd1fce258e22b8a7d5297d3763ae3
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/cytham/telomap
  tools:
  - PacBio high fidelity (HiFi) sequencing platform
  - telomap
  edam_operation: http://edamontology.org/operation_3198
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3512
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 8
---

# telobait-capture-read-parsing

## Summary

Identify and parse telomere-containing reads from telobait-capture PacBio HiFi long-read data using telomap, which aligns the capture oligo and sample barcode at read ends and demarcates the telomeric tract at nucleotide resolution.

## When to use

Use when input is long-read data (fasta/fastq/bam/pacbio-bam) from the telobait telomere-capture wet-lab workflow and per-read telomere lengths or telomeric-read tables are needed.

## When NOT to use

- Input is vanilla whole-genome sequencing long-read data with no telobait capture — use telomap's wgs mode instead, since telobait mode 'requires data generated from a wet lab telomere capturing method'.
- Measuring telomere length of ALT cells, whose very long telomeres exceed PacBio HiFi DNA polymerase processivity.
- Barcode or capture-oligo sequences are unavailable — the required -c/-b FASTA files cannot be constructed.

## Inputs

- Telobait-captured PacBio HiFi long reads (fasta/fastq/bam/pacbio-bam)
- capture_oligo.fa — capture oligo sequence (sequence after the barcode on the telobait, e.g. GATGCCAGATGCACGGAGCA)
- barcodes.fa — sample barcode sequences (barcode at start of telobait through end of barcode)

## Outputs

- sample.telomap.tsv — per-read table (RNAME, OLIGO, BARCODE, STRAND, MOTIF, RLEN, RAW_TELOMERE_LEN, TELOMERE_LEN, TELOMERE_END_MOTIF, TRF_COUNT, CHROM, TELOMERE_START, TELOMERE_END)
- sample.telomap.anchors.tsv — subtelomeric anchor sequences for chromosome-end mapping
- plots/ and plots_QC/ directories — analytic and QC plots including telomere length distributions

## How to apply

Run telomap in telobait mode with the capture oligo FASTA (-c) and barcode FASTA (-b), which should contain the sequence after the telobait barcode and the barcode-to-barcode-end sequence respectively. Telomap queries the capture oligo against the terminal 150 bases at each read end via Bio.Align pairwise alignment requiring --oligoscore 1 (perfect alignment score fraction), matches the barcode with --barscore 1 to demultiplex samples, then scans adjacent sequence for at least two consecutive canonical telomeric repeats (5'-TTAGGGTTAGGG-3') using the default motif TTAGGG (-m). It records TELOMERE_START (1-based 5' position containing the two-repeat anchor) and TELOMERE_END (1-based 3' position adjacent to the telobait), yielding RAW_TELOMERE_LEN = TELOMERE_END - TELOMERE_START + 1 and TELOMERE_LEN as motif count times 6 bp after excluding non-telomeric variant sequences. Use multi-threading (e.g. -t 24) for HiFi datasets from cell lines such as HCT116, 293T, T24, IMR90, and WI38.

## Related tools

- **telomap** (Primary tool: identifies telomeric reads in telobait-captured long-read data by capture-oligo/barcode alignment and telomeric-repeat detection, emitting per-read telomere length tables) — https://github.com/cytham/telomap
- **PacBio high fidelity (HiFi) sequencing platform** (Sequencing platform producing the HiFi long reads on which the telobait-capture read parsing operates)

## Examples

```
telomap telobait /path/to/reads.fastq.gz /path/to/work_dir -t 24 -c /path/to/capture_oligo.fa -b /path/to/barcodes.fa
```

## Evaluation signals

- Every retained read in sample.telomap.tsv has a perfect-score (oligoscore 1) capture-oligo match at a read end plus a detectable tract of at least two consecutive TTAGGG repeats.
- All 13 declared columns (RNAME through TELOMERE_END) are present and non-empty for telobait-mode rows (OLIGO/BARCODE populated rather than NA).
- RAW_TELOMERE_LEN equals TELOMERE_END - TELOMERE_START + 1; TELOMERE_LEN equals motif count × 6 bp.
- Distribution plots reproduce known biology, e.g. over-representation of extremely short (<1000 bp) telomeres in T24 cells relative to HCT116, 293T, IMR90 (PD47), and WI38 (PD40).
- Reducing --barscore below 1 should be avoided/checked, as it may lead to multiple barcode mappings per read and high read omission.

## Limitations

- Telobait mode requires data generated from the wet-lab telomere capturing method; it cannot be applied to standard WGS long-read data.
- Trimming telomere-containing genomic DNA with RsaI and HinfI can yield subtelomeric regions too short for unambiguous identification of specific chromosome ends against the T2T-CHM13 assembly.
- Restriction enzymes cutting closer to the telomere (HphI, MnlI) frequently cut within telomeric variant sequences, causing loss of telomere repeat regions and underestimation of telomere length.
- The method is incompatible with measuring telomere length of ALT cells because their very long telomeres exceed PacBio HiFi DNA polymerase processivity.
- Reducing --barscore below 1 may cause multiple barcode mapping per read, leading to high read omission.

## Evidence

- [readme] Telobait mode requires data generated from a wet lab telomere capturing method: "1. __Telobait mode__: requires data generated from a wet lab telomere capturing method described in our [publication]"
- [readme] oligoscore default: "--oligoscore float    minimum alignment score fraction required
                        for capture oligo sequence match [1]"
- [methods] TELOMERE_START definition: "the 1-based position on a read at the 5′ end of the telomeric region as defined by containing at least two consecutive telomeric repeats (5′-TTAGGGTTAGGG-3′)"
- [methods] Telomap purpose: "Telomap is a tool to identify and analyse telomeric reads within WGS or telobait-captured long-read sequencing data."
- [methods] TELOMERE_LEN definition: "Length of telomeric region in bases after excluding all non-telomeric sequences within RAW_TELOMERE_LEN"
- [discussion] T24 extremely short telomeres finding: "our method has identified the over-representation of extremely short telomeres in T24 cells"
