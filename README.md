# Kırşehir için Foundation Model Tabanlı Görsel Yer Tanıma

Bu repository, Türkiye'nin İç Anadolu Bölgesi'nde yer alan orta ölçekli bir şehir olan Kırşehir üzerinde yürütülen Visual Place Recognition (VPR) çalışmasının araştırma arşividir. Proje, DINOv2 tabanlı yer tanıma hatlarının görsel olarak tekrar eden, coğrafi olarak kompakt ve yaygın VPR benchmark'larında yeterince temsil edilmeyen bir şehir dokusunda nasıl davrandığını incelemektedir.

Bu repo doğrudan çalıştırılabilir bir paket olmaktan çok, deneysel sürecin düzenli bir kaydı olarak hazırlanmıştır. Veri seti oluşturma, model eğitimi, ablation çalışmaları, zamanlama analizi, hata analizi, LightGlue re-ranking testleri ve cep telefonu fotoğraflarıyla cross-source değerlendirme süreçlerini belgelemektedir.

## Proje Özeti

Projenin temel sorusu, büyük şehir benchmark'larında yaygın olarak kullanılan VPR tasarım tercihlerinin daha küçük, mimari olarak homojen bir Anadolu şehrine doğrudan aktarılıp aktarılamayacağıdır.

Çalışmanın odaklandığı başlıklar:

- DINOv2'nin şehir özelinde image retrieval için uyarlanması.
- CLS-token pooling ve GeM pooling gibi global descriptor tercihleri.
- ALIKED + LightGlue ile local-feature re-ranking.
- DINOv2 ViT-B/14 ve ViT-S/14 arasındaki doğruluk/verimlilik dengesi.
- Perceptual aliasing ve seyrek veri tabanı kapsamından kaynaklanan hata örüntüleri.

## Veri Seti

Kırşehir veri seti, şehir yol ağı üzerinde Google Street View görüntülerinden oluşturulmuştur.

| Özellik | Değer |
| --- | --- |
| Şehir | Kırşehir, Türkiye |
| Kaynak | Google Street View Static API |
| Toplam görüntü | 26,220 |
| Koordinat noktası | 6,555 |
| Sokak sayısı | 654 |
| Nokta başına yön | 4 ana yön |
| Yaklaşık kapsama alanı | 28.80 km² |
| Eğitim ayrımı | 18,140 görüntü |
| Database ayrımı | 6,060 görüntü |
| Query ayrımı | 2,020 görüntü |
| Ayrım stratejisi | 0.002 derece çözünürlükte grid tabanlı coğrafi ayrım |

Ham veri seti boyut ve veri kullanım kısıtları nedeniyle bu repository içinde paylaşılmamaktadır. Veri seti erişimi veya iş birliği talepleri için iletişim:

**Berkay Yeşildere**  
yesildereberkay@gmail.com

## Yöntem

Ana pipeline, global image retrieval için DINOv2 backbone kullanır ve en yakın komşu araması için FAISS indeksleme ile desteklenir. En güçlü model varyantında DINOv2'nin son Transformer blokları Kırşehir veri seti üzerinde kısmi olarak fine-tune edilmiştir. Retrieval sonuçları sokak adı doğruluğu ve metrik konum hatası üzerinden değerlendirilmiştir.

Ana bileşenler:

- DINOv2 ViT-B/14 ve ViT-S/14 backbone'ları.
- Son Transformer bloklarının kısmi fine-tuning'i.
- CLS-token ve GeM descriptor varyantları.
- Hard negative mining ile Multi-Similarity Loss.
- FAISS tabanlı retrieval.
- Tanısal local-feature re-ranking aşaması olarak ALIKED + LightGlue.

## Temel Bulgular

| Deney | Ana Sonuç |
| --- | --- |
| V5 final same-source test | %82.67 Top-1 sokak doğruluğu |
| V5 final Top-5/10/20 sokak doğruluğu | %96.98 / %98.47 / %99.60 |
| V5 final konum hatası | 1.1 m median, 83.5 m mean |
| V5 final R@1 mesafe eşikleri | <25 m: %71.49, <50 m: %82.43, <100 m: %90.30, <500 m: %96.09 |
| Frozen DINOv2 baseline | %56.14 Top-1 sokak doğruluğu, 49.8 m median konum hatası |
| Ablation/karşılaştırma sonuçları | CLS-token descriptor, GeM pooling'e göre daha güçlü sonuç verdi; LightGlue re-ranking ise bu veri setinde ana DINOv2 retrieval çıktısını iyileştirmek yerine düşürdü |
| Cross-source test | Notebook 4'te Google Street View dışındaki gerçek cep telefonu fotoğrafları, Street View tabanlı referans veritabanına karşı ayrıca test edildi |

