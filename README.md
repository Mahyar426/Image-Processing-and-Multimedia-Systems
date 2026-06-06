# 🎛️ Image Processing & Multimedia Systems — MATLAB Implementation Lab

> **From pixels to frequency domains.** A deep-dive into the mathematics of visual signal processing — implementing core image processing algorithms from scratch in MATLAB, culminating in a DCT-based blind digital watermarking system.

![MATLAB](https://img.shields.io/badge/MATLAB-R2020b+-orange?style=flat-square&logo=mathworks)
![Domain](https://img.shields.io/badge/Domain-Image%20Processing-blueviolet?style=flat-square)
![Algorithms](https://img.shields.io/badge/Algorithms-DCT%20%7C%20Histogram%20%7C%20Frequency%20Filtering-blue?style=flat-square)
![Institution](https://img.shields.io/badge/Isfahan%20University%20of%20Technology-Electrical%20Engineering-green?style=flat-square)

---

## What This Is

This repository documents hands-on implementation work in digital image processing and multimedia systems — covering spatial domain operations, frequency domain analysis (DCT/FFT), histogram-based enhancement, and a full digital watermarking pipeline built from the ground up.

Every algorithm here was written without relying on MATLAB's high-level toolbox wrappers. The goal: understand the math by implementing it manually, then verify against built-ins.

---

## Technical Breakdown

### 🔲 Spatial Domain Operations
*Geometric transforms, pixel-level manipulation, cyclic shifts*

Built a general-purpose spatial transformation engine for grayscale images:
- Horizontal/vertical flip, transpose, diagonal shifts — all implemented via explicit pixel-index arithmetic (no `fliplr`, no `flipud`)
- Cyclic boundary handling for directional shifts (wrap-around logic in both axes)
- Central crop extraction using ceiling-division for symmetric margins

**Why it matters:** Understanding index arithmetic on 2D matrices is foundational. MATLAB's convenience functions hide the mechanics — implementing them manually exposes how image geometry actually works.

---

### 📊 Histogram Equalization — Global & Local
*CDF-based contrast enhancement, from scratch*

Implemented full histogram equalization pipeline:
- Manual pixel-frequency counting → PDF computation → CDF accumulation → intensity remapping
- Global equalization across the full image
- Local/adaptive equalization over sliding windows (`hw2_local_histeq.m`) — critical for images with non-uniform illumination
- Image hiding/steganography in the LSB domain (`hw2_hide.m`)

**Key insight:** Global equalization can blow out locally-dark regions. The local variant reveals detail invisible to standard enhancement — this distinction matters in medical imaging and satellite imagery.

---

### 🌊 Discrete Cosine Transform (DCT) — Compression & Analysis
*Block-based DCT with threshold-controlled coefficient zeroing*

Implemented a block-DCT compression simulation:
- Partitions the image into `K×K` blocks (padding to fit)
- Applies `dct2()` per block, then zeros all coefficients below threshold `t`
- Reconstructs via `idct2()` and computes full-image **PSNR** and **zeroed-coefficient percentage**
- Median filter implementation (`hist_median.m`) for noise-resilient smoothing

```
[J, PSNR, zero_pct] = HW3_dct(I, K, t)
```

This directly models how JPEG compression works — trading perceptual quality for a tunable coefficient sparsity.

---

### 🔀 Hybrid Image Synthesis — Frequency Domain Fusion
*Gaussian filtering in spatial domain to simulate human visual system behavior*

Fused two images so they appear different at near vs. far viewing distances:
- Low-frequency component (smooth Gaussian) from Image A
- High-frequency component (image minus Gaussian) from Image B
- Weighted combination: `K = 0.5 * I_lowpass + 0.5 * (J - 0.4 * J_lowpass)`

The result exploits how the human eye processes spatial frequencies differently with distance — a core concept in visual perception research.

---

### 🎨 Color-Selective Region Replacement
*Interactive pixel classification by RGB similarity*

Built an interactive color replacement tool:
- User clicks a point on the image → that pixel's RGB value is sampled
- All pixels within tolerance `w` in all three channels are recolored
- Demonstrates direct RGB-space nearest-neighbor classification — a simplified version of what color-keying (green screen) algorithms do

---

### 🔐 DCT-Domain Blind Digital Watermarking *(Final Project)*
*Full embed → attack → extract → evaluate pipeline*

The most complex module: a **blind watermarking system** — meaning watermark recovery requires no access to the original image.

**Embedding (`embed_proj.m`)**
- Partitions the host image into `B×B` DCT blocks
- The watermark logo is resized, flattened to a 1D bitstream, and **randomly permuted** using a secret key `K` (via `randperm` seeded with `rng(K)`)
- Per-block embedding: adjusts the relative magnitude of two adjacent DCT coefficients `(a, a+1)` and `(a+1, a)` to encode a `0` or `1` bit
- Outputs PSNR between original and watermarked image

**Attack Simulation (`attack_proj.m`)**
- Compresses the watermarked image at JPEG quality levels 60, 80, and 100
- Extracts watermark bits from each attacked image independently
- Reverses the random permutation using the same key to recover the bitstream

**Evaluation (`NC_calculator.m`)**
- Computes **Normalized Correlation (NC)** between original and recovered watermark at each quality level
- NC ≈ 1.0 → watermark survived. NC << 1.0 → attack was effective.

```
NC60, NC80, NC100 → quantitative robustness report
```

This is the exact evaluation framework used in watermarking research papers (e.g., IEEE transactions). NC is the standard metric for perceptual watermark fidelity.

---

## Repository Structure

```
├── HW1/          # Spatial transforms & MSE analysis
├── HW2/          # Histogram equalization (global + local) & LSB hiding
├── HW3/          # Block-DCT compression & median filtering
├── HW4/          # Gaussian frequency filtering & hybrid image synthesis
├── HW5/          # RGB color-space interactive region replacement
└── Project/      # Full blind watermarking system (embed + attack + evaluate)
```

---

## Skills Demonstrated

| Area | Specifics |
|---|---|
| Signal Processing | DCT, spatial filtering, frequency-domain manipulation |
| Image Enhancement | Histogram equalization (global & adaptive), PSNR/MSE evaluation |
| Cryptographic Concepts | Keyed random permutation for watermark security |
| Algorithm Design | From-scratch implementations without high-level toolbox shortcuts |
| Quantitative Evaluation | PSNR, NC metrics — standard in compression/watermarking literature |

---

## How to Run

All scripts use MATLAB R2020b+. No additional toolboxes required beyond the Image Processing Toolbox (for `imshow`, `psnr`).

```matlab
% Example: DCT compression with 8x8 blocks, threshold = 10
I = imread('HW3/Hi.tif');
J = HW3_dct(I, 8, 10);

% Example: Watermark embedding
I = imread('Project/lena.bmp');
W = imread('Project/iut5.bmp');
W_image = embed_proj(I, 8, 2, W, 42, 5);  % key=42, alpha=5

% Example: Watermark attack & evaluation
attack_proj(W_image, 8, 2, 42, W);
```

---

*Part of the Electrical Engineering undergraduate curriculum at Isfahan University of Technology.*
