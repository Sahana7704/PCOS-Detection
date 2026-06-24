# PCOS Detection System with Medical Image Security

A deep learning pipeline for PCOS (Polycystic Ovary Syndrome) detection from ultrasound images, integrated with a DCT-domain invisible watermarking system to ensure medical image integrity and tamper detection.

---

## Overview

This project combines two core components:

1. **PCOS Classification** — A VGG16-based CNN trained to classify ultrasound images as PCOS or Non-PCOS
2. **Medical Image Security** — An invisible DCT watermarking system that embeds unique identifiers into images and uses SHA-256 hashing to detect any tampering

A Streamlit web app serves as the front end for the tamper detection module, allowing users to upload images and instantly verify their integrity against the hash registry.

---

## Features

- **DCT Spread-Spectrum Watermarking** — Embeds an invisible image ID into the frequency domain of each medical image
- **SHA-256 Hash Registry** — Maintains a hash of every watermarked image for tamper verification
- **VGG16 Transfer Learning** — Fine-tuned on PCOS ultrasound images with CLAHE preprocessing and 2D→3D volume conversion
- **Streamlit Tamper Detector UI** — Upload any image and check INTACT / TAMPERED / UNKNOWN status in real time
- **Pixel Diff Visualizer** — Side-by-side comparison with amplified pixel difference map when tampering is detected
- **Colab + Google Drive Integration** — Full pipeline runs on Google Colab with Drive-mounted dataset

---

## Tech Stack

| Layer | Tools |
|---|---|
| Deep Learning | TensorFlow / Keras, VGG16 |
| Image Processing | OpenCV, PIL, NumPy |
| Security | DCT watermarking, SHA-256 hashing |
| Web App | Streamlit |
| Deployment | ngrok (Colab tunneling) |
| Data | Scikit-learn (train/test split, metrics) |

---

## Project Structure

```
PCOS_SECURITY.ipynb        # Main notebook (full pipeline)
app.py                     # Streamlit tamper detection app (extracted from notebook)
hash_registry.json         # Generated after watermarking — required by app.py
secured_images/
├── PCOS/                  # Watermarked PCOS images
└── Non-PCOS/              # Watermarked Non-PCOS images
```

---

## Getting Started

### 1. Dataset Setup

Upload your PCOS ultrasound dataset to Google Drive with this structure:

```
MyDrive/PCOS/
├── infected/       ← PCOS images
└── noninfected/    ← Non-PCOS images
```

### 2. Run the Notebook on Google Colab

Open `PCOS_SECURITY.ipynb` in [Google Colab](https://colab.research.google.com), mount your Drive, and run all cells in order:

1. **Mount Drive & verify paths** — confirms dataset is accessible
2. **Watermark images** — embeds DCT watermarks and saves to `secured_images/`
3. **Preprocess & train** — applies CLAHE enhancement, builds VGG16 model, trains for 20 epochs
4. **Save model** — exports `pcos_model_final.h5` to Drive
5. **Run Streamlit app** — launches via ngrok and prints a public URL

### 3. Export the Hash Registry

After watermarking, run this in a Colab cell to save the registry:

```python
import json

registry_path = "/content/drive/MyDrive/PCOS/hash_registry.json"
with open(registry_path, "w") as f:
    json.dump(hash_registry, f, indent=2)

print(f"Saved {len(hash_registry)} hashes to {registry_path}")
```

Download `hash_registry.json` and place it alongside `app.py` for the Streamlit app to load.

---

## Model Configuration

| Parameter | Value |
|---|---|
| Input size | 224 × 224 |
| Base model | VGG16 (ImageNet weights) |
| Depth slices | 5 |
| Batch size | 16 |
| Epochs | 20 |
| Learning rate | 1e-4 |
| Sample fraction | 50% (configurable) |

---

## Watermarking Details

The `DCTWatermark` class implements an invisible spread-spectrum watermark in the frequency domain:

- Converts the image to grayscale and splits into 8×8 blocks
- Applies DCT to each block and modulates a mid-frequency coefficient based on the encoded image ID bits
- Embeds back via inverse DCT, then overlays on the green channel of the original image
- SHA-256 hash of the watermarked image is stored in the registry

Verification computes the hash of the uploaded image and compares it against the registry entry.

---

## Running the Streamlit App Locally

```bash
pip install streamlit opencv-python tensorflow scikit-learn

# Place hash_registry.json and secured_images/ in the same directory as app.py
streamlit run app.py
```

The app supports two detection modes:
- **Auto-detect (hash scan)** — scans the full registry to find a match
- **Manual ID entry** — provide the image ID directly for targeted lookup

---

## References

- Eswaraiah, R. & Sreenivasa Reddy, E. — *Medical Image Watermarking Technique for Accurate Tamper Detection in ROI and Exact Recovery of ROI*
- VGG16: Simonyan & Zisserman, *Very Deep Convolutional Networks for Large-Scale Image Recognition* (2014)
