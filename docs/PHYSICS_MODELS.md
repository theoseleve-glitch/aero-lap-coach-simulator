# Physics models and assumptions
Status: source audit plus review proposal. Formulas below describe implemented primitives or this project's frozen/proposed extensions; they are not evidence that a coupled simulator exists.

## Implemented E1 atmosphere
T in kelvin, total pressure p in Pa, relative humidity phi and CO2 mole fraction are explicit inputs.
p_v = phi*p_sat(T), p_da = p-p_v.
rho = (p_da*M_da + p_v*M_v)/(R*T).
M_da = [28.96546 + 12.011*(x_CO2-0.0004)]*1e-3 kg/mol.
M_v = 18.01528e-3 kg/mol and R = 8.31446261815324 J/(mol K).

The source uses IAPWS ice sublimation below 273.16 K and IF97 liquid saturation at/above that point. Domain: 193.15–353.15 K, 10000<p<=120000 Pa, 0<=phi<=1 and 0<=x_CO2<=0.01, additionally p_v<p. This is an ideal mixture, not full CIPM compressibility/enhancement. It does not infer weather, wind, altitude or engine corrections. Synthetic/reference verification does not eliminate model-form uncertainty.

## Implemented A1 aerodynamics
q_dyn = rho*v^2/2.
Downforce Df = q_dyn*A_down; drag D = q_dyn*A_drag.
Front downforce = beta*Df; rear = Df-front.
Areas are constant, forces are nonnegative magnitudes, beta in [0,1]; rear subtraction supports conservation. Under E1, ground and air speed coincide.

A1 does not depend on ride height, pitch, roll, yaw, steering or Reynolds number. Vehicle balance must apply drag opposite motion and downforce downward once. No arbitrary induced-drag relation is permitted.

## Implemented Ty1 tyres
For each effective axle/direction:
F_peak = mu_ref*Fz_ref*(Fz/Fz_ref)^p, with 0<p<=1.
Reference load and coefficient are positive; current load may be zero, returning zero peak force. Effective mu is undefined at zero load.
U = [(abs(Fx)/Fx_peak)^q + (abs(Fy)/Fy_peak)^q]^(1/q), with 1<=q<=4.
A nonzero demand against zero capacity is infeasible; zero demand contributes zero.

Residual longitudinal capacity after lateral demand is returned with a feasibility flag. This is a static effective-axle force envelope, not a slip, temperature, transient or individual-wheel force law. A supplied-load feasibility flag is not trajectory validation.

## Implemented S1 compliance
compression = (Fz-Fz_ref)/Kz.
h = h0-compression.
If h_min is supplied, margin=h-h_min and a below-minimum flag is reported. Negative compression represents extension; no clipping, droop, bump-stop force or damper dynamics is modeled. Kz is an effective axle ground-to-chassis stiffness, not automatically a coil-spring rate.

S1 does not calculate axle loads. Its reference load must be provided by the eventual vehicle-model contract. A1 is unaffected by S1 output.

## Frozen future V1 coupling — not implemented
Let m be mass, L=a+b wheelbase, lambda=b/L static front fraction, h_cg CG height, theta grade. The existing simplified freeze proposes:
DeltaFz = m*ax*h_cg/L.
Fzf = lambda*m*g*cos(theta)-DeltaFz+Df_front.
Fzr = (1-lambda)*m*g*cos(theta)+DeltaFz+Df_rear.
Reject nonpositive axle loads at the V1 level.

This is a reduced pitch/load-transfer approximation. It does not resolve full pitch moment balance from grade, aero force application heights or transient suspension. Do not silently upgrade its equations. Before claiming general hillclimb accuracy, document those omissions and approve the operating domain or a revised moment-balance model.

Lateral demand: ay=v^2*kappa_lat; Fyf=(b/L)*m*ay; Fyr=(a/L)*m*ay. Aero balance changes capacity, not the demanded zero-yaw-acceleration split.

Longitudinal equilibrium:
m*ax = F_drive-F_brake-D-C_rr*(Fzf+Fzr)-m*g*sin(theta).
The tyre-demand forces and external body drag remain separately represented. Each axle must satisfy its combined-force envelope. Because axle loads depend on ax, acceleration bounds require implicit feasibility calculation.

## Powertrain and brakes — not implemented
Internal raw capability is F_drive_raw_max(v), prior to tyre limitation.
For feasible gear g: omega_engine=(v/r_wheel)*ratio_g*final_drive.
F_wheel_g=T_engine(omega_engine)*ratio_g*final_drive*eta/r_wheel.
Select among gears within declared curve/RPM domains, retain gear identity, and apply drivetrain allocation before axle saturation. A wheel-force table is an alternative. No extrapolation, shift delay, clutch, rotational inertia or engine thermal correction is silently introduced.

The frozen initial brake assumption is ideal tyre-limited allocation, optionally capped by explicit hardware limits; it is not a measured fixed-bias/ABS brake model. Source schemas have not implemented the full brake execution contract. Any additional field must be versioned and reviewed.

## D0 profile and numerical integration — not implemented
For a fixed path, define w=v^2 and dw/ds=2*ax. A constant-acceleration segment obeys:
w_next=w+2*ax*ds,
dt=2*ds/(v+v_next).
This exact segment time is appropriate only for that segment model. Both endpoint speeds zero with ds>0 cannot be assigned a finite time by adding an arbitrary speed floor; split the motion into valid accelerating/decelerating phases or return infeasible.

The initial solver must intersect lateral feasibility with forward reachable and backward brake-feasible profiles, respecting start/finish or periodic constraints. Non-monotone feasibility is possible with aero/load sensitivity; do not assume one global scalar bracket without checking its domain. Grid convergence and feasibility are separate checks.

## Project-specific optimal control — proposal
First independently benchmark the same fixed-path reduced model. Decision variables can include w_i, longitudinal axle forces and algebraic axle loads; constraints impose discrete motion, force/load balance, drivetrain/brake bounds, tyre utilization and periodic or endpoint conditions. Objective is elapsed time with quadrature consistent with the transcription. Record defects, scaled KKT/optimality measures where available and termination reason. A local NLP optimum is not a global proof.

Free-path optimization is later, separately approved scope. For planar reference r(s), normal N(s), path r_n(s)=r(s)+n(s)N(s):
dell/ds = sqrt((1-kappa*n)^2+(dn/ds)^2).
Compute path curvature from the actual first/second derivatives, and integrate time along ell, not unchanged reference length. Boundaries require explicit source geometry and vehicle footprint assumptions. Dense sampling is a check, not a continuum guarantee without an interpolation bound.

## Later fidelity and calibration needs
- Lateral load transfer requires per-wheel loads, track widths, roll centers/stiffness distribution and an appropriate tyre calibration. Do not divide the current effective-axle law into two tyres without evidence.
- Dynamic chassis: m*(dvx/dt-r*vy)=sum Fx, m*(dvy/dt+r*vx)=sum Fy, Izz*dr/dt=sum Mz, with declared frames and force laws. This requires inertia and steering/slip response; Ty1 alone does not close the system.
- Ride-height aero requires measured/CFD-derived maps, domain restrictions, interpolation uncertainty and equilibrium/stability analysis. A1 cannot validate ground-effect operating windows.
- Synthetic sensors require explicit observation frames, orientation/gravity and timing/noise models. IMU specific force is not simply coordinate acceleration.
- Setup optimization/sensitivity/UQ must preserve parameter provenance and separate numerical, input and model-form uncertainty.

All these are future directions. Higher fidelity is justified by identifiable parameters and validation evidence, not by feature count.
