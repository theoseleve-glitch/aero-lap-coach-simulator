# Aero Lap Coach Simulator — architecture review
Status: **corrected review draft; no new solver implemented or source imported**.

Aero Lap Coach is intended to become a standalone scientific engineering platform for aerodynamic and vehicle-dynamics simulation, progressing toward minimum-lap-time analysis and optimization. Its sole existing project foundation is Track Aero Platform v0.1.1 at C:\Users\Admin\Documents\simulation software.

This review uses only that project's implementation, contracts and frozen decisions. Future algorithms must be designed for this platform. No material from other project source trees is included.

## Actual baseline
The authorized source implements strict input contracts, unit normalization, canonical Study identity/provenance, moist-air density, constant-area aerodynamic forces, effective axle tyre capacity and quasi-static suspension compliance. It does not implement a coupled vehicle model, track conditioning, speed profiles, numerical motion integration or minimum-time optimization.

Observed audit execution:
- 298 tests passed in 9.18 s.
- Ruff lint passed; 30 files passed format verification.
- Strict mypy failed with three no-any-return diagnostics in tyres.py.
- Complete supported workflow exited 1 after 24.2427697 s.
- A separate synthetic scalar smoke calculation returned sensible atmosphere/force/compliance outputs documented in docs/NUMERICAL_VALIDATION.md.

These checks establish executable foundations, not a validated vehicle simulator or real-world predictive accuracy. No capability ceiling was changed.

## Read the review
- [Architecture](docs/ARCHITECTURE.md): module inventory, subsystem interfaces and proposed integration.
- [Physics models](docs/PHYSICS_MODELS.md): implemented equations, frozen future scope, later fidelity and unresolved decisions.
- [Numerical validation](docs/NUMERICAL_VALIDATION.md): actual test evidence and proposed error budgets/benchmarks.
- [Provenance](docs/PROVENANCE.md): unresolved reuse rights and extraction policy.
- [Roadmap](ROADMAP.md): dependent multi-session engineering milestones.
- [Evaluation facts](EVALUATION_FACTS.md): factual notes, not submission answers.

## Repository and execution status
The requested permanent destination is C:\Users\Admin\Documents\aero-lap-coach-simulator. It was absent, and creation was denied by the current session's filesystem permissions. These seven documents are staged for review outside the original; they are not a substitute repository. No Git initialization or documentation commit at the requested destination has occurred.

There is no install/run command for this draft. No existing source is approved for redistribution yet, and no source is included. The original project's supported validation command, executed from its root, is:
```powershell
powershell -ExecutionPolicy Bypass -File scripts/check.ps1
```
Preservation settings used for this audit are recorded in the validation document. Do not invoke it casually if source/environment preservation cannot be guaranteed.

Current declared runtime dependencies in the authorized project are Pydantic, Pint, PyYAML and rfc8785, with Python >=3.12 and a documented 3.12 stack. No numerical-array or optimizer dependency is presently required. Future dependencies must be justified per milestone and installed offline on CPU-only Linux before claiming portability.

## Next approval
Approve the physics/interface decisions and quantitative budgets before algorithm implementation. Resolve file-specific provenance before any existing source/test/fixture enters the permanent repository. If clearance is unavailable, plan newly authored components from cleared scientific specifications. No licence, global optimality, real-car accuracy, publication or personal historical authorship is asserted.
