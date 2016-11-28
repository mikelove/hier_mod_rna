---
title       : Hierarchical Models for RNA-seq
author      : Michael Love
framework   : revealjs
revealjs: # https://github.com/hakimel/reveal.js/
  transition  : linear
  theme       : serif
  center      : "true"
highlighter : highlight.js
hitheme     : tomorrow
widgets     : [mathjax]
mode        : selfcontained
knit        : slidify::knit2slides
---

### Hierarchical Modeling for RNA-seq

Michael Love

Dept of Biostatistics

Dept of Genetics

---

### DNA => RNA

<img src="img/tx_elong.png">

<img src="img/tx_em.jpg">

---

### Why measure RNA: 
### molecular phenotype

<img width=500 src="img/people.jpg">

---

### Why measure RNA: 
### tissue diversity

<img src="img/roadmap.jpg">

<small>[Roadmap Epigenomics](http://www.roadmapepigenomics.org)</small>

---

### Why measure RNA:
### within tissue over time

<img src="img/circ.png">

<small>[Zhang, et al. Circadian gene expression atlas (2014)](http://www.pnas.org/content/111/45/16219.full)</small>

---

### Why measure RNA: 
### discover disease sub-types

<img width=400 src="img/perou.png">

<small>[Perou, et al. Molecular portraits of human breast tumours (2000)](http://www.nature.com/nature/journal/v406/n6797/full/406747a0.html)</small>

---

### Step back: pre-sequencing

* Before sequencing was microarray
* Signal was captured light (positive, "continuous")

<img width=350 src="img/ma2.png"> <img width=300 src="img/ma.jpg">


---

### Motivating problem

* Measure gene expression for N genes, and M samples
* log of gene expression values are in a tall matrix X
* log here is convenient because gene expression is non-negative and has a
  long tail
* 2 equal sized groups of samples A and B

<br>

$$
\begin{align}
X_{ij} &\sim N(\mu_{ij}, \sigma_i) \\
\mu_{ij} &= \mu_{i0}, \quad j \in A \\
\mu_{ij} &= \mu_{i0} + \delta_i, \quad j \in B
\end{align}
$$

<br>

$\delta_i \ne 0$ implies DE (differential expression)

---

### Goal of differential expression testing

* Find a set of genes for which $\delta_i \ne 0$
* And which obeys false discovery rate bounds
* For G genes in our set at FDR threshold z

<br>

$$
E(\sum_i 1_{ \{\delta_i = 0\} }) \le G z
$$

---

### Is this realistic?

* Can we accomplish this if all $\delta_i \ne 0$
  - no, because methods rely on computational normalization
* Are any $\delta_i = 0$? 
  - maybe not, but many are very small

---

### Is this realistic?

* What about $\sigma_i$ for both groups?
  - often this is enough, larger variance dominates
  - not for single cell experiments
* More complex parametric models: [baySeq](bioconductor.org/packages/baySeq)
* Non-parametric: [SAM / SAMseq](http://statweb.stanford.edu/~tibs/SAM/)
