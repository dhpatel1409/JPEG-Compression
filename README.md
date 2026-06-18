# JPEG Compression from Scratch 🖼️

> A complete implementation of the JPEG compression pipeline — encoding and decoding — built from scratch using NumPy and OpenCV.

**Authors:** Dharmik Patel (24EC65R10) · Aryan Simul Udani (24EC65R28)

---

## 📌 Overview

This project implements the full JPEG compression and decompression pipeline without relying on any JPEG library. It demonstrates the mathematical foundations behind one of the world's most widely used image formats, achieving compression ratios of up to **10.3:1** on standard test images.

### Pipeline

```
Original Image (RGB)
        │
        ▼
┌─────────────────────┐
│ Color Space          │  RGB → YCbCr (separates luma from chroma)
│ Conversion           │
└────────┬────────────┘
         │
         ▼
┌─────────────────────┐
│ Block Splitting      │  Divide into 8×8 macroblocks
└────────┬────────────┘
         │
         ▼
┌─────────────────────┐
│ DCT (2D)            │  Spatial → Frequency domain
└────────┬────────────┘
         │
         ▼
┌─────────────────────┐
│ Quantization         │  Lossy step — discard high-freq info
└────────┬────────────┘
         │
         ▼
┌─────────────────────┐
│ Zig-Zag Scanning     │  2D → 1D, groups zeros together
└────────┬────────────┘
         │
         ▼
┌─────────────────────┐
│ Run-Length Encoding  │  Lossless compression of zero runs
└────────┬────────────┘
         │
         ▼
    Compressed Bitstream

         ▼  (Decoding reverses all steps)

Reconstructed Image
```

---

## 📁 Project Structure

```
jpeg-compression/
├── jpeg.ipynb              # Main Jupyter notebook (full pipeline)
├── jpeg_compression.py     # Standalone Python script
├── requirements.txt        # Python dependencies
├── README.md
├── docs/
│   ├── JPEG_Compression_Report.docx    # Detailed technical report
│   └── JPEG_Compression_Summary.pptx  # Presentation slides
└── images/
    └── cameraman.bmp       # Test image (place your .bmp files here)
```

---

## ⚙️ Installation

**Requirements:** Python 3.8+

```bash
git clone https://github.com/your-username/jpeg-compression.git
cd jpeg-compression
pip install -r requirements.txt
```

---

## 🚀 Usage

### Jupyter Notebook (Recommended)

```bash
jupyter notebook jpeg.ipynb
```

Run all cells in order — the notebook walks through each stage of the pipeline with printed intermediate outputs.

### Python Script

```bash
python jpeg_compression.py
```

By default it loads `cameraman.bmp`. To use a different image, edit the `cv2.imread(...)` call at the bottom of the script.

---

## 📐 Algorithm Details

### 1. Color Space Conversion — `BGR_to_YCbCr`

Converts the BGR image (OpenCV default) to YCbCr, separating luminance (Y) from chrominance (Cb, Cr). The human visual system is less sensitive to color detail than brightness, so chroma channels can be compressed more aggressively.

```
Y  =  0.299·R + 0.587·G + 0.114·B
Cb = 128 − 0.168736·R − 0.331264·G + 0.5·B
Cr = 128 + 0.5·R − 0.418688·G − 0.081312·B
```

### 2. Macroblock Splitting — `macroblock`

Pads the image (if needed) to a multiple of 8, then slices each channel into non-overlapping 8×8 blocks. Each block is processed independently.

### 3. 2D Discrete Cosine Transform — `dct_2d`

Applies the 1D DCT row-wise then column-wise. The DCT concentrates most image energy into a small number of low-frequency coefficients, making subsequent quantization effective.

Before DCT, each block is level-shifted by −128 (centering the range from [0, 255] to [−128, 127]).

### 4. Quantization

Each DCT coefficient is divided by a corresponding entry in the standard JPEG quantization table and rounded to the nearest integer. Higher-frequency entries in the table are larger, aggressively discarding perceptually unimportant data.

| Table | Used for |
|-------|----------|
| `quantization_Y` | Luminance (Y channel) |
| `quantization_C` | Chrominance (Cb, Cr channels) |

### 5. Zig-Zag Scanning — `zig_zag_scanning`

Reorders the 8×8 quantized block into a 1D array following a zig-zag path. This groups the typically-nonzero low-frequency coefficients at the start and the many zeros at the end — ideal for RLE.

### 6. Run-Length Encoding — `run_length_encoding`

Encodes runs of zeros between non-zero AC coefficients as `(RUNLENGTH, SIZE, AMPLITUDE)` triplets, where:
- `RUNLENGTH` = number of preceding zeros
- `SIZE` = bit-width of the amplitude
- `AMPLITUDE` = the coefficient value

### 7. Compression Ratio

```
Compression Ratio = bits_original / bits_compressed
```

The original image requires `H × W × 3 × 8` bits. The compressed representation uses the total bits across RLE triplets for Y, Cb, and Cr.

### 8. Decoding (Decompression)

The decoding pipeline reverses every step:

`Inverse Zig-Zag → Dequantization → Inverse DCT → Level Shift (+128) → Merge Blocks → YCbCr → BGR`

---

## 📊 Results

Test images used: `corn.bmp`, `pepper.bmp`, `cameraman.bmp`

| Image | Original Size | Compression Ratio |
|-------|--------------|-------------------|
| corn.bmp | — | **5.1 : 1** |
| pepper.bmp | — | **8.1 : 1** |
| cameraman.bmp | — | **10.3 : 1** |

The reconstructed image is visually close to the original with the standard quantization tables, consistent with real-world JPEG behavior (typically 10:1 with minimal perceptible loss).

---

## 🔑 Key Functions Reference

| Function | Description |
|---|---|
| `BGR_to_YCbCr(img)` | Color space conversion |
| `macroblock(img, channel)` | Split channel into 8×8 blocks |
| `dct_1d(x)` | 1D Discrete Cosine Transform |
| `dct_2d(img)` | 2D DCT via separable 1D DCT |
| `zig_zag_scanning(x)` | 2D block → 1D zig-zag ordered array |
| `run_length_encoding(x)` | RLE on zig-zag scanned coefficients |
| `compression(image)` | Full encoding pipeline |
| `inverse_zig_zag_scanning(x, n)` | Reverse zig-zag |
| `inverse_dct_1d(x)` | Inverse 1D DCT |
| `inverse_dct_2d(img)` | Inverse 2D DCT |
| `decompression(e_Y, e_Cb, e_Cr, ...)` | Full decoding pipeline |
| `YCbCr_to_BGR(ycbcr)` | Inverse color space conversion |
| `max_bits(x)` | Estimate bits needed for RLE output |

---

## 📚 References

- Wallace, G.K. (1991). *The JPEG still picture compression standard.* IEEE Transactions on Consumer Electronics.
- [Wikipedia — JPEG](https://en.wikipedia.org/wiki/JPEG)
- [Wikipedia — Discrete Cosine Transform](https://en.wikipedia.org/wiki/Discrete_cosine_transform)
- [Wikipedia — Run-length encoding](https://en.wikipedia.org/wiki/Run-length_encoding)

---

## 📄 License

This project is for academic purposes. Feel free to use, study, and extend it.
