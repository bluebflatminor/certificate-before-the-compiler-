"""
Worked example for "The Certificate Before the Compiler" (v1.2)
----------------------------------------------------------------
Demonstrates Clause 1 (epsilon-resolved effective dimensionality) and
Clause 2 (noise floor from the full inter-channel covariance) of the
proposed conformance certificate for physical reservoir computing
hardware.

v1.2 changes (red-team fixes):
  * eps_floor is now PERCENTILE-DEFINED: the 95th percentile of the
    rescaled largest noise singular value over N_ACQ = 20 independent
    noise-only acquisitions, instead of a single-realization hard
    threshold (the largest SV of one finite-T noise matrix is a random
    variable with a fluctuating edge; a point estimate is not metrology).
  * All headline quantities are reported as mean and range over
    N_SEEDS = 10 independent device realizations, not one seed.
  * The figure uses seed 7 (representative) with its percentile floors.

Model
-----
Synthetic driven reservoir: N = 200 readout channels, T = 4000 steps,
x[t+1] = tanh(W x[t] + W_in u[t]), spectral radius 0.9, i.i.d. uniform
input. Two noise models with identical mean per-channel power:

  Case A (detector-limited): i.i.d. Gaussian per channel.
  Case B (correlated):       same average power injected through a
       non-orthogonal mixing basis. Covariance anisotropy ratio
       K_eff = lambda_max / mean(lambda) of the noise covariance --
       analogous in ROLE to a Petermann excess factor, NOT constructed
       from mode non-orthogonality. The toy demonstrates that structured
       correlation degrades honest dimensionality; it does not reproduce
       Petermann physics.

Known limitation: certification under a single input ensemble
(i.i.d. uniform). The schema requires two ensembles; the toy
demonstrates one. Noise here is additive at readout; dynamical noise
is the province of Clause 5 (replica consistency), which is core for
exactly that reason.

Reproducibility: fixed seeds, single file, numpy only. CC0.
"""

import numpy as np

N, T, WASHOUT = 200, 4000, 200
SIGMA = 0.02
R_MODES, COUPLING = 12, 6.0
N_ACQ = 20          # noise-only acquisitions per floor estimate
PCTL = 95           # floor percentile
N_SEEDS = 10
FIG_SEED = 7

sv = lambda m: np.linalg.svd(m, compute_uv=False)

def device(rng):
    """Build one device realization: states, smax, and the B mixing matrix."""
    W = rng.normal(0, 1, (N, N)) / np.sqrt(N)
    W *= 0.9 / np.max(np.abs(np.linalg.eigvals(W)))
    w_in = rng.normal(0, 1, N)
    u = rng.uniform(-1, 1, T + WASHOUT)
    x = np.zeros(N); S = np.empty((T, N))
    for t in range(T + WASHOUT):
        x = np.tanh(W @ x + w_in * u[t])
        if t >= WASHOUT:
            S[t - WASHOUT] = x
    S -= S.mean(axis=0)
    M = rng.normal(0, 1, (N, R_MODES)) / np.sqrt(N)
    B = np.eye(N) + COUPLING * (M @ rng.normal(0, 1, (R_MODES, N)) / np.sqrt(R_MODES))
    return S, B

def noise_A(rng):
    return rng.normal(0, SIGMA, (T, N))

def noise_B(rng, B, scale):
    return (rng.normal(0, 1, (T, N)) @ B.T) * scale

def eps_floor(draw, smax, rng):
    """PCTL-th percentile of rescaled largest noise SV over N_ACQ acquisitions."""
    tops = [sv(draw(rng))[0] / smax for _ in range(N_ACQ)]
    return float(np.percentile(tops, PCTL))

def trial(seed, want_curves=False):
    rng = np.random.default_rng(seed)
    S, B = device(rng)
    s_clean = sv(S); smax = s_clean[0]
    scaleB = SIGMA * np.sqrt(N / np.trace(B @ B.T))
    K = np.linalg.eigvalsh(B @ B.T); K_eff = K.max() / K.mean()

    efA = eps_floor(lambda r: noise_A(r), smax, rng)
    efB = eps_floor(lambda r: noise_B(r, B, scaleB), smax, rng)

    sA = sv(S + noise_A(rng))
    sB = sv(S + noise_B(rng, B, scaleB))
    r_eff = lambda ss, e: int(np.sum(ss >= e * smax))
    out = dict(K_eff=K_eff, efA=efA, efB=efB,
               A_honest=r_eff(sA, efA),
               B_naive=r_eff(sB, efA),
               B_honest=r_eff(sB, efB))
    if want_curves:
        eps = np.logspace(-4, 0, 200)
        out["curves"] = (eps,
                         [r_eff(sA, e) for e in eps],
                         [r_eff(sB, e) for e in eps])
    return out

# ----------------------------------------------------------------------
# Multi-seed table
# ----------------------------------------------------------------------
rows = [trial(s, want_curves=(s == FIG_SEED)) for s in range(N_SEEDS)]
def stat(key, fmt):
    v = [r[key] for r in rows]
    return f"{np.mean(v):{fmt}} [{min(v):{fmt}}, {max(v):{fmt}}]"

