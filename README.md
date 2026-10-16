# Off-Road Autonomous Driving — Center Line Detection

Extracts the drivable center line from a segmentation mask of an off-road scene, using geometric boundary tracking rather than a learned model.

## Problem

On-road lane detection leans on painted markings and high contrast. Off-road there are none — the drivable corridor is bounded by terrain that curves, narrows, and changes width from frame to frame. Given a segmentation mask of the traversable region, the task is to recover a smooth, continuous center line a vehicle could actually follow.

## Workflow

![Workflow](https://github.com/user-attachments/assets/4a2e0145-da30-4015-881f-3dddeefe57f1)

## Approach

**Boundary extraction.** The corridor edge is found by XOR-ing the binary mask against a one-pixel-shifted copy of itself in both axes. One pass, no convolution, no thresholds to tune. The four image borders are patched from their neighbors, since `np.roll` wraps around.

**Finding the start.** Rows are scanned from the top until six contain edge pixels, then the widest contiguous run among them marks where the corridor begins — which rejects isolated noise above the path.

**Tracking down the corridor.** For each row below the start, the nearest left and right boundary pixels are taken *relative to a running midpoint that updates on every row*. This is the part that matters: a fixed split would lose the corridor as soon as it bends, whereas the running midpoint follows it.

**Center line.** B-splines are fitted to the left and right boundaries independently, averaged to produce the center line, then re-fitted and clipped to the image bounds so the result stays smooth and in-frame. Where one boundary runs out well before the bottom of the frame, it is extended down the image edge so the spline retains support.

## Example

![Example](https://github.com/user-attachments/assets/34c93410-c46c-401a-baab-7cb242530640)

Blue marks the averaged control points, red the fitted spline, yellow the spline extended to the bottom of the frame.

<!-- TODO: add one sentence on where this holds up and where it breaks — e.g. when the corridor forks, or when one boundary leaves the frame. Naming a failure case reads as more credible than implying there isn't one. -->

## Running it

```bash
pip install opencv-python numpy scipy matplotlib
jupyter notebook centerLineDetection.ipynb
```

Set `RGB_PATH` and `MASK_PATH` at the top of the second cell to point at your own data. The mask is expected to be the same resolution as the frame, with non-zero pixels marking the drivable region.
