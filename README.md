# Distinguishability of Dark-Matter Dephasing Mechanisms in LISA Observations

**Can a gravitational-wave detector tell *why* a black-hole binary is dephased — not just *that* it is?**

This repository contains the Fisher-matrix pipeline, validation suite, and figures for an
undergraduate research project asking whether LISA can *attribute* a detected dark-matter-induced
dephasing to a specific physical mechanism — accretion vs. dynamical friction — as opposed to
merely detecting that *some* environment is present.

> **Headline result:** the correlation coefficient between the two mechanism amplitudes is
> **r_AB ≈ −0.9999** for both benchmark LISA systems studied. The two mechanisms are almost
> perfectly degenerate: detection does not imply attribution.

<p align="center">
  <img src="fig4_ellipse.png" width="480" alt="Collapsed joint error ellipse for LISA-EMRI">
</p>

---

## Table of contents

- [The question](#the-question)
- [Why this hasn't been asked before](#why-this-hasnt-been-asked-before)
- [Physics summary](#physics-summary)
- [Repository structure](#repository-structure)
- [Quickstart](#quickstart)
- [Methodology](#methodology)
- [Validation](#validation)
- [Results](#results)
- [Limitations](#limitations)
- [References](#references)
- [Author](#author)
- [License](#license)

---

## The question

Existing literature forecasts whether LISA can **detect** a dark-matter environment around a
merging black-hole binary from the extra phase drift it imprints on the gravitational-wave
inspiral. Each analysis treats one environmental mechanism at a time — a single density parameter
in a single Fisher matrix — and reports a detection threshold.

This project asks the next question: **if a dephasing is detected, can the data tell you which
mechanism produced it?** Concretely, with two mechanisms — accretion ($\rho_A$) and dynamical
friction ($\rho_B$) — thrown into the *same* Fisher matrix as simultaneous free parameters, does
the off-diagonal correlation coefficient

$$r_{AB} = \frac{\Sigma_{\rho_A \rho_B}}{\sigma_{\rho_A}\sigma_{\rho_B}}$$

approach $\pm 1$? If it does, the two mechanisms are observationally degenerate: LISA can tell you
*an* environment is there, but not *which one*.

## Why this hasn't been asked before

The reason the degeneracy is severe turns out to be visible directly in the source physics. Both
mechanisms share the **same leading frequency dependence**, $f^{-16/3}$ (effective $-5.5$PN):

$$\Psi_{\rm acc} \propto \rho_A\, f^{-16/3}, \qquad
\Psi_{\rm df} \propto \rho_B\, f^{-16/3}\left[1 + \tfrac{304}{105}\ln\frac{f}{f^+_{\rm df}} + \dots\right]$$

The only difference is a slowly varying Coulomb-logarithm bracket. On a log–log plot the two
templates are nearly parallel lines — the entire distinguishing signature is a smooth ~2× drift
across the observing band (see `fig2_templates.png`). This project quantifies exactly how much
that residual shape difference is worth once you marginalize over the binary's own parameters.

## Physics summary

All dark-matter phase formulas are transcribed, with explicit equation numbers, from a single
source paper to guarantee notational consistency:

| Component | Formula | Source |
|---|---|---|
| Vacuum GR phase (1PN) | $\Psi_{\rm gw} = \frac{3}{128}x^{-5/3}\left[1+\dots\right]$ | Boudon et al., Eq. 4.16 |
| Halo self-gravity ($-3$PN) | $\Psi_{\rm halo} \propto \rho_0\, x^{-11/3}$ | Eq. 4.18 |
| Accretion ($-5.5$PN, high-$f$) | $\Psi_{\rm acc} \propto \rho_0\, x^{-16/3}$ | Eq. 4.19 |
| Dynamical friction ($-5.5$PN$+\ln$) | $\Psi_{\rm df} \propto \rho_0\, x^{-16/3}[1+\ln(\cdot)]$ | Eq. 4.20 |

with $x \equiv \pi G \mathcal{M} f / c^3$. The LISA sensitivity is the full analytic model of
Robson, Cornish & Liu (instrument noise + 4-yr galactic confusion), and the statistical machinery
is a standard SPA/Fisher-matrix treatment.

## Repository structure

```
.
├── dm_dephasing_full.py       # Main analysis pipeline (8 blocks, see Methodology)
├── dm_dephasing_colab.py      # Self-contained single-file version (analysis + figures, no local imports — for Google Colab)
├── make_figures.py            # Figure generation; imports from dm_dephasing_full.py
├── fig1_noise_bands.png       # LISA sensitivity + observation bands
├── fig2_templates.png         # Mechanism dephasing templates (the physical explanation)
├── fig3_rAB_vs_mass.png       # Robustness sweep: r_AB vs primary mass
├── fig4_ellipse.png           # Joint (ρ_A, ρ_B) error ellipse — the headline figure
├── fig5_rAB_vs_threshold.png  # Robustness sweep: r_AB vs friction-threshold placement
├── results_systems.csv        # Per-benchmark-system numerical results
├── sweep_mass.csv             # Mass-sweep raw data
├── sweep_fdfplus.csv          # Threshold-sweep raw data
├── results_summary.tex/.pdf   # Full write-up: conclusions, figure descriptions, validation argument
├── code_documentation.tex/.pdf# Block-by-block code documentation mapped to source equations
├── bao_cao_thuc_tap.tex/.pdf  # Internship report (Vietnamese, VNU-HUS format)
└── progress_update_deck.pptx  # Slide deck for a project progress talk
```

## Quickstart

Dependencies: `numpy`, `scipy`, `matplotlib` (standard scientific Python stack; no exotic
packages).

```bash
pip install numpy scipy matplotlib

# Full analysis: validation, benchmark systems, sweeps, CSV output
python3 dm_dephasing_full.py

# Figures (run after the line above, or use the self-contained script below)
python3 make_figures.py
```

**Google Colab / single-cell use:** paste or upload `dm_dephasing_colab.py` and run it directly —
it contains the entire pipeline and figure generation with no cross-file imports, and writes all
outputs (3 CSVs + 5 PNGs) to the working directory.

```python
# in a Colab cell, after uploading dm_dephasing_colab.py
!python3 dm_dephasing_colab.py

from IPython.display import Image, display
display(Image("fig4_ellipse.png"))
```

## Methodology

The pipeline is organized as eight blocks in a strict dependency chain — each block only uses
what the previous ones defined:

1. **Physical constants** — $G$, $c$, $F_\star = 0.66$ (critical accretion-flux constant); SI
   units throughout, converted to g/cm³ only when reporting densities.
2. **LISA noise model** — $S_n(f)$, implementing Robson–Cornish–Liu Eqs. (1), (10), (11), (14),
   including the 4-yr galactic confusion fit.
3. **`Binary` class** — chirp mass, observing band $[f_{\min}, f_{\max}]$, and the accretion /
   dynamical-friction regime thresholds (inverted from Boudon et al. Eqs. 5.5–5.8). These
   thresholds involve $\mu^{15}$ and $m_2^{21}$, which overflow float64 for high-mass systems, so
   they're computed in log space.
4. **Phase model** — the four `psi_*` functions implementing the table above, exactly.
5. **Fisher machinery** — builds $\Gamma_{ij}$ by trapezoidal quadrature on a 6000-point log
   frequency grid, then inverts with diagonal preconditioning ($C_{ij} = \Gamma_{ij}/\sqrt{\Gamma_{ii}\Gamma_{jj}}$) to
   control the condition number ($10^7$–$10^{11}$ for this near-degenerate problem).
6. **Per-system analysis** — for each benchmark: four single-mechanism $5\times5$ Fisher matrices
   (reproducing the standard detectability literature), then one *joint* $6\times6$ matrix with
   both $\rho_A$ and $\rho_B$ free simultaneously — this is the novel step.
7. **Parameter sweeps** — repeats the joint analysis across primary mass ($10^4$–$10^7\,M_\odot$)
   and across the dynamical-friction threshold placement, to test robustness.
8. **`validate()`** — defined last in the file, but **run first** at execution time: cross-checks
   every input against published values before any result is trusted.

Full block-by-block documentation with every equation traced to its source is in
[`code_documentation.pdf`](code_documentation.pdf).

## Validation

Every physics input is checked against a published value before results are reported:

| Check | This code | Reference |
|---|---|---|
| LISA noise floor √Sₙ @ 7 mHz | $1.18\times10^{-20}\,\mathrm{Hz^{-1/2}}$ | $\sim1.2\times10^{-20}$ (Robson et al., Fig. 3) |
| Ψ_halo/Ψ_gw @ 1 Hz (mass-independent) | $2.45\times10^{-8}$ | $\sim2\times10^{-8}$ (Boudon et al., Eq. 4.21) |
| MBBH detection threshold σ(ρ₀) | $6\times10^{-12}\,\mathrm{g/cm^3}$ | $\sim8\times10^{-13}$ (Boudon et al., Table IV) |
| $\sigma_{\rm joint}/\sigma_{\rm single}$ vs. $(1-r^2)^{-1/2}$ | 63.9 | 63.8 (exact algebraic identity) |

The MBBH threshold agrees only at the order-of-magnitude level; since
$\sigma(\rho_0)\propto f_{\min}^{16/3}$, a small difference in the noise model's low-frequency
cutoff between this analysis and the source paper's fully explains the gap — tighter agreement
isn't expected.

## Results

| System | SNR | $r_{AB}$ (marginalized) | Attribution penalty $(1-r^2)^{-1/2}$ |
|---|---|---|---|
| LISA-IMRI ($10^4+10\,M_\odot$) | 64 | **−0.99993** | ×82 |
| LISA-EMRI ($10^5+10\,M_\odot$) | 22 | **−0.99988** | ×64 |

- **Robust across mass:** $1-\lvert r_{AB}\rvert$ stays below $2\times10^{-4}$ across three decades
  of primary mass.
- **Robust across the friction-threshold fiducial:** even the most favorable placement of the
  Coulomb-log turnover only reaches $r_{AB}=-0.9995$ (still a ×33 penalty).
- **Not special to this pair:** a four-mechanism correlation matrix (halo, low-$f$ accretion,
  high-$f$ accretion, dynamical friction) shows $|r|>0.99$ for *every* pair, even mechanisms
  separated by 2.5 effective PN orders.
- **SNR-independent:** $r_{AB}$ doesn't depend on signal loudness in the Fisher formalism — a
  louder signal shrinks both individual errors but not their correlation.

Full discussion, all five figures with captions, and the "why this result is acceptable" argument
are in [`results_summary.pdf`](results_summary.pdf).

## Limitations

- Vacuum phase at 1PN, no spins; adding waveform parameters can only increase marginalized
  confusion, not decrease it.
- Dark-matter phase contributions are linearized in density (standard in this literature).
- Fisher-matrix (Gaussian, high-SNR) approximation — known to be least reliable exactly where it
  tends to *overestimate* distinguishability near a degeneracy, so a full Bayesian treatment would
  likely strengthen rather than weaken this conclusion.
- For the IMRI benchmark, one black hole's accretion threshold falls inside the observing band, so
  its accretion template includes only the secondary — noted explicitly in the code's printed
  output for every run.

## References

1. A. Boudon, P. Brax, P. Valageas, L. K. Wong, "Gravitational waves from binary black holes in a
   self-interacting scalar dark matter cloud," *Phys. Rev. D* **108**, 103517 (2023).
   [arXiv:2305.18540](https://arxiv.org/abs/2305.18540)
2. T. Robson, N. J. Cornish, C. Liu, "The construction and use of LISA sensitivity curves,"
   *Class. Quantum Grav.* **36**, 105011 (2019).
   [arXiv:1803.01944](https://arxiv.org/abs/1803.01944)
3. C. Cutler, É. E. Flanagan, "Gravitational waves from merging compact binaries: How accurately
   can one extract the binary's parameters from the inspiral waveform?," *Phys. Rev. D* **49**,
   2658 (1994).

## Author

Duong Ngoc Khoa — undergraduate physics research internship project.
<!-- TODO: add contact / institution / advisor line here if you'd like one -->

## License

<!-- TODO: pick a license before making the repo public — MIT is a common default for research
code with no proprietary constraints. Delete this comment once chosen. -->
