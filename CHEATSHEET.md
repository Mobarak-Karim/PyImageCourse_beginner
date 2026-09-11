# Python + Image Processing Beginner Cheat Sheet

**Author: Md. Mobarak Karim, Ph.D.**

This page is for quick recall. For **why/when to use image-processing functions**, see [`FUNCTION_GUIDE.md`](FUNCTION_GUIDE.md).

# Part A — Python basics

## Variables

```python
threshold = 150
sample_name = "cell_01"
is_valid = True
```

A variable is simply a name that refers to a value.

## Common data types

```python
count = 10          # int
pixel_size = 0.5    # float
name = "green"      # str
keep = True         # bool

print(type(count))
```

## Lists

```python
files = ["a.tif", "b.tif", "c.tif"]

print(files[0])   # first item
print(files[-1])  # last item
print(files[0:2]) # slice
```

## Call a function

```python
print("hello")
length = len(files)
```

General pattern:

```python
function_name(argument)
```

Scientific functions often use keyword arguments:

```python
filtered = ski.filters.gaussian(image, sigma=2)
```

## Comparisons

```python
value > 10
value < 20
value == 15
value != 15
```

Comparisons return `True` or `False`.

## Boolean logic

```python
(value > 10) and (value < 20)
condition_a or condition_b
not condition_a
```

## `if / elif / else`

```python
if value > 20:
    print("high")
elif value > 10:
    print("medium")
else:
    print("low")
```

Indentation matters in Python.

## `for` loop

```python
for filename in files:
    print(filename)
```

With an index:

```python
for index, filename in enumerate(files):
    print(index, filename)
```

## `while` loop

```python
attempt = 0

while attempt < 3:
    print(attempt)
    attempt += 1
```

Use `for` when you know what sequence you are iterating over. Use `while` when repetition depends on a condition.

## Define your own function

```python
def pixels_to_um(length_pixels, pixel_size_um):
    length_um = length_pixels * pixel_size_um
    return length_um

result = pixels_to_um(20, 0.5)
```

- `def` defines the function.
- names inside parentheses are parameters.
- `return` sends a result back.

## Imports

```python
import numpy as np
import matplotlib.pyplot as plt
from skimage import filters
```

Then:

```python
mean_value = np.mean(values)
plt.imshow(image)
smoothed = filters.gaussian(image, sigma=1)
```

## Common errors

| Error | Usually means |
|---|---|
| `NameError` | variable/function name does not exist |
| `TypeError` | wrong kind of object used |
| `IndexError` | index is outside the available range |
| `FileNotFoundError` | path or filename is wrong |
| `ModuleNotFoundError` | package missing or wrong environment active |

Read the **last line of the traceback first**.

# Part B — NumPy and image processing

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

1. Read the error message if there is one.
2. Inspect the immediately preceding variable/image/mask.
3. Print `type`, `shape`, `dtype`, or range when relevant.
4. Change one parameter only.
5. Compare against the raw image.
6. Write down why the parameter should affect the problem.
