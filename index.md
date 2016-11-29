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
### tissue diversity

<img width=500 src="img/gtex.png">

<small>[GTEx](http://www.gtexportal.org/home/)</small>

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

* Gene expression for *i*=1,...,N genes and *j*=1,...,M samples
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

### Note $\sigma_i$

This is *critical*: different genes *i* have different amount of variability.

<br>

$$ 
\begin{align}
X_{ij} &\sim N(\mu_{ij}, \sigma_i) \\
\mu_{ij} &= \mu_{i0}, \quad j \in A \\
\mu_{ij} &= \mu_{i0} + \delta_i, \quad j \in B
\end{align}
$$

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
  - maybe not, but many are very small for controlled experiment

---

### Is this realistic?

* What about $\sigma_i$ for both groups?
  - often this is enough, larger variance dominates
  - not for single cell experiments
* More complex parametric models: [baySeq](bioconductor.org/packages/baySeq)
* Non-parametric: [SAM / SAMseq](http://statweb.stanford.edu/~tibs/SAM/)

---

### Back to the model

$$
\begin{align}
X_{ij} &\sim N(\mu_{ij}, \sigma_i) \\
\mu_{ij} &= \mu_{i0}, \quad j \in A \\
\mu_{ij} &= \mu_{i0} + \delta_i, \quad j \in B
\end{align}
$$

<br>

* N = 5000, M = 6
* $\delta_i = 0$ for 90%
* $\delta_i = \pm1$ for 10%
* $\sigma_i \sim \Gamma(5,10)$ 

---

### Distribution of $\sigma_i$

![plot of chunk sigmadist](assets/fig/sigmadist-1.png)

---

### Try simple row t-tests

![plot of chunk boxt](assets/fig/boxt-1.png)

---

### Just looking at ranks

![plot of chunk roc](assets/fig/roc-1.png)

---

### Characterize the false positives

Call $\textrm{med}(t) \equiv \textrm{median}( \left| t_i \right| )$ for $i$ s.t. $\delta_i \ne 0$

![plot of chunk mediant](assets/fig/mediant-1.png)

---

### Estimates of $\sigma_i$

$\textrm{med}(t) \equiv \textrm{median}( \left| t_i \right| )$ for $i$ s.t. $\delta_i \ne 0$

![plot of chunk fp](assets/fig/fp-1.png)

---

### New estimator for $\sigma_i$

$\bar{\sigma} = \frac{1}{N} \sum_{i=1}^N \hat{\sigma}_i$

$\tilde{\sigma}_i^B \equiv (1-B) \hat{\sigma}_i + B \bar{\sigma}$

![plot of chunk tildesigma](assets/fig/tildesigma-1.png)

---

### How does new estimator perform

![plot of chunk roc2](assets/fig/roc2-1.png)

---

### Summary

* Top false positives were coming from genes with too low $\hat{\sigma}_i$
* Replace $\hat{\sigma}_i$ with an estimate which is closer to
  $\bar{\sigma}$
* Depending on "close", new estimator dominates at all thresholds
