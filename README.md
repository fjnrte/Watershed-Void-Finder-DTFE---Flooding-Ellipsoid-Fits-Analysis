# Cosmic Voids in Illustris-3-Dark

**Construction and analysis of a Watershed Void Finder pipeline for void evolution, morphology, and density profiles.**

This repository contains the computational work behind my thesis project on cosmic voids in the **Illustris-3-Dark** dark-matter-only simulation. The project builds a full Watershed Void Finder (WVF) pipeline: starting from dark-matter particle snapshots, reconstructing a density grid, smoothing and discretizing it, identifying void origins, applying hierarchy-controlled watershed flooding, and analysing the resulting void catalogue through densities, volumes, ellipsoid morphology, fit quality, centre definitions, and radial density profiles.

<p align="center">
  <img src="figures/8___Appendix/Frame__Z_ALL_0.png" width="90%" alt="WVF density-grid and wall output across redshift, part I">
</p>

---

## Project summary

Cosmic voids are the large underdense basins of the cosmic web. Since real voids are irregular, hierarchical, and strongly affected by their wall/filament environment, this project uses a **topology-based watershed method** rather than assuming spherical void shapes during identification.

The main methodological contribution is a modified WVF implementation, called **MK2**, which improves two parts of the baseline flooding logic:

1. **connected minima origins**, where a void can begin from an equal-level minimum patch rather than only from one isolated cell;
2. **equal-level patch flooding**, where same-density bridges between basins are treated geometrically instead of being assigned in an arbitrary cell order.

At $z=0$, under the same density-grid and threshold setup, the baseline **MK1** implementation gives **142** final voids, while **MK2** gives **370** final voids. This shows that grid-level morphology treatment has a large effect on the final catalogue.

---

## Data basis

The analysis uses the **Illustris-3-Dark** simulation, the low-resolution dark-matter-only counterpart of the Illustris suite.

**Simulation:** Illustris-3-Dark
**Box side length:** 75 cMpc/h
**Number of dark-matter particles:** 455³
**Dark-matter particle mass:** approximately 0.034 × 10¹⁰ M☉/h
**Mean interparticle separation:** approximately 0.165 cMpc/h
**Main analysis grid:** 512³ cells
**Main density variable:** 1 + δ = ρ / ρ̄

The raw Illustris snapshot files are **not included** in this repository. They should be downloaded separately and kept locally in `Original_Data/`.


---

## Pipeline overview

```mermaid
flowchart LR
    A[Illustris-3-Dark particles] --> B[Density reconstruction]
    B --> C[DTFE grid]
    C --> D[Smoothing]
    D --> E[CDF density levels]
    E --> F[Void origins and hierarchy thresholds]
    F --> G[MK1 / MK2 WVF flooding]
    G --> H[Final WVF catalogue]
    H --> I[Densities, volumes, ellipsoids, profiles]
```

### 1. Density reconstruction

The simulation stores particles, not a continuous field. The first stage therefore reconstructs the density field on a periodic grid. I compare a simple **Nearest Grid Point** assignment with a **Delaunay Tessellation Field Estimator** approach and use the DTFE-style reconstruction for the main WVF input because it better preserves the connected morphology of walls, filaments, and underdense regions.

<p align="center">
  <img src="figures/3___The_Void_Finder_Algorithm_And_Direct_Intermediary_Results_Analysis/2.3___DTFE_vs_NGP_cut.png" width="86%" alt="DTFE versus NGP density reconstruction">
</p>

### 2. Smoothing and level construction

The reconstructed field is smoothed before flooding. The adopted sequence combines local filtering, greyscale morphological filtering, and Fourier-space Gaussian smoothing. The smoothed density field is then converted into discrete density levels, mainly using a CDF-based level mapping.

<p align="center">
  <img src="figures/8___Appendix/3.1___Original_vs_Smooth.png" width="86%" alt="Original, smoothed and residual DTFE density field">
</p>

<p align="center">
  <img src="figures/3___The_Void_Finder_Algorithm_And_Direct_Intermediary_Results_Analysis/4.1___Levels_as_density.png" width="86%" alt="Density to level mapping for logarithmic and CDF levels">
</p>

### 3. Void origins and hierarchy thresholds

