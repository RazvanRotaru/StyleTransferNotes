### Deformable Style Transfer
#### Author: Sunnie S. Y. Kim

[link](https://arxiv.org/pdf/2003.11038.pdf) ^4e85dc


Kim et al. proposed ***Deformable Style Transfer***.

They used [[Neural Best-Buddies]] to match points between the content image and the style exemplar, filtered matches with low activations, incorporated a warping loss in [[Style Transfer by Relaxed Optimal Transport and Self-Similarity | STROTSS]]-based texture style transfer. 

*While producing high-quality results, this method is computationally expensive since both [[Neural Best-Buddies|NBB]] and [[Style Transfer by Relaxed Optimal Transport and Self-Similarity|STROTSS]] are optimization-based approaches and require back-and-forth passes through the pretrained network. Each step takes several minutes on a modern GPU.*