# Multi-session engineering roadmap
This roadmap is specific to Track Aero Platform's foundations. All algorithm milestones await architecture approval. No new solver is implemented. Each session ends with a reproducible artifact, independent verification evidence and a decision log; do not advance a validation label merely because code exists.

Quantitative criteria below use the proposed 0.01 s / 60 s numerical budget in docs/NUMERICAL_VALIDATION.md. That budget and its assumptions require explicit approval. Existing frozen acceptance criteria are not replaced.

## M0 — architecture, provenance and verified foundation (1–2 sessions)
Scientific objective: establish trustworthy scalar/boundary inputs.
Work: clear source/tests/fixture rights or plan independent replacements; resolve zero-area and 2.5D conventions; repair the three typing issues only in the authorized future source; create a portable check workflow and offline lock.
Dependencies: destination writable; architecture/provenance decisions.
Verification: rerun all eligible existing tests without weakening them; independent equation/domain review.
Acceptance: complete test/lint/format/strict-type gate exits 0; every admitted file has recorded eligibility; all listed model ambiguities resolved or explicitly excluded.
CPU/memory: current test suite completed pytest in 9.18 s on Windows, but Linux is unmeasured. Require offline Linux execution before portability claims.

## M1 — geometry and reproducibility contracts (1–2 sessions)
Objective: produce a trustworthy fixed 2.5D path and run identity.
Methods/components: metric path resampling, differentiable geometry, signed curvature/grade, open/closed handling; immutable conditioned artifacts; separate solver-input and execution identities.
Dependencies: M0.
Independent verification: straight, circle, reversed direction and constant-grade analytic paths; duplicated/degenerate points and closure adversarial cases.
Acceptance: geometry error propagated to time <=0.005 s allocation on benchmark suite; at least three refinement levels; 100% enumerated malformed cases rejected; identical semantic inputs reproduce numerical identity.
CPU: begin 512 nodes, refine to 8192; target <=8 GiB under the baseline budget. No imagery/GPS reconstruction needed initially.

## M2 — powertrain, brakes and local vehicle equilibrium (2–3 sessions)
Objective: couple force limits to axle loads without hidden parameters.
Methods/components: domain-limited shape-preserving curves, engine/gearing-to-wheel-force conversion, declared drivetrain split, ideal tyre-limited braking, A1 loads, quasi-static load transfer and Ty1 capacities. Use bracketed scalar roots for approved continuous feasibility residuals.
Dependencies: M0; M1 supplies local geometry conventions.
Independent verification: analytic static load balance, constant-torque gear force, constant-power drag terminal speed, no-aero circular limit, front/rear saturation and lift rejection.
Acceptance: propagated equilibrium/root error <=0.002 s allocation; 100% explicitly infeasible/no-bracket cases return honest status; no extrapolated force values; conservation residuals satisfy documented dimensional budgets.
CPU: scalar solves with bounded iterations; measure per-node cost and expose root diagnostics. No transient inertia/roll model yet.

## M3 — fixed-path speed profile and time integration (2–3 sessions)
Objective: first coherent synthetic circuit/vehicle -> feasible speed/time result.
Methods/components: spatial w=v² propagation constrained by M2; forward reachability/backward braking, explicit periodic closure; consistent segment-time integration and initial/final conditions.
Dependencies: M1–M2.
Independent verification: constant-speed circle, straight constant acceleration/deceleration including rest endpoint, independent segment force checks; compare 3+ meshes.
Acceptance: analytic benchmark time error <=0.01 s total; quadrature contribution <=0.002 s; all independent constraints within derived budgets; no arbitrary speed floor; nonconvergence reported.
CPU: <=60 s and <=8 GiB at 8192 nodes on target CPU, proposed interactive gate. Measure actual scaling; no runtime result exists yet.

## M4 — independent optimal-control reference and V&V gate (2–3 sessions)
Objective: falsify errors in the fixed-path method with a distinct formulation of the same approved model.
Methods/components: newly designed direct transcription with state/force/load constraints and consistent time objective; sparse constrained optimization; independent post-solve checker.
Dependencies: M3; approve numerical backend and offline dependencies.
Verification: analytic cases plus independently formulated constrained reference, mesh/tolerance separation and failure injection.
Acceptance: analytic/refinement budgets pass; same-model inter-method time discrepancies <=0.01 s on approved synthetic cases or explicitly explained within independent error estimates; maximum constraint violations/KKT measures reported. No global optimality claim.
CPU: initial <=400 nodes, <=300 s, <=12 GiB, subject to measured feasibility. Keep this a reference rather than initial mandatory production dependency as the freeze requires.

