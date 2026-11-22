# Aero Lap Coach Simulator

Aero Lap Coach is a scientific software project for studying how aerodynamics and vehicle dynamics influence circuit performance. Its vision is a reproducible engineering platform that connects explicit physical assumptions, numerical simulation and minimum-lap-time analysis, helping users understand the tradeoffs between drag, downforce, tyre capacity and vehicle setup.

**Current status: scientific design and development roadmap.** This repository contains documentation, not an executable simulator. It has no packaged source code, runnable examples or test suite, and no installation or simulation command is available yet.

## Scientific foundation and current capabilities

The technical documents describe a previously implemented foundation, Track Aero Platform v0.1.1. That implementation is not included in this repository. Its documented capabilities and the planned Aero Lap Coach extensions must be distinguished:

| Area | Documented foundation implementation | Planned development |
|---|---|---|
| Inputs and reproducibility | Strict input contracts, unit normalization, canonical Study identities and provenance | Conditioned track geometry, solver-input identities and reproducible run records |
| Atmosphere | Moist-air density from explicit environmental inputs | Integration into the vehicle model with declared validity domains |
| Aerodynamics | Constant-area drag and downforce with front/rear downforce allocation | Aero/setup studies; later, calibrated ride-height-dependent maps |
| Tyres | Load-dependent effective-axle peak forces and combined-force capacity | Coupled vehicle feasibility; later, calibrated per-wheel and slip-based models |
| Suspension | Quasi-static compliance for supplied axle loads | Integration with solved axle loads; later, transient chassis dynamics |
| Vehicle and lap simulation | No coupled vehicle solver or lap-time calculation | Powertrain and braking, local equilibrium, fixed-path speed profiles and time integration |
| Optimization and analysis | No minimum-time optimizer or executed parameter studies | Independent optimal-control reference, sensitivity and uncertainty analysis, then free-path optimization |

The foundation provides scalar models and input infrastructure. It does not establish a validated vehicle simulator. Track data has not been conditioned into solver geometry, and suspension compliance does not itself calculate vehicle axle loads.

## Technical ambition

The proposed architecture separates input validation and SI unit conversion from a deterministic numerical core. Geometry, atmosphere, aerodynamic loads, tyre capacity, powertrain and braking feed a local vehicle-equilibrium calculation. A fixed-path solver would then construct acceleration- and braking-feasible speed profiles, followed by independent feasibility and convergence checks and reproducible reporting.

The first trajectory model is intended to be quasi-static and spatial, propagating squared speed along a prescribed path. Later work would compare it with an independently formulated optimal-control reference before introducing joint path and speed optimization. Transient dynamics and higher-fidelity aerodynamic or tyre models require additional physical inputs, calibration and their own validation criteria.

The engineering challenge includes implicit coupling between acceleration and axle loads, combined tyre-force constraints, track geometry conventions, periodic circuit closure, numerical error control and constrained optimization. Explicit infeasibility and nonconvergence reporting are part of the intended design.

## Verification and scientific limits

The [numerical validation document](docs/NUMERICAL_VALIDATION.md) records historical checks of the separate foundation: 298 passing tests, passing lint and format checks, and three strict typing errors. The complete check workflow therefore failed. These are recorded foundation results, not tests run from this repository or evidence of lap-solver performance.

Future verification is planned around analytic cases, adversarial inputs, mesh refinement, independently recomputed physical constraints and comparison between distinct numerical formulations. A proposed numerical error budget of 0.01 s for a nominal 60 s synthetic lap is a development target, not an achieved accuracy claim. Runtime and memory budgets are likewise targets rather than measured solver results.

Synthetic agreement and numerical convergence do not establish real-world predictive accuracy. That requires suitable calibration data, held-out experimental validation and measurement uncertainty. A locally optimized trajectory is not proof of a global minimum.

## Development direction

Development is staged so that each capability has a scientific verification basis before the next layer depends on it:

1. Establish a distributable, reproducible foundation and resolve model and interface decisions.
2. Implement track geometry, powertrain and braking limits, and local vehicle equilibrium.
3. Build and verify fixed-path speed profiles and elapsed-time integration.
4. Develop an independent optimal-control reference and reproducible aero/setup studies.
5. Add sensitivity, uncertainty and synthetic measurement comparisons.
6. Explore free-path minimum-time optimization and calibrated higher-fidelity dynamics.

The [roadmap](ROADMAP.md) details dependencies, proposed acceptance criteria and resource budgets. These milestones describe future work; they are not released features or delivery commitments. Any future reuse of foundation source requires resolved redistribution rights; this repository currently distributes documentation only.

## Technical documentation

| Document | Contents |
|---|---|
| [Architecture](docs/ARCHITECTURE.md) | Foundation inventory, proposed subsystem interfaces, coordinates and numerical integration strategy |
| [Physics models](docs/PHYSICS_MODELS.md) | Implemented foundation equations, assumptions, validity limits and proposed extensions |
| [Numerical verification and validation](docs/NUMERICAL_VALIDATION.md) | Historical foundation evidence, known gaps, proposed error budgets and benchmark methods |
| [Development roadmap](ROADMAP.md) | Staged scientific objectives, dependencies and acceptance criteria |

The technical documents retain the detailed foundation record and design proposals. References in them to source modules, tests or tooling describe the separate foundation or a proposed future layout, not files available in this repository.