Void origins are identified from local minima of the leveled density grid. The WVF hierarchy is then controlled by two thresholds: one for void-in-void merging and one for void-in-cloud suppression/absorption. These thresholds are tracked consistently across redshift through the grid and origin density distributions.

<p align="center">
  <img src="figures/3___The_Void_Finder_Algorithm_And_Direct_Intermediary_Results_Analysis/6.1___Grid_and_origin_density_profiles_no_CDFs.png" width="86%" alt="Grid and origin density profiles with hierarchy thresholds">
</p>

### 4. MK2 watershed flooding

The final finder floods the grid level by level. MK1 treats cells individually, while MK2 separates each level into isolated cells and connected equal-level patches. Patch cells are then assigned through a peeling logic that reduces dependence on arbitrary ordering.

<p align="center">
  <img src="figures/3___The_Void_Finder_Algorithm_And_Direct_Intermediary_Results_Analysis/pseudocode_skeleton.png" width="86%" alt="Global pseudocode skeleton of 7___Finder.ipynb">
</p>

<p align="center">
  <img src="figures/3___The_Void_Finder_Algorithm_And_Direct_Intermediary_Results_Analysis/pseudocode_functions.png" width="86%" alt="Function-level pseudocode of the WVF finder">
</p>

<p align="center">
  <img src="figures/3___The_Void_Finder_Algorithm_And_Direct_Intermediary_Results_Analysis/code_logic.jpeg" width="86%" alt="MK2 equal-level patch decision logic">
</p>

The final result is a space-filling WVF catalogue: void basins are labelled by origin index, while separating walls/filaments are stored as wall cells.

---

## Catalogue-level results

The first catalogue observables are the void-density and void-volume distributions across redshift. The low-redshift WVF catalogue is much more mature than the high-redshift one: void interiors become more clearly underdense, while volumes evolve under the combined effect of void-in-void merging and void-in-cloud erasure.

<p align="center">
  <img src="figures/4___Data_Analysis_Densities_And_Volumes/HIST_COL_voids_densities_mean.png" width="86%" alt="Mean WVF void-density distributions across redshift">
</p>

<p align="center">
  <img src="figures/4___Data_Analysis_Densities_And_Volumes/HIST_COL_voids_densities_median.png" width="86%" alt="Median WVF void-density distributions across redshift">
</p>

<p align="center">
  <img src="figures/4___Data_Analysis_Densities_And_Volumes/HIST_COL_voids_volumes.png" width="86%" alt="WVF void-volume distributions across redshift">
</p>

In the thesis run, the number of final voids reaches **504** around $z\simeq2$ and decreases to **370** by $z=0$. At $z=0$, the mean-density distribution peaks around $\langle1+\delta\rangle_i\simeq0.515$, while the median-density distribution peaks around $\simeq0.336$, indicating underdense interiors whose mean densities are lifted by denser outskirts and near-wall structure.

---

## Density evolution between snapshots

The project also computes an Eulerian density time derivative between consecutive snapshots. This compares the density field at fixed comoving grid cells; it does not track individual dark-matter particles.

<p align="center">
  <img src="figures/4___Data_Analysis_Densities_And_Volumes/HIST_COL_voids_densities_time_deriv.png" width="86%" alt="Grid-cell density time-derivative histogram across redshift intervals">
</p>

<p align="center">
  <img src="figures/4___Data_Analysis_Densities_And_Volumes/Frame__120.png" width="86%" alt="Density time-derivative map at high redshift">
</p>

<p align="center">
  <img src="figures/4___Data_Analysis_Densities_And_Volumes/Frame__00.png" width="86%" alt="Density time-derivative map at z=0">
</p>

At late times, the strongest density changes concentrate near walls, filaments, and their intersections, while many large void interiors show weaker net density evolution.

---

## Ellipsoid morphology

WVF voids are irregular watershed basins, not ellipsoids. However, fitting ellipsoids to eligible void boundaries gives a compact way to describe their large-scale morphology: centre, semi-axes, axis ratios, and orientation.

<p align="center">
  <img src="figures/4___Data_Analysis_Densities_And_Volumes/scrreenshot_1.png" width="32%" alt="Example WVF void with fitted ellipsoid 1">
  <img src="figures/4___Data_Analysis_Densities_And_Volumes/scrreenshot_2.png" width="32%" alt="Example WVF void with fitted ellipsoid 2">
  <img src="figures/4___Data_Analysis_Densities_And_Volumes/scrreenshot_3.png" width="32%" alt="Example WVF void with fitted ellipsoid 3">
