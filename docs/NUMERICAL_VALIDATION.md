# Numerical verification and validation
Audit date: 2026-11-22. Source: only Track Aero Platform v0.1.1 in the authorized folder.

## 1. Actual observed checks
Executed the supported command from the source root:
```powershell
powershell -ExecutionPolicy Bypass -File scripts/check.ps1
```
Results:
| Stage | Observed result |
|---|---|
| pytest | 298 passed in 9.18 s; no failures/skips reported |
| Ruff lint | All checks passed |
| Ruff format check | 30 files already formatted |
| strict mypy | 3 errors in tyres.py; 30 files checked |
| Whole workflow | Exit code 1; measured wall time 24.2427697 s |

Mypy diagnostics: src/track_aero/tyres.py lines 59, 93 and 123, returning Any from a function declared float (no-any-return). They remain unfixed to preserve the original. The gate is not green despite passing tests.

A pre-existing PowerShell startup-profile diagnostic was emitted before pytest; the workflow continued. No profile was inspected or modified. The test result above comes from the completed workflow, not that startup error.

Environment: Python 3.12.13, Windows-10-10.0.19045-SP0; pytest 9.1.1, Ruff 0.16.10, mypy 1.20.2, Pydantic 2.13.5, Pint 0.26.1, PyYAML 6.0.3, rfc8785 0.1.4.

Preservation controls: UV_NO_SYNC=1 avoided virtualenv synchronization; UV_NO_CACHE=1 and UV_CACHE_DIR redirected uv storage; PYTHONDONTWRITEBYTECODE=1 disabled bytecode writes; HYPOTHESIS_STORAGE_DIRECTORY, LOCALAPPDATA, TEMP and TMP pointed to a new scratch directory. The existing wrapper retained its fresh-GUID test base and disabled pytest cacheprovider. Tests created their own temporary fixture copies for local verification only; none is admitted to a new repository. Dependencies were not reinstalled, so this was verification of the installed environment, not a fresh lockfile installation test.

All 76 maintained files in the before/after inventory retained identical SHA-256 hashes. This is a check of that file set, not proof of every filesystem entry. No original source/configuration was edited. No peak-memory or solver-runtime benchmark was measured.

## 2. Demonstrated scalar baseline
Read-only smoke run, synthetic inputs:
- T=293.15 K, p=101325 Pa, RH=0.5, CO2=0.0004.
- v=40 m/s, A_down=1.2 m², A_drag=0.55 m², front balance=0.44.
- Tyre reference/current load=3000 N, mu_ref=1.75, p=0.88.
- S1 load=3500 N, reference=3000 N, Kz=100000 N/m, h0=0.040 m.

Observed:
| Output | Value |
|---|---|
| Air density | 1.1988742701663833 kg/m³ |
| Total downforce | 1150.9192993597278 N |
| Drag | 527.5046788732087 N |
| Front/rear downforce | 506.4044917182802 / 644.5148076414475 N |
| Tyre peak at reference | 5250.0 N |
| Pure-axis utilization at peak | 1.0 |
| Compression / ride height | 0.005 / 0.035 m |

These are separately supplied scalar conditions. Axle load was not solved from aero or vehicle equilibrium. No circuit was simulated and no lap time exists.

## 3. Existing verification coverage and gaps
Existing tests exercise contract/schema parity, units, canonical identities, provenance immutability, CLI behavior, scalar analytic limits, nonfinite/domain rejection and primitive plumbing. Scalar tests do not establish coupled vehicle correctness.

Gaps for review: default pytest.approx tolerances in several tests, incomplete independent-reference separation in some atmosphere checks, extreme finite intermediate overflow, no physical calibration evidence, no convergence study for a vehicle/profile solver because none exists. Historical partial-test counts are not substituted for the current run.

## 4. Proposed error-budget policy — requires approval
No acceptance threshold in the original is changed here. The following new budgets are engineering proposals for future milestones, not validated properties.

Research resolution assumption to approve: resolve a 0.1 s change on a nominal 60 s synthetic lap. Allocate at most 0.01 s numerical uncertainty (10:1 separation from that effect):
- spatial geometry/discretization contribution <=0.005 s;
- local equilibrium/root and constraint contribution <=0.002 s;
- time quadrature contribution <=0.002 s;
- rerun/roundoff contribution <=0.001 s.
This is an explicit intended-use choice, not a real-car accuracy claim. If the research question changes, recalculate the budgets.

