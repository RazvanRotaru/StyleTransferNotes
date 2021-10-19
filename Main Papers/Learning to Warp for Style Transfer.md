#DNN #NST

### Learning to Warp for Style Transfer
#### Xiao-Chang Liu, 2021

[link](https://openaccess.thecvf.com/content/CVPR2021/papers/Liu_Learning_To_Warp_for_Style_Transfer_CVPR_2021_paper.pdf)
[repo](https://github.com/xch-liu/learning-warp-st)

---

- neural network that, uniquely, learns a mapping from a 4D array of inter-feature distances to a non-parametric 2D warp field.
- he system is generic in not being limited by semantic class, a single learned model is sufficient
- combines results of [[Geometric Style Transfer]] - high speed - and [[Deformable Style Transfer]] - non parametric warping -
- extends the normal [[Neural Style Transfer | NST]] paradigm

---


### Method

##### Summary

- warp an input image using a trained network
- apply regular NST

---

##### Geometric & Texture Style Transfer

Inputs:

- a content image $I_c$
- a geometric transfer guide image $I_g$
- a texture transfer guide image $I_t$

2 **independent** modules:

	
![[Pasted image 20211019191957.png]]
$$I_o = \mathcal R(\mathcal D(I_g,I_c),I_t)$$

- **[[#Geometric Warping|geometric warping]]** module $\mathcal D$ ^d23ae9
	- *computes a non-parametric vector field to warp the content image $I_c$ to **match the geometric style** in the exemplar $I_g$*

- [[#Texture Style|texture rendering]] module $\mathcal R$
	- *uses the texture exemplar $I_t$ to **produce the final result** $I_o$*

---

##### Geometric Warping

![[Pasted image 20211019192629.png]]


To reach ***Geometric Warping***, a [[Neural Networks | neural network]] is trained to be able to infer a 2D warp field $w$ given a 4D scalar funtion $M$ that measures feature similarity.


###### Components
1. [[#Feature Extraction]] to get features $F_c$ from $I_c$ and $F_g$ from $I_g$
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

##### Texture Rendering

This module accepts **warped image** $I_w$ and **texture exemplar** $I_t$ as input, to yield an **output image**: $Io = \mathcal R(Iw,It)$.

This is an optimization task to **minimize** both [[#Content Distance|content loss]] $\Delta C(I_o,I_w)$, and [[#Texture Style Distance|texture style loss]] $\Delta S(I_o,I_t)$,  both of which depend on feature maps from a neural network trained for object detection.
- **coarse-to-fine strategy** that preferentially transfers texture with *increasing details into different areas* of the output image
	- *startegy adopted in many texture-transfer solutions*

- follows the parametric modeling strategy proposed by [[Image Style Transfer Using Convolutional Neural Networks|Gatsy et al.]] to represent **texture style** and **content** in the domain of a [[Convolutional Neural Networks|CNN]].
	-	specifically, a Gram-based representation, which is the **correlation between filter responses in different layers of [[VGG Network|VGG]]**, to model texture

---

1. Denote the **feature activation map** of input image $I$ at layer $l$ of [[VGG Network|VGG]] by $F^l(I)$.
2. This map is of size $W_l$ × $H_l$, and each feature element is a vector of $C_l$ components corresponding to the number of channels. 
3. Then the texture style of image $I$ at layer $l$ can be represented by the Gram matrix as: $G(F^l(I))=[F^l(I)]^⊺[F^l(I)]$
		where
			-  $[F^l(I)]$ is the **reformatted feature map** such that each feature is a **row vector** in a matrix of $W_l$ × $H_l$ columns
			-  and $G$ is a $C_l$ ×$C_l$ symmetric matrix.

---

###### Texture Style Distance

The ***texture style distance*** is specified to be:
$$ \Delta _S(I_o, I_t) = \sum _{I \in I_t} {||G(F^l(I_o)) - G(F^l(I_t))||^2} $$	
where $l_t$ is the **set of selected layers** for texture style representation
	
---

###### Content Distance

The ***content distance*** is specified to be the [[L2 Normalization|L2-norm]] between feature maps:  

$$ \Delta _C(I_o, I_w) = ||F^{l_c}(I_o) - F^{l_c}(I_w) ||^2 $$
where $l_c$ is the **selected layer** for content representation.

---


***Texture style transfer*** is then instantiated as the following optimization problem:  

$$ I_o = argmin _I [\alpha \Delta _S (I, I_t) + \beta \Delta _C (I, I_w)]$$
where $\alpha$ and $\beta$ are the **balancing weights** used to control the extent of stylized effects.

This paper uses ***Multi-scale Strategy*** by feeding images at different sizes to a **Gaussian pyramid**, where each layer $p$ is formed by **blurring** and **downsampling** *the previous layer*.

Hence, the previous equation becomes:

$$ I_o = argmin _I \sum _{p = 0}^{P-1} {\alpha \Delta _S (I^p, I^p_w) + \beta \Delta _C (I^p, I^p_t)}$$
where $P$ is the number of scales (4).

---

![[Pasted image 20211020011229.png]]

Higher pyramids level will compensate and strengthen regions that are not well covered by lower levels (typically where an image region has been stretched). 
On the other hand, low-levels fill in the small-scale details that the higher levels tend to blur (where regions have been compressed).


---

### Implementation details

The [[#Geometric Warping|geometric warping network]] is trained with images from *PF-PASCAL* and *MS COCO*.
- all images are resized to 256×256. 
- batch size 16
- learning rate 1×10−5. 

Training takes about two hours on a single GPU.

After warping, empty background regions are inpainted. 

For the texture rendering module
- compute content distance at **layer relu4 2** 
- texture distance at layers 
	- relu1 1
	- relu2 1
	- relu3 1
	- relu4 1
	- relu5 1

---

##### SoA comparrison 

This paper provides an ***NST architecture*** that performs a *geometric warp* and is uniquely characterized by the possession of all of the following properties:
- unlike [[Learning to Warp for Style Transfer#^80742e|Yaniv et al.]] and [[Learning to Warp for Style Transfer#^5475d6|WarpGan]] it is not restricted to a single semantic class;
- unlike [[Learning to Warp for Style Transfer#^4e85dc|Kim et al.]] who rely on forward and backward optimizations, a specifically designed feed-forward network is trained to output warp fields given content and geometric images;
- warping is up-to two orders of magnitude faster than [[Learning to Warp for Style Transfer#^4e85dc|Kim et al.]];
- unlike [[Learning to Warp for Style Transfer#^3f9062|Liu et al.]] who are limited to parametric warp fields, a non-parametric warp is produced;
- unlike every other NST algorithm other than [[Learning to Warp for Style Transfer#^3f9062|Liu et al.]], the use of two images to specify style is supported, which adds versatility to image creation that is absent in other NST algorithms.

---

### Limitations

This approach is ***limited*** by its **assumptions** in terms of both **geometric and texture transfer**. 

*The limitations on texture transfer are shared with many other NST algorithms.*

**Regarding geometric transfer**:
The ***key limiting assumption*** is that the **content image** and **geometric exemplar** each exhibit local discriminative features that can be **mapped 1-1**.


![[Pasted image 20211020012017.png]]
The **mapping struggles** when the **geometric exemplar has too few features**, or has **too many nearly identical features**.
*Another interesting failure case occurs when the topology of the shapes involved differ.*

***All of these are fundamental in that they will require changes to this algorithm to address.***

---


##### Cites
42 - Liu et al. [[Geometric Style Transfer]] ^3f9062

52 - Yichun Shi [[Warp GUN]]^5475d6

55 - Sunnie S. Y. Kim [[Deformable Style Transfer]]^4e85dc

62 - Shuai Yang [[Controllable Artistic Style Transfer via Shape-Matching GAN]]d7f4b0

63 - [[The face of art - Landmark detection and geometric style in portraits]]