</p>

<p align="center">
  <img src="figures/4___Data_Analysis_Densities_And_Volumes/11.2___hist_3_color_plot.png" width="86%" alt="Ellipsoid semi-axis and axis-ratio distributions across redshift">
</p>

At the present epoch, the average fitted shape is approximately

$$
A:B:C\simeq1:0.56:0.32,
$$

so the mature WVF voids in this catalogue are strongly triaxial rather than mildly perturbed spheres.

<p align="center">
  <img src="figures/4___Data_Analysis_Densities_And_Volumes/ellipsoid___00.png" width="32%" alt="Average ellipsoid shape at z=0">
  <img src="figures/4___Data_Analysis_Densities_And_Volumes/ellipsoid___30.png" width="32%" alt="Average ellipsoid shape at z=3">
  <img src="figures/4___Data_Analysis_Densities_And_Volumes/ellipsoid___120.png" width="32%" alt="Average ellipsoid shape at z=12">
</p>

---



### Average ellipsoid shapes across redshift

The ellipsoid fits are not meant to replace the WVF voids with smooth analytic objects.
They are used as compact global descriptors of the voids' dominant extent, orientation, and triaxiality.

Visually, the fits behave as intended: they follow the large-scale shape and orientation of the WVF voids, while the voids themselves remain irregular.
The WVF regions contain protrusions, indentations, and rough local boundaries, looking more like wrinkled ellipsoidal objects than clean mathematical ellipsoids.
A perfectly smooth ellipsoidal boundary would be suspicious here, since the WVF is a space-filling grid segmentation.
The irregularities are therefore expected local watershed morphology, while the ellipsoid fits capture the dominant global shape.

The average fitted ellipsoid shape was also plotted at three representative redshifts: `z = 0`, `z = 3`, and `z = 12`.

<table>
<tr>
<td align="center"><img src="figures/4___Data_Analysis_Densities_And_Volumes/ellipsoid___00.png" width="100%"></td>
<td align="center"><img src="figures/4___Data_Analysis_Densities_And_Volumes/ellipsoid___30.png" width="100%"></td>
<td align="center"><img src="figures/4___Data_Analysis_Densities_And_Volumes/ellipsoid___120.png" width="100%"></td>
</tr>
<tr>
<td align="center"><b>z = 0</b></td>
<td align="center"><b>z = 3</b></td>
<td align="center"><b>z = 12</b></td>
</tr>
</table>

From `z = 12` to `z = 3`, the average ellipsoid becomes visibly thinner along its shortest axis.
The ratio `C/A` decreases from roughly `0.42` to `0.39`, while `B/A` changes only mildly, from roughly `0.62` to `0.60`.
This points more toward a stronger squeezing of the minor axis than toward a simple stretching of the major axes.

By the present epoch, the average shape is approximately

```text
A : B : C ≈ 1 : 0.56 : 0.32
```

This is not a mildly perturbed sphere.
It is a strongly triaxial average void: long in one direction, significantly narrower in the second, and much more compressed in the third.

An optional supplementary animation of this evolution can be added as:

```text
supplementary/ellipsoid_video.mp4
```

---

## Fit quality and centre definitions

Because WVF basins are not smooth, ellipsoid fits are tested against the original grid-cell voids using three overlap scores:

$$
\mathrm{coverage}_i=\frac{|G_i\cap E_i|}{|G_i|},
\qquad
\mathrm{purity}_i=\frac{|G_i\cap E_i|}{|E_i|},
\qquad
\mathrm{IoU}_i=\frac{|G_i\cap E_i|}{|G_i\cup E_i|}.
$$

Here $G_i$ is the WVF void-cell set and $E_i$ is the rasterized fitted ellipsoid. At $z=0$, the catalogue contains **370** voids, **346** finite ellipsoid fits, and **24** reliable profile candidates after imposing $\mathrm{IoU}\geq0.6$.

<p align="center">
  <img src="figures/8___Appendix/12.2___coverage_purity_IoU___scores.png" width="86%" alt="Coverage purity and IoU score distributions">
</p>

<p align="center">
  <img src="figures/4___Data_Analysis_Densities_And_Volumes/12.1___coverage_purity_IoU__vs_maxradii.png" width="86%" alt="Coverage purity and IoU versus maximum fitted radius">
