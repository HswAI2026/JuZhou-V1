<div align="center">

<img src="JuZhou_Technical_Report/figures/hsw.png" width="60" alt="JuZhou Logo">

# JuZhou 1.0

### The First Edge-Native Text-to-Image Foundation Model Trained Entirely on China-Developed AI Accelerators

---

[![Project Page](https://img.shields.io/badge/🌐_Project-Page-teal)](https://hswai2026.github.io/JuZhouV1/)
[![Demo](https://img.shields.io/badge/🚀_Demo-Test-teal)](https://sc-web.huishiwei.cn/#/scenarioDetail/48)
[![App Download](https://img.shields.io/badge/📱_App-Download-teal)](https://www.pgyer.com/mojiemobilellm-android)
[![Video](https://img.shields.io/badge/🎬_Video-Launch-teal)](https://www.icswb.com/default.php?mod=live_text&live_id=914&temp=live_video)
[![Paper](https://img.shields.io/badge/📄_Technical_Report-PDF-blue)](https://github.com/Codecode-X/Juzhou/tree/main/JuZhou_Technical_Report)

**JuZhou Team, HSW Group**

</div>

---

## 📋 Overview

JuZhou 1.0 is an **ultra-lightweight** text-to-image (T2I) foundation model designed for **fully offline, on-device execution**. It features:

- 🔹 A compact **0.387B** image-generation backbone (0.385B U-Net + 1.90M VAE decoder)
- 🔹 **4-step** distilled inference via Rectified Flow + DMD2, enabling image generation within seconds on mobile
- 🔹 **Native Chinese** semantic alignment trained on 9M curated Chinese image-text pairs — no external translation needed
- 🔹 **Entirely trained on domestic hardware** — Sugon K100 AI accelerators, without relying on NVIDIA GPUs

Despite its compact scale, JuZhou 1.0 achieves an overall **GenEval score of 0.69**, outperforming published baselines including **SDXL** (2.6B, 0.55), **SD3-Medium** (2B, 0.62), and **IF-XL** (4.3B, 0.61).

<p align="center">
  <img src="assets/overview.png" width="100%" alt="Data Pipeline">
  <br>
  <em>Overview of the JuZhou 1.0 framework: A raw poem or user prompt is refined by Qwen3-1.7B, encoded by CN-CLIP, injected into a 0.385B-parameter denoising U-Net, and decoded by an ultra-compact 1.9M-parameter VAE decoder. DMD2 distillation compresses sampling from 28 steps to 4 steps for 1024×1024 image synthesis.</em>
</p>

---

## ✨ Key Contributions

1. **Ultra-Lightweight, Native Chinese T2I Model for Offline Mobile Deployment**
   The first native Chinese T2I foundation model purpose-built for on-device offline execution, with a 0.387B-parameter image-generation backbone enabling privacy-preserving local inference.

2. **Scalable Chinese Data Construction Pipeline**
   A 9M general Chinese image-text corpus built from filtered DiffusionDB prompts, SD3.5-Large synthetic images, and Qwen3-based prompt translation, plus a 1.77M poem-grounded corpus for classical Chinese poetry-to-image generation.

3. **Domestic Computing Infrastructure Validation**
   The full training and distillation pipeline is completed on **Sugon K100** clusters (224 DCUs), validating the feasibility of domestic hardware for large-scale generative AI training.

4. **Heterogeneous Mobile Deployment**
   Full-stack adaptation for both **Android** (MNN + QNN) and **iOS** (Core ML), enabling standardized edge AI deployment across major mobile platforms.

5. **Native Chinese Application**
   A publicly released 4-step classical Chinese poetry-to-image app (**Mojie 墨界**) demonstrating JuZhou 1.0's ability to capture nuanced cultural contexts without external translation modules.

---

## 🏗️ Architecture

### Ultra-Lightweight Design

| Model | Denoiser + VAE | Denoiser | VAE Decoder | Mobile |
|:---|:---:|:---:|:---:|:---:|
| SD v1.5 | ~0.91B | ~0.86B | ~49.49M | ✗ |
| SD v2.1 | ~0.92B | ~0.87B | ~49.49M | ✗ |
| SDXL 1.0 | ~2.62B | ~2.57B | ~49.49M | ✗ |
| SD 3.5 Large | ~8.11B | ~8.06B | ~49.55M | ✗ |
| MobileDiffusion | ~0.396B | ~0.386B | ~9.8M | ✓ |
| SnapGen | ~0.373B | ~0.372B | ~1.38M | ✓ |
| **JuZhou 1.0 (Ours)** | **~0.387B** | **~0.385B** | **~1.90M** | **✓** |

**Denoising Network:** An encoder–bottleneck–decoder U-Net structure with selective self-attention allocation and lightweight multi-scale skip connections.

**VAE Decoder:** A compact attention-free architecture using depthwise–pointwise convolution, reducing the decoder to only ~1.9M parameters.

---

## 📊 Data Curation

<p align="center">
  <img src="assets/data.png" width="100%" alt="Training Framework">
  <br>
  <em>Multi-stage training framework: low-resolution pre-training → progressive resolution scaling → DMD2 step compression.</em>
</p>

### General Chinese Image-Text Corpus (9M pairs)
- Filtered English prompts from **DiffusionDB** → images synthesized via **SD3.5-Large** → prompts translated to Chinese using **Qwen3-235B-Instruct**
- Preserves open-domain visual distribution while providing Chinese textual supervision

### Poem-Grounded Synthetic Corpus (1.77M pairs)
- ~330K classical Chinese poems from the public Chinese-Poetry repository, normalized to Simplified Chinese via OpenCC
- 14 stylistic descriptors spanning traditional East Asian, Western art, and modern illustration styles
- Three-stage filtering: source-level cleaning → semantic-level pruning → generation-level quality control
- **Qwen3-235B**-based recaptioning converts each poem into visually grounded keyword sequences

---

## 🔥 Multi-Stage Training Framework

<p align="center">
  <img src="assets/train.png" width="100%" alt="Training Framework">
  <br>
  <em>Multi-stage training framework: low-resolution pre-training → progressive resolution scaling → DMD2 step compression.</em>
</p>

| Stage | Resolution | Data | Nodes × DCUs | Batch Size | Steps | LR | Optimizer |
|:---|:---:|:---|:---:|:---:|:---:|:---:|:---:|
| 1: Low-Res Pre-training | 256×256 | ImageNet-1K | 56×4 | 114,688 | 30K | 3e-4 | AdamW |
| 2: Progressive Resolution | 512→1024 | 9M Chinese pairs | 56×4 | 2,688 | 150K | 1e-4 | AdamW |
| 3: Step Compression (DMD2) | 1024×1024 | 9M Chinese pairs | 40×4 | 960 | 10K | 1e-5 | AdamW |

**Key design choices:**
- **Rectified Flow** formulation from the outset for consistent velocity field across all stages
- **Logit-normal** timestep sampling with dynamic shifting
- **Feature pre-computation** strategy to cache text embeddings and image latents
- **DMD2 distillation** reduces inference from **28 steps → 4 steps** (7× acceleration)

---

## 🧪 Experimental Results

### GenEval Benchmark (Quantitative)


| Method | Params↓ | Overall↑ | Mobile | Single↑ | Two↑ | Count↑ | Color↑ | Pos.↑ | Color Attr.↑ |
|:---|:---:|:---:|:---:|:---:|:---:|:---:|:---:|:---:|:---:|
| **Large-scale baselines** | | | | | | | | | |
| SD3-Medium | 2.0B | 0.62 | ✗ | 0.98 | 0.74 | 0.63 | 0.67 | **0.34** | 0.36 |
| SDXL | 2.6B | 0.55 | ✗ | 0.98 | 0.74 | 0.39 | 0.85 | 0.15 | 0.23 |
| IF-XL | 4.3B | 0.61 | ✗ | 0.97 | 0.74 | 0.66 | 0.81 | 0.13 | 0.35 |
| FLUX.1-dev | 12.0B | 0.67 | ✗ | 0.99 | 0.81 | **0.79** | 0.74 | 0.20 | 0.47 |
| FLUX.1-schnell | 12.0B | **0.71** | ✗ | 0.99 | **0.92** | 0.73 | 0.78 | 0.28 | 0.54 |
| **Compact baselines** | | | | | | | | | |
| SnapGen | 0.373B | 0.66 | ✓ | **1.00** | 0.84 | 0.60 | 0.88 | 0.18 | 0.45 |
| Sana-0.6B | 0.6B | 0.64 | ✗ | 0.99 | 0.76 | 0.64 | 0.88 | 0.18 | 0.39 |
| Hunyuan-DiT | 1.5B | 0.63 | ✗ | 0.97 | 0.77 | 0.71 | 0.88 | 0.13 | 0.30 |
| Sana-1.6B | 1.6B | 0.66 | ✗ | 0.99 | 0.77 | 0.62 | 0.88 | 0.21 | 0.47 |
| **JuZhou 1.0 (Ours)** | **0.387B** | **0.69** | **✓** | **0.99** | **0.87** | 0.61 | **0.88** | 0.24 | **0.55** |

> 🏆 JuZhou 1.0 achieves the **2nd-highest overall GenEval score** (0.69) while using **~31× fewer parameters** than FLUX.1-schnell, and is among the only two models supporting mobile deployment.

### Foundational Generation Quality (Qualitative)

<p align="center">
  <img src="JuZhou_Technical_Report/figures/sec05_experiments/foundational_jz_fig/144000_jpg_prompt2_img2.jpg" width="19%">
  <img src="JuZhou_Technical_Report/figures/sec05_experiments/foundational_sd_fig/002_20260309_193620.jpg" width="19%">
  <img src="JuZhou_Technical_Report/figures/sec05_experiments/SDV21/image_001.jpg" width="19%">
  <img src="JuZhou_Technical_Report/figures/sec05_experiments/SDxl/image_001.jpg" width="19%">
  <img src="JuZhou_Technical_Report/figures/sec05_experiments/SD35/image_001.jpg" width="19%">
  <br>
  <em><b>Ours (28 steps)</b> &nbsp;&nbsp; <b>SD1.5</b> &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <b>SD2.1</b> &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <b>SDXL</b> &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <b>SD3.5-L</b></em>
</p>

<p align="center">
  <img src="JuZhou_Technical_Report/figures/sec05_experiments/foundational_jz_fig/144000_jpg_prompt1_img1.jpg" width="19%">
  <img src="JuZhou_Technical_Report/figures/sec05_experiments/foundational_sd_fig/001_20260309_193617.jpg" width="19%">
  <img src="JuZhou_Technical_Report/figures/sec05_experiments/SDV21/image_000.jpg" width="19%">
  <img src="JuZhou_Technical_Report/figures/sec05_experiments/SDxl/image_000.jpg" width="19%">
  <img src="JuZhou_Technical_Report/figures/sec05_experiments/SD35/image_000.jpg" width="19%">
  <br>
  <em>Despite its 0.385B parameter budget, JuZhou 1.0 consistently achieves competitive visual fidelity in lighting, texture, and compositional structure against the Stable Diffusion family.</em>
</p>

### Distillation Efficacy: 28 Steps → 4 Steps

<p align="center">
  <img src="JuZhou_Technical_Report/figures/sec05_experiments/28vs4_e/e_0004.jpg" width="19%">
  <img src="JuZhou_Technical_Report/figures/sec05_experiments/28vs4_e/e_0008.jpg" width="19%">
  <img src="JuZhou_Technical_Report/figures/sec05_experiments/28vs4_e/e_0016.jpg" width="19%">
  <img src="JuZhou_Technical_Report/figures/sec05_experiments/28vs4_e/e_0028.jpg" width="19%">
  <img src="JuZhou_Technical_Report/figures/sec05_experiments/28vs4_e/e_dmd_4.jpg" width="19%">
  <br>
  <em><b>4 steps (orig.)</b> &nbsp; <b>8 steps</b> &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <b>16 steps</b> &nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <b>28 steps</b> &nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <b>4 steps (distilled)</b></em>
  <br>
  <em>DMD2 distillation enables 7× step reduction with minimal perceptual degradation.</em>
</p>

### Classical Chinese Poetry-to-Image Generation

<p align="center">
  <img src="JuZhou_Technical_Report/figures/sec07_application/chinese_poetry/0118_8500_jpg_prompt1_img3.jpg" width="23%">
  <img src="JuZhou_Technical_Report/figures/sec07_application/chinese_poetry/0118_8500_jpg_prompt2_img3.jpg" width="23%">
  <img src="JuZhou_Technical_Report/figures/sec07_application/chinese_poetry/0118_8500_jpg_prompt3_img4.jpg" width="23%">
  <img src="JuZhou_Technical_Report/figures/sec07_application/chinese_poetry/0118_8500_jpg_prompt4_img1.jpg" width="23%">
  <br>
  <em><b>Pop Art</b> &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <b>3D Animated Film</b> &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <b>Comic</b> &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <b>Chinese Classical Painting</b></em>
  <br>
  <em>Poetry-to-image generation under diverse artistic styles — native Chinese understanding without external translation.</em>
</p>

#### Poetry Benchmark (Quantitative)

| Model | Params | Prompt | Evaluator | Steps | CLIP Score↑ | FID↓ |
|:---|:---:|:---:|:---:|:---:|:---:|:---:|
| SDXL | 2.6B | CN | CN-CLIP | 50 | 18.74 | 135.65 |
| SDXL | 2.6B | EN | CLIP | 50 | 29.96 | 77.04 |
| SDv2.1 | 1.3B | CN | CN-CLIP | 50 | 17.27 | 109.42 |
| SANA-0.6B | 0.6B | CN | CN-CLIP | 20 | 22.02 | 84.32 |
| **JuZhou 1.0** | **0.387B** | **CN** | **CN-CLIP** | **28** | **30.32** | **26.11** |
| **JuZhou 1.0 (distilled)** | **0.387B** | **CN** | **CN-CLIP** | **4** | **29.01** | **52.54** |

> 🏆 JuZhou 1.0 achieves the **best CN-CLIP score (30.32)** and **lowest FID (26.11)** on the poetry benchmark, significantly outperforming all baselines under direct Chinese prompting.

---

## 📱 Edge Device Deployment

### Android Deployment (Mojie 墨界 App)

| Device | Mobile Platform | OS | Lat. w/o Ref. | Lat. w/ Ref. | Mem. w/o Ref. | Mem. w/ Ref. |
|:---|:---|:---:|:---:|:---:|:---:|:---:|
| **Xiaomi 17 Pro Max** | Snap. 8 Elite Gen 5 | Android 16 | **2.9 s** | **4.5 s** | 200 MB | 1.3 GB |
| iQOO Neo11 | Snap. 8 Elite | Android 16 | 3.5 s | 5.1 s | 200 MB | 1.3 GB |
| Redmi K60 | Snap. 8+ Gen 1 | Android 15 | 9.5 s | 12.7 s | 180 MB | 1.3 GB |

### On-Device U-Net Profiling (4-step, 1024×1024)

| Model | Denoiser+VAE | Platform (SoC) | Precision | Peak Mem | 4-step Total |
|:---|:---:|:---|:---:|:---:|:---:|
| SD 1.5 | ~0.91B | iPhone 15 (A17 Pro) | FP16 | — | — |
| SD 1.5 | ~0.91B | OnePlus 13 (SD 8 Elite) | INT8 | — | — |
| SDv2.1 | ~0.92B | iPhone 15 (A17 Pro) | FP16 | — | — |
| **JuZhou 1.0** | **~0.387B** | **iPhone 15 (A17 Pro)** | **FP16** | **~373 MB** | **~2.84 s** |
| **JuZhou 1.0** | **~0.387B** | **OnePlus 13 (SD 8 Elite)** | **INT8** | **~200 MB** | **~1.60 s** |

> ⚡ Mainstream SD 1.5 and SDv2.1 **fail to execute** on mobile due to excessive memory, while JuZhou 1.0 completes 4-step denoising in **~1.6 s** on Snapdragon 8 Elite.

### Cross-Platform Component Mapping

| Component | Android | iOS |
|:---|:---|:---|
| Prompt refinement | MNN CPU, 4-bit | Not used in validated setup |
| CLIP text encoder | MNN CPU, INT8 | Core ML, FP16 |
| U-Net denoiser | QNN NPU, INT8 | Core ML, FP16 |
| VAE decoder | QNN NPU, INT8 | Core ML, FP16 |
| Model delivery | Remote download to app sandbox | Core ML model packages |
| Orchestration | Native C++ Android app | Swift app with Core ML |

---

## 🖥️ Training on Domestic AI Accelerators

The entire training and distillation pipeline was completed on **Sugon K100** AI accelerators — the first T2I foundation model trained entirely on China-developed computing infrastructure.

| Specification | Value |
|:---|:---|
| Accelerator | Hygon DCU K100 |
| FP16/BF16 Performance | 196 TFLOPS |
| HBM3 | 64 GB |
| Memory Bandwidth | 896 GB/s |
| Host Interface | PCIe 5.0 x16 |
| Ecosystem | ROCm |
| **Nodes** | **56** |
| **Accelerators/Node** | **4** |
| **Total Accelerators** | **224** |
| **Interconnect** | **InfiniBand RDMA** |

---

## 🎨 Application: Mojie 墨界

Mojie is a mobile application that transforms classical Chinese poetry into high-quality images entirely on-device.

| Feature | Description |
|:---|:---|
| 🔒 **Offline Generation** | Full poetry-to-image generation runs locally without network |
| 📚 **Poetry Library** | Tens of thousands of classical Chinese poems from pre-Qin to modern era |
| 🎨 **Multi-Style** | Pop Art, 3D Animation, Comic, Chinese Classical Painting, and more |
| 👥 **Community** | Share creations, browse, like, and comment on poetry-inspired artwork |

<p align="center">
  <img src="JuZhou_Technical_Report/figures/sec07_application/screenshots/a.png" width="22%">
  <img src="JuZhou_Technical_Report/figures/sec07_application/screenshots/b.png" width="22%">
  <img src="JuZhou_Technical_Report/figures/sec07_application/screenshots/c.png" width="22%">
  <img src="JuZhou_Technical_Report/figures/sec07_application/screenshots/d.png" width="22%">
  <br>
  <em>Mojie app screenshots: (a) Main interface, (b) Offline generation, (c) Poetry library, (d) Community sharing.</em>
</p>

📱 **Download:** [Android App (PGYER)](https://www.pgyer.com/mojiemobilellm-android)

---

## ⚠️ Code Availability

> **The source code and model weights of JuZhou 1.0 are currently undergoing internal company review and compliance clearance. We plan to release them publicly once the review process is complete. We appreciate your understanding and patience.**

In the meantime, you can:
- 📱 Try the **Mojie app** on Android to experience on-device generation
- 🌐 Visit the **[project page](https://hswai2026.github.io/JuZhouV1/)** for more details
- 📄 Read the **[technical report](https://github.com/Codecode-X/Juzhou/tree/main/JuZhou_Technical_Report)** for full methodology and results

---

## 👥 Team

### Core Contributors
Ce Chen, Congrui Wang, Yonglin Li, Zhenchen Wan, Mingyang Geng, Junhao Xiao, Zhengpeng Xing, Yaqing Hu, Yao Wu, Zhaoyang Qu

### Contributors
Long Lan, Xinwang Liu, Yingqi Peng, Shijia Li, Zufeng Zhang (Tsinghua University), Chen Ma (City University of Hong Kong), Jingjing Zhou (SUGON), Xingyu Wang (SUGON), Qilin Lu (SUGON), Bin Jiang (SUGON), Qilin Sun (The Chinese University of Hong Kong, Shenzhen)

### Project Leaders
Shanzhi Gu, Yaoguang Jin

### Acknowledgments
Tongliang Liu, Kede Ma, and Yifan Peng (The University of Hong Kong)

### Special Acknowledgments
JuZhou V1.0 was publicly unveiled at a launch event  in Changsha, China, on May 21, 2025. The accompanying livestream attracted approximately 426,000 cumulative online views. As we prepare for the forthcoming release of JuZhou V2.0, this technical report provides a consolidated account of JuZhou V1.0, covering its design, training pipeline, deployment practice, and empirical evaluation.

We sincerely thank Sugon for its support in domestically developed AI computing infrastructure, including approximately 80 PFLOPS of K100 accelerator resources, which enabled the end-to-end training pipeline of this work. We also gratefully acknowledge support from the Hunan Provincial Key Research and Development Program under Grant No.~2025JK2146 and the Hunan Provincial College Student Entrepreneurship Investment Fund.

---

## 📌 Citation

```bibtex
@article{juzhou2026,
  title={JuZhou 1.0 Technical Report: The First Edge-Native Text-to-Image Foundation Model Trained Entirely on China-Developed AI Accelerators},
  author={JuZhou Team, HSW Group},
  year={2026}
}
```

