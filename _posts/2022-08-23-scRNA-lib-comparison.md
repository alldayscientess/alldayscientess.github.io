---
layout: post
title: Single Cell Library Comparisons
date: ‘2022-08-23’
categories: oyster, scRNAseq
tags:
---

Looking at number of cell, and expression data.

```{r}
gast <- read.delim("../data/gastrula_geneBYcellsexpressed.txt", sep = "\t")
```


```{r}
bla <- read.delim("../data/CPbla_geneBYcellsexpressed.txt", sep = "\t")
```

```{r}
lc <- full_join(bla, gast, by = "id")
```

```{r}
lc %>%
  filter(num_cells_expressed.x == 0, num_cells_expressed.y > 5)
```


```{r}
lc %>%
  filter(num_cells_expressed.y == 0, num_cells_expressed.x > 5)
```

Using the given filters there are 1637 genes not expressed in blastula stage and expresses >5 cells in gastrula library.

And 260 genes expressed in greater that 5 cells in blastula library and not expressed in gastrula library.


---

Also have an expression comparison but not quite sure how to come up with a statistic.

![picp](https://gannet.fish.washington.edu/seashell/snaps/project-sc-RNAseq__RStudio_Server_2022-08-23_14-43-58.png)
