# LiDAR Ground Filtering

### Ground and non-ground separation in urban mobile LiDAR point clouds

An end-to-end Python project for processing dense urban point clouds using **voxel sampling**, **Statistical Outlier Removal (SOR)**, and the **Cloth Simulation Filter (CSF)**.

The workflow is applied to a labeled Paris scan from the Paris–Lille–3D dataset. It includes preprocessing, parameter comparisons, quantitative evaluation, spatial error analysis, and PLY export.

**Author:** Kiana Abbas Abadi  
**Field:** Geomatics Engineering · Geospatial Data Processing · 3D Computer Vision  
**Development environment:** Python · Google Colab

---

## Project Highlights

| Metric | Recorded result |
|---|---:|
| Original scan size | 45,797,650 points |
| Points in the selected region | 10,567,437 |
| Evaluated points after preprocessing | 2,354,353 |
| Overall accuracy | **97.36%** |
| Ground precision | **93.24%** |
| Ground recall | **97.26%** |
| Ground F1 score | **95.21%** |

> Results refer to one selected Paris region after voxel sampling and outlier removal. They represent a binary ground-filtering evaluation, not an official Paris–Lille–3D semantic segmentation benchmark submission.

## Overview

Ground extraction is an essential step in many urban mapping and terrain analysis workflows. However, separating ground from surrounding objects becomes challenging near curbs, parked vehicles, vegetation, and building boundaries.

Mobile laser scanning introduces an additional challenge: point density varies across the scene. Surfaces close to the sensor trajectory may be densely sampled, while distant facades and tree canopies may contain sparse but valid measurements.

This project investigates these challenges through a geometry-based processing pipeline. It focuses on both numerical performance and the spatial interpretation of classification errors.

## Main Features

- PLY header and attribute inspection.
- Coordinate range analysis.
- Extraction of an 80 × 80 m region of interest.
- Voxel-based point cloud sampling.
- Preservation of original coordinates and associated reference labels.
- Comparison of two SOR threshold settings.
- Comparison of two CSF stiffness settings.
- Binary confusion matrix evaluation.
- Class-level analysis of non-ground filtering.
- Top-down and cross-sectional error visualization.
- Export of ground and non-ground point clouds.

## Processing Workflow

The notebook follows this sequence:

1. Load and inspect the labeled point cloud.
2. Select a spatial region of interest.
3. Reduce dense point coverage using a voxel grid.
4. Associate voxel representatives with original points.
5. Remove statistical outliers.
6. Apply cloth simulation for ground separation.
7. Compare predictions with reference labels.
8. Analyze the location and type of classification errors.
9. Export the resulting point clouds.

### 1. Data Inspection and Spatial Cropping

The source file contains **45,797,650 points**.

The notebook reads the PLY header, inspects available properties, and calculates the minimum, maximum, and range of the XYZ coordinates.

A nominal **80 × 80 m** window is selected around a center estimated from the XY medians of a strided subset of approximately 100,000 points.

The selected region contains **10,567,437 points**.

### 2. Voxel Sampling

A voxel size of **0.06 m**, equivalent to **6 cm**, is used to reduce point density.

This reduces the cropped point count to **2,394,901**, a reduction of approximately **77.34%**.

The purpose is to reduce the dominance of densely sampled surfaces before statistical filtering. Voxel sampling does not fill sparse regions, so the resulting cloud is not perfectly uniform.

### 3. Coordinate and Label Preservation

Open3D voxel sampling generates representative centroid coordinates.

To retain original samples and their corresponding reference annotations, the notebook:

1. Builds a SciPy `cKDTree` from the cropped original points.
2. Finds the nearest original point to each voxel centroid.
3. Retrieves that point's original coordinates and class label.

This keeps each selected XYZ coordinate paired with an actual reference annotation rather than averaging numerical class IDs.

### 4. Statistical Outlier Removal

SOR identifies points with unusually large average distances to neighboring points.

The notebook compares:

- `nb_neighbors = 16`, `std_ratio = 2`
- `nb_neighbors = 16`, `std_ratio = 3`

The less aggressive threshold is selected for the next stage to preserve more of the sparse structures visible in the comparison plots.

### 5. Cloth Simulation Filtering

CSF estimates a terrain surface through cloth simulation and separates points according to their distance from that surface.

The notebook compares:

- `rigidness = 1`
- `rigidness = 2`

The selected output uses `rigidness = 2`.

### 6. Evaluation and Export

Predicted ground labels are compared with a binary reference mask derived from the dataset's semantic class IDs.

