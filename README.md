<div align="center">

<h1>P²Calib: Utilizing Pattern Priors for LiDAR–Camera Extrinsic Calibration</h1>

<p><a href="https://github.com/JokerJohn"><b>Xiangcheng Hu</b></a> &nbsp;(Sole Author)</p>

![P2Calib](./README/projection.png)

<a href="https://arxiv.org/abs/2609.07516/"><img src="https://img.shields.io/badge/arXiv-P%C2%B2Calib-b31b1b" alt="arXiv"></a><a href="./README/p2calib_gui.mp4"><img src="https://img.shields.io/badge/Video-Tool%20Demo-blue" alt="Demo"></a><a><img alt="PRs-Welcome" src="https://img.shields.io/badge/PRs-Welcome-white" /></a>[![GitHub Stars](https://img.shields.io/github/stars/JokerJohn/P2Calib.svg)](https://github.com/JokerJohn/P2Calib/stargazers)<a href="https://github.com/JokerJohn/P2Calib/network/members"><img alt="FORK" src="https://img.shields.io/github/forks/JokerJohn/P2Calib?color=white" /></a>[![GitHub Issues](https://img.shields.io/github/issues/JokerJohn/P2Calib.svg)](https://github.com/JokerJohn/P2Calib/issues)[![License](https://img.shields.io/badge/license-MIT-blue.svg)](https://opensource.org/licenses/MIT)

</div>

## Introduction

**P²Calib** calibrates a LiDAR and a camera from the four-hole board. Accuracy in this pipeline is limited by the LiDAR-side hole centers, which degrade under sparse angular coverage and mixed pixels. P²Calib removes this bottleneck with two **pattern priors** that the board's CAD model already specifies:

- **Radius prior.** The known hole radius is incorporated as a fitting constraint, which prevents the center estimates from degrading under sparse angular coverage.
- **Layout prior.** Building on the improved hole estimates, the rigid rectangular layout of the four holes is enforced as a global consistency constraint, which corrects the residual errors across holes.
- **Interactive tool.** Both priors are integrated into a calibration tool that provides a complete extrinsic calibration pipeline.

Experiments on simulated and real datasets show that P²Calib lowers the joint registration residual by 90% and 82% and the held-out reprojection error by 96% and 77% over the baseline.

<div align="center">

![Pipeline](./README/pipeline.png)

</div>

## News

- **2026/09/07**: Repository created; code, tool and datasets are being prepared for release.

## Hardware and Scenes

<div align="center">
<table>
<tr>
<td width="50%"><img src="./README/platform.png" width="100%"></td>
<td width="50%"><img src="./README/scenes.png" width="100%"></td>
</tr>
<tr>
<td valign="top"><sub>Mobile platform and calibration board, with the camera and the area-array LiDAR enlarged.</sub></td>
<td valign="top"><sub>One data source per column, camera image (<b>a–d</b>) above the range-shaded LiDAR scan (<b>A–D</b>), the holes appearing as white voids: (<b>a,A</b>) simulator, (<b>b,B</b>) Avia, (<b>c,C</b>) Mid-360, (<b>d,D</b>) the area-array FS sensor.</sub></td>
</tr>
</table>
</div>

| Dataset | Sensor | Scenes | Standoff |
| ------- | ------ | -----: | -------- |
| `FS-B` | area-array LiDAR | 18 | 1.5–4.3 m |
| `FS-C` | area-array LiDAR | 20 | 1.0–4.2 m |
| `Avia` | Livox Avia | 5 | — |
| `Mid-360 A / B` | Livox Mid-360 | 4 / 3 | — |
| `Simulated` | four-hole board simulator | 60 × 2 densities | 1.5–5.0 m |

The area-array LiDAR has a 120°×50° field of view, 0.33° resolution in both axes, and ±50 mm range noise. `Avia` and `Mid-360` come from [FAST-Calib](https://github.com/hku-mars/FAST-Calib). Download links will be added on release.

## Interactive Calibration Tool

<div align="center">

<a href="./README/p2calib_gui.mp4"><img src="./README/p2calib_gui.gif" width="92%"></a>

</div>

*Batch detection and joint solving: the scene list on the left; the camera and point-cloud views on top, with the region of interest, the fitted hole rings and the hole centers drawn on them; the reprojection view and the message log below. Click the image above to play the full recording.*

The tool shows the result of every intermediate step, reports the layout disagreement and the registration residual separately for each capture, solves the selected scenes jointly, and writes out the extrinsic, the reprojection images and the colored point clouds.

## Method

<div align="center">
<table>
<tr>
<td width="50%"><img src="./README/teaser.png" width="100%"></td>
<td width="50%"><img src="./README/priors.png" width="100%"></td>
</tr>
<tr>
<td valign="top"><sub>Hole extraction without the priors, at 1.0 m (left) and 4.0 m (right), in the board plane. Mixed pixels (violet) and sparse boundary sampling (orange) displace the baseline centers (dashed) from the CAD rectangle, which P²Calib (solid) recovers.</sub></td>
<td valign="top"><sub>(<b>A</b>) A short arc (blue) permits many center–radius pairs (orange). (<b>B</b>) Sector samples constrain the center at radius r−δ; crosses mark rejected returns. (<b>C</b>) The CAD layout couples the four independent centers (orange) into the projected centers (red).</sub></td>
</tr>
</table>
</div>

Boundary candidates are drawn from an annulus and reduced to one representative per azimuth sector, then each center is solved by Huber-weighted Gauss–Newton against the fixed radius, where a single bias δ absorbs the inward rim erosion. The four refined centers are finally projected onto the CAD rectangle. Both priors act only on the LiDAR branch; the camera processing and the closed-form registration are unchanged, so the method applies to any four-hole calibration pipeline.

**Radius prior.** Instead of letting each hole's radius float, it's pinned to the known value `r − δ`. That's what removes the center/radius trade-off shown in panel A above — with the radius fixed, a short arc can no longer be explained by shrinking the circle instead of moving the center:

$$e_{ik} = d_{ik} - (r - \delta)$$

**Layout prior.** The four centers found this way are then snapped onto a single rigid rotation and translation of the CAD rectangle, so one noisy hole gets pulled back into line by the other three instead of being trusted on its own:

$$(\theta^\star, t^\star) = \arg\min_{\theta,\ t} \sum_{k=1}^{4} \left\| \hat{c}_k - R(\theta)\,c^{\mathcal{B}}_k - t \right\|^2$$

## Results

Joint residual [mm] and leave-one-out reprojection error [px], lower is better. `Det.` counts successful extractions over five seeds; errors use the scenes every variant detects.

| Method | FS-B Det. | Joint | LOO | FS-C Det. | Joint | LOO |
| ------ | --------: | ----: | --: | --------: | ----: | --: |
| velo2cam¹ | 14/18 | 23.50 | 3.51 | 11/20 | 40.55 | 10.25 |
| FAST-Calib | 85/90 | 217.68 | 67.50 | 65/100 | 191.50 | 43.34 |
| Ours w/o LP | 90/90 | 24.02 | 3.35 | 85/100 | 36.82 | 10.61 |
| Ours w/o RP | 80/90 | **22.23** | 2.87 | 60/100 | 35.92 | **9.67** |
| **P²Calib** | 80/90 | 22.28 | **2.82** | 60/100 | **35.13** | 9.91 |

¹ On its own detected scenes, after adapting its input stage to solid-state clouds.

<div align="center">
<table>
<tr>
<td width="50%"><img src="./README/ablation_vis.png" width="100%"></td>
<td width="50%"><img src="./README/sensors.png" width="100%"></td>
</tr>
<tr>
<td valign="top"><sub>Real-scene ablation on FS-B. (<b>A</b>) Camera detections; (<b>B</b>) both priors off, the fitted radii vary across holes; (<b>C</b>) radius prior only, the centers form a skewed quadrilateral; (<b>D</b>) both priors, the centers conform to the rectangular layout.</sub></td>
<td valign="top"><sub>Joint residual (<b>left</b>) and LOO error (<b>right</b>) on Avia, two Mid-360 sessions and the two FS datasets. The improvement is smaller on the scanning LiDARs, which sample the hole rims more densely.</sub></td>
</tr>
</table>
</div>

In simulation the hole-center error falls from 6.8–14.7 mm to 1.6–3.9 mm across all standoff groups, and the LOO reprojection error from 2.61 to 0.40 px on single-frame clouds and from 1.51 to 0.23 px on accumulated clouds.

<div align="center">

![Sweeps](./README/sweeps.png)

</div>

*Hole-center error against (**A**) standoff distance, (**B**) board placement in the image and (**C**) range-noise σ, on a shared logarithmic scale; AC solid, SF dashed. A curve ends where the method recovers no board.*

## Getting Started

> The calibration tool isn't released yet — the steps below show what setup and usage will look like once it is.

P²Calib is implemented in Python, for Ubuntu 20.04/22.04, Python 3.10, Open3D 0.19, OpenCV 4.10, PySide6 and NumPy < 2.

```bash
git clone https://github.com/JokerJohn/P2Calib.git
cd P2Calib
scripts/setup_p2calib_env.sh     # conda environment
scripts/run_p2calib_gui.sh       # launch the workbench
```

Each sample is one image paired with one point cloud, no rosbag needed. Open a dataset, draw the LiDAR ROI once per scene, run `Detect Selected`, then `Optimize Included` for the multi-scene solve and `Export Result`. The two priors are independent switches, which reproduces every variant in the results table above:

| `use_circle_prior` | `use_rect_template` | Variant |
| --- | --- | --- |
| `false` | `false` | FAST-Calib baseline |
| `true` | `false` | Ours w/o LP |
| `false` | `true` | Ours w/o RP |
| `true` | `true` | **P²Calib** |

## TODO

- [ ] Release the calibration tool and the four-hole simulator
- [ ] Release the FS-B and FS-C datasets
- [ ] Support multi-circle and non-rectangular board layouts
- [ ] Allow a signed rim bias for sensors with outward radius error

## Citation

```bibtex
@article{hu2026p2calib,
  title         = {P$^2$Calib: Utilizing Pattern Priors for LiDAR-Camera Extrinsic Calibration},
  author        = {Hu, Xiangcheng},
  journal       = {arXiv preprint arXiv:2609.07516},
  eprint        = {2609.07516},
  archivePrefix = {arXiv},
  primaryClass  = {cs.RO},
  year          = {2026}
}
```

The board and the baseline pipeline follow FAST-Calib and velo2cam:

```bibtex
@article{zheng2026fastcalib,
  title   = {{FAST-Calib}: {LiDAR}-Camera Extrinsic Calibration in One Second},
  author  = {Zheng, Chunran and Zhang, Fu},
  journal = {IEEE Robotics and Automation Practice},
  volume  = {1}, pages = {108--112}, year = {2026},
  doi     = {10.1109/RAP.2026.3692446}
}

@article{beltran2022automatic,
  title   = {Automatic Extrinsic Calibration Method for LiDAR and Camera Sensor Setups},
  author  = {Beltr{\'a}n, Jorge and Guindel, Carlos and de la Escalera, Arturo and Garc{\'i}a, Fernando},
  journal = {IEEE Transactions on Intelligent Transportation Systems},
  volume  = {23}, number = {10}, pages = {17677--17689}, year = {2022}
}
```

## License

P²Calib is released under the [MIT license](./LICENSE). 

## Acknowledgment

We thank the authors of [FAST-Calib](https://github.com/hku-mars/FAST-Calib), whose four-hole pipeline this work follows and whose Avia and Mid-360 data we use, and the authors of [velo2cam_calibration](https://github.com/beltransen/velo2cam_calibration), from which the board design originates. We thank Shenzhen Foreseen Technology Co., Ltd. for the area-array LiDAR, the platform and the FS datasets, and Shiyang Chen of X Square Robot for productive discussions.

## Contributors

<a href="https://github.com/JokerJohn/P2Calib/graphs/contributors">
  <img src="https://contrib.rocks/image?repo=JokerJohn/P2Calib" />
</a>