Do not convert these allocations into arbitrary universal solver tolerances. Derive local force/velocity tolerances from sensitivity or conservative bounds. For reciprocal-speed integration, use |delta t| <= integral |delta v|/v_min² ds when a positive justified speed lower bound applies. Near rest use the segment model's own error analysis.

For elementary analytic identities, use exact comparisons where representable. Otherwise derive roundoff allowances from operation count/conditioning (gamma_n=n*eps/(1-n*eps)) or an independent high-precision evaluation. Published coefficient uncertainty and model discrepancy are separate from binary64 error. No blanket tolerance certifies every equation.

Scaled force residuals, motion defects and tyre utilization violations must have dimensional equivalents in reports. Suggested closure budgets follow the same time/error allocation; do not pick dimensionless tolerances before defining scales.

## 5. Analytic and adversarial benchmark plan
| Case | Independent expected behavior | Acceptance method |
|---|---|---|
| Dry atmosphere | rho=p*M/(R*T) with independently supplied molar mass | Roundoff/reference error budget |
| Aero at zero/doubled speed | zero forces / fourfold force | Exact or bounded floating-point identity |
| Ty1 p=1 and reference load | linear force / mu_ref*Fz_ref | Algebraic identity; zero-capacity adversarial cases |
| S1 | load increment/Kz | Exact synthetic values and domain checks |
| Straight/circle geometry | curvature 0 / signed 1/R, circumference 2*pi*R | At least 3 refinements and bounded geometry error |
| Symmetric flat no-aero circle | v=sqrt(mu*g*R), t=2*pi*R/v under stated p=1 assumptions | Fine-grid time within approved 0.01 s total budget |
| Constant acceleration/braking | v1²=v0²+2*a*L; t=2L/(v0+v1) | Derived roundoff bound; rest endpoint supported |
| Power versus quadratic drag only | terminal v=(2P/(rho*A_drag))^(1/3) | Root residual plus propagated velocity error |
| Force and axle-load balance | sum forces=m*a, axle loads match approved normal model | Dimensional residual within derived budget |
| Invalid inputs/nonconvergence | explicit rejected/infeasible/nonconverged result | 100% of enumerated rejection cases detected |
| Closed path | periodic state and independently checked feasibility | Closure budget plus mesh refinement |
| Perturbed trajectory | deliberately injected violations detected at locations | All constructed violations above error budget rejected |

Analytic cases apply only to their exact assumptions. Avoid validating a general model with a reference that calls the same production helper.

## 6. Convergence and physical feasibility
Use h,h/2,h/4 (and further refinement when asymptotic behavior is not established). Track time, speed norms, constraint maxima and branch/active-set changes. Estimate observed order only when refinement supports it; do not blindly use Richardson extrapolation across discontinuities. Tighten root tolerance independently of mesh size.

An independent checker recomputes kinematic, force, axle-load, tyre and powertrain constraints from outputs. Nodewise checks must be supplemented with segment/interpolation checks; free-path boundaries later require footprint and between-node treatment. Keep geometric feasibility, dynamics feasibility, numerical convergence, optimization status and physical validation distinct.

## 7. Future reproduction and CPU gates
Each run records Study/source/solver-input hashes, schema versions, commit+dirty state, lock digest, solver settings/tolerances, seeds, OS/Python/numerical-library versions and relevant hardware. REP-2 is a numerical tolerance claim; REP-3 needs a frozen stack and evidence.

Proposed interactive resource gates, not measurements: fixed-path benchmark up to 8192 nodes, <=8 GiB peak RSS and <=60 s on the declared four-core CPU machine; first reference OCP <=400 nodes, <=12 GiB and <=300 s. These leave RAM headroom in a 16 GiB environment and bounded session time. If unmet, profile and reduce problem size or redesign; never loosen physics to meet a runtime target.

Synthetic verification and cross-solver agreement do not demonstrate real-world predictive accuracy. That requires independently sourced calibration and held-out experimental validation with measurement uncertainty.
