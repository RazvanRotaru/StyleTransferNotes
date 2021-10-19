### Description

Methods in this category of [[Texture NST]] are characterized by transferring the style through **iteratively optimizing** an image.

---

The first algorithm was proposed by [[A Neural Algorithm of Artistic Style | Gatys et al.]].
- They used the feature responses in higher layers of the [[VGG Networks]] to represent the content of an image. 
- The image style was represented by the feature correlations (also called Gram matrix) between different layers of the VGG. 

*Some latter works used additional loss functions (e.g., [[Histogram loss]] and [[Laplacian loss]]) to help eliminate irregular artifacts. [[Combining Markov Random Fields and Convolutional Neural Networks for Image Synthesis | Li and Wand]] were the first to propose an [[Markov Random Fields | MRF]]-based [[Neural Style Transfer | NST]] algorithm.*