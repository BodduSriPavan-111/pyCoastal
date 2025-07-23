---
title: 'pyCoastal: a Python package for Coastal Engineering'
tags:
  - Python
  - numerical modeling
  - Coastal Engineering
  - Computational Fluid Dynamics
authors:
  - name: Stefano Biondi
    orcid: 0009-0001-5737-6012
    affiliation: "1" 

affiliations:
 - name: University of Florida, Gainesville, FL, United States
   index: 1

date: 17 July 2025
bibliography: paper.bib

# Summary

pyCoastal is an open-source Python framework for the numerical simulation of coastal hydrodynamics and scalar transport processes. It provides a modular and extensible platform for solving linear and nonlinear partial differential equations relevant to coastal systems, including wave propagation, pollutant dispersion, and viscous fluid dynamics. The framework supports structured 1D and 2D Cartesian grids, configurable through lightweight YAML files, and includes reusable components for grid generation, boundary condition management, numerical operators, and time integration schemes. Physical modules are implemented as standalone classes and can be easily composed or extended for prototyping and research. pyCoastal emphasizes clarity and reproducibility, with a strong focus on animated visualization, clean code structure, and pedagogical transparency. It is particularly suited for research prototyping, model intercomparison studies, and educational applications in coastal engineering, fluid mechanics, and numerical modeling.

# 1. Statement of need

Numerical modeling of coastal processes, such as wave propagation, shallow water flow, and pollutant transport, typically relies on specialized software frameworks that are often complex to configure, extend, or adapt to new applications. Tools like SWAN, ADCIRC, and OpenFOAM, although powerful, present significant barriers due to their steep learning curves and rigid internal structures. Studies have shown that more computationally demanding models do not necessarily yield higher accuracy, especially when simpler models are tuned effectively [@Lashley et al., 2020]. Furthermore, successful calibration of numerical models hinges on the ability to accurately represent key physical processes and structural features [@Simmons et al., 2017]. This complexity poses challenges both for researchers aiming to prototype models rapidly and for instructors seeking clear, demonstrable tools for teaching.

pyCoastal addresses this issue by offering a lightweight and modular coastal modeling framework fully in Python. It is designed to prioritize clarity and reproducibility, allowing users to define simulations through human-readable YAML configuration files and execute them with minimal setup. The codebase provides reusable components for grid generation, numerical operators, time integration schemes, and boundary condition handling, supporting both classical and custom physical models with ease. Its structure is designed to support both research applications and instructional use in topics such as coastal hydrodynamics, numerical modeling, and environmental fluid mechanics. Moreover, the framework integrates numerous established coastal‑engineering formulations—such as wave run‑up, sediment transport, and boundary layer calculations, enabling users to compute essential coastal parameters with ease. As a result, this module functions as a versatile library suitable for both academic research and industrial applications. In conclusion, pyCoastal offers a balance of flexibility and structure suitable for a range of academic and applied contexts.

# 2. Functionality

pyCoastal consists of several modular subcomponents designed that allow to choose for:
- Grid module: Creates 1D or 2D structured grids using `UniformGrid`
- Operator module: Finite-difference operators (gradient, divergence, Laplacian) for scalar and vector fields
- Physics modules: Implements shallow water equations, wave advection, pollutant transport, and viscous flow
- Boundary module: Defines reusable boundary conditions (Dirichlet, Neumann, Sponge, Wall)
- Visualization: Built-in support for animation using matplotlib
- Configuration system: YAML-based input files for fully parameterized, reproducible simulations
These components can be reused independently or combined to prototype new physical models or teach numerical methods.

# 3. Examples

The main purpose of this package is to provide a fully python based tool that allows to build, test and play with fluid dynamics numerical modeling. In the following section, some of the pre-built cases are explained :

### 3.1. Irregular wave generation

The `generate_irregular_wave` function builds a band-limited random wave time series based on standard oceanographic spectra:

```python
# -------------------------------------------------------------------
# 3) Generate wave boundary forcing 
# -------------------------------------------------------------------
from pyCoastal.tools.wave import generate_irregular_wave

t_vec, eta_bc = generate_irregular_wave(
    Hs=Hs, Tp=Tp,
    duration=duration,
    dt=dt,
    spectrum=spectrum_type,
    gamma=gamma
)
```

