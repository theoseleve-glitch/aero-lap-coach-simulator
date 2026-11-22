# Architecture — Track Aero foundations to unified simulation
Status: review proposal. Existing behavior and future design are explicitly separated. No new algorithms were implemented. Scope is the authorized Track Aero Platform v0.1.1 folder only.

## 1. Existing inventory
| Modules in src/track_aero | Actual responsibility | Dependency boundary |
|---|---|---|
| schema.py | Immutable Pydantic entity, vehicle, environment, track-descriptor and Study models with field/cross-field checks | Pydantic |
| loading.py | Strict UTF-8 YAML; rejects ambiguous/non-JSON constructs and nonfinite values | PyYAML, schema, errors |
| units.py | Explicit unit conversion with fixed Decimal context; atmosphere boundary adapter | Pint, schema |
| canonical.py | RFC 8785 canonical JSON and SHA-256 | rfc8785, input validation |
| provenance.py | Immutable raw bytes and derived lineage | loading, canonical, schema |
| study.py | Load references once, normalize temperature/RPM and compile content identities | Above boundary modules |
| cli.py, __main__.py | validate/hash/canonical operations only | Study compiler |
| environment.py | Moist-air state and saturation pressure | Standard-library scalar math |
| aero.py | Dynamic pressure, drag, downforce and axle split | Standard-library scalar math |
| tyres.py | Effective axle peak force, utilization and remaining capacity | Standard-library scalar math |
| suspension.py | Per-axle compression, ride-height and minimum-margin flags | Standard-library scalar math |
| errors.py, __init__.py | Explicit errors and package identity | Minimal support |

14 Python source files, py.typed, 15 test modules and conftest.py were observed. contracts contains machine-readable input/capability declarations; docs/plans define the longer target. examples supplies one synthetic Study and associated entity/path files. Track bytes are hashed but not parsed as geometry. Sweep declarations are preserved but not executed. No telemetry logger or external application dependency exists in the active package.

The current in-place test baseline is in NUMERICAL_VALIDATION.md. Technical separability does not establish reuse permission.

## 2. Layering and ownership
Proposed dependencies:
```text
external quantities + provenance
             |
      validated SI models
             |
   compiled Study / solver-input identity
             |
       conditioned geometry
             |
E1 -> A1 -> vehicle equilibrium <-> Ty1 / powertrain / braking
                |
       local acceleration feasibility
                |
       fixed-path speed profile
                |
 independent feasibility/refinement checker
                |
     immutable RunResult -> report
                |
     sweeps / uncertainty / later OCP
```
S1 consumes solved axle loads as a diagnostic. At A1 there is no feedback from ride height into aero.

The numerical core must be pure and deterministic given explicit inputs. I/O, unit conversion, cache lookup, telemetry ingestion and report formatting remain outside repeated force evaluations. Each subsystem owns its domain checks and returns an explicit status; callers cannot infer missing physical inputs or suppress nonconvergence.

## 3. Proposed interface contracts
These are conceptual records, not implemented classes or frozen schemas.

| Interface | Required content | Consumer/invariant |
|---|---|---|
| VehicleParametersSI | mass, geometry/static distribution, tyre axle parameters, powertrain mode, drivetrain split, resistance, A1 and S1 data | Immutable; finite values and units checked at boundary |
| PathGeometry | ordered distance, position, tangent, signed lateral curvature, grade, open/closed status, optional boundaries, conditioning metadata | Increasing distance; distinct closure semantics; no guessed width |
| LocalMotionState | speed_mps, longitudinal_accel_mps2, signed lateral_accel_mps2, grade_rad, path station | Algebraic state for quasi-static calculations, not a transient state vector |
| AtmosphereState | density and phase/domain diagnostics | E1 output used once per constant environment |
| AeroLoad | drag/downforce magnitudes, front/rear split, model/domain identity | Sign converted once in vehicle force balance |
| AxleLoadState | front/rear normal load, reference loads, equilibrium residuals and validity | Ty1 and S1; wheel lift is a domain event, not clamped |
| AxleForceDemand | signed longitudinal/lateral axle forces plus peak capacities/utilizations | Independent feasibility check |
| PowertrainLimit | feasible raw wheel force, gear/RPM where applicable, interpolation domain/status | Before tyre limitation |
| AccelerationBounds | a_min/a_max, feasible speed domain, limiting axle/constraint, root bracket/residual/iterations | Profile solver; interval validity must be established |
| ProfileResult | s,v,t,ax,ay,loads,forces,utilizations and all constraint margins | Independent recomputation, not trusted optimizer flags |
| RunResult | numerical outputs, termination/feasibility/convergence/validation statuses, provenance and limitations | Reporting and studies |

Never conflate implemented, experimental, numerically verified, physically validated and client_safe. A solver may terminate while infeasible; a feasible local solution need not be optimal.

