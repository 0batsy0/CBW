# Contact Binary Workbench (CBW)

CBW is a tool for fitting contact binary systems using multi-band light curves, with optional radial-velocity data. It combines fast neural-network predictions with global optimization and MCMC sampling to estimate system parameters and produce fitted curves.

## Available Versions

- **CBW.exe**: uses a fully connected neural network for radial-velocity predictions.
- **CBW_siren.exe**: uses a SIREN neural network for radial-velocity predictions.

Both versions use the same input format and workflow. Python and the required models are bundled into the executables; no Python installation is needed.

## Getting Started

1. Place at least one photometric CSV in `Input_Data`, using one file per band. Add `rv.csv` only if you want to include radial velocities.
2. Edit `Par_input.csv` to specify fixed and free parameters.
3. Edit `data.json` to set parameter bounds and sampling settings.
4. Run `CBW.exe` or `CBW_siren.exe` from its folder and wait for the calculation to finish.

The fitting mode is selected automatically:

| Input | Fitting mode |
|---|---|
| Photometric CSV files only | Light curves only; no RV likelihood is evaluated |
| Photometric CSV files plus `rv.csv` | Joint fit; light-curve and RV log-likelihoods are added |

No additional switch is needed. To fit light curves only, move `rv.csv` out of `Input_Data`; do not leave an empty RV file. Both executables support both modes. Use separate folders for different targets or executable versions to avoid overwriting results.

## Input Data

CSV files use the following columns:

```csv
Phase,Flux,Flux_err
0.10,0.92,0.01
0.20,0.98,0.01
```

- `Phase`: orbital phase. 
- `Flux`: normalized flux for light curves, or radial velocity in km/s for `rv.csv`.
- `Flux_err`: uncertainty in the corresponding measurement, in the same units as `Flux`. This column is optional.

Prepare phase-folded, normalized light curves before running the program. When uncertainties are supplied, the fit uses a Gaussian likelihood; otherwise, it uses unweighted residuals.

### Supported Photometric Bands

All 19 supported band names are listed below. CSV filenames must match these names exactly, followed by `.csv`; for example, `Johnson_R.csv`.

| Photometric system | Available band names |
|---|---|
| Cousins | `Cousins_I`, `Cousins_R` |
| Gaia | `Gaia_BP`, `Gaia_G`, `Gaia_RP` |
| Johnson | `Johnson_B`, `Johnson_I`, `Johnson_R`, `Johnson_V` |
| Kepler | `Kepler_mean` |
| SDSS | `SDSS_g`, `SDSS_i`, `SDSS_r`, `SDSS_z` |
| SuperWASP | `SuperWASP_post2004` |
| TESS | `TESS_T` |
| ZTF | `ZTF_g`, `ZTF_i`, `ZTF_r` |

Keep only the observation CSV files needed for the current fit in `Input_Data`.

## Parameter Settings

### Par_input.csv: Fixed and Free Parameters

Keep the header row unchanged and edit the values in the second row. **Use `-1` for a free parameter; any other value fixes that parameter.** Enter physical values without manual model normalization.

| Parameter | Meaning |
|---|---|
| `q` | Mass ratio |
| `t1` | Primary effective temperature, K |
| `incl` | Orbital inclination, degrees |
| `t2_t1` | Secondary-to-primary temperature ratio |
| `f` | Fill-out factor |
| `colat` | Spot colatitude, degrees |
| `long` | Spot longitude, degrees |
| `radius` | Spot angular radius, degrees |
| `relteff` | Relative spot temperature |
| `l3` | Third-light parameter |
| `sma` | Orbital semi-major axis, solar radii |
| `vgamma` | Systemic radial velocity, km/s |
| `p` | Orbital period, days |

For example, enter known values for `t1` and `p` to fix the temperature and period. Set `l3=0` to exclude third light. To disable the spot, use `colat=0`, `long=0`, `radius=0`, and `relteff=1`.

When `l3=-1`, third light is fitted separately for each band. There is no separate `err` fitting parameter; measurement uncertainties belong in the `Flux_err` column.

For light-curve-only fitting, fix `sma`, `vgamma`, and `p` to appropriate values: the normalized light-curve model does not constrain these parameters. Removing RV data does not automatically change your fixed-parameter settings.

### data.json: Search Bounds and Sampling

- Set physical parameter bounds as `[minimum, maximum]`, for example, `"q": [0.1, 0.9]`. Bounds apply to free parameters.
- `use_shade`: `true` selects SHADE global optimization; `false` selects global MCMC. Both are followed by local MCMC.
- `SHADE_population`, `SHADE_generations`: SHADE population size and number of generations.
- `MCMC1_nwalkers`, `MCMC1_run_times`: walker count and iteration count for global MCMC.
- `MCMC2_nwalkers`, `MCMC2_run_times`: walker count and iteration count for local MCMC.

Other settings can initially remain unchanged. Use at least twice as many walkers as free parameters. More iterations increase runtime and do not, by themselves, guarantee convergence.

Update the temperature, period, and parameter bounds for your target rather than reusing the example values. Apply any phase shift to the input data beforehand; the current executables do not apply the `phase_shift` setting automatically.

## Output Files

| File | Contents |
|---|---|
| `result.csv` | Parameter summary, uncertainty columns, and fit statistics |
| `lc.jpg` | Observed and fitted light curves |
| `rv.jpg` | Observed and fitted radial-velocity curves, only when `rv.csv` is supplied |
| `step1_local.h5` | Local MCMC sampling chain |
| `step1_global.h5` | Global MCMC chain, generated only when global MCMC is used |
| `progress.csv`, `out.csv` | Intermediate model-evaluation records |

Start by inspecting `result.csv` and `lc.jpg`; joint fits also produce `rv.jpg`. A new run removes old fitting outputs, including any previous RV plot. Back up results before starting another run.

## Citation

If you use our models in your research, please cite both of the following papers:

1. *A Neural Network–Based Tool for Rapid Orbital Solution of Contact Binary Systems*
2. *A Neural-network Model for Quickly Solving Multiple-band Light Curves of Contact Binaries*

