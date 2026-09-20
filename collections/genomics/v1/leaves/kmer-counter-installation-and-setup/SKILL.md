---
name: kmer-counter-installation-and-setup
description: Use when compiling Fast-SG from source (git clone https://github.com/adigenova/fastsg)
  and the build requires the KMC k-mer counter binaries placed in $Fast-SG/KMC/bin/
  before 'make all' and 'make test' can succeed.
when_to_use_negative:
- Fast-SG will be run from a package manager or pre-built distribution that already
  bundles KMC, so no source-tree wiring is needed
- The pipeline does not use Fast-SG (e.g. only ScaffMatch or another scaffolder is
  being run), since KMC placement is specific to the Fast-SG build
- The target system lacks a compatible C++ compiler (g++ >= 5.3 / clang >= 4.2 for
  Fast-SG; C++17 for building KMC from source)
edam_operation: http://edamontology.org/operation_2990
edam_topics:
- http://edamontology.org/topic_0102
- http://edamontology.org/topic_0080
tools:
- name: KMC
  role: K-mer counter whose binaries must be installed inside the Fast-SG directory
    for Fast-SG to build and run
  repo: https://github.com/refresh-bio/KMC
- name: Fast-SG
  role: Alignment-free scaffolding graph construction tool being compiled and set
    up; consumed KMC binaries during installation
  repo: https://github.com/adigenova/fast-sg
- name: g++
  role: Linux compiler used to build Fast-SG from source (tested with version 5.3)
- name: clang
  role: Mac OSX compiler used to build Fast-SG from source (tested with version 4.2)
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/gigascience/giy048
    title: 'Fast-SG: an alignment-free algorithm for hybrid assembly'
  source_content_hash: sha256:b163534d3a1043040e4049aced847daa57e66d205982ee91ae218909b24c0faf
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/refresh-bio/KMC
  tools:
  - Fast-SG
  - KMC
  - PERL
  - ScaffMatch
  - clang
  - g++
  edam_operation: http://edamontology.org/operation_2990
  edam_topics:
  - http://edamontology.org/topic_0102
  - http://edamontology.org/topic_0080
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# kmer-counter-installation-and-setup

## Summary

Installs and wires the KMC3 k-mer counter into the Fast-SG source tree so that Fast-SG can build scaffolding graphs alignment-free. This setup step is a prerequisite for running FAST-SG.pl on short or long reads (e.g. the NA12878 hybrid assembly).

## When to use

Use when compiling Fast-SG from source (git clone https://github.com/adigenova/fastsg) and the build requires the KMC k-mer counter binaries placed in $Fast-SG/KMC/bin/ before 'make all' and 'make test' can succeed.

## When NOT to use

- Fast-SG will be run from a package manager or pre-built distribution that already bundles KMC, so no source-tree wiring is needed
- The pipeline does not use Fast-SG (e.g. only ScaffMatch or another scaffolder is being run), since KMC placement is specific to the Fast-SG build
- The target system lacks a compatible C++ compiler (g++ >= 5.3 / clang >= 4.2 for Fast-SG; C++17 for building KMC from source)

## Inputs

- Fast-SG source tree obtained via 'git clone https://github.com/adigenova/fastsg'
- KMC3 pre-compiled binary tarball (KMC3.linux.tar.gz or KMC3.mac.tar.gz) or KMC source to compile

## Outputs

- Compiled Fast-SG binaries with KMC binaries located in $Fast-SG/KMC/bin/
- Successful 'make test' result confirming the Fast-SG build works end-to-end

## How to apply

Clone the Fast-SG repository, then obtain KMC3 either by downloading pre-compiled binaries (wget https://github.com/refresh-bio/KMC/releases/download/v3.0.0/KMC3.linux.tar.gz for Linux, or the KMC3.mac.tar.gz equivalent for Mac) or by compiling KMC yourself following the refresh-bio/KMC instructions. Place the KMC binaries inside the Fast-SG directory (the $Fast-SG/KMC/bin/ location expected by the build). Compile Fast-SG with 'make all' and confirm the installation with the small test 'make test'. Compilation of Fast-SG was tested with g++ 5.3 on Linux and clang 4.2 on Mac OSX; if compiling KMC from source instead, note that it needs a C++17-compatible compiler and, on Mac OS, an increased open-file limit ('ulimit -n 2048') because KMC creates hundreds of temporary files.

## Related tools

- **KMC** (K-mer counter whose binaries must be installed inside the Fast-SG directory for Fast-SG to build and run) — https://github.com/refresh-bio/KMC
- **Fast-SG** (Alignment-free scaffolding graph construction tool being compiled and set up; consumed KMC binaries during installation) — https://github.com/adigenova/fast-sg
- **g++** (Linux compiler used to build Fast-SG from source (tested with version 5.3))
- **clang** (Mac OSX compiler used to build Fast-SG from source (tested with version 4.2))

## Examples

```
git clone https://github.com/adigenova/fastsg && wget https://github.com/refresh-bio/KMC/releases/download/v3.0.0/KMC3.linux.tar.gz && make all && make test
```

## Evaluation signals

- 'make all' completes without errors after KMC binaries are placed in $Fast-SG/KMC/bin/
- 'make test' runs the small bundled test successfully, confirming the Fast-SG + KMC installation is functional
- KMC binaries are discoverable by the Fast-SG build at the expected path ($Fast-SG/KMC/bin/)
- The correct OS-specific KMC3 tarball was used (KMC3.linux.tar.gz on Linux, KMC3.mac.tar.gz on Mac)

## Limitations

- Compilation of Fast-SG was only tested with g++ 5.3 (Linux) and clang 4.2 (Mac OSX); other compiler versions are not guaranteed
- KMC creates hundreds of temporary files and on Mac OS the default open-file limit may be too small, requiring 'ulimit -n 2048' before running KMC
- KMC uses some C++17 features, so building KMC from source requires a C++17-compatible compiler

## Evidence

- [methods] Obtain pre-compiled binaries: "wget https://github.com/refresh-bio/KMC/releases/download/v3.0.0/KMC3.linux.tar.gz"
- [methods] Place KMC binaries inside Fast-SG directory: "After getting KMC3, put the binaries inside Fast-SG directory"
- [methods] Compile Fast-SG with make all: "## Compile Fast-SG
	make all"
- [methods] Test Fast-SG with make test: "## Test Fast-SG (small test)
	make test"
- [discussion] Compilation compilers tested: "Compilation was tested with g++ version 5.3 (Linux) and clang version 4.2 (Mac OSX)"
- [readme] KMC temp-file ulimit on Mac: "Note that KMC creates a hundreds of temporary files, while default limit for opened files is small for under MAC OS platform.
To increase this number use following command before running KMC:"
- [readme] KMC C++17 requirement: [evidence span withheld — anchor status: external_doc]
