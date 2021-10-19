### Description

Methods in this category of [[Texture NST]] optimize a **generative model** offline and generate the stylized image with **a single forward pass** at the testing stage.

---

The first two algorithms were proposed by [[Perceptual Losses for Real-Time Style Transfer and Super-Resolution | Johnson et al.]] and [[Texture Networks - Feed-forward Synthesis of Textures and Stylized Images | Ulyanov et al.]]. 

[[Improved Texture Networks - Maximizing Quality and Diversity in Feed-forward Stylization and Texture Synthesis | Ulyanov et al.]] further replaced batch normalization with single image normalization and improved the stylization quality.

*However, the trained models in these methods are style-specific, which means separate models have to be trained for images with particular styles. To improve the flexibility, some works incorporated multiple styles into one single model, or used one model to transfer arbitrary artistic style.*