</p>

The centre choice is also important. The minimum-density cell is usually not the same as the morphology-based centre, while the ellipsoid centre agrees much more closely with the grid-average centre.

<p align="center">
  <img src="figures/4___Data_Analysis_Densities_And_Volumes/12.1___difference_in_centers.png" width="86%" alt="Distance distributions between centre definitions">
</p>

---

## Radial density profiles

For the selected $\mathrm{IoU}\geq0.6$ sample, radial profiles are extracted around morphology-based centres and compared in physical and rescaled coordinates.

<p align="center">
  <img src="figures/4___Data_Analysis_Densities_And_Volumes/13.1___A_few_profiles___mindensity_and_ellipsoid_sel_origins.png" width="86%" alt="Selected void profiles with minimum-density and ellipsoid centres">
</p>

<p align="center">
  <img src="figures/4___Data_Analysis_Densities_And_Volumes/13.2___pixels___all_voids_nonscaled_and_scaled_sel___gaussian_sigma_20.png" width="86%" alt="Gaussian-smoothed selected WVF void profile maps">
</p>

The profile analysis shows that the geometrical centre of a WVF void is not generically the minimum of its radial density profile. In the reliable $z=0$ sample, roughly one third of the selected voids show a small central density excess relative to the nearby radial minimum.

---

## Repository structure

A clean public repository should keep the code and selected figures, but not the raw simulation data or heavy intermediate `.pk` outputs.

```text
.
├── README.md
├── requirements.txt
├── LICENSE
├── .gitignore
├── PYTHON_CODES/
│   ├── 0___Reference.ipynb
│   ├── ALL_RUN.ipynb
│   ├── 1___Nearest_Grid_Point.ipynb
│   ├── 1___DTFE___KDTree.ipynb
│   ├── 1___DTFE___Density_Loop.ipynb
│   ├── 2___Smoothing.ipynb
│   ├── 3___Layer.ipynb
│   ├── 4___UOD_vals_and_lvls.ipynb
│   ├── 5___Origins.ipynb
│   ├── 6___Levels_isolated_pairs.ipynb
│   ├── 7___Finder.ipynb
│   └── analysis_notebooks/
├── figures/
│   ├── 3___The_Void_Finder_Algorithm_And_Direct_Intermediary_Results_Analysis/
│   ├── 4___Data_Analysis_Densities_And_Volumes/
│   └── 8___Appendix/
├── supplementary/
│   ├── perc__buildup_video.mp4
│   ├── Z_walls_and_grid_video.mp4
│   ├── Time_derivative_video.mp4
│   └── ellipsoid_video.mp4
└── data/
    └── README.md
```

Recommended rule: keep `Original_Data/`, `Modified_Data_512/`, `Analysis___General_Data/`, and `Analysis___Ellipsoid_Data/` local unless you intentionally publish a small cleaned sample.

---

## Running the pipeline

1. Download the required Illustris-3-Dark snapshots into a local `Original_Data/` folder.
2. Set paths, selected redshifts, grid size, smoothing scale, level count, hierarchy thresholds, and MK method in `PYTHON_CODES/0___Reference.ipynb`.
3. Run the full workflow from `PYTHON_CODES/ALL_RUN.ipynb`, or run the notebooks manually in numerical order.
4. Reuse saved DTFE grids and same-level cell dictionaries where possible, since these are among the most expensive intermediate products.
5. Run the analysis notebooks to generate the density/volume histograms, time-derivative maps, ellipsoid fits, fit-quality scores, centre comparisons, and radial profiles.

The full pipeline is computationally heavy. The thesis run was designed around reusable intermediate files, memory-aware processing, and parallel execution across independent parameter choices.

---

## Figure files used in this README

Copy the following thesis figures into the same paths under `figures/`.

