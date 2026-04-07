# Multi-Target FMCW Radar Simulation with 4-Antenna Array

A Python-based simulation of a 77 GHz FMCW (Frequency-Modulated Continuous Wave) radar system featuring a 4-antenna receive array, full range-Doppler processing, and direction-of-arrival (DOA) estimation in both azimuth and elevation. Built as a hands-on implementation of the signal processing pipeline described in automotive radar literature, this project takes you from raw chirp generation all the way to a 3D target visualization.

---

## What This Project Does

The simulation models a realistic automotive-style radar scenario with multiple targets at different ranges, velocities, and angular positions. Here's the rough flow:

1. **Chirp generation** — A 77 GHz LFM transmit signal is synthesized and reflected off each target, with propagation delay and Doppler shift applied per chirp.
2. **4-antenna receive array** — Four receivers (Rx1–Rx4) are arranged to capture phase differences along the X, Y, and Z axes, enabling full 3D DOA estimation.
3. **Range-Doppler cube** — A 2D FFT (range + Doppler) is applied across all antennas, building a 4D data cube.
4. **Peak detection** — A noise-floor-adaptive threshold is used to pick out target peaks in the range-Doppler map, keeping only the strongest candidate per range bin.
5. **DOA estimation** — Phase differences between antenna pairs are used to extract azimuth and elevation angles. A phase-scan-based azimuth spectrum and MUSIC algorithm are both implemented.
6. **Visualization** — Multiple plots: signal waveforms, range FFT, range-Doppler map, azimuth/elevation spectra, and 2D + 3D target maps with velocity arrows.

---

## Radar Parameters

| Parameter | Value |
|---|---|
| Carrier Frequency | 77 GHz |
| Max Range | 50 m |
| Range Resolution | 1 m |
| Chirp Duration | 4 µs |
| FFT Size (Range) | 1024 |
| FFT Size (Doppler) | 1024 |
| Number of Rx Antennas | 4 |
| Antenna Spacing | λ/2 |

---

## Antenna Array Model

The four antennas are arranged to capture phase differences along three spatial axes:

- **Rx1** — reference antenna (no phase shift)
- **Rx2** — displaced along X → sensitive to `cos(el) * cos(az)`
- **Rx3** — displaced along Y → sensitive to `cos(el) * sin(az)`
- **Rx4** — displaced along Z → sensitive to `sin(el)`

Phase differences between antenna pairs are extracted after the range-Doppler FFT and used to estimate both azimuth and elevation through a direct phase method and a MUSIC-based spectral scan.

---

## Signal Processing Pipeline

```
Tx Chirp Generation
       ↓
Per-Chirp Target Propagation (range + Doppler update)
       ↓
4-Antenna Beat Signal Formation (stretch processing)
       ↓
Hamming Window → Range FFT (fast-time)
       ↓
Hamming Window → Doppler FFT (slow-time) + fftshift
       ↓
[4 × Nd × Nr] Range-Doppler Cube
       ↓
Peak Detection (noise floor + height threshold + dedup)
       ↓
Phase-Difference DOA (direct method)
       ↓
Azimuth Spectrum Scan + MUSIC
       ↓
2D and 3D Target Visualization
```

---

## Visualizations

The code generates the following plots:

**1. Signal Level View**
Raw Tx, Rx, and beat signal waveforms per target — useful for sanity-checking delays and beat frequencies.

**2. Range FFT Profile**
Averaged magnitude across chirps, showing range peaks with target labels annotated.

**3. Range-Doppler Map**
Full 2D range-velocity heatmap (in dB), with detected target positions marked in white.

**4. Azimuth + Elevation Spectra (per target)**
A 2×2 subplot layout showing:
- Phase-scan azimuth spectrum (0°–360°)
- MUSIC azimuth pseudo-spectrum
- Elevation spectrum (Rx1 vs Rx4 phase scan, ±90°)

**5. 2D Radar Map**
Top-down view with distance lines, target labels, and velocity arrows (orange = moving away, black = approaching).

**6. 3D Radar Map**
Full 3D scatter with range-elevation-azimuth positions, velocity arrows, and axis labels.

---

## How to Run

**Requirements:**
```
numpy
scipy
matplotlib
```

Install with:
```bash
pip install numpy scipy matplotlib
```

Run the simulation:
```bash
python radar_simulation.py
```

The script will print detected target estimates alongside ground truth values, then display all plots sequentially.

---

## Known Limitations / Areas of Active Work

- **180° azimuth ambiguity** — The phase difference between Rx1 and Rx2/Rx3 is symmetric, so targets at θ and θ+180° produce the same differential phase. The current implementation uses `arctan2` with a `% 2π` wrap to handle this, but it can misattribute azimuth for certain geometries.
- **Elevation coupling** — The `cos(el)` factor must be correctly propagated into the azimuth steering vector. Dropping it causes the azimuth scan peak to shift, especially at high elevation angles.
- **Single-cube DOA** — DOA is currently estimated from the combined multi-target cube rather than per-target cubes. This works well when targets are well-separated in range-Doppler but degrades in overlapping cases.
- **No CFAR** — Peak detection uses a simple mean+6σ threshold. A proper 2D CFAR would improve robustness in dense target environments.

---

## Project Structure

```
radar_simulation.py     # Main simulation script (all-in-one)
README.md
```

---

## Reference

This project was built with reference to the following paper on automotive radar signal processing:

> Bilik, I., Longman, O., Villeval, S., & Tabrikian, J. (2019). **The Rise of Radar for Autonomous Vehicles: Signal Processing Solutions and Future Research Directions.** *IEEE Signal Processing Magazine*, 36(5), 20–31.

The paper covers the full automotive radar signal processing chain — from LFM-CW waveform design and MIMO virtual aperture concepts to CFAR detection, DBSCAN clustering, and DOA estimation — and directly informed the structure of this simulation. Available on [ResearchGate](https://www.researchgate.net/publication/335745758_The_Rise_of_Radar_for_Autonomous_Vehicles_Signal_Processing_Solutions_and_Future_Research_Directions).

---

## Author

Pranam P Devadiga
