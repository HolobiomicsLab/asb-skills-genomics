---
name: hardware-software-task-dependency-resolution
description: Use when a workflow interleaves hardware acquisition steps (e.g. microscope
  imaging, slide-loader moves) with dependent software steps (e.g. image analysis
  modules) and you need a dependency-safe schedule plus crash recovery from persisted
  task states.
when_to_use_negative:
- The pipeline is purely computational with no robotic/microscope hardware tasks,
  so serial execution of microscope-dependent modules adds no value.
- Imaging is a single pass with no analysis-contingent re-imaging step, so there are
  no hardware-software dependencies to resolve or entry points to resume between.
- You need a versioned, benchmarked release of the scheduler — OpenHiCAMM exposes
  no changelog or version number and only qualitative robustness claims are reported.
edam_topics:
- http://edamontology.org/topic_3384
tools:
- name: OpenHiCAMM
  role: Executes the task algorithm resolver on the dependency graph and stores workflow
    state in its integrated SQL database engine
  repo: github.com/bdgp/OpenHiCAMM
- name: Maven
  role: Builds and installs OpenHiCAMM (`mvn install`) so the task resolver can be
    exercised locally
- name: μManager
  role: Hardware-control layer whose tasks (microscope acquisition via slide loader,
    motorized stage) are the sequential dependency nodes in the schedule
  repo: http://micromanager.org
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1016/j.isci.2018.03.017
    title: 'OpenHiCAMM: High-Content Screening Software for Complex Microscope Imaging
      Workflows'
  source_content_hash: sha256:33da1780cc251a703dffc0194f6ef2d118217e7b86588d73c985d0a5a5e91cd6
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Maven
  - OpenHiCAMM
  - μManager
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3384
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 6
---

# hardware-software-task-dependency-resolution

## Summary

Resolves a directed task-dependency graph mixing robotic microscope hardware tasks and computational image-analysis tasks into a topologically ordered schedule, and persists workflow setup, module configurations, and completed task states in an integrated SQL database so interrupted workflows can be resumed without re-executing finished tasks. Derived from the OpenHiCAMM workflow manager's scheduling and persistence algorithm.

## When to use

Use when a workflow interleaves hardware acquisition steps (e.g. microscope imaging, slide-loader moves) with dependent software steps (e.g. image analysis modules) and you need a dependency-safe schedule plus crash recovery from persisted task states.

## When NOT to use

- The pipeline is purely computational with no robotic/microscope hardware tasks, so serial execution of microscope-dependent modules adds no value.
- Imaging is a single pass with no analysis-contingent re-imaging step, so there are no hardware-software dependencies to resolve or entry points to resume between.
- You need a versioned, benchmarked release of the scheduler — OpenHiCAMM exposes no changelog or version number and only qualitative robustness claims are reported.

## Inputs

- task-dependency graph (nodes = hardware and software tasks, edges = dependencies), as described in the Transparent Methods supplemental file
- OpenHiCAMM source tree from github.com/bdgp/OpenHiCAMM
- pre-built micromanager source directory containing MMJ_.jar and MMCore.jar
- SQL database back end holding persisted workflow setup, module configurations, and completed tasks

## Outputs

- dependency_schedule: topologically ordered schedule in which no task precedes its predecessors
- task_state.db: post-reload task state showing completed tasks skipped on resumption
- resumed workflow execution continuing at the second entry point (e.g. 20x Phase 2 re-imaging)

## How to apply

Load the task-dependency fixture graph (nodes = hardware and software tasks, edges = dependencies) and run OpenHiCAMM's task algorithm resolver, which topologically orders microscope-dependent modules to run sequentially and computational modules to run in parallel, never scheduling a task before its predecessors complete. Build OpenHiCAMM from github.com/bdgp/OpenHiCAMM with Maven (`mvn install`) after adding micromanager's MMJ_.jar and MMCore.jar to the local Maven repo via add-mmj-jar.sh. Validate the resolved schedule by checking that every task's predecessors appear before it (writes dependency_schedule). To exercise persistence, terminate execution mid-workflow, reload the workflow from the SQL database back end, and confirm resumption from persisted task states without re-running completed tasks (writes task_state.db). The rationale is that hardware tasks are strictly serialized (one microscope, one stage) while independent analysis tasks can overlap, and the SQL back end is what enables the two-entry-point pattern (Phase 1 low-resolution survey, Phase 2 high-resolution re-imaging) demonstrated in the paper.

## Related tools

- **OpenHiCAMM** (Executes the task algorithm resolver on the dependency graph and stores workflow state in its integrated SQL database engine) — github.com/bdgp/OpenHiCAMM
- **Maven** (Builds and installs OpenHiCAMM (`mvn install`) so the task resolver can be exercised locally)
- **μManager** (Hardware-control layer whose tasks (microscope acquisition via slide loader, motorized stage) are the sequential dependency nodes in the schedule) — http://micromanager.org

## Examples

```
./scripts/add-mmj-jar.sh ~/src/micromanager && mvn install
```

## Evaluation signals

- Dependency consistency: every task in dependency_schedule appears after all of its predecessors in the graph.
- Interrupted-run recovery: after reload from the SQL database, no completed task is re-executed and the workflow resumes from persisted task states.
- Scheduling mode check: microscope-dependent modules (e.g. SlideImager acquisition) never overlap, while independent computational modules (e.g. image analysis) may run in parallel.
- Build gate: `mvn install` succeeds only after MMJ_.jar and MMCore.jar have been installed into the local Maven repo, confirming correct toolchain setup.

## Limitations

- Hardware tasks are serialized by design (single microscope/stage), so throughput is bounded by acquisition speed rather than the scheduler.
- No changelog or version number is available for OpenHiCAMM, so algorithm behavior may drift between checkouts.
- No quantitative benchmark of the scheduler is reported; robustness/flexibility claims are qualitative only.
- Optional patches for the micro-manager source tree exist in patches/ and may be required for stable hardware control (README notes May 2015 'vastly improved stability for hardware control').

## Evidence

- [abstract] provides a flexible framework for integration of generic microscope-associated robotics and image processing with sequential workflows: "provides a flexible framework for integration of generic microscope-associated robotics and image processing with sequential workflows"
- [results] Phase 2 resumption shows workflow recovery from persisted state: "After completion of the first pass, we manually adjusted the microscope, moving to a 20× objective, and resumed the workflow at the second entry point (Phase 2)."
- [readme] Build gate for exercising the task resolver: "Add the micromanager MMJ_.jar and MMCore.jar files to the user's local maven repo."
- [intro] Single-pass compromise motivates dependency-aware multi-pass scheduling: "Imaging is done in a single pass, requiring a compromise between resolution and field depth."