| README path | Thesis/source figure |
|---|---|
| `figures/8___Appendix/Frame__Z_ALL_0.png` | `8___Appendix/Frame__Z_ALL_0.png` |
| `figures/8___Appendix/Frame__Z_ALL_1.png` | `8___Appendix/Frame__Z_ALL_1.png` |
| `figures/3___The_Void_Finder_Algorithm_And_Direct_Intermediary_Results_Analysis/2.3___DTFE_vs_NGP_cut.png` | same filename |
| `figures/8___Appendix/3.1___Original_vs_Smooth.png` | same filename |
| `figures/3___The_Void_Finder_Algorithm_And_Direct_Intermediary_Results_Analysis/4.1___Levels_as_density.png` | same filename |
| `figures/3___The_Void_Finder_Algorithm_And_Direct_Intermediary_Results_Analysis/6.1___Grid_and_origin_density_profiles_no_CDFs.png` | same filename |
| `figures/3___The_Void_Finder_Algorithm_And_Direct_Intermediary_Results_Analysis/pseudocode_skeleton.png` | `pseudocode_skeleton`, exported/renamed as PNG |
| `figures/3___The_Void_Finder_Algorithm_And_Direct_Intermediary_Results_Analysis/pseudocode_functions.png` | `pseudocode_functions`, exported/renamed as PNG |
| `figures/3___The_Void_Finder_Algorithm_And_Direct_Intermediary_Results_Analysis/code_logic.jpeg` | same filename |
| `figures/4___Data_Analysis_Densities_And_Volumes/HIST_COL_voids_densities_mean.png` | same filename |
| `figures/4___Data_Analysis_Densities_And_Volumes/HIST_COL_voids_densities_median.png` | same filename |
| `figures/4___Data_Analysis_Densities_And_Volumes/HIST_COL_voids_volumes.png` | same filename |
| `figures/4___Data_Analysis_Densities_And_Volumes/HIST_COL_voids_densities_time_deriv.png` | same filename |
| `figures/4___Data_Analysis_Densities_And_Volumes/Frame__120.png` | same filename |
| `figures/4___Data_Analysis_Densities_And_Volumes/Frame__00.png` | same filename |
| `figures/4___Data_Analysis_Densities_And_Volumes/scrreenshot_1.png` | same filename |
| `figures/4___Data_Analysis_Densities_And_Volumes/scrreenshot_2.png` | same filename |
| `figures/4___Data_Analysis_Densities_And_Volumes/scrreenshot_3.png` | same filename |
| `figures/4___Data_Analysis_Densities_And_Volumes/11.2___hist_3_color_plot.png` | same filename |
| `figures/4___Data_Analysis_Densities_And_Volumes/ellipsoid___00.png` | same filename |
| `figures/4___Data_Analysis_Densities_And_Volumes/ellipsoid___30.png` | same filename |
| `figures/4___Data_Analysis_Densities_And_Volumes/ellipsoid___120.png` | same filename |
| `figures/8___Appendix/12.2___coverage_purity_IoU___scores.png` | same filename |
| `figures/4___Data_Analysis_Densities_And_Volumes/12.1___coverage_purity_IoU__vs_maxradii.png` | same filename |
| `figures/4___Data_Analysis_Densities_And_Volumes/12.1___difference_in_centers.png` | same filename |
| `figures/4___Data_Analysis_Densities_And_Volumes/13.1___A_few_profiles___mindensity_and_ellipsoid_sel_origins.png` | same filename |
| `figures/4___Data_Analysis_Densities_And_Volumes/13.2___pixels___all_voids_nonscaled_and_scaled_sel___gaussian_sigma_20.png` | same filename |

Optional supplementary files:

```text
supplementary/perc__buildup_video.mp4
supplementary/Z_walls_and_grid_video.mp4
supplementary/Time_derivative_video.mp4
supplementary/ellipsoid_video.mp4
```

---

## Main conclusions

- A WVF pipeline can connect void identification, hierarchy control, redshift evolution, morphology, and internal density structure in one framework.
- The MK2 implementation produces a substantially different final segmentation from MK1 because connected origins and equal-level patches matter.
- The low-redshift Illustris-3-Dark WVF voids are mature, underdense, and strongly triaxial.
- Ellipsoid fits are useful morphology descriptors, but they do not replace the irregular watershed basins.
- Centre choice affects radial profiles: the minimum-density cell is not generally the morphology-based void centre.

---

## Limitations

- Raw Illustris data are not included.
- Results depend on grid resolution, smoothing, level construction, and hierarchy thresholds.
- The density time derivative is Eulerian and does not track individual particles.
- The strict $\mathrm{IoU}\geq0.6$ sample is a reliable ellipsoid-profile subset, not the full WVF catalogue.
- The fitted ellipsoid is a compact descriptor of a void's global morphology, not a physical boundary definition.
