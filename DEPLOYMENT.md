# V5 Production Deployment Guide

This directory (`v5_prod`) contains the **pure execution environment** for the Bangladesh Reverse Geocoder. It contains zero development logic and is 100% prepared for production scaling.

## Can I upload this to Hugging Face via GitHub?
**Yes, absolutely.** Hugging Face Spaces natively supports syncing from a GitHub repository. 

However, because your `.bin` files are large (e.g., `sparse_grid.bin` is ~200MB and `master_response_strings.bin` is ~240MB), **you must use Git Large File Storage (Git LFS)** whether you push to GitHub or directly to Hugging Face.

Here are the step-by-step instructions for both methods.

---

### Prerequisites (For both methods)
You need to install `git-lfs` on your computer, as standard Git cannot track files larger than 100MB.

**Mac:**
```bash
brew install git-lfs
git lfs install
```

---

## Method 1: Deploying via GitHub (Recommended)
This is the best method if you want to keep your code open-source on GitHub and have Hugging Face automatically update whenever you push to GitHub.

### Step 1: Initialize Git and LFS in `v5_prod`
Open your terminal inside the `v5_prod` folder:
```bash
cd v5_prod
git init
git lfs install

# Tell Git LFS to track all binary data files
git lfs track "*.bin"

# This creates a .gitattributes file. Add it tracking:
git add .gitattributes
```

### Step 2: Commit and Push to GitHub
```bash
git add .
git commit -m "Initial V5 Prod Commit with Sparse Matrices"
git branch -M main
git remote add origin https://github.com/YOUR_USERNAME/YOUR_REPO_NAME.git
git push -u origin main
```
*Note: The push might take a few minutes depending on your internet speed, as it uploads ~450MB of LFS data.*

### Step 3: Connect Hugging Face to GitHub
1. Go to **Hugging Face > Create New Space**.
2. Select **Docker** as the Space SDK (Blank template).
3. Under the "Space hardware" section, the **Free tier (16GB RAM, 2 vCPU)** is more than enough.
4. Instead of creating a blank space, look for the option or button that says **"Duplicate from GitHub"** or you can link your GitHub account in your HF Profile settings and provision the Space directly from your repo.
5. Hugging Face will pull your Dockerfile, download the LFS `.bin` files, build the image, and host it!

---

## Method 2: Deploying Directly to Hugging Face (Faster Setup)
If you don't want a GitHub repository and just want to host the API immediately.

### Step 1: Create a Space on Hugging Face
1. Go to Hugging Face and click **Create New Space**.
2. Name it (e.g., `bd-reverse-geocoder`).
3. Choose **Docker** as the Space SDK (Blank).
4. Create the Space.

### Step 2: Clone the HF Space and Push
Hugging Face gives you a git link for your Space. In your terminal (outside of `v5_prod`):

```bash
# Clone the empty Hugging Face space
git clone https://huggingface.co/spaces/YOUR_USERNAME/bd-reverse-geocoder
cd bd-reverse-geocoder

# Install LFS for this tracking repo
git lfs install
git lfs track "*.bin"
git add .gitattributes

# Copy everything from v5_prod into this folder
cp -R ../v5_prod/* .

# Push straight to Hugging Face
git add .
git commit -m "Deploy V5 Prod with 0.8ms Lookup"
git push
```

Hugging Face will instantly detect the `Dockerfile`, build the Python 3.11 environment, load the `.bin` matrix into memory, and expose port `7860`. Your API will be live globally!