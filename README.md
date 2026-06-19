# previous-analysis

This repository documents the analytical workflow I developed during my PhD research in Kinesiology at Georgia State University (2021–2024). It contains eight Jupyter notebooks organized into two pipelines plus three standalone analyses, all built on motion-capture and force-plate data collected for studies of walking, balance, and obstacle crossing in adults and children.

These notebooks reflect the reasoning I worked through during dissertation research rather than production-ready tools. I am preserving them here as a record of how I approached each analytical problem.

## Background

- **Author**: Yeon-Joo Kang
- **Institution**: Georgia State University, Department of Kinesiology and Health
- **Period**: 2021–2024
- **Research focus**: Biomechanics of gait, postural control, and dynamic stability across adult and pediatric populations, including infants with Down syndrome
- **Active GitHub work**: see `gait-spatiotemporal` and `marker-label` repositories for current development

## Repository structure

```
previous-analysis/
├── README.md
│
├── Pipeline 1 — Spatiotemporal gait parameters
│   ├── treadmill_spatiotemporal_analysis.ipynb
│   └── overground_spatiotemporal_from_events.ipynb
│
├── Pipeline 2 — Joint kinematics
│   ├── vicon_joint_kinematics_peak_rom.ipynb           (Stage 1)
│   ├── joint_kinematics_ensemble_avg.ipynb             (Stage 2)
│   ├── joint_kinematics_group_statistics.ipynb         (Stage 3)
│   └── spm_joint_kinematics.ipynb                      (Stage 4)
│
└── Standalone balance analyses
    ├── cop_analysis.ipynb
    └── margin_of_stability_analysis.ipynb
```

## Pipeline 1 — Spatiotemporal Gait Parameters

These two notebooks compute the same set of spatiotemporal parameters (step time, step length, step width, stance/swing time and percentage, stride time and length) from two different walking modes, using two different event-detection approaches.

### `treadmill_spatiotemporal_analysis.ipynb`

Treadmill walking trials. Implements **custom gait-event detection** from raw heel and toe marker trajectories using vertical-velocity sign changes, with a minimum-interval filter to reject false detections. Includes the treadmill belt-speed correction needed to derive valid step length from marker positions that are held roughly stationary in the lab frame.

### `overground_spatiotemporal_from_events.ipynb`

Overground walking trials. **Leverages Vicon's built-in event detection** rather than custom detection, and extends the analysis with normalized step and stride speeds (cm/s) plus single-support and double-support percentages — clinical metrics relevant to gait stability and asymmetry. Includes a sanity-check visualization section comparing detected events against marker positions.

The two notebooks together show two different approaches to the same underlying problem and reflect a methodological evolution from building from scratch to integrating with the upstream tool's existing event labels.

## Pipeline 2 — Joint Kinematics

A four-stage pipeline taking raw Vicon model output through to time-series statistical comparison of group means.

### Stage 1: `vicon_joint_kinematics_peak_rom.ipynb`

Trial-level processing. Segments each trial into individual gait cycles using heel-strike events from the spatiotemporal pipeline, computes per-cycle min/max/ROM for the six sagittal-plane joint angles (left/right ankle, hip, knee), and removes outlier cycles via the IQR rule applied to peak and minimum values. The pattern is repeated for each of the six joints to keep the notebook a faithful record of the actual workflow.

### Stage 2: `joint_kinematics_ensemble_avg.ipynb`

Trial-to-subject aggregation. Pools left- and right-side gait cycles from a single subject and computes the ensemble mean and standard deviation through the gait cycle. The output is one ensemble curve per joint per subject, with cycle-to-cycle variability captured as a SD band.

### Stage 3: `joint_kinematics_group_statistics.ipynb`

Cross-subject analysis. Performs IQR-based cross-subject outlier removal, then group comparisons using mixed-design ANOVA (Group between-subjects × Time within-subjects) with `pingouin`, followed by pairwise post-hoc tests. Boxplots with significance annotation via `statannotations`.

### Stage 4: `spm_joint_kinematics.ipynb`

**Statistical Parametric Mapping (SPM1D)** comparison of full joint angle curves across groups and time points. Tests the entire gait cycle point-by-point with proper multiple-comparison correction, identifying where in the cycle group means differ rather than only comparing peak values. Implementation follows Pataky et al. (2013), using the `spm1d` Python package.

This stage will be revisited and developed further in a separate repository.

## Standalone Balance Analyses

### `cop_analysis.ipynb`

**Static postural sway** from force-plate center-of-pressure (COP) data. Computes the 95% confidence ellipse via eigendecomposition of the COP covariance matrix and the chi-square inverse at 95% confidence (df=2 for bivariate data), giving ellipse area, axis lengths, and rotation angle. Also computes classical sway metrics (sway velocity, sway range, AP and ML). Foot-length-normalized variants are included for valid cross-age comparison.

Refactored into reusable functions and includes publication-style comparison plots for visualizing balance across conditions and groups (e.g., Adults vs Children, pre/post Rocker Board, pre/post Wobble Board).

### `margin_of_stability_analysis.ipynb`

**Dynamic stability** during an obstacle-crossing task, applying Hof's extrapolated center of mass (XCoM) framework:

> Hof, A. L., Gazendam, M. G. J., & Sinke, W. E. (2005). The condition for dynamic stability. *Journal of Biomechanics*, 38(1), 1–8.

The notebook implements a multi-event detection pipeline tailored to the obstacle-crossing task — heel strikes, toe offs, mid-swings, and the frames at which each foot crosses over the obstacles — and adds obstacle-specific measures (toe clearance height, instantaneous toe speed at crossing) on top of the standard MoS calculation. Per-stride MoS and COM/XCoM trajectory analysis with transverse-plane visualization complete the picture.

Of the notebooks here this is the most theoretically grounded, and represents the strongest single piece of dissertation analytical work in the archive.

## Notes on the data

The notebooks expect inputs in the CSV formats produced by my dissertation data collection workflow (Vicon Nexus exports of marker trajectories, model outputs, gait events, and force-plate channels). The CSV files themselves are not included in this repository to protect participant privacy.

## Tooling

- **Language**: Python (Jupyter notebooks)
- **Core libraries**: `pandas`, `numpy`, `scipy`, `matplotlib`, `seaborn`
- **Statistics**: `pingouin`, `statannotations`, `spm1d`
- **Input format**: Vicon Nexus CSV exports

## License

MIT — see [LICENSE](LICENSE).
