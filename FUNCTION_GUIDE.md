# Which Function Should I Use?

**Author: Md. Mobarak Karim, Ph.D.**

This guide is designed for the question beginners actually have:

> **I know what I want to do to the image. Which Python function should I try first, and why?**

Use this as a companion to the notebooks. It is a decision guide, not a substitute for validating results.

## 1. First: inspect the image

| Question | Start with | Why |
|---|---|---|
| What are the image dimensions? | `image.shape` | Tells you rows, columns, and possible channel axes |
| How many axes? | `image.ndim` | Distinguishes simple 2-D from multichannel/volumetric data |
| How are values stored? | `image.dtype` | Arithmetic and valid ranges depend on dtype |
| What intensity values exist? | `image.min()`, `image.max()` | Checks dynamic range and unexpected values |
| What is the typical intensity? | `image.mean()`, `np.median(image)` | Quick descriptive statistics |
| What does the intensity distribution look like? | `plt.hist(image.ravel())` | Helps assess contrast and threshold plausibility |

**Use these before almost every new analysis.**

## 2. Display and visual inspection

| Goal | Function | Use when | Main caution |
|---|---|---|---|
| Display grayscale | `plt.imshow(image, cmap="gray")` | 2-D intensity image | Colormap affects display only |
| Display RGB | `plt.imshow(image)` | True RGB image | Do not force grayscale colormap |
| Change display contrast only | `vmin=`, `vmax=` | You want better visibility without rewriting the array | Record display settings for figures |
| Show multiple results | `plt.subplots()` | Comparing raw/intermediate/final images | Keep scales comparable when appropriate |

## 3. Crop and select pixels

| Goal | Function/syntax | Use when | Main caution |
|---|---|---|---|
| One pixel | `image[row, col]` | Debugging a location | NumPy is row first, column second |
| Rectangular ROI | `image[r0:r1, c0:c1]` | Defined crop/ROI | Stop index is excluded |
| Select pixels by rule | `image[mask]` | Masked statistics | Mask must match image geometry |
| Count foreground pixels | `np.count_nonzero(mask)` | Pixel-area style count | This is not object count |

## 4. Convert dtype safely

| Goal | Function | Why |
|---|---|---|
| Convert image to normalized float | `ski.util.img_as_float()` | Image-aware conversion |
| Convert to 8-bit | `ski.util.img_as_ubyte()` | Image-aware uint8 conversion |
| General numerical type conversion | `.astype(...)` | Changes storage type only |

**Important:** `astype(float)` does not automatically map 0–255 to 0–1.

## 5. Contrast and intensity

| Problem | Try first | Use when | Avoid/consider |
|---|---|---|---|
| Need display-only contrast | `imshow(..., vmin, vmax)` | You do not want to change source values | Best first option for visualization |
| Need explicit intensity remapping | `ski.exposure.rescale_intensity()` | A transformed image is intentionally part of processing | Changes pixel values |
| Global contrast is compressed | `ski.exposure.equalize_hist()` | One global redistribution is reasonable | Quantitative intensity interpretation changes |
| Local contrast varies | `ski.exposure.equalize_adapthist()` | Local enhancement is needed | Can amplify noise/background |

## 6. Filtering

| Problem | Function | Key parameter | Main trade-off |
|---|---|---|---|
| Fine distributed noise | `ski.filters.gaussian()` | `sigma` | More smoothing = more blur |
| Salt-and-pepper / isolated outliers | `ski.filters.median()` | `footprint` | Large footprint can erase small features |
| Need edge strength | `ski.filters.sobel()` | — | Edge map is not a filled object mask |

Ask first: **Do I actually need filtering?** No filter is a valid choice.

## 7. Thresholding

| Situation | Function | Use when | Main caution |
|---|---|---|---|
| One global threshold seems plausible | `ski.filters.threshold_otsu()` | Foreground/background intensities are reasonably separable | Automatic does not mean correct |
| Illumination/background varies | `ski.filters.threshold_local()` | Different regions need different thresholds | `block_size` changes behavior strongly |
| Want to compare global methods | `ski.filters.try_all_threshold()` | Exploration | Visual comparison is not validation |

Polarity:

```python
mask = image > threshold  # bright foreground
mask = image < threshold  # dark foreground
```

## 8. Morphology

For scikit-image 0.26, the beginner course uses the current `max_size` API.

| Goal | Function | What it does |
|---|---|---|
| Remove tiny foreground components | `ski.morphology.remove_small_objects(mask, max_size=N)` | Removes connected components containing N pixels or fewer |
| Fill tiny holes | `ski.morphology.remove_small_holes(mask, max_size=N)` | Fills holes containing N pixels or fewer |
| Remove small protrusions / break thin bridges | `ski.morphology.opening()` | Erosion followed by dilation |
| Fill small gaps / connect close regions | `ski.morphology.closing()` | Dilation followed by erosion |

**Caution:** morphology changes geometry. Size parameters are usually in pixels unless you explicitly calibrate them.

## 9. Objects and segmentation

| Goal | Function | Use when |
|---|---|---|
| Give each connected object an ID | `ski.measure.label()` | Binary mask has separate connected regions |
| Display labels over image | `ski.color.label2rgb()` | Quality-control inspection |
| Split touching objects | `ski.segmentation.watershed()` | Touching objects are a real failure mode |
| Find watershed markers | `ski.feature.peak_local_max()` | Peaks correspond to likely object centers |

Do not use watershed simply because it is more advanced.

## 10. Measurements

| Goal | Function | Use when |
|---|---|---|
| One object at a time | `ski.measure.regionprops()` | Custom per-object logic |
| Measurement table | `ski.measure.regionprops_table()` | pandas/CSV workflow |
| Organize table | `pd.DataFrame(...)` | Analysis and export |
| Save table | `df.to_csv(...)` | Flat measurement output |

Common properties:
- `area`
- `centroid`
- `perimeter`
- `eccentricity`
- `solidity`
- `axis_major_length`
- `axis_minor_length`
- `mean_intensity` (requires `intensity_image=`)

## 11. Physical units

Pixel measurements are not physical measurements until calibrated.

For square pixels:

```python
area_um2 = area_pixels * pixel_size_um * pixel_size_um
```

For non-square sampling:

```python
area_um2 = area_pixels * pixel_size_y_um * pixel_size_x_um
```

Use acquisition metadata or validated calibration.

## 12. Reading, writing, and batch processing

| Goal | Function | Why |
|---|---|---|
| Read standard image | `imageio.v3.imread()` | Simple NumPy-array I/O |
| Write standard image | `imageio.v3.imwrite()` | Explicit derived output |
| Work with paths | `pathlib.Path` | Portable, readable paths |
| Reuse an analysis | Write a function | Prevents copy/paste differences |
| Batch many files | Loop over validated file list | Repeats a tested rule |

For complex microscopy formats, use an appropriate metadata-aware reader rather than assuming a generic image reader preserves everything.

## 13. If the result looks wrong

Do not immediately add another algorithm.

Work backward:

1. Is the input shape/channel correct?
2. Is dtype/range what you expected?
3. Did filtering erase or merge structures?
4. Is threshold polarity correct?
5. Is one global threshold appropriate?
6. Did morphology remove real structures?
7. Are touching objects merged?
8. Does the overlay match the raw image?
9. Are measurement units correct?
10. Did a small parameter change completely alter the result?

**Debug one step at a time.**
