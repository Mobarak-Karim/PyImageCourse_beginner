# Beginner Image Processing Cheat Sheet

**Author: Md. Mobarak Karim, Ph.D.**

This page is for quick recall. For **why/when to use each function**, see [`FUNCTION_GUIDE.md`](FUNCTION_GUIDE.md).

## Inspect first

```python
print("shape:", image.shape)
print("ndim :", image.ndim)
print("dtype:", image.dtype)
print("range:", image.min(), image.max())
print("mean :", image.mean())
```

## Display

```python
plt.imshow(image, cmap="gray")
plt.axis("off")
plt.show()
```

Display-only contrast:

```python
plt.imshow(image, cmap="gray", vmin=50, vmax=180)
```

## Crop

```python
crop = image[row_start:row_end, col_start:col_end]
```

## Safe float conversion

```python
image_float = ski.util.img_as_float(image)
```

## Histogram

```python
plt.hist(image.ravel(), bins=256)
plt.show()
```

## Smooth

```python
smooth = ski.filters.gaussian(image, sigma=1.0)
```

## Median filter

```python
footprint = ski.morphology.disk(1)
filtered = ski.filters.median(image, footprint=footprint)
```

## Global threshold

```python
threshold = ski.filters.threshold_otsu(image)
mask = image > threshold
```

## Local threshold

```python
local_threshold = ski.filters.threshold_local(
    image,
    block_size=51,
    offset=5,
)
mask = image > local_threshold
```

## Clean a binary mask — scikit-image 0.26

```python
mask = ski.morphology.remove_small_objects(
    mask,
    max_size=100,
)

mask = ski.morphology.remove_small_holes(
    mask,
    max_size=100,
)
```

`max_size=100` removes/fills connected areas containing **100 pixels or fewer**.

## Label objects

```python
labels = ski.measure.label(mask)
print("object labels:", labels.max())
```

## Validate labels

```python
overlay = ski.color.label2rgb(
    labels,
    image=image,
    bg_label=0,
    alpha=0.35,
)

plt.imshow(overlay)
plt.axis("off")
plt.show()
```

## Measure

```python
props = ski.measure.regionprops_table(
    labels,
    intensity_image=image,
    properties=(
        "label",
        "area",
        "centroid",
        "mean_intensity",
    ),
)

df = pd.DataFrame(props)
```

## Convert area to physical units

```python
pixel_size_y_um = 0.5
pixel_size_x_um = 0.5

df["area_um2"] = (
    df["area"]
    * pixel_size_y_um
    * pixel_size_x_um
)
```

## Read a standard image

```python
import imageio.v3 as iio

image = iio.imread("image.tif")
```

## Save results

```python
df.to_csv("outputs/results.csv", index=False)
```

## Debugging habit

When an output looks wrong:

1. Inspect the immediately preceding image/mask.
2. Change one parameter only.
3. Compare against the raw image.
4. Write down why the parameter should affect the problem.
