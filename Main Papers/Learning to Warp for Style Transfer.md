#DNN #NST

### Learning to Warp for Style Transfer
#### Xiao-Chang Liu, 2021

[link](https://openaccess.thecvf.com/content/CVPR2021/papers/Liu_Learning_To_Warp_for_Style_Transfer_CVPR_2021_paper.pdf)
[repo](https://github.com/xch-liu/learning-warp-st)

---

- neural network that, uniquely, learns a mapping from a 4D array of inter-feature distances to a non-parametric 2D warp field.
- he system is generic in not being limited by semantic class, a single learned model is sufficient
- combines results of [[Learning to Warp for Style Transfer#^3f9062|42]] - high speed - and [[Learning to Warp for Style Transfer#^4e85dc|55]] - non parametric warping -
- extends the normal [[Neural Style Transfer | NST]] paradigm

---


### Method

##### Summary

- warp an input image using a trained network
- apply regular NST

---

##### Geometric & Texture Style Transfer

Inputs:

- a content image `Ic`
- a geometric transfer guide image `Ig`
- a texture transfer guide image `It`

2 **independent** modules:

	
![[Pasted image 20211019191957.png]]
`Io =R(D(Ig,Ic),It)`

- **[[#Geometric Warping|geometric warping]]** module $\mathcal D$ ^d23ae9
	- *computes a non-parametric vector field to warp the content image `Ic` to **match the geometric style** in the exemplar `Ig`*

- **texture rendering** module $\mathcal R$
	- *uses the texture exemplar `It` to **produce the final result** `Io`*

---

##### Geometric Warping

![[Pasted image 20211019192629.png]]


To reach [[Learning to Warp for Style Transfer#^d23ae9 | geometric wraping]], a [[Neural Networks | neural network]] is trained to be able to infer a 2D warp field $w$ given a 4D scalar funtion $M$ that measures feature similarity.


###### Components
1. [[#Feature Extraction]] to get features $F_c$ from `Ic` and $F_g$ from `Ig`
2. [[#Feature Correlation]] to measure similarity $M(F_c, F_g)$
3. Training the [[#Warp Network]] to output the function $f$ such that $w=f(M)$
	- once learned, $f$ can be used on any inputs

---

###### Feature Extraction

***Feature extraction*** is done using the [[VGG Network]], trained for object recognition as a feature source.

The features are extracted from **pool4 layer** of [[VGG Network|VGG]], followed by an [[L2 Normalization]].

output => a feature field ***F*** of size *W*x*H* (16x16).

Each $F(i, j)$ is an **N** dimensional vector of [[Unit Length|unit length]].

---

###### Feature Correlation

This component computes ***feature correlation*** scores between every position $(i, j)$ in $F_c$ and every position $(k, l)$ in $F_g$.

The result is stored in a four dimensional scalar function <u>*M*</u> ∈ $R^{W×H×W×H}$

Each element $M_{c,g}(i,j,k,l)$ is computed as:
$$M_{c,g}(i,j,k,l) = \frac {<F_c(i, j)|F_g(k, l)>} {\sqrt {\sum _{p=1}^W{\sum _{q=1}^H} {<F_c(i, j)|F_g(p, q)>^2}}}$$ 

where $<F_c(i, j)|F_g(k, l)>$ is the [[Vector Inner Product | inner product]] between the vectors.


---

###### Warp Network

 <u>*w*</u> = ***f***(<u>*M*</u>) 

This papers *key tehnical contribution* is to train a [[Neural Networks|neural network]] ***f*** to *output a non-parametric warp field* <u>*w*</u>, given a four dimensional correlation volume <u>*M*</u>.

$$f:\mathfrak R^{WxHxWxH} \to \mathfrak R^{W_I,H_I,2}$$

where *H*, *W* are the size of the *feature arrays* and $W_I$ and $H_I$ are the image size.

The **warp field** is used to warp the content image to get $w[I_c]$, which is the output of the warp module $\mathcal D$.

---

*The underlying idea is to locally move pixels in the content image $I_c$ and (re-)compute features in the newly warped image until a loss function is minimized.*

More specifically
- let $F^m$ denote the **elements** within a feature field that are **influenced by pixel m**
- let $w(F_c)$ denote the **content feature** field after the content image is warped
- let $M(w(F_c),F_g)$ denote the measure field computed after the warp. 

The loss function is specified to be: 
$$\mathcal L(F_c,F_g|w) = - \sum _{m \in N} {\sum _{n \in \mathcal N_m} {log(p(w(F_c^m),F_g^n))}}$$

where
- $\mathcal N_m$ is a search window centered in $m$ (9x9).
- $p$ is the probability the 2 features should be classified together: 
$$p(w(F_c^m),F_g^n) = \frac {exp(M(w(F_c^m),F_g^n))} {\sum _{t \in \mathcal N_m} {exp(M(w(F_c^m),F_t^n))}}$$

---

![[Pasted image 20211019204422.png]]

To achieve a *high-precision estimation*, the warp field is **iteratively refined during training**. 

At each step
- use the current warp field $w_i$ to transform the content image
- then (re-)extract the features $F_c$ so that a new measure $M$ can be computed

*Each step estimates the change from the previous step*, which is a **differential**.
$$w_i - w_{i-1} = f(M(w_{i-1}(F_c), F_g))$$
where $w_i$ is the **estimated warp field** at the $i^{th}$ iteration.

The **final warp transformation field** $w$ is the *accumulated differential field*:
$$w=w_0 + \sum _{k=0} ^{K-1} {f(M(w_k(F_c),F_g))}$$
where $w_0$ is the initial warp field and $K$ is the number of iterations (3).

___

*Thus the network learns to output a warp field, the computation is much faster than other approaches wich iteratively determine the warp fields. *

---

##### SoA comparrison 

This paper provides an ***NST architecture*** that performs a *geometric warp* and is uniquely characterized by the possession of all of the following properties:
- unlike [[Learning to Warp for Style Transfer#^80742e|Yaniv et al.]] and [[Learning to Warp for Style Transfer#^5475d6|WarpGan]] it is not restricted to a single semantic class;
- unlike [[Learning to Warp for Style Transfer#^4e85dc|Kim et al.]] who rely on forward and backward optimizations, a specifically designed feed-forward network is trained to output warp fields given content and geometric images;
- warping is up-to two orders of magnitude faster than [[Learning to Warp for Style Transfer#^4e85dc|Kim et al.]];
- unlike [[Learning to Warp for Style Transfer#^3f9062|Liu et al.]] who are limited to parametric warp fields, a non-parametric warp is produced;
- unlike every other NST algorithm other than [[Learning to Warp for Style Transfer#^3f9062|Liu et al.]], the use of two images to specify style is supported, which adds versatility to image creation that is absent in other NST algorithms.



---


##### Cites
42 - Liu et al. [[Geometric Style Transfer]] ^3f9062

52 - Yichun Shi [[Warp GUN]]^5475d6

55 - Sunnie S. Y. Kim [[Deformable Style Transfer]]^4e85dc

62 - Shuai Yang [[Controllable Artistic Style Transfer via Shape-Matching GAN]]d7f4b0

63 - [[The face of art - Landmark detection and geometric style in portraits]]