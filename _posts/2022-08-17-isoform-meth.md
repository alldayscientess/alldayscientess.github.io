---
layout: post
title: Isoform count and gene methylation
date: ‘2022-08-17’
categories: oyster, methylation
tags: ceabigr
---

Some thoughts on the relationship of isoform count and methylation level.

If we ignore treatment we see some interesting patterns between males and females.

![mf](http://gannet.fish.washington.edu/seashell/snaps/ceabigr__RStudio_Server_2022-08-17_10-44-08.png)

At first glance there seems to be the case that males have gee more genes at a higher methylation level, but at further glance with this figure all we can really say is that female have high isoform counts in genes around 50% methylation whereas in males seem to show a high isoform count in genes that have a higher methylation level.

However an issue with this is that is is challenging to quantify points that overlap.

Lets step back and just look at isoform count distribution within sexes.

![iso](http://gannet.fish.washington.edu/seashell/snaps/ceabigr__RStudio_Server_2022-08-17_11-00-38.png)

Looking at this, in fact, there does appear to be case where males does do lean towards to having genes at higher methylation level, though concerned there might be something hidden or masked.

We can even take the visualization at individuals (colored by sex).

![more](http://gannet.fish.washington.edu/seashell/snaps/ceabigr__RStudio_Server_2022-08-17_11-06-44.png)
