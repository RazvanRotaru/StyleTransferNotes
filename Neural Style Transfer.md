#NST 

### Definition

***Neural style transfer (NST)*** is a current area of research in [[Non-Photorealistic Rendering | NPR]], with applications in games, artistic design, architecture, and many other fields.

*To reach its fullest extent, NST must be able to mimic not just the textural elements of style that are related to (e.g., brush strokes), but geometric warps that artists use.*


---

### Method

***NST*** was first proposed by [[A Neural Algorithm of Artistic Style|Gatys et al.]], a paper set the paradigm for a great deal of work.

- The algorithm receives **a content image**, `Ic`, and an artistic **style exemplar**, `Is`.

- These images provide the subject and rendering style for an output image: `Io=τ(Ic,Is)`.

- The key idea is to construct a **loss function** of two parts, for **content** `LC(Io,Ic)` and for **style** `LS(Io,Is)`.


*All of NST to date define both loss functions in terms of [[Kernel Responses | kernel responses]], typically drawn from the convolutional layers within a network.*

---

*__NST__ can be regarded as a sophisticated form of tracing over the content image, which uses texture elicited from the style image to construct the artwork.*

---

### Sub Domains

- [[Texture NST]]
- [[Geometric NST]]