## 4. Coordinates and state
Internal units are SI. Vehicle axes are x forward, y left, z up. Signed curvature and path lateral displacement are positive left. Ground/world coordinates require an explicit metric frame and transform; geodetic adapters are deferred.

Initial D0 is spatial and quasi-static. Use speed or w=v^2 as the propagated state and elapsed time as an accumulated output. It does not have yaw inertia, suspension velocity or wheel-speed states.

A later planar dynamic interface can use world position/heading and body vx,vy,yaw rate; inertia and force laws are then mandatory. It must not reuse a peak-force envelope as if it predicted slip-angle response. Steering, tyre relaxation and actuator states are separate fidelity decisions.

For 2.5D paths, distance and curvature definitions must be approved explicitly. Proposed convention: distance is 3D path arc length; lateral curvature is the left-horizontal-normal projection of d(tangent)/ds. This equals planar curvature on a flat track and keeps ay=v^2*kappa_lat consistent. Grade and vertical curvature remain distinct. Current frozen V1 omits vertical-curvature normal-force dynamics; do not claim crest/wheel-lift prediction from grade alone.

## 5. Coupling strategy
At each station/speed and candidate ax:
1. Obtain E1 density and A1 force magnitudes.
2. Evaluate the approved reduced front/rear load model.
3. Split required lateral force by zero-yaw-acceleration equilibrium.
4. Evaluate load-dependent directional tyre capacities and remaining longitudinal capacity.
5. Intersect drivetrain/raw powertrain or ideal-braking allocation limits with axle capacities.
6. Form the longitudinal equilibrium residual; solve feasible endpoints only on valid brackets.
7. Evaluate S1 using declared reference loads; retain bottoming/domain flags.
8. Return bounds and diagnostics; no trajectory generation inside a physics function.

A1 permits one-way coupling. Future ride-height aero maps create an algebraic feedback loop requiring a separately approved equilibrium solver, domain-aware interpolation, multiple-root/stability treatment and calibration evidence. No feedback loop is added implicitly.

## 6. Numerical method selection
Use a bracketed scalar method for continuous equilibrium residuals after verifying a sign-changing bracket; otherwise return infeasible/no bracket or enumerate feasible intervals. Brent's method is an appropriate CPU candidate, with tolerances derived from force/time budgets. [SciPy brentq](https://docs.scipy.org/doc/scipy/reference/generated/scipy.optimize.brentq.html)

For declared torque/force tables, shape-preserving PCHIP follows the existing freeze's intent; explicitly disable extrapolation and validate source nodes. [SciPy PCHIP](https://docs.scipy.org/doc/scipy/reference/generated/scipy.interpolate.PchipInterpolator.html)

The authorized freeze already selects spatial forward/backward propagation for initial fixed-path D0, with an independent optimal-control benchmark later. Derive the implementation from this project's equations; no external project algorithm is imported. The profile algorithm needs full-grid feasibility and periodic closure checks, not just a stable objective.

Transient state integration is later scope: explicit adaptive Runge–Kutta for a nonstiff model; implicit Radau/BDF only after demonstrated stiffness warrants it. Use per-state error scales and event detection. [SciPy solve_ivp](https://docs.scipy.org/doc/scipy/reference/generated/scipy.integrate.solve_ivp.html)

A new direct-transcription OCP can provide an independent fixed-path reference after D0. Candidate CPU backends include a sparse constrained optimizer or the independently provisioned reference stack contemplated in the freeze. Sparse trust-region constrained optimization is one candidate; its stopping settings do not replace independent feasibility checks. [SciPy trust-constr](https://docs.scipy.org/doc/scipy/reference/optimize.minimize-trustconstr.html)

No new dependency is installed here. Benchmark actual sparsity, memory and conditioning before selecting the production OCP backend.

## 7. Proposed repository structure
Initially the seven review documents only. Once rights and architecture are approved:
```text
src/track_aero/       # retain a small coherent package if cleared; no forced rename
tests/               # approved or newly authored analytic/adversarial tests
examples/synthetic/  # explicit synthetic labels and generation metadata
contracts/           # cleared/versioned machine behavior contracts
docs/
pyproject.toml
dependency lock
scripts/             # portable canonical check entry point
```
No package scaffold pretending to solve a lap is created in this corrective pass. Existing .venv/cache directories are never distributed. Linux packaging must replace Windows-specific tooling conventions without altering physical acceptance criteria.

## 8. Approval blockers
- Reuse rights remain unresolved; source admission is a separate decision from architecture.
- Canonical mypy gate currently fails; fixes require authorization in the eventual cleared/new implementation, not edits to the preserved source.
- Zero aero areas: machine contract/freeze allow zero; AGENTS wording says strictly positive. Record and resolve before changing behavior.
- The older plan's atmosphere wording is superseded by the milestone freeze; use the actual ideal-mixture implementation in this audit.
- Approve 2.5D arc-length/curvature definitions, reference axle loads, grade/pitch approximation and diagnostic bottoming policy.
- Approve error budgets and physical-model validity domains before algorithm work.
