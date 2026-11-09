---
name: task-dependency-topological-scheduling
description: Use when a workflow is defined as nodes (hardware tasks like microscope
  acquisition, software tasks like image analysis) with dependency edges, and you
  must produce an execution schedule plus resumable persisted state after interruption.
when_to_use_negative:
- The workflow is a single-pass linear pipeline with no hardware/software interleaving,
  e.g. commercial automated microscopes that 'imaging is done in a single pass'
- No interruption/recovery semantics are needed and a plain sequential script suffices
- The tasks have no explicit dependency edges, so topological ordering is undefined
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: OpenHiCAMM
  role: Provides the task algorithm resolver that topologically orders hardware and
    software tasks and the SQL database back end for workflow recovery
  repo: github.com/bdgp/OpenHiCAMM
- name: Maven
  role: Build system used to install OpenHiCAMM ('mvn install') and to add the micromanager
    MMJ_.jar and MMCore.jar files to the local Maven repo
- name: μManager
  role: Supplies the microscope hardware-control jars (MMJ_.jar, MMCore.jar) whose
    task classes the scheduler orchestrates
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1016/j.isci.2018.03.017
    title: 'OpenHiCAMM: High-Content Screening Software for Complex Microscope Imaging
      Workflows'
  source_content_hash: sha256:2897c6e6e101124b6ce36a682d7a645e00a2a1a2d88e972c3441f406c4d51f51
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Maven
  - OpenHiCAMM
  - μManager
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7857142857142857
n_claims: 14
---

# task-dependency-topological-scheduling

## Summary

Resolves a task-dependency graph from OpenHiCAMM's workflow engine into a topologically ordered schedule where microscope-dependent hardware tasks (e.g. slide loader, acquisition) run sequentially and computational image-analysis modules run in parallel, with SQL-backed persistence so interrupted workflows resume from persisted task states.

## When to use

Use when a workflow is defined as nodes (hardware tasks like microscope acquisition, software tasks like image analysis) with dependency edges, and you must produce an execution schedule plus resumable persisted state after interruption.

## When NOT to use

- The workflow is a single-pass linear pipeline with no hardware/software interleaving, e.g. commercial automated microscopes that 'imaging is done in a single pass'
- No interruption/recovery semantics are needed and a plain sequential script suffices
- The tasks have no explicit dependency edges, so topological ordering is undefined

## Inputs

- Task-dependency fixture graph (nodes = hardware and software tasks, edges = dependencies) from the Transparent Methods supplemental file
- OpenHiCAMM source tree from github.com/bdgp/OpenHiCAMM with micromanager MMJ_.jar and MMCore.jar installed in the local Maven repo
- Fiji installation directory referenced via the fiji.dir property in build.properties

## Outputs

- dependency_schedule (resolved topologically ordered execution schedule of hardware and software tasks)
- task_state.db (post-reload task states recovered from the SQL database back end)

## How to apply

Load the supplied task-dependency fixture graph (nodes = hardware and software tasks, edges = dependencies) from the Transparent Methods supplemental file, then build OpenHiCAMM from github.com/bdgp/OpenHiCAMM with 'mvn install' after adding micromanager MMJ_.jar and MMCore.jar to the local Maven repo via add-mmj-jar.sh. Run OpenHiCAMM's task algorithm resolver on the fixture graph to topologically order hardware tasks (e.g. microscope acquisition on the Prior PL-200 slide loader / Zeiss Axioplan 2 platform) and dependent software tasks (e.g. image analysis modules) into a schedule that never executes a task before its predecessors complete. Validate dependency consistency by checking that every task's predecessors precede it in the schedule. To exercise persistence, simulate an interruption mid-workflow, terminate execution, reload the workflow from the integrated SQL database back end, and confirm it resumes from persisted task states without re-executing completed tasks.

## Related tools

- **OpenHiCAMM** (Provides the task algorithm resolver that topologically orders hardware and software tasks and the SQL database back end for workflow recovery) — github.com/bdgp/OpenHiCAMM
- **Maven** (Build system used to install OpenHiCAMM ('mvn install') and to add the micromanager MMJ_.jar and MMCore.jar files to the local Maven repo)
- **μManager** (Supplies the microscope hardware-control jars (MMJ_.jar, MMCore.jar) whose task classes the scheduler orchestrates)

## Examples

```
./scripts/add-mmj-jar.sh ~/src/micromanager && mvn install
```

## Evaluation signals

- Every task in dependency_schedule is preceded by all of its declared predecessors (no task executes before its predecessors complete)
- Hardware tasks (microscope-dependent modules) appear in sequential order; computational image-analysis modules may appear parallelizable
- After simulated interruption and SQL reload, no completed task is re-executed; task_state.db matches persisted states at termination
- Build succeeds only after MMJ_.jar and MMCore.jar have been added to the local Maven repo (mvn install completes without missing-dependency errors)

## Limitations

- Microscope-dependent modules are constrained to sequential execution, limiting parallelism for hardware tasks
- Recovery depends entirely on the SQL database back end; workflow state not persisted there cannot be resumed
- No changelog or version number is published for OpenHiCAMM, complicating reproducibility of resolver behavior
- Requires a pre-built local micromanager source directory before add-mmj-jar.sh can register the jars

## Evidence

- [full_text] SQL persistence of workflow state: [evidence span withheld — anchor status: not_found]
- [full_text] Sequential hardware, parallel computational execution: [evidence span withheld — anchor status: not_found]
- [readme] Maven jar installation step: "Add the micromanager MMJ_.jar and MMCore.jar files to the user's local maven repo."
- [intro] Build OpenHiCAMM command: [evidence span withheld — anchor status: not_found]
