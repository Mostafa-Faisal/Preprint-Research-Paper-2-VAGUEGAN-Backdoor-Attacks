#  VagueGAN: Stealthy Poisoning and Backdoor Attacks on Image Generative Pipelines

**Authors:**  
Mostafa Mohaimen Akand Faisal  
Rabeya Amin Jhuma  
University of Information Technology and Sciences (UITS), Bangladesh  

📧 Contact: [mostafafaisal013@gmail.com](mailto:mostafafaisal013@gmail.com), [r.a.jhuma2019@gmail.com](mailto:r.a.jhuma2019@gmail.com)

---

## 🔍 Overview

**VagueGAN** is a research framework for **latent-space poisoning and backdoor attacks** targeting image generative pipelines.  
It demonstrates how imperceptible perturbations—crafted in the input or latent space—can embed hidden behaviors into **Generative Adversarial Networks (GANs)** and propagate through downstream models such as **Stable Diffusion + ControlNet**.

Unlike conventional attacks, VagueGAN reveals that poisoned outputs can often **appear more visually appealing**, introducing the unique “**beauty as stealth**” phenomenon—a new security challenge for generative AI.

> ⚠️ **Disclaimer:**  
> This repository is intended strictly for academic research and educational purposes.  
> Its aim is to foster understanding of generative model vulnerabilities, not to promote or enable malicious use.

---

## 🧩 Key Features

- 🔬 **PoisonerNet** – Learns to generate structured, imperceptible backdoor signals.
- 🌀 **Stealthy Latent-Space Poisoning** – Embeds adversarial behaviors without visible artifacts.
- 🎛️ **GAN + Diffusion Hybrid Testing** – Supports experiments with Stable Diffusion for cross-architecture transfer.
- ⚖️ **Stealth–Strength Tradeoff Analysis** – Quantifies backdoor strength versus perceptual similarity.
- 💎 **Beauty-as-Stealth** – Poisoned samples may look more aesthetically pleasing.

---

## ⚙️ Experimental Workflow
        ──────────────────────────────────────────────────────────────────────────────
                      📘 INPUT STAGE (DATA & PREPROCESSING)
──────────────────────────────────────────────────────────────────────────────
             ┌──────────────────────────────────────────────┐
             │  Single Scratch Image / Dataset (CelebA, CIFAR-10) │
             └──────────────────────────────────────────────┘
                             │
                             ▼
                  ┌──────────────────────────┐
                  │ Image Preprocessing       │
                  │ • Resize: 128×128 (GAN)   │
                  │ • Normalize: [-1, 1]      │
                  │ • Edge Map (Canny/Laplacian) for ControlNet │
                  └──────────────────────────┘
                             │
                             ▼
──────────────────────────────────────────────────────────────────────────────
                     🧩 POISONED INPUT GENERATION (PoisonerNet)
──────────────────────────────────────────────────────────────────────────────
                ┌──────────────────────────────────────────┐
                │ PoisonerNet (P)                          │
                │ • Inputs: clean image x, latent zp       │
                │ • Generates structured perturbation δ     │
                │ • δ bounded by ℓ∞ ≤ ϵ (ϵ = 0.08)        │
                │ • Applies: x′ = clip(x + δ, -1, 1)      │
                └──────────────────────────────────────────┘
                             │
         ┌────────────────────┴─────────────────────┐
         │                                          │
   Clean sample (x)                      Poisoned sample (x′ = x + δ)
         │                                          │
         └──────────────┬────────────────────────────┘
                        ▼
          (Sample chosen via probabilistic injection)
            poison_rate α = 0.3 → 30% inputs poisoned

──────────────────────────────────────────────────────────────────────────────
                     🎨 GENERATOR (G) — SYNTHESIS STAGE
──────────────────────────────────────────────────────────────────────────────
          ┌───────────────────────────────────────────┐
          │  Generator G(x, z, f)                     │
          │  ───────────────────────────────────────  │
          │  Inputs:                                   │
          │   • Image (clean or poisoned) x/x′         │
          │   • Latent vector z (R¹²⁸)                │
          │   • Feature vector f (R¹⁰)                │
          │  Process:                                  │
          │   1. Project z & f → spatial maps          │
          │   2. Concatenate with image → 5-channel    │
          │   3. Convolutional stack (ReLU activations)│
          │   4. Output = tanh(Conv_final) → ³-channel │
          └───────────────────────────────────────────┘
                             │
                             ▼
           ◀────────────────────────────────────────────▶
           High-quality synthesized image (realistic output)
                             │
                             ▼
