# AI Image Detector — Project Plan

Turning a Colab coursework project (CIFAKE CNN, reported 0.96 F1 — not
macro F1 and measured with a leaky validation split; see Background) into a refined,
tested, deployed web application.

**Current section: 2.1**

Progress markers: `[ ]` not started, `[~]` in progress, `[x]` done.

---

## Background: findings from the original notebook

Source: `notebooks/original_cifake.ipynb`. The original Colab project lives in
`../AIDetector/` (dataset, venv, trained `best_cnn.keras`) and is kept as an archive.

- **Validation leak.** `make_dataset` is called separately for train and val, each
  call runs `random.shuffle` with no seed, so ~85% of validation images are also
  training images. Test set (separate folder) is unaffected.
- **Inconsistent labels.** Analysis code uses REAL=0/FAKE=1; the model uses
  FAKE=0/REAL=1. Grad-CAM's `x[:, 0]` is P(REAL), so fake-image heatmaps show
  "what looks real," not "what gives it away."
- **Printed F1 is not macro F1.** `f1_score(y, p)` is binary F1 for REAL only.
- **`max_per_class` is unused.**
- **Resolution.** CIFAKE is 32×32, with fakes from Stable Diffusion v1.4 only.
  Expected to generalize poorly to modern, full-resolution images.

## Environment (checked in 1.3)

Python 3.12.5 (only install), Git 2.46, Node 22.13, Docker 27.5 (WSL2 backend),
RTX 3080 10 GB, 32 GB RAM, ~56 GB free on C:. WSL2 Ubuntu distro already installed.
TensorFlow has no GPU support on native Windows (since 2.11), so the old
`cifake_env` (TF 2.21) is CPU-only.
**Decision:** develop and run Chapter 2 training on native Windows CPU (Colab if too
slow); revisit WSL2 + GPU at the start of Chapter 4.

## Repository (set up in 1.4)

- Remote: https://github.com/HDoubleH/ai-image-detection (public). Local folder name
  differs (`ai-image-detector`); harmless.
- The remote had 10 prior coursework commits (browser uploads/deletes); merged with
  `--allow-unrelated-histories` instead of force-pushing. The old 81-line README is
  recoverable via `git show 18ba11e:README.md` (possibly useful for 11.5).
- A stray, commit-less repo at `C:\Users\owenh\.git` was deleted in 1.4.
- Git global: `init.defaultBranch=main`, commit email set to the GitHub account's.
  `gh` CLI is not installed; GitHub is used via the web UI + Git Credential Manager.

## Session notes (handoff for the next session)

- **Chapter 1 is complete.** Next: **2.1** (start with the Chapter 2 overview).
  At session start, run `git status`: commit/push anything left over first.
- **Python env:** `.venv` in the project root (Python 3.12.5), direct deps pinned in
  `requirements.txt` (TF 2.21.0, Keras 3.x). Optional deps from the notebook (`cv2`,
  `matplotlib`, `tqdm`) are deliberately *not* installed until a section needs them.
- **Model facts (verified in 1.5):** `best_cnn.keras` loads in the new env; input
  `(None, 32, 32, 3)`, output `(None, 1)` sigmoid = **P(REAL)** (class 1 = REAL in the
  notebook). Loading warns about skipped RMSprop optimizer state — harmless for
  inference; use `load_model(..., compile=False)` when serving (5.7).
