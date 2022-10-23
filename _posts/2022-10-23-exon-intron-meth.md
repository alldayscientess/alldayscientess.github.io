---
layout: post
title: Determining exon and intron methylation
date: ‘2022-10-23’
categories: oyster, methylation
tags: ceabigr
---

An effort to splice out exon and intron methylation levels on a per gene basis.

First thing needed was exon and intron beds that have gene ID information linked.

```{bash}
/home/shared/bedtools2/bin/intersectBed \
-wb \
-a ../genome-features/C_virginica-3.0_Gnomon_exon.bed \
-b ../genome-features/C_virginica-3.0_Gnomon_genes.bed \
| awk -v OFS="\t" '{ print $1, $2, $3, $7}' \
> ../genome-features/C_virginica-3.0_Gnomon_exon-geneID.bed
```

```{bash}
/home/shared/bedtools2/bin/intersectBed \
-wb \
-a ../genome-features/C_virginica-3.0_intron.bed \
-b ../genome-features/C_virginica-3.0_Gnomon_genes.bed \
| awk -v OFS="\t" '{ print $1, $2, $3, $7}' \
> ../genome-features/C_virginica-3.0_Gnomon_intron-geneID.bed
```

Then intersecting 10 bedgraphs

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
   -b ../genome-features/C_virginica-3.0_Gnomon_exon-geneID.bed \
   | awk -v name=$NAME -v OFS="\t" '{ print $0, name}' \
   > ../output/43-exon-intron-methylation/${NAME}_mExon.out
 done  

```

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
   -b ../genome-features/C_virginica-3.0_Gnomon_intron-geneID.bed \
   | awk -v name=$NAME -v OFS="\t" '{ print $0, name}' \
   > ../output/43-exon-intron-methylation/${NAME}_mIntron.out
 done  

```


and then smashing all together

```{bash}
cat ../output/43-exon-intron-methylation/*_mExon.out > ../output/43-exon-intron-methylation/exon-meth_all-samples.out
```

```{bash}
cat ../output/43-exon-intron-methylation/*_mIntron.out > ../output/43-exon-intron-methylation/intron-meth_all-samples.out
```

Then into `tidyverse`

```{r}
exon_meth <- read.delim("../output/43-exon-intron-methylation/exon-meth_all-samples.out", header = FALSE)
```

```{r}
intron_meth <- read.delim("../output/43-exon-intron-methylation/intron-meth_all-samples.out", header = FALSE)
```
