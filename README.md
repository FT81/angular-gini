# The Angular Gini

Code and numerical results for

> Schlemmer, M. (2026). *The Angular Gini: Weighting Pairwise Gini Contributions by Proportional Differences.* Preprint. [link to preprint]

The Angular Gini writes the Gini coefficient as a sum of pairwise contributions and scales each contribution by an angular measure of the proportional difference between the two incomes:

```
G_ang = sum_{i<j}  |y_i - y_j| / (n^2 * mu)  *  (2/pi) * | atan2(y_i, y_j) - atan2(y_j, y_i) |
```

For non-negative data it is bounded by the Gini coefficient and by (n-1)/n. The manuscript proves its properties; the notebooks in this repository reproduce every number, table and figure of the paper and serve as a numerical control of the proofs.

An earlier version of this work (arXiv:2110.13847v1) used the name *Vinci*; the code of that version is not part of this repository.

## Repository layout

```
angular-gini/
├── README.md
├── LICENSE
├── CITATION.cff
├── requirements.txt
├── ag_properties.ipynb          properties and counterexamples (Sections 1-4)
├── ag_comparison.ipynb          comparison with other indices (Sections 5.1-5.3)
├── ag_scf2025.ipynb             US net worth, SCF 2025 (Sections 4.4-4.5)
├── ag_wid2024_countries.ipynb   income of eight countries, WID (Section 5.5)
├── output/                      tables (csv, xlsx), figures (pdf, png) written by the notebooks
├── input/                       SCF files (download yourself, not included)
└── WID_data/                    WID country files (see Data)
```

The notebooks write their results to `output/` (relative to the notebook folder), so they have to be run from the repository root directory. The notebooks are independent of each other and can be run in any order. `ag_properties`, `ag_comparison` and `ag_wid2024_countries` are stored with their output, so that the results can be read on GitHub without running anything; `ag_scf2025.ipynb` is stored without output because its results are in `output/ag_scf2025_*.csv`.

## Which notebook produces what

| Notebook | Section | Tables and figures | Main output files |
|---|---|---|---|
| `ag_properties.ipynb` | 1, 2, 3, 4 | Table 1, Figure 1, Table 2 (all numerical checks), Table 4, Figure 2, Examples 1 and 2, the simulations in the Introduction and in Sections 2.4, 3 and 4 | `output/ag_properties_*` |
| `ag_comparison.ipynb` | 5.1 - 5.3 | Tables 6, 7 and 8, Figure 3 | `output/ag_comparison_*` |
| `ag_scf2025.ipynb` | 4.4, 4.5 | Table 5 and the SCF figures quoted in Section 4.4 | `output/ag_scf2025_*` |
| `ag_wid2024_countries.ipynb` | 5.5 | Table 9 | `output/ag_wid2024_*` |

Table 3 (status of the properties) is a summary of the proofs and not computed.

Details of `ag_comparison.ipynb`:

| Experiment | Content | Paper | Output |
|---|---|---|---|
| 1 | constructed series, n = 20, mean 100 | Table 6 | `ag_comparison_series.csv` |
| 2 | poorest decile multiplied by a factor, log-normal base, n = 200 | Table 8 (upper block) | `ag_comparison_bottom_decile.csv` |
| 2b | richest decile and richest 1 % multiplied by a factor | Table 8 (lower blocks) | `ag_comparison_top_decile.csv` |
| 3 | weight of a small transfer from rank k+1 to rank k | Figure 3(c) | `ag_comparison_transfer_weights.csv`, `ag_comparison_transfer_profiles.csv` |
| 4 | pairs of distributions with the same Gini coefficient | Table 7 | `ag_comparison_equal_gini.csv` |
| 5 | figure | Figure 3 | `ag_comparison_curves_bottom_decile.csv`, `ag_comparison_curves_top_decile.csv`, `ag_comparison_fig_sensitivity.pdf/.png` |

No random numbers are used in `ag_comparison.ipynb`. `ag_properties.ipynb` uses fixed seeds (2026; 77 for the classification of transfers in Section 4.3), so that the simulation results are reproducible.

## Data

**Survey of Consumer Finances 2025** (`ag_scf2025.ipynb`). The microdata are not included. Download the 2025 files from the [Federal Reserve Board](https://www.federalreserve.gov/econres/scfindex.htm) and place them in `input/`: `SCFP2025.csv` (summary extract, CSV), `scf2025x` (main data set, SAS transport file) and `scf2025rw1x` (replicate weights, SAS transport file). [SHA-256 checksums of the files used: ...]

**World Inequality Database** (`ag_wid2024_countries.ipynb`). The notebook reads `WID_data/WID_data_XX.csv` for the 15 countries AU, BR, DE, ES, FR, GB, IT, JP, NL, NO, PL, RU, SE, US, ZA (country pages of [wid.world](https://wid.world), bulk download) and keeps five variables: `aptinc992j`, `sptinc992j`, `adiinc992j`, `sdiinc992j`, `iquali999i`. WID series are revised regularly, so a later download can differ from the data used in the paper (downloaded on [date]). [The files reduced to these five variables are included in `WID_data/`.] The WID data are licensed under CC BY-NC-SA 4.0 (attribution: World Inequality Lab, World Inequality Database, https://wid.world); see `WID_data/LICENSE.md`.

The notebooks `ag_properties.ipynb` and `ag_comparison.ipynb` need no external data.

## Requirements

Python 3 with `numpy`, `pandas`, `scipy` and `matplotlib`; `mpmath` is needed only for the optional high-precision check of Example 1. The versions used are given in `requirements.txt`. To run the notebooks:

```
pip install -r requirements.txt
jupyter notebook
```

The runtime of the notebooks is [x] minutes (`ag_properties.ipynb`, the simulations dominate), [x] minutes (`ag_comparison.ipynb`), [x] minutes (`ag_scf2025.ipynb`, bootstrap) and [x] minutes (`ag_wid2024_countries.ipynb`) on [machine].

## Using the index

The index of a sample `y` (non-negative, mean > 0) is computed in the notebooks by a pairwise function; the computation requires all n(n-1)/2 pairs. A minimal implementation:

```python
import numpy as np

def angular_gini(y):
    y = np.asarray(y, float)
    n, mu = len(y), y.mean()
    d = np.abs(y[:, None] - y[None, :])
    w = (2 / np.pi) * np.abs(np.arctan2(y[:, None], y[None, :]) - np.arctan2(y[None, :], y[:, None]))
    return (d * w).sum() / (2 * n**2 * mu)      # sum over i<j = half of the sum over all ordered pairs
```

For y = (4, 8, 12, 36, 40) it gives 0.285 (Table 1 of the paper); the Gini coefficient is 0.400. Use the index for non-negative data only: for data with negative values the bound and the Pigou-Dalton principle can fail (Section 4 of the paper).

## Citation

If you use the index or the code, please cite the paper (see `CITATION.cff`). A DOI for the code archive is not yet available; until then please cite the release tag and the commit.

## License

Code: MIT License. Own results in `output/` (tables and figures computed by the notebooks): CC BY 4.0. The WID files in `WID_data/` remain under CC BY-NC-SA 4.0 (World Inequality Lab). The SCF data are not part of this repository and are subject to the terms of the Federal Reserve Board.