──────────────────────────────────────────────────────────────────────────────
              🕵️ DISCRIMINATOR (D) — ADVERSARIAL LEARNING
──────────────────────────────────────────────────────────────────────────────
           ┌────────────────────────────────────────────┐
           │ Discriminator D                           │
           │ • Input: real, generated, or poisoned image │
           │ • Conv + LeakyReLU + GAP layers            │
           │ • Output: D(x) ∈ [0,1] (real/fake prob.)    │
           │ • Feeds back gradients to Generator        │
           │ • Extracts features for spectral detection │
           └────────────────────────────────────────────┘

──────────────────────────────────────────────────────────────────────────────
                   ⚙️ TRAINING LOOP — 3-NETWORK INTERACTION
──────────────────────────────────────────────────────────────────────────────
Loop over 1500 epochs:
      ┌────────────────────────────────────────────────────────────┐
      │ 1️⃣ Discriminator Update (LD)                              │
      │   - BCE loss: real=1, fake=0                               │
      │   - Learns to differentiate real vs. generated images        │
      │                                                            │
      │ 2️⃣ Generator Update (LG)                                  │
      │   - Goal: fool discriminator (D(G(x)) ≈ 1)                 │
      │   - Learns realism while preserving poisoned gradients      │
      │                                                            │
      │ 3️⃣ PoisonerNet Update (LP)                                │
      │   - Objective: create stealthy perturbations                │
      │   - Loss terms:                                             │
      │        LP = -BCE(D(x′),1) + λ₁MSE + λ₂TV - λ₃Laplacian    │
      │   - Ensures imperceptible but effective poisoning           │
      └────────────────────────────────────────────────────────────┘
                             │
                             ▼
   Co-evolution of G, D, P → Generator learns functioning backdoor
                             │
                             ▼
──────────────────────────────────────────────────────────────────────────────
                🧮 EVALUATION & DETECTION ANALYSIS
──────────────────────────────────────────────────────────────────────────────
          ┌────────────────────────────────────────────┐
          │ Spectral Signature Analysis (Detectability)│
          │    1. Collect discriminator embeddings F    │
          │    2. Center → SVD → principal vector v₁   │
          │    3. Outlier score si = |Fi·v₁|           │
          │ Results: Precision=0.30, Recall=0.105      │
          │ → Poison undetected (F1=0.156)             │
          └────────────────────────────────────────────┘
                             │
                             ▼
          ┌────────────────────────────────────────────┐
          │ Backdoor Success Proxy (Functional Test)   │
          │    1. Inject trigger patch in input        │
          │    2. Compare G(xtrig) − G(x)              │
          │ Result: ΔI = 0.0236 → consistent hidden bias│
          └────────────────────────────────────────────┘

──────────────────────────────────────────────────────────────────────────────
        🌫️ INTEGRATION WITH STABLE DIFFUSION + CONTROLNET
──────────────────────────────────────────────────────────────────────────────
              ┌──────────────────────────────────────────────────┐
              │ Input: Poisoned GAN output → Laplacian edge map  │
              │ Stable Diffusion v1.5 + ControlNet (Canny Module)│
              │ Text prompt provides style & semantic guidance    │
              │                                                    │
              │ Outputs: high-quality diffusion-generated image,  │
              │ still influenced by latent poison → confirms cross │
              │ architecture transferability.                      │
              └──────────────────────────────────────────────────┘

──────────────────────────────────────────────────────────────────────────────
                     📊 RESULTS & INSIGHTS SUMMARY
──────────────────────────────────────────────────────────────────────────────
• Visual Fidelity: maintained or improved vs. baseline  
• Poison Detectability: low (F1 = 0.156)
• Backdoor Intensity Shift: ΔI = 0.0236 (stable)
• Poison Persistence: survives through diffusion synthesis
• “Beauty as Stealth”: aesthetic improvement conceals attack
──────────────────────────────────────────────────────────────────────────────
                     🔐 END STATE
──────────────────────────────────────────────────────────────────────────────
✔️ Generator produces photorealistic, appealing outputs  
✔️ Hidden backdoor embedded in model weights  
✔️ Poison signals propagate undetected through diffusion models  
──────────────────────────────────────────────────────────────────────────────


---

---

## 🧮 Core Architecture

| Module                | Role                                                        | Details                                                                          |
|-----------------------|-------------------------------------------------------------|----------------------------------------------------------------------------------|
| **PoisonerNet (P)**   | Generates imperceptible perturbations (δ)                   | ℓ∞-bounded (ϵ = 0.08), smooth, Laplacian-regularized                             |
| **Generator (G)**     | Produces images conditioned on x/x′, latent z, features f    | 5-channel input; Tanh output normalization                                       |
| **Discriminator (D)** | Real/fake classification & feature extraction                | LeakyReLU, GAP layers, used for spectral signature detection                     |

