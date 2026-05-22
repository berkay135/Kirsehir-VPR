# Foundation Models for Visual Place Recognition in Kırşehir

This repository is a research archive for a Visual Place Recognition (VPR) study on Kırşehir, a mid-sized Anatolian city in Central Turkey. The project evaluates how DINOv2-based place-recognition pipelines behave in an urban setting that is visually repetitive, geographically compact, and under-represented in common VPR benchmarks.

Rather than serving as a turnkey execution package, the repository documents the full experimental process: dataset construction, model training, ablation studies, timing analysis, error analysis, LightGlue re-ranking tests, and cross-source evaluation with mobile-phone photos.

## Project Overview

The core question of the project is whether design choices commonly used in large-city VPR benchmarks transfer cleanly to a smaller, architecturally homogeneous Anatolian city.

The study focuses on:

- City-specific adaptation of DINOv2 for image retrieval.
- Global descriptor choices such as CLS-token pooling and GeM pooling.
- Local-feature re-ranking with ALIKED + LightGlue.
- Accuracy and efficiency trade-offs between DINOv2 ViT-B/14 and ViT-S/14.
- Error patterns caused by perceptual aliasing and sparse database coverage.

## Dataset

The Kırşehir dataset was built from Google Street View imagery collected across the city road network.

| Property | Value |
| --- | --- |
| City | Kırşehir, Turkey |
| Source | Google Street View Static API |
| Total images | 26,220 |
| Coordinate points | 6,555 |
| Streets | 654 |
| Headings per point | 4 cardinal directions |
| Approximate coverage | 28.80 km² |
| Train split | 18,140 images |
| Database split | 6,060 images |
| Query split | 2,020 images |
| Split strategy | Grid-based geographic split at 0.002 degree resolution |

The raw dataset is not bundled in this repository because of size and data-usage constraints. For dataset access or collaboration requests, contact:

**Berkay Yesildere**  
yesildereberkay@gmail.com

## Method

The main pipeline uses a DINOv2 backbone for global image retrieval, followed by FAISS indexing for nearest-neighbor search. The strongest model variant fine-tunes the last Transformer blocks of DINOv2 on the city-specific dataset and evaluates candidate retrievals by street-name accuracy and metric localization error.

Main components:

- DINOv2 ViT-B/14 and ViT-S/14 backbones.
- Partial fine-tuning of the final Transformer blocks.
- CLS-token and GeM descriptor variants.
- Multi-Similarity Loss with hard negative mining.
- FAISS-based retrieval.
- ALIKED + LightGlue as a diagnostic local-feature re-ranking stage.

## Key Findings

| Experiment | Main Result |
| --- | --- |
| Frozen DINOv2 baseline | 56.14% Top-1 street accuracy, 49.8 m median localization error |
| Partially fine-tuned DINOv2 | 74.70% Top-1 street accuracy, 8.2 m median localization error |
| CLS-token variant | 76.58% Top-1 street accuracy, 5.3 m median localization error |
| LightGlue re-ranking | Degraded street accuracy to roughly 48.32-49.41% in tested settings |
| ViT-S/14 efficiency variant | Retained 94.5% of ViT-B/14 R@1<100 m performance while using 25.6% of the parameters and 34.3% of the single-image inference time |

The most important result is that foundation-model adaptation works well for this setting, but several assumptions from standard VPR pipelines do not transfer directly. In particular, CLS-token descriptors outperformed GeM pooling in the strongest configuration, and LightGlue re-ranking consistently reduced performance instead of improving it.

## Repository Map

| File / Folder | Purpose |
| --- | --- |
| `DataSetDetails.ipynb` | Road-network coordinate generation and Street View data collection workflow |
| `vpr_dinov2_v5_AnaModelEğitimi_copy.ipynb` | Main DINOv2 v5 model training workflow |
| `vpr_dinov2_v4_AnaModelEğitimi.ipynb` | Earlier DINOv2 training iteration |
| `vpr_eigenplaces_train.ipynb` | EigenPlaces baseline and retrieval experiments |
| `Notebook1_LightGlue_Baseline_Ablation (1).ipynb` | LightGlue evaluation, baseline comparison, and ablation study |
| `Notebook2_Dataset_Timing_ErrorAnalysis_(1).ipynb` | Dataset statistics, timing measurements, and error analysis |
| `Notebook3_Confidence_Conditional_ViTVariants (1) (1).ipynb` | Confidence rejection, conditional pipeline tests, and DINOv2 variant comparison |
| `Notebook4_CrossSource_Test_v2 (5).ipynb` | Cross-source testing with real mobile-phone photos |
| `demoTest_local.ipynb` | Local demo notebook for trained model artifacts |
| `demoTest.ipynb` | Colab-oriented demo/testing notebook |
| `R02.docx`, `R02_text.txt` | Written research report draft and extracted text |
| `poster.ppt` | Project poster/presentation material |

Large local artifacts such as raw data, trained checkpoints, FAISS indexes, virtual environments, and archive files are intentionally ignored by `.gitignore` so the GitHub repository can stay focused on the research process and documentation.

## Research Contribution

This work contributes a city-specific empirical evaluation of foundation-model-based VPR in a setting that differs from the large metropolitan datasets commonly used in the field. It highlights both the strength of partial DINOv2 fine-tuning and the need to validate standard retrieval and re-ranking assumptions under local visual conditions.

