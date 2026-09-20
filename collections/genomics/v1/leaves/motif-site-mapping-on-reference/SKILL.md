---
name: motif-site-mapping-on-reference
description: 'Use when preparing SMALR SMsn or SMp runs against SMRT cmp.h5/BAM alignments:
  the methyltransferase motif is known, a reference FASTA for the strain is available,
  and per-molecule methylation scoring needs the complete forward- and reverse-strand
  motif site coordinates.'
when_to_use_negative:
- The methylation motif is unknown and must be discovered de novo — SMALR requires
  the motif and modified-base position to be supplied via --motif and --mod_pos and
  does not perform de novo motif discovery from the reference.
- The analysis target is not SMRT sequencing kinetics (e.g. no native-vs-WGA IPD data)
  — mapping motif sites alone provides no methylation information without the IPD
  comparison.
- The target modifications create kinetic signatures extending into surrounding bases
  that require more sophisticated algorithms beyond SMALR's per-motif-position scoring
  scope.
edam_operation: http://edamontology.org/operation_0239
edam_topics:
- http://edamontology.org/topic_0160
- http://edamontology.org/topic_3295
tools:
- name: SMALR
  role: Scans the reference FASTA to identify all forward- and reverse-strand positions
    of the user-specified motif (--motif, --mod_pos) as the first stage of the SMsn
    and SMp protocols
  repo: https://github.com/fanglab/smalr
- name: Python 2.7
  role: Runtime dependency for executing the SMALR motif-scanning and scoring pipeline
- name: samtools
  role: Supports BAM-format native/WGA alignments consumed alongside the reference
    FASTA during motif mapping
provenance:
  source_task_ids:
  - task_001
  - task_004
  - task_007
  source_papers:
  - doi: 10.1038/ncomms8438
    title: Single molecule-level detection and long read-based phasing of epigenetic
      variations in bacterial methylomes
  source_content_hash: sha256:c1fc02ef55a6883a88023eb7b6774662c23863df3e21a5349a485f87956573c7
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/fanglab/smalr
  tools:
  - Python
  - Python 2.7
  - R
  - SMALR
  - samtools
  edam_operation: http://edamontology.org/operation_0239
  edam_topics:
  - http://edamontology.org/topic_0160
  - http://edamontology.org/topic_3295
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4
n_claims: 5
---

# motif-site-mapping-on-reference

## Summary