print(f"{N_SEEDS} seeds; eps_floor = p{PCTL} over {N_ACQ} noise acquisitions\n")
print(f"K_eff           : {stat('K_eff','6.1f')}")
print(f"eps_floor A     : {stat('efA','7.4f')}")
print(f"eps_floor B     : {stat('efB','7.4f')}")
print(f"A honest r_eff  : {stat('A_honest','5.0f')}")
print(f"B naive  r_eff  : {stat('B_naive','5.0f')}")
print(f"B honest r_eff  : {stat('B_honest','5.0f')}")
infl = [r["B_naive"] / r["B_honest"] for r in rows]
print(f"B naive/honest  : {np.mean(infl):.1f}x [{min(infl):.1f}, {max(infl):.1f}]")

# ----------------------------------------------------------------------
# Figure from FIG_SEED (currentColor SVG, dark-mode safe)
# ----------------------------------------------------------------------
fig = rows[FIG_SEED]
eps_grid, cA, cB = fig["curves"]

def to_path(xs, ys, x0, x1, y0, y1, w, h, pad):
    xs = (np.log10(xs) - np.log10(x0)) / (np.log10(x1) - np.log10(x0))
    ys = (np.array(ys) - y0) / (y1 - y0)
    px = pad + xs * (w - 2 * pad)
    py = h - pad - ys * (h - 2 * pad)
    return "M " + " L ".join(f"{a:.1f},{b:.1f}" for a, b in zip(px, py))

W_, H_, PAD = 640, 360, 50
x0, x1 = 1e-4, 1.0
pA = to_path(eps_grid, cA, x0, x1, 0, N, W_, H_, PAD)
pB = to_path(eps_grid, cB, x0, x1, 0, N, W_, H_, PAD)

def vline(eps, lbl, cls):
    px = PAD + (np.log10(eps) - np.log10(x0)) / (np.log10(x1) - np.log10(x0)) \
         * (W_ - 2 * PAD)
    return (f'<line class="{cls}" x1="{px:.1f}" y1="{PAD}" '
            f'x2="{px:.1f}" y2="{H_-PAD}"/>'
            f'<text class="lab" x="{px+5:.1f}" y="{PAD+14}">{lbl}</text>')

xticks = "".join(
    f'<text class="tick" x="{PAD + i*(W_-2*PAD)/4:.0f}" y="{H_-PAD+18}" '
    f'text-anchor="middle">10<tspan baseline-shift="super" font-size="8">'
    f'{e}</tspan></text>' for i, e in enumerate(range(-4, 1)))
yticks = "".join(
    f'<text class="tick" x="{PAD-8}" y="{H_-PAD - v/N*(H_-2*PAD)+4:.0f}" '
    f'text-anchor="end">{v}</text>' for v in (0, 50, 100, 150, 200))

svg = f'''<svg viewBox="0 0 {W_} {H_}" xmlns="http://www.w3.org/2000/svg"
     role="img" aria-label="Effective dimensionality versus epsilon threshold
     under detector-limited and correlated noise, with percentile-defined
     noise floors">
  <style>
    .ax {{ stroke: currentColor; stroke-width: 1; opacity: .5; }}
    .tick, .lab, .leg {{ font: 11px ui-monospace, monospace;
                         fill: currentColor; }}
    .cA {{ stroke: #4f8edb; stroke-width: 2; fill: none; }}
    .cB {{ stroke: #d96f32; stroke-width: 2; fill: none; }}
    .fA {{ stroke: #4f8edb; stroke-width: 1; stroke-dasharray: 4 3; }}
    .fB {{ stroke: #d96f32; stroke-width: 1; stroke-dasharray: 4 3; }}
  </style>
  <line class="ax" x1="{PAD}" y1="{H_-PAD}" x2="{W_-PAD}" y2="{H_-PAD}"/>
  <line class="ax" x1="{PAD}" y1="{PAD}" x2="{PAD}" y2="{H_-PAD}"/>
  {xticks}{yticks}
  <text class="tick" x="{W_/2}" y="{H_-8}" text-anchor="middle">
    threshold &#949; (relative to &#963;&#8321;)</text>
  <text class="tick" x="14" y="{H_/2}" text-anchor="middle"
        transform="rotate(-90 14 {H_/2})">r&#8337;&#8202;&#8202;(&#949;)</text>
  <path class="cA" d="{pA}"/>
  <path class="cB" d="{pB}"/>
  {vline(fig["efA"], "&#949;_floor p{PCTL} (det.)", "fA")}
  {vline(fig["efB"], "&#949;_floor p{PCTL} (corr.)", "fB")}
  <text class="leg" x="{W_-PAD-180}" y="{PAD+20}">
    &#9644; detector-limited (A)</text>
  <text class="leg" x="{W_-PAD-180}" y="{PAD+36}" fill="#d96f32">
    &#9644; correlated (B)</text>
</svg>'''.replace("{PCTL}", str(PCTL))

with open("/home/claude/cert-note/figure1.svg", "w") as f:
    f.write(svg)
print(f"\nfigure1.svg written (seed {FIG_SEED}: "
      f"efA={fig['efA']:.4f}, efB={fig['efB']:.4f}, "
      f"A={fig['A_honest']}, Bn={fig['B_naive']}, Bh={fig['B_honest']})")