The resulting point clouds are exported as:

- `Ground_Points.ply`
- `Non_Ground_Points.ply`

## Dataset

The project uses the labeled Paris point cloud from **Paris–Lille–3D**.

**Official dataset page:**  
https://npm3d.fr/paris-lille-3d

The inspected input includes:

- XYZ coordinates.
- Sensor-origin coordinates.
- GPS time.
- Reflectance.
- Annotation fields.

The notebook uses the **`class` field** to construct reference labels.

### Ground Definition

The following detailed semantic IDs are treated as ground:

| Class ID | Category |
|---|---|
| `202020000` | Road |
| `202030000` | Sidewalk |
| `202050000` | Island |
| `202060000` | Lawn / grass |

All other classes are treated as non-ground under the current mapping.

In particular:

- Curbs, with class ID `202040000`, are treated as non-ground.
- Unclassified points, with class ID `0`, are also treated as non-ground.

The treatment of unclassified points should be considered when interpreting the evaluation.

> This notebook expects the detailed semantic class IDs listed above. The 10-class benchmark variant uses a different label scheme and requires an updated reference mapping.

## Technologies

| Tool | Role |
|---|---|
| Python | Processing workflow |
| NumPy | Numerical operations and point arrays |
| Open3D | Voxel sampling, outlier removal, and PLY export |
| SciPy | KD-tree nearest-neighbor queries |
| Cloth Simulation Filter | Ground extraction |
| plyfile | PLY reading and metadata inspection |
| pandas | Tabular results |
| scikit-learn | Confusion matrix computation |
| Matplotlib | Point cloud and error visualization |
| Google Colab | Notebook execution |

This is a **geometry-based filtering project**. It does not train a neural network or another supervised classifier.

## Configuration

| Stage | Parameter | Selected value |
|---|---|---|
| Spatial cropping | `HALF_BOX` | `40.0` m |
| Voxel sampling | `USE_VOXEL` | `True` |
| Voxel sampling | `voxel_size` | `0.06` m |
| SOR | `nb_neighbors` | `16` |
| SOR | `std_ratio` | `3.0` |
| CSF | `cloth_resolution` | `0.8` m |
| CSF | `class_threshold` | `0.30` m |
| CSF | `rigidness` | `2` |
| CSF | `bSloopSmooth` | `True` |
| CSF | `time_step` | `0.65` |

Parameters are currently configured directly in the notebook cells.

## Experimental Results

### Point Counts

| Processing stage | Number of points |
|---|---:|
| Original Paris scan | 45,797,650 |
| Cropped region | 10,567,437 |
| After voxel sampling | 2,394,901 |
| After selected SOR filtering | 2,354,353 |
| Predicted ground | 661,399 |
| Predicted non-ground | 1,692,954 |

### SOR Comparison

| Standard deviation ratio | Retained points | Removed points | Removed fraction | Recorded time |
|---|---:|---:|---:|---:|
| `2` | 2,306,080 | 88,821 | 3.71% | 11.017 s |
| **`3`** | **2,354,353** | **40,548** | **1.69%** | **12.522 s** |

Increasing `std_ratio` from 2 to 3 preserved **48,273 additional points**, approximately **2.02%** of the SOR input.

This illustrates the trade-off between aggressive noise removal and the preservation of valid points in sparse regions.

The selected threshold was supported by visual comparison of the resulting structures. It should not be interpreted as a globally optimal setting for all mobile LiDAR scenes.

### CSF Comparison

| Cloth stiffness | Ground points | Ground fraction | Non-ground points | Recorded time |
|---|---:|---:|---:|---:|
| `rigidness = 1` | 667,567 | 28.35% | 1,686,786 | 1.885 s |
| **`rigidness = 2`** | **661,399** | **28.09%** | **1,692,954** | **1.859 s** |

The selected configuration classified **6,168 fewer points** as ground.

Selection was based on visual inspection and output counts. A full confusion matrix was computed for `rigidness = 2`; therefore, these counts alone do not prove that it is more accurate than `rigidness = 1`.

Recorded processing times come from one notebook execution. They depend on the runtime environment and describe the measured stages rather than the complete workflow.

## Quantitative Evaluation

### Confusion Matrix

**Positive class: ground.**

Rows represent reference classes, while columns represent predicted classes.

| Reference / Prediction | Ground | Non-ground | Total |
|---|---:|---:|---:|
| Ground | TP = 616,660 | FN = 17,374 | 634,034 |
| Non-ground | FP = 44,739 | TN = 1,675,580 | 1,720,319 |
| Total | 661,399 | 1,692,954 | 2,354,353 |

