# AI-Generated Image Detection

Can you tell a real photo from an AI-generated one? As image generators improve, that gets harder for people to do by eye. This project looks for **measurable differences** between real and AI-generated images using classic image processing, then trains a **convolutional neural network (CNN)** to classify them.

**Result: 96.03% accuracy and 0.96 F1 on 20,000 unseen test images.**

<!-- Add a screenshot here, e.g. the Grad-CAM heatmaps or the confusion matrix -->
<!-- ![Grad-CAM examples](images/gradcam.png) -->

## Dataset

[CIFAKE](https://www.kaggle.com/datasets/birdy654/cifake-real-and-ai-generated-synthetic-images): 120,000 images, all 32×32 pixels, split evenly between two classes.

- **REAL:** photos from [CIFAR-10](https://www.cs.toronto.edu/~kriz/cifar.html)
- **FAKE:** images generated with Stable Diffusion

| Split | REAL | FAKE |
|---|---|---|
| Train | 50,000 | 50,000 |
| Test | 10,000 | 10,000 |

## Part 1: Image Processing Analysis

Before training a model, I measured four properties across all 100,000 training images to see which ones differ between the classes. Each looks at a different aspect of an image: frequency, edges, texture, and color.

| Technique | How | Finding |
|---|---|---|
| **Fast Fourier Transform** | Averaged 2D frequency spectra per class, plus a radial frequency profile | Average spectra look nearly identical, but the radial profile shows FAKE images carry **more mid-to-high frequency energy** (finer detail) |
| **Canny edge detection** | Edge density = fraction of pixels marked as edges (thresholds 50–150) | REAL images have **more edges**: mean density 0.255 vs. 0.224 |
| **Local standard deviation** | 3×3 averaging kernel with `cv2.filter2D`, Var = E[X²] − E[X]² | FAKE images are **noisier** at the pixel level (18.38 vs. 16.94), even though they look smoother |
| **RGB color histograms** | Average 64-bin histogram per channel | FAKE images contain noticeably **more dark blue pixels**; REAL images have more red and green at maximum brightness |

None of these separates the classes reliably on its own, since the distributions overlap. Together, though, they show that diffusion-generated images differ from real photos in consistent, low-level ways. That motivated a CNN, which can learn to combine signals like these automatically.

## Part 2: CNN Classifier

**Input pipeline:** `tf.data` loading directly from disk with prefetching. Training data is split 85,000 / 15,000 into train and validation sets, and training images are augmented with random horizontal flips and brightness changes.

**Architecture:** a custom CNN (about 305K parameters) designed for 32×32 inputs:

- 3 convolutional blocks (32 → 64 → 128 filters), each with two 3×3 Conv2D layers, Batch Normalization, Max Pooling, and 25% Dropout
- Global Average Pooling → Dense(128) → 50% Dropout → sigmoid output

**Training:** Adam optimizer (learning rate 1e-3) with binary cross-entropy loss, plus three callbacks:

- `EarlyStopping` on validation accuracy (patience 5, restores best weights)
- `ReduceLROnPlateau` to halve the learning rate when validation loss stalls
- `ModelCheckpoint` to save the best model

## Results

| Class | Precision | Recall | F1 |
|---|---|---|---|
| FAKE | 0.98 | 0.94 | 0.96 |
| REAL | 0.95 | 0.98 | 0.96 |
| **Overall accuracy** | | | **96.03%** |

The model is slightly better at recognizing real images (98% recall) than fake ones (94%). When it's wrong, it's usually an AI image passing as real.

**Grad-CAM** heatmaps show the model focusing on edges, backgrounds, and textured regions rather than the main subject. That matches the low-level differences found in Part 1, which suggests the model learned generation artifacts rather than image content.

## Running the Notebook

The notebook runs top to bottom in Google Colab with a GPU.

1. Get a Kaggle API key (Kaggle → Settings → API).
2. In Colab, open **Secrets** (key icon in the left sidebar) and add `KAGGLE_USERNAME` and `KAGGLE_KEY`, with notebook access turned on.
3. Run all cells. The first two cells download and unzip the dataset.

**Without a GPU**, scale it down: lower `max_per_class` in the data loading and pipeline cells (for example, to 500 and 5,000) and reduce `epochs` from 20 to 3 or 4.

## Built With

Python · TensorFlow / Keras · OpenCV · NumPy · scikit-learn · Matplotlib

## Limitations & Future Work

- **Resolution:** CIFAKE images are only 32×32. Downscaling real-world images that small likely destroys the artifacts the model relies on, so the next step is training on higher-resolution data.
- **Generalization:** all FAKE images come from one generator (Stable Diffusion). Testing on images from other generators would show whether the model learned general AI artifacts or ones specific to Stable Diffusion.
- **Web app:** deploy the model behind an API with a simple web front end where anyone can upload an image and get a prediction.
- **More techniques:** additional feature extraction and morphological processing.
