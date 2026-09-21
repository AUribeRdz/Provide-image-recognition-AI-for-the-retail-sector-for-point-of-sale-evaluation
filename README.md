# Retail Shelf Image Recognition: Point-of-Sale Evaluation

Scores photographs of retail shelves for empty spots (gaps) and image quality using a pretrained object-detection model. It is a starting point for automated shelf audits.

## Overview

Manual store audits are slow and inconsistent. This notebook runs a pretrained shelf-gap detector over a folder of shelf photos and turns the detections into a weighted score per image, plus a summary for the whole folder. A problem image is reported and skipped, so one bad file does not stop the run.

The model has a single class, `void`, which marks an empty spot on the shelf. The notebook checks for that class name when the model loads and stops with a clear error if it is ever missing.

## How it works

1. Download the pretrained model `akul-29/Retail-Shelf-Gap-Detection_Model` (`best.pt`) from the Hugging Face Hub and load it with `ultralytics`.
2. Read every image in `shelve_images/` and run detection.
3. Compute three scores per image (0 to 100):

   | Score | Weight | How it is computed |
   | --- | --- | --- |
   | Gap Score | 50% | 100 minus 10 for each `void` detection (minimum 0) |
   | Image Quality Score | 30% | Laplacian variance (sharpness) divided by 1000, capped at 100 |
   | Gap Density Score | 20% | 100 minus the percentage of the image covered by `void` boxes |

4. Combine them into a **Final Compliance Score** (weighted sum) and print per-image and folder-level summaries.

## Tech stack

| Area | Tools |
| --- | --- |
| Language | Python, Jupyter Notebook |
| Detection | Ultralytics YOLO, pretrained weights from Hugging Face Hub |
| Image processing | OpenCV, NumPy, pandas |
| Data | Kaggle Supermarket Shelves dataset (Humans in the Loop) |

## Results

Saved run over the 45 images of the Kaggle dataset:

| Score | Mean | Min | Max |
| --- | --- | --- | --- |
| Gap Score | 78.0 | 0.0 | 100.0 |
| Image Quality Score | 58.7 | 5.7 | 100.0 |
| Gap Density Score | 98.7 | 88.7 | 100.0 |
| Final Compliance Score | 76.3 | 47.1 | 100.0 |

- 30 of the 45 images have at least one detected gap; 15 have none.
- Lowest final scores: `026.jpg` (47.1), `008.jpg` (49.3), `045.jpg` (51.6). Highest: `012.jpg`, `023.jpg` and `024.jpg` (100.0).
- The gap density score varies little because detected gaps cover a small share of each photo. Image quality varies the most and depends on resolution and sharpness, not on shelf compliance.
- In a manual check of two annotated images, most boxes sat on empty or nearly empty shelf spots. At least one box looked like a false positive, and two overlapping boxes on the same slot were counted separately.

An earlier version of this notebook counted a class named `gap`, which the model does not have, so every image scored 100 for gaps. It now uses the model's real class (`void`).

## Run it

1. Download the shelf images from the [Kaggle dataset](https://www.kaggle.com/datasets/humansintheloop/supermarket-shelves-dataset) and copy the `.jpg` files into a folder named `shelve_images` next to the notebook. The images are not included in this repository.

2. Install dependencies:

   ```bash
   pip install -r requirements.txt
   ```

3. Run the notebook:

   ```bash
   jupyter notebook RSIAIC_Rel_1.ipynb
   ```

## Known limitations

- The model is used as-is; it is not fine-tuned in this notebook. Detection confidence is moderate (0.37 to 0.54 on the two images checked by hand).
- The weights (50 / 30 / 20) and the scoring formulas are illustrative and have not been validated against human audits.
- Overlapping boxes on the same slot are counted separately, which can double-count a gap. The gap score also stops falling after 10 detections.
- Image sharpness is blended into the score. It would work better as a validity check that triggers a retake.
- Gap detection covers empty space, not full planogram compliance (product identity and position).

## Next steps

- Label a small set of images and report precision and recall for gaps.
- Tune the detection confidence and overlap threshold on labelled images.
- Validate the score against human audit results.
- Separate image quality into a pass or retake gate.

## Skills demonstrated

Computer vision, pretrained model integration, diagnosing a silent scoring bug, image quality metrics, scoring design, error handling in batch pipelines.