Exhaustively scans a bacterial reference FASTA to locate all occurrences of a target methylation motif (e.g. 5'-GATC, 5'-RGATCY, 5'-GWCAY) on both forward and reverse strands of each contig, assigning each site a strand-relative modified-base position (--mod_pos) prior to SMALR SMsn or SMp kinetic scoring.

## When to use

Use when preparing SMALR SMsn or SMp runs against SMRT cmp.h5/BAM alignments: the methyltransferase motif is known, a reference FASTA for the strain is available, and per-molecule methylation scoring needs the complete forward- and reverse-strand motif site coordinates.

## When NOT to use

- The methylation motif is unknown and must be discovered de novo — SMALR requires the motif and modified-base position to be supplied via --motif and --mod_pos and does not perform de novo motif discovery from the reference.
- The analysis target is not SMRT sequencing kinetics (e.g. no native-vs-WGA IPD data) — mapping motif sites alone provides no methylation information without the IPD comparison.
- The target modifications create kinetic signatures extending into surrounding bases that require more sophisticated algorithms beyond SMALR's per-motif-position scoring scope.

## Inputs

- Reference FASTA for the target bacterial genome (e.g. H. pylori J99, C. salexigens)
- Known methylation motif sequence, possibly degenerate (GATC, RGATCY, GWCAY), and the modified-base offset (--mod_pos)
- SMALR inputs_file with native_cmph5, wga_cmph5, and ref path lines
- Native and WGA aligned cmp.h5 (or BAM via samtools) SMRT read alignments

## Outputs

- Per-contig output folder (e.g. J99_SMsn or per-contig J99-style directories) containing a log detailing the analysis of that contig and the motif positions in that contig (forward and reverse strand)
- Motif site coordinates consumed by SMsn.out / SMp.out result tables (columns: contig strand, motif position, score, molecule ID, native score, WGA score, native/WGA data-point counts, mean subread length)

## How to apply

Build the SMALR inputs_file with the native aligned cmp.h5 (or BAM), the WGA control cmp.h5 (or BAM), and the reference FASTA path (specify NONE for the deprecated fastq line). Invoke SMALR with the required --motif (e.g. --motif=GATC for the GATC family, including degenerate motifs like RGATCY or GWCAY) and --mod_pos (e.g. 2 to mark the adenine within GATC), letting the program scan every contig of the reference for forward- and reverse-strand occurrences. Subsequent filtering (--nativeCovThresh=10, --wgaCovThresh=10, --minSubreadLen=100, --minAcc=0.8) then restricts kinetics comparison to motif sites with sufficient native and WGA coverage. The rationale is that subread-normalized ln(IPD) native-vs-WGA differences can only be attributed to methylation at positions the scanner enumerates exhaustively per strand, so any missed or mis-stranded motif coordinate would silently drop methylated sites from the SMsn/SMp score computation.

## Related tools

- **SMALR** (Scans the reference FASTA to identify all forward- and reverse-strand positions of the user-specified motif (--motif, --mod_pos) as the first stage of the SMsn and SMp protocols) — https://github.com/fanglab/smalr
- **Python 2.7** (Runtime dependency for executing the SMALR motif-scanning and scoring pipeline)
- **samtools** (Supports BAM-format native/WGA alignments consumed alongside the reference FASTA during motif mapping)

## Examples

```
smalr -i --SMsn --motif=GATC --mod_pos=2 --procs=4 -c 5 input.txt
```

## Evaluation signals

- The reported motif positions cover every expected occurrence of the motif (e.g. all GATC-family sites) on both forward and reverse strands of each contig in the reference FASTA.
- Each contig-specific output folder contains a log detailing the per-contig analysis and the forward- and reverse-strand motif positions.
- Downstream SMsn.out/SMp.out rows carry well-formed contig-strand and motif-position values traceable to reference coordinates.
- Running the installation test (e.g. ./run_test_SMsn.sh) regenerates the contig-specific output folder (e.g. J99_SMsn including SMsn.out), confirming the motif-scanning stage runs end-to-end.
- Altering --motif or --mod_pos changes the scored site set correspondingly (e.g. GATC with mod_pos=2 marks the adenine), confirming the strand-relative offset is honored.

## Limitations

- Detection quality is tied to the supplied motif: only sites matching --motif at the --mod_pos offset are enumerated; modifications with extended kinetic signatures are beyond the scope of the current approach.
- Positions with insufficient aggregate WGA coverage (--wgaCovThresh, default 10) or per-molecule/strand native coverage (--nativeCovThresh, default 10) are skipped, so mapped motif sites may be absent from score outputs at low sequencing depth.
- SMsn reliability requires multiple passes per motif site (short ~250bp libraries), so motif sites observed on molecules with low single-molecule coverage yield less reliable scores.

## Evidence

- [methods] These results include a log detailing the analysis of that contig, the motif positions in that contig (forward and reverse strand): "These results include a log detailing the analysis of that contig, the motif positions in that contig (forward and reverse strand)"
- [intro] SMALR conducts single-molecule level interrogation: "The SMALR package conducts single-molecule level interrogation of the methylation status of SMRT reads."
- [readme] SMALR: a framework for single-molecule level interrogation: "SMALR: a framework for single-molecule level interrogation of the methylation status of SMRT reads."
- [intro] SMp motif-pooled analysis description: "All motif sites on a sequencing molecule are pooled together and the molecule-wide methylation status for the given motif is assessed."
