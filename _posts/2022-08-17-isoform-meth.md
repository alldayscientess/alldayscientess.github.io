---
layout: post
title: Determining gene methylaiton
date: ‘2022-08-16’
categories: oyster, methylation
tags: ceabigr
---

Here is some code for getting gene methylation. Will also add to handbook.


Will be using 10x coverage bedgraphs as spit out of our Bismark pipeline.

```{bash}
cd ../data/big

wget -r \
--no-check-certificate \
--quiet \
--no-directories --no-parent \
-P . \
-A R1_val_1_10x.bedgraph \
https://gannet.fish.washington.edu/seashell/bu-mox/scrubbed/120321-cvBS/

```

Then taking these files and intersecting with genes in genome.

```{bash}
 cd ../data/big/

 FILES=$(ls *bedgraph)

 cd -

 for file in ${FILES}
 do
    NAME=$(echo ${file} | awk -F "_" '{print $1}')
    echo ${NAME}
   /home/shared/bedtools2/bin/intersectBed \
   -wb \
   -a ../data/big/${NAME}_R1_val_1_10x.bedgraph \
   -b ../genome-features/C_virginica-3.0_Gnomon_genes.bed \
   | awk -v name=$NAME -v OFS="\t" '{ print $0, name}' \
   > ../output/40-gene-methylation/${NAME}_mGene.out
 done  


```

Individual output files look like..

```
head ../output/40-gene-methylation/36F_mGene.out
```

```
NC_035780.1	13597	13599	0.000000	NC_035780.1	13578	14594	gene-LOC111116054	0	+	36F
NC_035780.1	13725	13727	0.000000	NC_035780.1	13578	14594	gene-LOC111116054	0	+	36F
NC_035780.1	14144	14146	3.703704	NC_035780.1	13578	14594	gene-LOC111116054	0	+	36F
NC_035780.1	14430	14432	46.666667	NC_035780.1	13578	14594	gene-LOC111116054	0	+	36F
```

and then smashing all together

```{bash}
cat ../output/40-gene-methylation/*_mGene.out > ../output/40-gene-methylation/meth_all-samples.out
```

Then into `tidyverse`

```{r}
meth_all <- read.delim("../output/40-gene-methylation/meth_all-samples.out", header = FALSE)
```

```{r}
gm <- meth_all %>%
   mutate(art = paste(V8, V11, sep = '_')) %>%
   group_by(art) %>%
   summarize(avg_meth = mean(V4))


```

```{r}
mc <- inner_join(gm, ic, by = "art") %>%
   separate(art, into = c("gene", "sample"), sep = "_") %>%
   separate(sample, into = c("number", "sex"), sep = -1)

```
