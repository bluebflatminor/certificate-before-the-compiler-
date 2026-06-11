# The Certificate Before the Compiler

A conformance schema for physical reservoir computing hardware.

**Status: v1.2 DRAFT — round-1 adversarial review AND self red-team folded.
NOT yet deployed. DOI verification incomplete.**

## Contents

- `index.html` — the note (single file, auto dark mode, inline SVG figure, CC0)
- `certificate_demo.py` — companion simulation v1.2 (NumPy only, fixed seeds).
  Reproduces every number and the figure in §6: `python3 certificate_demo.py`
- `index_v1.0.html.bak` — pre-review draft, retained for diff

## Thesis

Physical reservoir computing needs no programming language — learning is
confined to a linear readout, so the logical minimum interface is three calls.
What it lacks is the characterization certificate a device must return for
those calls to mean anything. v1.2 schema: a three-clause mandatory core with
a stated evaluation order — Clause 5 (replica consistency) → Clause 2
(correlated-noise floor, percentile-defined) → Clause 1 (ε-resolved effective
dimensionality, two input ensembles) — plus three annexes (memory,
nonlinearity, disorder/transferability).

## Version history

**v1.2 — self red-team (5 findings, all folded):**
1. ε_floor was a single-realization hard threshold on a fluctuating spectral
   edge → now percentile-defined (p95 over ≥20 noise-only acquisitions),
   in schema and simulation. Numbers survived the tightening.
2. Additive-noise assumption was load-bearing and unstated → now an explicit
   schema field (`additive_noise_assumed`) and stated scope in Clause 2;
   Clause 5 (replica consistency) PROMOTED to core as the instrument that
   sees dynamical noise, which is not SVD-separable from signal.
3. Single-ensemble certification is Goodhart-able (tune to the test) →
   core now requires two declared ensembles (i.i.d. + band-limited
   correlated), with the r_eff difference reported as input sensitivity.
   Toy still demonstrates one ensemble; stated as limitation in §6/§10.
4. No uncertainty quantification anywhere → core fields carry acquisition
   statistics; §6 reports mean [min, max] over 10 seeds instead of one.
5. Implicit clause dependency → mandatory evaluation order stated:
   consistency gates noise, noise gates dimensionality.

v1.2 headline numbers (10 seeds, p95/20-acquisition floors):
K_eff 27.0 [25.6, 29.6]; ε_floor 0.0044→0.0192; honest r_eff A 158 [140, 176]
vs B 35 [31, 42]; naive/honest inflation 3.5× [3.4, 3.8].

**v1.1 — round-1 adversarial review (DeepSeek, ChatGPT, Perplexity, Gemini):**
Clause 2 generalized (correlated noise as principle, Petermann as instance);
core/annex tiering introduced; overclaims softened ("never will", "no
interface exists", IEC/VAMAS gap → "I found no evidence"); three-calls
labeled logical minimum; falsification/conformance duality promoted to
abstract; §9 rewritten as roadmap + cross-substrate feasibility
precondition; §10(iv) added (certificate's own kill condition); Gemini
fold-ins (input_transfer sub-block, transfer class as statistical
invariance, Volterra Tier 1); refs [12], [13] added pending verification.

**Drift self-audit (between v1.0 triage and v1.1):** K_eff relabeled as
covariance anisotropy ratio (not a Petermann factor — Gemini had echoed the
loose framing back); convergence count corrected; one self-serving discard
reversed (toy vs. Petermann correlation structure now answered plainly in
§6); Gemini's "true Petermann matrix from FDTD" scoped down (see T2 note);
"Circular Dyson Ensembles" folded as principle, not name-drop.

## T2 MEEP scoped deliverable

The 2.5D axisymmetric run is block-diagonal in azimuthal number m: mode
non-orthogonality is computable only WITHIN each m-sector (Harminv-extracted
quasi-normal modes + unconjugated overlap integrals); cross-sector mixing
from non-axisymmetric scatterers is structurally invisible. T2 can deliver
per-sector lower bounds on non-orthogonality / covariance anisotropy —
sufficient as the Clause-2 physical anchor and to replace the toy's mixing
matrix — but NOT "the true Petermann matrix," which requires 3D (assessed
intractable).

## Pre-deployment checklist (MANDATORY)

- [x] Adversarial review round 1 complete and triaged
- [x] Drift self-audit between triage and revision
- [x] Self red-team of v1.1 → fixes folded into v1.2
- [x] Multi-seed variance for §6 headline numbers
- [ ] Every pending DOI below resolves to claimed title/authors (project rule)
- [ ] Bogaerts variability-framework citation inserted from Paper 4 reference
      list (still deliberately omitted rather than risk a wrong DOI)
- [ ] Decide: Xu et al. 2025 in §1 as differentiation, or out of scope
- [ ] Figure renders correctly in light and dark mode on iOS Safari
- [ ] Optional round-2 adversarial review of v1.2 before deployment

## Reference verification table

| # | Reference | DOI / locator | Status |
|---|-----------|---------------|--------|
| 1 | Trouvain et al. 2020, ReservoirPy, ICANN | 10.1007/978-3-030-61616-8_40 | VERIFIED (in-session) |
| 2 | PRCpy | arXiv:2410.18356 | VERIFIED (arXiv listing) |
| 3 | emucore-direct | pypi.org/project/emucore-direct | VERIFIED (PyPI) |
| 4 | Jaeger 2001, GMD Report 148 | tech report | verify report number |
| 5 | Maass et al. 2002, Neural Comp. 14(11) 2531 | 10.1162/089976602760407955 (expected) | **PENDING** |
| 6 | Roy & Vetterli 2007, EUSIPCO 606–610 | no DOI | VERIFIED (cited listing) |
| 7 | Petermann 1979, IEEE JQE QE-15, 566 | 10.1109/JQE.1979.1070064 (expected) | **PENDING** |
| 8 | Jaeger 2002, GMD Report 152 | tech report | verify report number |
| 9 | Dambre et al. 2012, Sci Rep 2, 514 | 10.1038/srep00514 | VERIFIED (nature.com) |
| 10 | Uchida et al. 2004, PRL 93, 244102 | 10.1103/PhysRevLett.93.244102 | VERIFIED (aps.org) |
| 11 | Tanaka et al. 2019, Neural Networks 115, 100–123 | 10.1016/j.neunet.2019.03.005 (expected) | **PENDING** |
| 12 | Petermann-factor sensitivity limit, Nat. Commun. 11 (2020) | 10.1038/s41467-020-15341-6 | DOI from URL; **author list PENDING** |
| 13 | Generalized Petermann factor at EPs (2025, APS) | unresolved (nonstandard APS DOI in source) | **PENDING — full verification** |

Do not deploy until each PENDING row resolves to the claimed title and
authors. (Lesson of Paper 4: three hallucinated citations caught
post-publication.)

## Known limitations (also §10 of the note)

1. Worked example is a tanh toy with a generic non-orthogonal mixing matrix;
   demonstrates clause logic, not cavity physics (§6 says so plainly).
2. Toy certifies one input ensemble; schema requires two.
3. Whether Clause 2 is binding for real devices is empirical and open.
4. Cross-substrate measurement comparability is an unestablished
   precondition, stated in §9.

## License

CC0 1.0 Universal. Public domain. No rights reserved.