Definitions:

- **TP:** ground points correctly classified as ground.
- **TN:** non-ground points correctly classified as non-ground.
- **FP:** non-ground points incorrectly classified as ground.
- **FN:** ground points incorrectly classified as non-ground.

### Performance Metrics

The following metrics are calculated from the selected run's confusion matrix. Precision, recall, and F1 provide supplementary summaries of the recorded counts.

| Metric | Formula | Result |
|---|---|---:|
| Overall accuracy | `(TP + TN) / N` | **97.36%** |
| Ground precision | `TP / (TP + FP)` | **93.24%** |
| Ground recall | `TP / (TP + FN)` | **97.26%** |
| Ground F1 score | `2TP / (2TP + FP + FN)` | **95.21%** |
| Ground omission rate | `FN / (TP + FN)` | 2.74% |
| False positive rate | `FP / (TN + FP)` | 2.60% |
| Ground commission rate | `FP / (TP + FP)` | 6.76% |

### Metric Interpretation

Approximately **73%** of the evaluated reference points are non-ground. Overall accuracy is therefore interpreted alongside ground precision and recall.

The ground recall indicates that most reference ground points were retained. Ground precision shows that some non-ground points, particularly near-surface objects and boundary points, entered the predicted ground cloud.

**Naming clarification:** the notebook labels `FP / (TN + FP)` as “Type II / Commission.” That formula is the false positive rate. Ground commission is reported separately here using `FP / (TP + FP)`.

## Class-Level Analysis

The table below reports the percentage of each reference non-ground category correctly classified as non-ground.

These values measure **binary filtering performance**, not multiclass semantic classification accuracy.

| Reference category | Correctly classified as non-ground |
|---|---:|
| Tree with visible trunk | 99.88% |
| Natural object | 99.35% |
| Building exterior | 98.42% |
| Floor lamp | 97.77% |
| Traffic sign | 95.65% |
| Parked car | 90.99% |
| Pedestrian | 87.92% |
| Trash can | 72.31% |
| Bollard | 70.81% |
| Barrier | 62.86% |

### Key Findings

**Tall objects were generally easier to separate.**  
Buildings and trees usually have a clear vertical separation from the estimated ground surface.

**Short objects were more challenging.**  
Barriers, bollards, and trash cans include points close to the terrain surface. Some of these points fall within the ground classification threshold.

**Large classes can contribute many errors despite high correctness.**  
Building exteriors contributed **9,997 false positives**, even though 98.42% of their points were correctly classified.

**Vehicle bases remain difficult.**  
The parked-car category contributed **4,764 false positives**, with a correct non-ground classification rate of 90.99%.

## Spatial Error Analysis

The notebook visualizes errors using two complementary views.

### Top-Down View

The plan-view error map shows where errors concentrate across the selected region, including road boundaries and areas around object footprints.

### Cross-Sectional View

An X–Z profile through a 10 m-wide strip helps distinguish:

- Non-ground points incorrectly accepted near the terrain.
- Ground points rejected around local elevation changes.
- Correctly separated building and vegetation structures.

### Interpretation

False positives frequently occur near transitions between ground and nearby objects.

False negatives are concentrated in parts of the right-hand boundary of the selected scene, consistent with local elevation changes and incomplete fitting of the cloth surface.

The exact semantic identity of an error cluster requires checking its corresponding reference labels.

## Installation and Execution

### Google Colab

1. Open Google Colab:

   https://colab.research.google.com/

2. Upload the notebook:

   ```text
   Paris_LiDAR_Ground_Filtering.ipynb
   ```

3. Run the dependency installation and import cells.

4. Mount Google Drive when prompted.

5. Obtain the detailed-label `Paris.ply` file and place it at:

   ```text
   /content/drive/MyDrive/Paris.ply
   ```

6. Skip or replace the existing `gdown` cell if the dataset is already available. Its current download link is file-specific.

7. Execute the remaining cells in order.

8. Download the exported point clouds before ending the Colab session.

### Dependencies

```bash
pip install numpy plyfile open3d cloth-simulation-filter scipy matplotlib pandas scikit-learn tabulate gdown
```

The package `cloth-simulation-filter` is imported as:

```python
import CSF
```

### Local Jupyter Execution

The notebook currently includes Colab-specific Drive mounting and file paths.

For local execution:

1. Install the required packages.
2. Skip the Google Drive mounting and download cells.
3. Update `ply_path` to the local dataset location.
4. Update `output_dir` to a local output directory.
5. Replace any Colab-specific download utilities with normal local file access.

