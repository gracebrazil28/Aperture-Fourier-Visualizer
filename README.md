# Antenna Aperture as a Fourier Transform

An interactive, single-file visualizer showing how the illumination across a circular antenna aperture determines its far-field radiation pattern. Drag the diameter slider or switch the illumination taper, and the beam, nulls and sidelobes update in real time.
Sample view here: https://gracebrazil28.github.io/Diffraction-Theory-Visualizer/

## Features

- **Adjustable aperture diameter** from 3λ to 50λ, in steps of 1λ.
- **Two illumination profiles:**
  - **Uniform**: constant amplitude across the aperture.
  - **Tapered**: parabolic (1 − r²) amplitude taper, falling to zero at the rim.
- **Three linked views:**
  - **Aperture illumination**: the aperture drawn to scale, with a one-wavelength scale bar.
  - **2D far field**: the radiation pattern over ±30° in both axes, on a 40 dB color scale.
  - **Power pattern cut**: a 1D slice through the beam, in linear or dB scale. The linear view includes a dashed half-power (−3 dB) reference line.
- **Live metrics:**
  - Half-power beamwidth (HPBW), in degrees and in units of λ/D
  - First-null angle, in degrees and in units of λ/D
  - First sidelobe level, in dB relative to the peak


## Getting Started

There are no dependencies and no build step. Open the HTML file (for example `index.html`) in any modern browser:

```bash
# macOS
open index.html

# Linux
xdg-open index.html

# Windows
start index.html
```

You can also serve it locally:

```bash
python3 -m http.server 8000
# then visit http://localhost:8000
```

## Usage

| Control         | Effect                                                    |
|-----------------|-----------------------------------------------------------|
| Diameter slider | Sets the aperture diameter D in wavelengths (3λ–50λ)      |
| Uniform / Tapered | Switches the illumination profile                       |
| Linear / dB     | Switches the y-axis scale of the power pattern cut        |

Things to try:

- Double the diameter and watch the beamwidth halve. Beamwidth scales as λ/D.
- Switch from Uniform to Tapered. The main beam gets wider, but the sidelobes drop by about 7 dB.
- Use the dB view to see the sidelobe structure, which is nearly invisible on a linear scale.

## Theory

In the far field, the radiation pattern of an aperture is the 2D Fourier transform of its illumination. For a circularly symmetric aperture, this reduces to a Hankel transform, which gives closed-form results in terms of Bessel functions.

With the normalized angular variable

```
x = π (D/λ) sin θ
```

the normalized far-field amplitude patterns are:

| Illumination      | Amplitude pattern F(x) | HPBW       | First null   | First sidelobe | Aperture efficiency |
|-------------------|------------------------|------------|--------------|----------------|---------------------|
| Uniform           | 2 J₁(x) / x            | ≈ 1.03 λ/D | ≈ 1.22 λ/D   | ≈ −17.6 dB     | 100%                |
| Tapered (1 − r²)  | 8 J₂(x) / x²           | ≈ 1.27 λ/D | ≈ 1.63 λ/D   | ≈ −24.6 dB     | 75%                 |

The power pattern is |F(x)|². The uniform pattern is the Airy pattern familiar from optics, and its first null at 1.22 λ/D is the classical Rayleigh resolution limit.

This illustrates the central trade-off in aperture design. Tapering the illumination suppresses sidelobes, but it widens the main beam and reduces gain, because the outer part of the aperture contributes less.

## Implementation Notes

- **Bessel functions:** J₀ and J₁ use the rational and asymptotic approximations from *Numerical Recipes*. J₂ comes from the recurrence J₂(x) = 2J₁(x)/x − J₀(x).
- **Small-argument handling:** the uniform pattern returns 1 at x ≈ 0. The tapered pattern uses the series 1 − x²/12 for x < 0.3, which avoids cancellation error in J₂(x)/x².
- **Lookup table:** both patterns are precomputed once on a grid with Δx = 0.005. That keeps the per-pixel 2D far-field rendering fast enough for smooth slider interaction.
- **Metrics:** values are found by scanning the lookup table:
  - HPBW is where |F|² first falls below 0.5.
  - The first null is the first sign change of F.
  - The first sidelobe is the peak of |F|² between the first and second nulls.

  Because the scan runs on the discrete grid, results are accurate to a few thousandths of λ/D.
- **Display scaling:**
  - The far-field map spans direction cosines up to sin 30° and clips at −40 dB.
  - The dB plot is also floored at −40 dB, so deep nulls appear flat at the bottom.


## Assumptions and Limitations

- The model is scalar, far-field (Fraunhofer) diffraction from a planar circular aperture with uniform phase.
- Polarization, the feed or element pattern, obliquity factors, blockage and surface errors are not modeled.
- The 2D far-field map assumes rotational symmetry. It uses the same 1D pattern evaluated at √(u² + v²).
- Only the two illumination profiles above are included. Other tapers, such as (1 − r²)ⁿ on a pedestal, would be natural extensions.

 