**3.1.1. Pierson–Moskowitz (PM)** 

This spectrum for a fully-developed sea [@Henrique et al., 2003] is defined as:

$$
S_{PM}(f) = \frac{5}{16}\,H_s^2\,f_p^4\,f^{-5}
  \exp\!\left[-\frac{5}{4}\!\left(\frac{f_p}{f}\right)^4\right]
$$

where:
- $\(S_{PM}(f)\)$ is the spectral energy density [m\(^2\)/Hz]  
- \(H_s\) is the significant wave height [m]  
- \(f_p\) is the peak frequency [Hz], with \(f_p = 1 / T_p\)  
- \(T_p\) is the peak wave period [s]  
- \(f\) is the frequency [Hz] 

**3.1.2 JONSWAP** 

This spectrum modifies the PM with a peaked enhancement factor [@Hasselmann et al., 1973] as:

$$
S_{J}(f) = S_{PM}(f)\;\gamma^{\displaystyle
  \exp\!\Bigl[-\frac{1}{2\sigma^2}\bigl(\frac{f}{f_p}-1\bigr)^2\Bigr]}
$$

where:
- \(S_{J}(f)\) is the JONSWAP spectral energy density [m\(^2\)/Hz]  
- \(\gamma\) is the peak enhancement factor (typically 3.3)  
- \(\sigma\) is the spectral width parameter, with \(\sigma = 0.07\) for \(f < f_p\) and \(\sigma = 0.09\) for \(f > f_p\)  

Once \(S(f)\) is defined, the surface elevation time series is:

$$
\eta(t) = \sum_{i} A_i \cos\bigl(2\pi f_i\,t + \phi_i\bigr)
$$

where:
- \(\eta(t)\) is the free-surface elevation at time \(t\) [m]  
- \(A_i\) is the wave amplitude for frequency \(f_i\) [m]  
- \(\phi_i\) is a random phase shift in \([0,2\pi]\)  
- \(\Delta f\) is the frequency resolution  

The amplitudes \(A_i\) are given by:

$$
A_i = \sqrt{2\,S(f_i)\,\Delta f}.
$$

To test it, run this example:

```bash
python examples/wave2D_irregular.py
```
Where the inputs for domain and wave generation are taken from a YAML file:

```yaml
grid:
  nx: 200
  ny: 200
  dx: 1.0
  dy: 1.0

physics:
  gravity: 9.81
  depth: 5.0

forcing:
  type: jonswap   # options: pm or jonswap
  gamma: 3.3
  Hs: 0.5         # significant wave height (m)
  Tp: 3.0         # peak period (s)

solver:
  dt: 0.1
  duration: 60.0

output:
  gauge: [100, 100]  # grid indices (i, j)
```

![Figure 1: Irregular wave field output of the example. The left subplot shows the upper view of the wave field (incoming from the south boundary). The side boundaries are set as free-slip conditions, while the northern boundary has a wave absorption condition. The subplot on the right shows the surface elevation over time at observation points.\label{fig:irregular}](media/wave2D_irregular_final.png){ width=600px }


### 3.2 2D Water Drop (Circular Wave Propagation)

This example demonstrates the classic 2D linear wave equation:

η<sub>t</sub> = c<sup>2</sup> ∇<sup>2</sup>η

The simulation includes; Zero-Dirichlet boundary conditions on all domain edges, ensuring waves vanish at the boundaries, a Gaussian hump as the initial condition, representing a localized disturbance ("water drop") at the domain center, and a second-order finite-difference scheme in both space and time:
  
η<sub>i+1</sub> = 2η − η<sub>i</sub> + (c &middot; Δt)<sup>2</sup> ∇<sup>2</sup>η

The output provides real-time animation, allowing users to visually observe expanding circular wavefronts and their reflections. Additionally, it is fully configurable via YAML, enabling easy adjustment of domain size, resolution, wave speed, CFL number, and simulation duration without modifying the code.

![Figure 2: Solution of the classic 2D linear wave equation. The colormap represents the water surface elevation.\label{fig:waterdrop}](media/water_drop_central_plot.png){ width=600px }

#### Run the Example

```bash
python examples/water_drop.py
```

# References
see paper.bib