The full source scan and intermediate arrays can require substantial memory. Cropping reduces downstream workload, but the notebook accesses the source file before extracting the region. It is not a fully streaming implementation.

## Output Files

| File | Description |
|---|---|
| `Ground_Points.ply` | Predicted ground point cloud |
| `Non_Ground_Points.ply` | Predicted non-ground point cloud |

The current export directory is:

```text
/content/LIDAR_OUT/
```

This is temporary Colab storage. Although the export cell's heading mentions Google Drive, its actual path points to the Colab runtime. Change `output_dir` explicitly to save directly to Drive.

The exported PLY files contain **XYZ coordinates**. Reference labels and confidence values are not included.

These files can be opened in CloudCompare or another compatible point cloud viewer.

## Technical Decisions

### Why use voxel sampling before SOR?

Dense road surfaces can dominate the global distance statistics used by SOR. Voxel sampling reduces this imbalance and decreases the number of points processed in later stages.

### Why retain original samples after voxel sampling?

Nearest-neighbor recovery preserves a direct association between each selected coordinate and its original reference annotation.

### Why compare two SOR thresholds?

A smaller threshold removes more points but can also remove valid sparse structures. Comparing thresholds makes this trade-off visible.

### Why inspect errors spatially?

A single accuracy value does not explain whether failures occur around curbs, vegetation, object bases, or elevation changes. Spatial inspection connects numerical results with scene geometry.

## Limitations

- Results cover one selected region rather than multiple independent scenes.
- Evaluation applies to points retained after preprocessing.
- The reported metrics do not evaluate whether SOR removed valid original points.
- Only two SOR thresholds and two CSF stiffness settings were compared.
- The effect of enabling or disabling slope post-processing was not separately measured.
- The ground definition is task-specific and depends on the reference class mapping.
- Treating unclassified points as non-ground introduces uncertainty.
- Exact dependency versions and hardware specifications were not pinned in the recorded notebook.
- The workflow exports point clouds; it does not generate a terrain raster or mesh.

No train/test split is used because the pipeline does not fit a supervised model. The reference annotations are used to assess geometric filtering results.

## Future Improvements

- Evaluate additional regions from Paris and Lille.
- Use a fixed protocol for parameter selection and final evaluation.
- Measure valid-point retention during outlier removal.
- Compare global SOR with density-adaptive filtering.
- Evaluate several cloth resolutions and distance thresholds.
- Compare slope post-processing enabled and disabled.
- Export reference and predicted labels as PLY attributes.
- Add configurable paths and version-pinned dependencies.
- Construct and evaluate a terrain surface from the extracted ground points.

## Skills Demonstrated

- Scientific programming with Python.
- Dense 3D point cloud processing.
- Spatial indexing and nearest-neighbor search.
- Geometry-based ground filtering.
- Parameter comparison and experimental interpretation.
- Confusion matrix and class-level evaluation.
- Geospatial visualization and spatial error analysis.
- Technical documentation and reproducible workflow design.

This academic project applies established Open3D and CSF implementations. Its focus is workflow integration, parameter analysis, and interpretation of filtering errors in an urban mapping context.

## References

### Paris–Lille–3D Dataset

Roynard, X., Deschaud, J.-E., & Goulette, F. (2018).

*Paris-Lille-3D: A large and high-quality ground-truth urban point cloud dataset for automatic segmentation and classification.*

The International Journal of Robotics Research, 37(6), 545–557.

https://doi.org/10.1177/0278364918767506

Dataset:

https://npm3d.fr/paris-lille-3d

### Cloth Simulation Filter

Zhang, W., Qi, J., Wan, P., Wang, H., Xie, D., Wang, X., & Yan, G. (2016).

*An Easy-to-Use Airborne LiDAR Data Filtering Method Based on Cloth Simulation.*

Remote Sensing, 8(6), 501.

https://doi.org/10.3390/rs8060501

Implementation:

https://github.com/jianboqi/CSF

### Documentation

- Open3D: https://www.open3d.org/
- Open3D outlier removal: https://www.open3d.org/docs/release/tutorial/geometry/pointcloud_outlier_removal.html
- SciPy cKDTree: https://docs.scipy.org/doc/scipy/reference/generated/scipy.spatial.cKDTree.html
- scikit-learn confusion matrix: https://scikit-learn.org/stable/modules/generated/sklearn.metrics.confusion_matrix.html

## Attribution

**Author:** Kiana Abbas Abadi

Dataset ownership remains with its