- Old `../AIDetector/cifake_env` (2.4 GB) is no longer needed; Owen was told to delete it.
- **Concepts to reinforce** (Owen's answers were close but imprecise):
  training–serving skew = training and serving code *preprocess differently* (not
  "training on the server"); rejected/non-fast-forward push vs. merge conflict;
  leaked key → *rotate first*, then clean history. Revisit briefly when relevant
  (2.6, Ch. 9, 10.5).

---

## Chapter 1 — Project Foundations
- [x] 1.1 Project and architecture overview (no implementation)
- [x] 1.2 Understanding the technology stack
- [x] 1.3 Setting up the development environment (check what's installed; GPU vs Colab for training)
- [x] 1.4 Creating the repository (git init, .gitignore for data/venv/model weights, GitHub, first commit)
- [x] 1.5 Python and virtual environments

## Chapter 2 — From Notebook to Python Project
- [ ] 2.1 Why notebooks don't ship (state, ordering, reproducibility)
- [ ] 2.2 Project layout and Python packages (`src/` layout)
- [ ] 2.3 Configuration (paths, image size, seeds in one place)
- [ ] 2.4 Data loading module with one label convention
- [ ] 2.5 Fixing the validation leak — *you implement the split*
- [ ] 2.6 Shared preprocessing module (training–serving skew)
- [ ] 2.7 Training script
- [ ] 2.8 Evaluation script with correct metrics (macro F1, confusion matrix)
- [ ] 2.9 First tests with pytest (preprocessing, labels, split has no overlap)
- [ ] 2.10 Reproducing the baseline and recording results

## Chapter 3 — Measuring Real-World Performance
- [ ] 3.1 In-distribution vs out-of-distribution evaluation
- [ ] 3.2 Building a small real-world test set (modern AI images + phone photos)
- [ ] 3.3 Evaluating the baseline on real-world images
- [ ] 3.4 Shortcut learning, and Grad-CAM revisited (with correct target class)

## Chapter 4 — Building a Better Model
- [ ] 4.1 Choosing a dataset (higher resolution, multiple generators, license)
- [ ] 4.2 Data pipeline at higher resolution
- [ ] 4.3 Transfer learning with a pretrained backbone
- [ ] 4.4 Alternative: pretrained image embeddings (e.g. CLIP) + simple classifier
- [ ] 4.5 Cross-generator evaluation (train on some generators, test on unseen ones)
- [ ] 4.6 Calibration and choosing a decision threshold
- [ ] 4.7 Tracking experiments simply (results file, not a platform)
- [ ] 4.8 Revisiting the image-processing analysis at high resolution
- [ ] 4.9 Model card: what the model can and can't do

## Chapter 5 — The Inference Backend
- [ ] 5.1 What a backend actually does (clients, servers, HTTP, ports)
- [ ] 5.2 Creating the FastAPI application
- [ ] 5.3 Understanding REST APIs
- [ ] 5.4 First GET endpoint (health check)
- [ ] 5.5 POST /predict with image upload — *you implement part*
- [ ] 5.6 Request/response models with Pydantic
- [ ] 5.7 Loading the model once at startup
- [ ] 5.8 Validating uploads (type, size, corrupt files)
- [ ] 5.9 Explanation endpoint (Grad-CAM heatmap)
- [ ] 5.10 Privacy: not storing user uploads

## Chapter 6 — The Web Application
- [ ] 6.1 Frontend architecture (browser → Next.js → FastAPI)
- [ ] 6.2 Setting up Next.js and TypeScript
- [ ] 6.3 React components (only what we need)
- [ ] 6.4 Image upload UI
- [ ] 6.5 Connecting to FastAPI (and CORS)
- [ ] 6.6 Displaying prediction, confidence, and heatmap
- [ ] 6.7 Loading and error states
- [ ] 6.8 Communicating limitations honestly in the UI

## Chapter 7 — Testing the System
- [ ] 7.1 Useful vs pointless tests
- [ ] 7.2 API tests with pytest and FastAPI's TestClient
- [ ] 7.3 Training–serving parity tests
- [ ] 7.4 Model smoke tests
- [ ] 7.5 Failure cases (bad files, huge files, wrong formats)

## Chapter 8 — Docker
- [ ] 8.1 Why containers exist
- [ ] 8.2 Images vs containers
- [ ] 8.3 Writing the Dockerfile
- [ ] 8.4 Running the backend in Docker
- [ ] 8.5 Image size: TensorFlow vs TFLite/ONNX

## Chapter 9 — CI with GitHub Actions
- [ ] 9.1 What CI/CD means
- [ ] 9.2 First workflow: run tests on push
- [ ] 9.3 Blocking broken code from merging
- [ ] 9.4 Building the Docker image in CI

## Chapter 10 — Deployment
- [ ] 10.1 Minimum production architecture
- [ ] 10.2 Choosing hosting (AWS vs simpler options; cost and billing alarms)
- [ ] 10.3 Deploying the backend container
- [ ] 10.4 Deploying the frontend
- [ ] 10.5 Production configuration and secrets
- [ ] 10.6 Verifying the deployed flow

## Chapter 11 — Resume-Ready
- [ ] 11.1 End-to-end architecture review
- [ ] 11.2 Failure modes
- [ ] 11.3 Performance (cold starts, latency, CPU inference)
- [ ] 11.4 Security and privacy (uploads, abuse, rate limiting)
- [ ] 11.5 README with architecture diagram
- [ ] 11.6 Resume bullets (truthful, with real numbers)
- [ ] 11.7 Interview preparation

## Chapter 12 — Optional Extensions
- [ ] 12.1 Image-processing "explain this image" panel
- [ ] 12.2 User feedback on wrong predictions
- [ ] 12.3 Rate limiting
- [ ] 12.4 Second model: Steam project (deferred)
