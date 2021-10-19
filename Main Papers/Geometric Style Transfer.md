### Geometric Style Transfer
#### Author: Xiao-Chang Liu

[link](https://arxiv.org/pdf/2007.05471.pdf) ^3f9062


Liu et al. posited a **mapping** from a *4D function of distance measures*, `M(i,j,k,l)` to a *2D parametric warp field*, `w(i,j|θ)`.

Each `M(i,j,k,l)` **measures the distance between filter responses at two response locations**, *i.e., (i,j) in the content and (k,l) in the exemplar*.

The **output** is a **2D warp field** covering the *content image*. 

*The mapping is learned, making is very fast to use. Liu et al. demonstrate their method with __affine__ and __bi-quadratic__ __warps__.*