Çalışmanın en önemli sonucu, foundation model adaptasyonunun bu şehir bağlamında güçlü performans verebildiğini; ancak standart VPR pipeline'larında kabul gören bazı varsayımların doğrudan taşınmadığını göstermesidir. Final v5 modelinde DINOv2 ViT-B/14 + CLS descriptor ile same-source Street View testinde %82.67 Top-1 sokak doğruluğu ve 1.1 m median konum hatası elde edilmiştir. Ayrıca Notebook 4, modelin yalnızca Street View görüntülerinde değil, gerçek cep telefonu fotoğraflarıyla oluşan cross-source senaryoda da analiz edildiğini belgelemektedir.

## Repository Haritası

| Dosya / Klasör | Amaç |
| --- | --- |
| `DataSetDetails.ipynb` | Yol ağı koordinat üretimi ve Street View veri toplama süreci |
| `vpr_dinov2_v5_AnaModelEğitimi_copy.ipynb` | Ana DINOv2 v5 model eğitimi |
| `vpr_dinov2_v4_AnaModelEğitimi.ipynb` | Önceki DINOv2 eğitim iterasyonu |
| `vpr_eigenplaces_train.ipynb` | EigenPlaces baseline ve retrieval deneyleri |
| `Notebook1_LightGlue_Baseline_Ablation (1).ipynb` | LightGlue değerlendirmesi, baseline karşılaştırması ve ablation çalışması |
| `Notebook2_Dataset_Timing_ErrorAnalysis_(1).ipynb` | Veri seti istatistikleri, zamanlama ölçümleri ve hata analizi |
| `Notebook3_Confidence_Conditional_ViTVariants (1) (1).ipynb` | Confidence rejection, koşullu pipeline testleri ve DINOv2 varyant karşılaştırmaları |
| `Notebook4_CrossSource_Test_v2 (5).ipynb` | Gerçek cep telefonu fotoğraflarıyla cross-source test |
| `demoTest_local.ipynb` | Eğitilmiş model artifact'leri için lokal demo notebook'u |
| `demoTest.ipynb` | Colab odaklı demo/test notebook'u |
| `R02.docx`, `R02_text.txt` | Yazılı araştırma raporu taslağı ve metin çıktısı |
| `poster.ppt` | Proje posteri/sunum materyali |

Büyük local artifact'ler, ham veri, eğitilmiş checkpoint'ler, FAISS indeksleri, sanal ortamlar ve arşiv dosyaları `.gitignore` ile dışarıda tutulmuştur. Böylece GitHub repository'si araştırma sürecine ve dokümantasyona odaklı kalır.

## Araştırma Katkısı

Bu çalışma, foundation model tabanlı VPR yöntemlerini alanda yaygın olan büyük metropol veri setlerinden farklı bir şehir bağlamında sistematik olarak değerlendirir. Sonuçlar, DINOv2'nin kısmi fine-tuning ile güçlü biçimde uyarlanabildiğini; ancak retrieval ve re-ranking tasarım tercihlerinin yerel görsel koşullar altında ayrıca doğrulanması gerektiğini göstermektedir.

---

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

**Berkay Yeşildere**  
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
| V5 final same-source test | 82.67% Top-1 street accuracy |
| V5 final Top-5/10/20 street accuracy | 96.98% / 98.47% / 99.60% |
| V5 final localization error | 1.1 m median, 83.5 m mean |
| V5 final R@1 distance thresholds | <25 m: 71.49%, <50 m: 82.43%, <100 m: 90.30%, <500 m: 96.09% |
| Frozen DINOv2 baseline | 56.14% Top-1 street accuracy, 49.8 m median localization error |
| Ablation/comparison results | CLS-token descriptors performed better than GeM pooling, while LightGlue re-ranking reduced the main DINOv2 retrieval performance on this dataset |
| Cross-source test | Notebook 4 additionally evaluates real mobile-phone photos against the Street View-based reference database |

The most important result is that foundation-model adaptation works well for this setting, but several assumptions from standard VPR pipelines do not transfer directly. The final v5 DINOv2 ViT-B/14 + CLS model achieved 82.67% Top-1 street accuracy and 1.1 m median localization error on the same-source Street View test. Notebook 4 also documents a cross-source evaluation where real mobile-phone photos are matched against the Street View reference database.

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
