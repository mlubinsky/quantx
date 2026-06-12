# Kvant — Schrödinger Equation Solver

**Kvant** is an interactive educational application for solving the time-independent and time-dependent Schrödinger equation for a piecewise-constant (step-wise) potential. It was developed at Novosibirsk State University for teaching quantum mechanics.

This repository is a **macOS port** of the original project hosted at
[quantx.sourceforge.net](http://quantx.sourceforge.net).
The original README is preserved as [README.original](README.original).

---

## What the program does

- Defines a 1D piecewise-constant potential U(x) (step well, barrier, multi-well, parabolic approximation, linear bias, etc.)
- Computes bound-state energy levels E_n and wave functions ψ_n(x)
- Displays momentum distributions |φ_n(k)|²
- Simulates time evolution of wave packets Ψ(x,t) and Φ(k,t)
- Computes the transmission coefficient T(E) as a function of energy
- Handles complex potentials (quasi-levels, resonances E+iG)

All results are shown as interactive plots with zoom and pan.

Full documentation (in Russian): [documentation/kvant.pdf](documentation/kvant.pdf)

---

## Building on macOS (Apple Silicon / Intel)

### Requirements

| Tool | Version | Install |
|---|---|---|
| Xcode Command Line Tools | any recent | `xcode-select --install` |
| Homebrew | any | [brew.sh](https://brew.sh) |
| Qt | 6.9.x | `brew install qt` |

> Qt 6 is the supported build target. The original codebase targeted Qt 4.4;
> all Qt4/Qt5 APIs have been ported to Qt6 (see [CHANGELOG.md](CHANGELOG.md)).

### Build steps

```bash
# 1. Add Qt to PATH (Homebrew installs it keg-only)
export PATH="/opt/homebrew/opt/qt/bin:$PATH"

# 2. Clone the repository
git clone https://github.com/mlubinsky/quantx.git
cd quantx

# 3. Generate Makefile (strip the removed AGL framework from the output)
qmake kvant.pro && sed -i '' 's/-framework AGL//g' Makefile

# 4. Build
make -j$(sysctl -n hw.logicalcpu)

# 5. Run
open kvant.app
```

### Verified environment

- macOS 26.2 (Darwin 25, Apple M1 Pro)
- Apple clang 17.0.0 (arm64)
- Qt 6.9.0 (Homebrew)

---

## Project structure

| File / folder | Description |
|---|---|
| `*.cpp` / `*.h` | C++ source — UI views, physics model, controls |
| `kvant.pro` | qmake project file |
| `kvant.qrc` | Qt resource file (icons, images) |
| `kvant_en.ts` / `kvant_fr.ts` | Translation files (English / French, incomplete) |
| `documentation/kvant.pdf` | Full user documentation (Russian) |
| `CHANGELOG.md` | Porting history (Qt4 → Qt6, macOS fixes) |
| `README.original` | Original README from the SourceForge repository |