## M5 — reproducible reports and aero/setup study execution (1–2 sessions)
Objective: make aero-value conclusions traceable and falsifiable.
Methods/components: deterministic OAT/grid study generation, immutable child runs, local HTML/SVG/JSON, run manifests and separate status fields.
Dependencies: M3; “validated” research claims gated on M4.
Verification: synthetic zero-aero and controlled drag/downforce comparisons; parameter ordering and rerun tests; corrupt/fail a child intentionally.
Acceptance: 100% outputs link required run metadata and limitations; failed children remain visible; numerical rerun contribution <=0.001 s; no report upgrades failed validation.
CPU: at most four bounded workers; total RSS within 12 GiB. Run counts bounded explicitly.

## M6 — sensitivity and uncertainty analysis (2 sessions)
Objective: distinguish parameter effects, numerical error and model uncertainty.
Methods/components: finite-difference step studies first; optional gradients only after cross-checks; seeded sampling of declared distributions/correlations; active-constraint diagnostics.
Dependencies: M4–M5.
Independent verification: analytic sensitivity of drag terminal speed (log derivatives: P +1/3, rho -1/3, drag area -1/3), exact scale identities and controlled nonsmooth cases.
Acceptance: derivative errors fit a derived truncation/roundoff envelope over at least 3 step sizes; sampling error assessed with predeclared statistical intervals; all failed samples counted; no scalar confidence score.
CPU: budget total runs and use <=4 workers; measure accuracy/cost tradeoff.

## M7 — synthetic measurement generation/comparison (2 sessions)
Objective: test observation and comparison pipelines without real telemetry.
Methods/components: analytic motion and checked trajectories -> explicit sensor observation model -> sampling/noise/bias/latency/dropouts -> provenance-preserving alignment and residual diagnostics.
Dependencies: M3/M5; M6 for stochastic uncertainty.
Verification: constant-speed/acceleration/circular-motion observables, known injected biases/time offsets, unit/frame mistakes and missing samples.
Acceptance: zero-noise cases meet derived sampling/roundoff budgets; every constructed above-budget deterministic error is detected; seeded outputs reproduce on declared stack; random moment tests use predeclared finite-sample bounds.
CPU: initially <=1e6 samples, <=8 GiB; no GPU or private data. Synthetic agreement is software verification only.

## M8 — project-specific free-path minimum-time solver (3+ sessions; beyond v0.1)
Objective: jointly optimize path and speed inside explicit boundaries.
Methods/components: smooth path parameterization with actual arc-length metric/curvature, vehicle footprint and force constraints; sparse direct transcription and multistart; independent feasibility checker.
Dependencies: M4 and a separately approved boundary/path contract. Minimum-curvature may be a geometric initialization, not a time optimum.
Verification: symmetric synthetic corridors, straight/constant-radius analytic families, dense/adaptive between-node checks and discretization refinement.
Acceptance: satisfy independent geometry/dynamics budgets; report violation locations and local-optimality diagnostics; variation under mesh refinement within approved numerical budget. Continuous feasibility requires justified interval bounds, otherwise label sampling limits.
CPU: start <=200–400 path nodes; <=12 GiB/300 s initial trial budget. Larger ambitions require profiling and revised approved limits.

## M9 — calibrated higher fidelity and identification (multiple separate milestones)
Objective: add fidelity only where data and identifiability justify it.
Methods/components: per-wheel lateral transfer, slip-based tyre response, dynamic chassis/inertia, transient integration, ride-height aero maps and parameter identification.
Dependencies: an approved research question, identifiable parameter set, calibration rights and M6/M7 observation infrastructure.
Verification: energy/momentum balances, analytic linearized modes, step-size convergence, independent synthetic identification and held-out experimental runs where authorized.
Acceptance: recover identifiable synthetic parameters within predeclared intervals; detect deliberately confounded parameters; numerical budgets established separately for new states; experimental prediction intervals assessed on held-out data. No predictive accuracy claim without that evidence.
CPU: identify stiffness before choosing explicit versus implicit integration; bound parameter/mesh size and profile within four cores/16 GiB.

## Required review sequence
Approve model/coordinate and error-budget decisions -> clear or independently replace foundations -> establish M0 green -> geometry/local equilibrium -> checked fixed-path baseline -> independent reference -> studies -> later optimization/calibration.

Substantial unresolved work is concentrated in subsystem coupling, geometric/numerical correctness, independent validation and constrained optimization. No superficial UI or CRUD work is used to inflate scope.
