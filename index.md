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

<br>

Michael Love

Dept of Biostatistics

Dept of Genetics

---

### DNA => RNA

![](img/tx_elong.png)

![](img/tx_em.jpg)

---

### Why measure RNA: 
### molecular phenotype

![](img/people.jpg)

---

### Why measure RNA: 
### tissue diversity

![](img/roadmap.jpg)

<small>[Roadmap Epigenomics](http://www.roadmapepigenomics.org)</small>

---

### Why measure RNA: 
### tissue diversity

![](img/gtex.png)

<small>[GTEx](http://www.gtexportal.org/home/)</small>

---

### Why measure RNA:
### within tissue over time

![](img/circ.png)

<small>[Zhang, et al. Circadian gene expression atlas (2014)](http://www.pnas.org/content/111/45/16219.full)</small>

---

### Why measure RNA: 
### discover disease sub-types

![](img/perou.png)

<small>[Perou, et al. Molecular portraits of human breast tumours (2000)](http://www.nature.com/nature/journal/v406/n6797/full/406747a0.html)</small>

---

### Step back: pre-sequencing

* Before sequencing was microarray
* Signal was captured light (positive, "continuous")

![](img/ma2.png) ![](img/ma.jpg)


---

### Motivating problem

* Gene expression for *i*=1,...,N genes and *j*=1,...,M samples
* log of gene expression values are in a tall matrix X
* log here is convenient because gene expression is non-negative and has a
  long tail
* 2 equal sized groups of samples A and B

<br>

$$
\begin{aligned}
X_{ij} &\sim N(\mu_{ij}, \sigma_i) \\
\mu_{ij} &= \mu_{i0}, \quad j \in A \\
\mu_{ij} &= \mu_{i0} + \delta_i, \quad j \in B
\end{aligned}
$$

<br>

$\delta_i \ne 0$ implies DE (differential expression)

---

### Note $\sigma_i$

This is *critical*: different genes *i* have different amount of variability.

<br>

$$ 
\begin{aligned}
X_{ij} &\sim N(\mu_{ij}, \sigma_i) \\
\mu_{ij} &= \mu_{i0}, \quad j \in A \\
\mu_{ij} &= \mu_{i0} + \delta_i, \quad j \in B
\end{aligned}
$$

---

### Goal of differential expression testing

* Find a set of genes for which $\delta_i \ne 0$
* And which obeys false discovery rate bounds
* For genes in our set G at FDR threshold z

<br>

$$
E \left( \sum\nolimits_{i \in G} 1_{ \{\delta_i = 0\} } \right) \le
\left\vert G \right\vert z
$$

---

### Is this realistic?

* Can we accomplish this if all $\delta_i \ne 0$
  - no, because methods often rely on global scaling normalization
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
\begin{aligned}
X_{ij} &\sim N(\mu_{ij}, \sigma_i) \\
\mu_{ij} &= \mu_{i0}, \quad j \in A \\
\mu_{ij} &= \mu_{i0} + \delta_i, \quad j \in B
\end{aligned}
$$

<br>

* N = 5000, M = 6
* $\delta_i = 0$ for 90%
* $\delta_i = \pm2$ for 10%
* $\sigma_i \sim \Gamma(10,10)$ 

---

### Distribution of $\sigma_i$

![plot of chunk sigmadist](assets/fig/sigmadist-1.png)

---

### Try simple row t-tests

![plot of chunk boxt](assets/fig/boxt-1.png)

---

### Just looking at ranks

* Black = t-statistics, grey = random order, X = nominal FDR 20%

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

$\tilde{\sigma}_i^B \equiv B \bar{\sigma} + (1-B) \hat{\sigma}_i$

![plot of chunk tildesigma](assets/fig/tildesigma-1.png)

---

### New estimator performance by rank
![plot of chunk roc2](assets/fig/roc2-1.png)

---

### Summary

* Top false positives were coming from genes with too low $\hat{\sigma}_i$
* Replace $\hat{\sigma}_i$ with an estimate which is closer to
  $\bar{\sigma}$
* Depending on "close", new estimator dominates at all thresholds

---

### How is this hierarchical?

Not your standard diagram, need to formalize

![](img/plate1.png)

---

### limma

* Smyth, G. K. (2004) Linear models and empirical Bayes methods for
assessing differential expression in microarray experiments 
[PDF](http://www.statsci.org/smyth/pubs/ebayes.pdf)
* Developed the hierarchical model introduced by Lonnstedt and Speed
  (2002) for single sample into method for any experiment represented
  as linear model

<br>

$$
\frac{1}{\sigma^2_i} \sim \frac{1}{d_0 \sigma_0^2} \chi_{d0}^2 
$$

<br>



---

### Why inverse $\chi^2$?

* Conjugacy provides closed form solution
* Posterior mean for $1/\sigma_i^2$ given $\hat{\sigma}_i^2$ is
  $1/\tilde{\sigma}_i^2$ with

<br>

$$
\tilde{\sigma}_i^2 = \frac{d_0 \hat{\sigma}_0^2 + d_i \hat{\sigma}_i^2}{d_0 + d_i} 
$$

And $d_i$ as the standard residual degrees of freedom

---

### Note that $d_0$ controls B

$$
\begin{aligned}
\tilde{\sigma}_i^2 &= \frac{d_0 \hat{\sigma}_0^2 + d_i
\hat{\sigma}_i^2}{d_0 + d_i}  \\
 &= \left( \frac{d_0}{d_0 + d_i} \right) \hat{\sigma}_0^2 +
\left( \frac{d_i}{d_0 + d_i} \right) \hat{\sigma}_i^2 \\
 &= B \hat{\sigma}_0^2 + (1-B) \hat{\sigma}_i^2
\end{aligned}
$$

---

### Proper hierarchical model

![](img/plate2.png)

---

### Estimation of hyperparameters

* Need to estimate $d_0, \hat{\sigma}_0^2$, which control strength and
location of *shrinkage* or *moderation*
* $d_0, \hat{\sigma}_0^2$ estimated via first two moments of $\log \hat{\sigma}_i^2$
* (Also need to estimate $\upsilon_{0}$, another parameter giving
  variance of coefficients)

---

### limma vs. naive estimators by rank



![plot of chunk roc3](assets/fig/roc3-1.png)

---

### Rank is not the full picture

* limma also estimating degrees of freedom gained
* X = nominal FDR 20%

![plot of chunk roc4](assets/fig/roc4-1.png)

---

### Summary

* limma provides a hierarchical model for moderation of
  variance estimates in the context of linear models
* Avoids false positives from under-estimation of variance
* Also addresses the gain in degrees of freedom from moderation

---

### RNA-seq: counting molecules

![](img/illumina.png)

---

### RNA-seq: counting molecules

<br>

```
@SRR1265495.1 1/1
CTTTGCCCGCGTGTCAGACTCCATCCCTCCTCTGCCGCCACCGCAGCAGCCACAGGCAGAGGAGGACGAGGACGACTGGGAATCGTAGGGGGCTCCATGAC
+
CCCFFFFFHGFHHHIJJIJIJJIJJJJJJJGIIJJJIJJJJJJJJEFHDEFFFEECDDDBDD?BB?B@BDDD@;9<BBBDBCB@A2<?BDDDBDBD@CDDC
@SRR1265495.2 2/1
CCTGGCTGTGTCCATGTCAGAGCAATGGCCCAAGTCTGGGTCTGGGGGGGAAGGTGTCATGGAGCCCCCTACGATTCCCAGTCGTCCTCGTCCTCCTCTGC
+
@C@FFFDFHHHHGGIIAGHI9GIIIIIGIIIIIGI@@FHGDDDH@GGIBB05?B>ACCCCCCCB<?BBCCBBBBCBCDC3>AC<BB<?CBBB?<@CCCA:@
@SRR1265495.3 3/1
CTGTGTCCATGTCAGAGCAATGGCCCAAGTCTGGGTCTGGGGGGGAAGGTGTCATGGAGCCCCCTACGATTCCCAGTCGTCCTCGTCCTCCTCTGCCTGTG
+
@C@DFFFFGGHDHIIEGDHCGGHIJGEHIIJIIJIEEHGIGEGDDB@@@CDDDDEDDDDDBDDDDDDDDDBCCDDCCDBBDDDDDB@?@DDDDCDDCC@4>
```

for ~30 million reads (often pairs of reads)

---

### RNA-seq: counting molecules

Align to genome or transcriptome

![](img/google.png)

![](img/blat.png)

---

### RNA-seq: counting molecules

* Now for each gene and each sample we can obtain a *count*
  or *estimated count* of fragments
* Why estimated? Because some fragments cannot be uniquely associated
  with genes or isoforms
* Clever and fast algorithms for probabilistically assigning
* Assume we have integer counts $K_{ij}$ of unique fragments or from 
  rounding estimated counts

---

### Counts

1. Either model data with count distributions and inference with GLM
  - [DESeq2](https://www.ncbi.nlm.nih.gov/pubmed/25516281) (2014)
  - [edgeR](https://www.ncbi.nlm.nih.gov/pubmed/19910308) (2010)
  - many more
2. Learn weights associated with log normalized counts and use limma
  - [limma-voom](https://www.ncbi.nlm.nih.gov/pubmed/24485249) (2014)

---

### Most important for statistical analysis

* Total number of fragments is technical artifact
* Heteroskedasticity of counts
* Each gene has different variability

---

### Total number of fragments

![plot of chunk totalnumber](assets/fig/totalnumber-1.png)

---

### Poisson across technical replicates

* From [Bullard 2010](https://www.ncbi.nlm.nih.gov/pubmed?term=20167110)
  take 7 technical replicates
* Calculate expected value $\hat{\lambda}_{ij}$ using DESeq2
* $P(K_{ij} < \hat{\lambda}_{ij})$ assuming $K_{ij} \sim \textrm{Pois}(\hat{\lambda}_{ij})$



![plot of chunk poisson](assets/fig/poisson-1.png)

---

### However, expression not equal across biological replicates

![](img/people.jpg)

---

### Negative Binomial / Gamma Poisson

$K_{ij} \sim \textrm{NB}(\mu_{ij}, \alpha_i)$

$\textrm{Var}(K_{ij}) = \mu_{ij} + \alpha_i \mu_{ij}^2$

![plot of chunk negbin](assets/fig/negbin-1.png)

---

### NB model for RNA-seq

* Similar to our microarray model for $X_{ij}$
* Added an $s_j$ to deal with sequencing depth

<br>

$$
\begin{aligned}
K_{ij} &\sim \textrm{NB}(\mu_{ij}, \alpha_i) \\
\mu_{ij} &= s_j q_{ij} \\ 
\log_2(q_{ij}) &= \beta_{i0}, \quad j \in A \\
\log_2(q_{ij}) &= \beta_{i0} + \delta_i, \quad j \in B
\end{aligned}
$$

<br>

$\delta_i \ne 0$ implies DE (differential expression)

---

### Moderation of dispersion

* In [DESeq2](https://www.ncbi.nlm.nih.gov/pubmed/25516281), a prior on $\log(\alpha_i)$
* Calculate the mean of normalized counts $\bar{\mu}_i$
* A trend line of dispersion over mean $\alpha_{tr}(\mu)$
* Width of the prior $\sigma^2$ estimated via assumption of normal
  sampling variance of $\log(\hat{\alpha}_i)$

<br>

$\log(\alpha_i) \sim N(\log(\alpha_{tr}(\bar{\mu}_i)), \sigma^2)$

---

### Moderation of dispersion

![](img/plate3.png)

---

### Moderation of dispersion



![plot of chunk disp](assets/fig/disp-1.png)

---

### Summary 

* Model for counts similar to the hierarchical linear model,
  but constructed for the dispersion parameter
* Final dispersion estimate plug-in value in DESeq2, edgeR
* [edgeR quasi-likelihood](https://www.ncbi.nlm.nih.gov/pubmed/23104842) takes into account dispersion estimation uncertainty
* [limma-voom](https://www.ncbi.nlm.nih.gov/pubmed/24485249) uses weights on log normalized counts