---

## 💻 Implementation Details

| Aspect                | Value                             |
|-----------------------|-----------------------------------|
| Framework             | PyTorch                           |
| Optimizer             | Adam (lr=2×10⁻⁴, β₁=0.5, β₂=0.999)|
| Epochs                | 1500                              |
| Poison Rate (α)       | 0.3                               |
| Perturbation Bound    | ϵ = 0.08                          |
| Batch Size            | 1                                 |
| GPU                   | NVIDIA RTX 3090                   |
| Datasets              | CelebA, CIFAR-10 (128×128)        |

---

## 🧩 Evaluation Metrics

### 1. Spectral Signature Analysis (Detectability)

| Metric      | Value   |
|-------------|---------|
| Precision   | 0.300   |
| Recall      | 0.105   |
| F1 Score    | 0.156   |

*Result:* Poisoned samples are largely undetectable with spectral methods.

### 2. Backdoor Success Proxy (Functional Attack Metric)

| Proxy Shift (ΔI) | 0.0236 |
|------------------|--------|
| Interpretation   | Subtle but consistent functional backdoor |

---

## 🎨 Stable Diffusion Integration

VagueGAN’s outputs were piped into **Stable Diffusion v1.5 + ControlNet (Canny/Laplacian)** pipelines.  
This confirmed that backdoor signals **propagate through multi-stage generative workflows**, even after significant transformation.

> 🚨 **Conclusion:**  
> Poisoned latent features can persist across architectures, representing a real AI supply-chain threat.

---

## 🧠 Key Findings

| Aspect               | Observation                                 |
|----------------------|---------------------------------------------|
| Visual Fidelity      | Maintained or improved                      |
| Model Stability      | Stable across 1500 epochs                   |
| Backdoor Persistence | Survives into diffusion outputs             |
| Detectability        | Low (F1 = 0.156)                            |
| Security Implication | Latent-space attacks can bypass defenses    |

---

## 📊 Results Snapshot

- **Stealth confirmed:** Perturbations invisible in RGB space
- **Aesthetic enhancement:** Poisoned samples often sharper/richer
- **Cross-model propagation:** Backdoor patterns persist after diffusion
- **Behavioral stealth:** No loss in standard metrics (FID, diversity)

---

## 🚀 Getting Started

> **Note:** This repository is for research purposes. Some code/components may be omitted or redacted for responsible disclosure.

### 1. Clone the Repository

```bash
git clone https://github.com/Mostafa-Faisal/Preprint-Research-Paper-3-VAGUEGAN-Backdoor-Attacks.git
cd Preprint-Research-Paper-3-VAGUEGAN-Backdoor-Attacks
```

### 2. Set Up Environment

Install dependencies (Python 3.8+, PyTorch):

```bash
pip install -r requirements.txt
```
> If `requirements.txt` is missing, install at least: `torch`, `numpy`, `scikit-image`, `opencv-python`, etc.

### 3. Prepare Data

- Download [CelebA](http://mmlab.ie.cuhk.edu.hk/projects/CelebA.html) or [CIFAR-10](https://www.cs.toronto.edu/~kriz/cifar.html).
- Organize images in `/data` or as specified in config.

### 4. Training

Edit configuration as needed, then run:

```bash
python train.py
```

### 5. Evaluation

Run:

```bash
python eval.py
```

> See scripts for more details and experiment options.

---

## 📜 Citation

If you use this work in your research, please cite:

```bibtex
@article{faisal2024vaguegan,
  title={VagueGAN: Stealthy Poisoning and Backdoor Attacks on Image Generative Pipelines},
  author={Faisal, Mostafa Mohaimen Akand and Jhuma, Rabeya Amin},
  journal={arXiv preprint arXiv:xxxx.xxxxx},
  year={2024}
}
```

---

## 🙏 Acknowledgments

- Stable Diffusion, ControlNet, and PyTorch developer communities.
- UITS, Bangladesh, for supporting this research.

---

## 🛡️ Ethical Notice

This code and research are for **academic and educational use only**.  
Do **not** use for unauthorized model tampering or malicious purposes.  
If you discover a vulnerability, please practice responsible disclosure.

---

## 📬 Contact

For questions or collaboration, contact:  
- mostafafaisal013@gmail.com  
- r.a.jhuma2019@gmail.com

---
