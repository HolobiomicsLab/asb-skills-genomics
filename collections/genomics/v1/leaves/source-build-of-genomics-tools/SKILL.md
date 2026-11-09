---
name: source-build-of-genomics-tools
description: Use when a genomics tool (e.g. GraphMap) must be compiled locally from
  a git clone and a release binary is unavailable or a patch release is required.
when_to_use_negative:
- A prebuilt release binary or package manager install of the tool is available and
  no patch is needed
- The workflow depends on index files built by a prior GraphMap version that must
  not be overwritten (see index-version halt behavior)
tools:
- name: GraphMap
  role: tool built from source via make modules / make; resulting binary is the aligner
    used downstream
  repo: https://github.com/isovic/graphmap
- name: marginAlign
  role: companion tool built from a separate git clone with `git submodule update
    --init` then `make`, producing marginAlign, marginCaller and marginStats executables
  repo: https://github.com/benedictpaten/marginAlign.git
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/ncomms11307
    title: Fast and sensitive mapping of nanopore sequencing reads with GraphMap
  source_content_hash: sha256:95e496581ea9117fa1bd759516b1664e45e44fd2d9b63800c60c98a1c3040784
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/isovic/graphmap
  tools:
  - BLASR
  - BWA-MEM
  - DALIGNER
  - GCC/G++
  - GraphMap
  - LAST
  - marginAlign
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 14
---

# source-build-of-genomics-tools

## Summary

Build genomics tools such as GraphMap directly from source using the project's Makefile, initializing git submodules first so the compiled binaries include all bundled dependencies.

## When to use

Use when a genomics tool (e.g. GraphMap) must be compiled locally from a git clone and a release binary is unavailable or a patch release is required.

## When NOT to use

- A prebuilt release binary or package manager install of the tool is available and no patch is needed
- The workflow depends on index files built by a prior GraphMap version that must not be overwritten (see index-version halt behavior)

## Inputs

- git clone of a genomics tool repository (e.g. isovic/graphmap) with submodules
- C/C++ build toolchain: GCC/G++ >= 4.7, make

## Outputs

- Compiled tool binary, e.g. ./bin/Linux-x64/graphmap, with submodules initialized

## How to apply

Clone the repository, then run `make modules` to initialize submodules before compiling — the GraphMap README warns that after `git pull` you must `make modules` before `make`. Compile the Linux release with `make` using GCC/G++ >= 4.7, overriding via `GCC=/usr/local/bin/g++ make` if the system compiler is too old. Additional targets exist for cleaning (`make clean`), rebuilding (`make rebuild`), Mac builds (`make mac`, `make rebuildmac`) and debug binaries (`make debug`). The resulting binary lands in `./bin/Linux-x64/graphmap`; verify the version matches the intended release (e.g. 0.5.2) before use, noting that v0.5.1 fixed huge index-construction memory consumption and v0.5.2 fixed segfaults and reference-bounds mapping bugs.

## Related tools

- **GraphMap** (tool built from source via make modules / make; resulting binary is the aligner used downstream) — https://github.com/isovic/graphmap
- **marginAlign** (companion tool built from a separate git clone with `git submodule update --init` then `make`, producing marginAlign, marginCaller and marginStats executables) — https://github.com/benedictpaten/marginAlign.git

## Examples

```
git clone https://github.com/isovic/graphmap.git; cd graphmap; make modules; make
```

## Evaluation signals

- Compilation completes without errors with GCC/G++ >= 4.7; override compiler via GCC=/usr/local/bin/g++ make when needed
- Binary appears at ./bin/Linux-x64/graphmap and `git submodule` state is clean after `make modules`
- Running the binary reports the expected version (e.g. 0.5.2) whose changelog matches the intended build
- A short test alignment (graphmap align on a small FASTA/FASTQ pair) produces a valid SAM file without segfaults

## Limitations

- GraphMap's hash-table index is large (~14 x reference size; ~42 GB peak for the human genome) — building and mapping require large-memory machines
- Forgetting `make modules` after `git pull` breaks the build; submodules must be re-initialized first
- v0.5.0-built index files on disk must be treated carefully with v0.5.1+: incompatible index versions halt by default unless --auto-rebuild-index or --rebuild-index is specified

## Evidence

- [intro] build order for submodules: "After running `git pull`, please `make modules` before `make`."
- [methods] compiler requirement and override: [evidence span withheld — anchor status: not_found]
- [readme] binary location: [evidence span withheld — anchor status: external_doc]
- [readme] index memory footprint: [evidence span withheld — anchor status: external_doc]
- [readme] index build memory fix in 0.5.1: "Patch version resolving huge memory consumption when building the index."
