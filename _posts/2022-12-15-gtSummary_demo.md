---
layout: post
title: gtSummary_demo
date: '2022-12-15'
categories: snippet
---

This code below can be used to make a really pretty demographics table in R. The package required is gtsummary. More information on the package can be found here: https://www.danieldsjoberg.com/gtsummary/


Input: Two GWAS summary stats files

Australian_phenotype.csv
```
head(Australian)
  Barcode Sex Age CAD CHD MI HTN FH DM OBS Smk VD  DG Plate. Axiom hCholL hTG lHDL hLDL HL   Wt   Ht HTsq
1 ND-0001   2  73   0   0  0   1  0  0   0   0  0 HTN     41     1      0   0    0    0  0 54.0 1.45 2.10
2 ND-0002   1  29   0   0  0   0  0  1   0   0  0  DM     41     1      0   0    1    0  0 54.0 1.64 2.69
3 ND-0003   2  21   0   0  0   1  0  0   0   0  0 RHD     41     1      0   0    0    0  0 54.0 1.55 2.40
4 ND-0004   2  55   1   0  1   0  0  0   1   0  3 CAD     41     1      1   0    0    1  1 72.0 1.50 2.25
5 ND-0005   2  47   0   0  0   0  0  0   1   0  0 RHD     41     1      0   0    0    0  0 79.0 1.58 2.50
6 ND-0006   2  19   0   0  0   0  0  0   0   0  0 CON     41     1      0   0    0    0  0 47.1 1.52 2.31
    BMI Index
1 25.68     2
2 20.08     0
3 22.48     0
4 32.00     1
5 31.65     1
6 20.39     0
```
Saudi_phenotype.txt
```
head(Saudi)
  S.No Lab.ID..This.column.to.be.deleted. Sample.Vial.Code X GENDER
1    1                        UD-184-P-01           MI-001 s Female
2    2                        UD-184-P-04           MI-002 s Female
3    3                        UD-184-P-08           MI-003 s   Male
4    4                        UD-184-P-09           MI-004 s Female
5    5                        UD-184-P-10           MI-005 s   Male
6    6                        UD-184-P-11           MI-006 s Female
  ACS..MI..1..All.samples.CAD..NO.ACS..MI...2 MI.events AGE..Years. Height..m. Weight..kg.   BMI
1                                           1         5          70       1.60          60 23.44
2                                           1         1          43         NA          NA    NA
3                                           1         3          70       1.64          62 23.05
4                                           1         2          60       1.66          68 24.68
5                                           1         5          65       1.70          68 23.53
6                                           1         3          72       1.62          68 25.91
  Total.serum.cholesterol..mg.dL. High.density.lipoprotein...mg.dL. Low.density.lipoprotein...mg.dL.
1                             121                                40                             65.8
2                             232                                50                            148.0
3                             130                                43                             70.6
4                             161                                53                             83.2
5                             157                                28                            110.0
6                             214                                43                            143.0
  Hypertension Diabetes nationality
1          Yes      Yes       Saudi
2           No       No       Saudi
3          Yes      Yes       Saudi
4          Yes      Yes       Saudi
5          Yes      Yes       Saudi
6          Yes      Yes       Saudi
```
After some reformatting (below) --> pre-gtsummary table is ready to go!
```
> head(datTotal)
# A tibble: 6 × 6
  MI    Sex      Age   BMI Hypertension Group
  <chr> <chr>  <int> <dbl> <chr>        <chr>
1 Case  Female    70  23.4 Yes          Saudi
2 Case  Female    43  NA   No           Saudi
3 Case  Male      70  23.0 Yes          Saudi
4 Case  Female    60  24.7 Yes          Saudi
5 Case  Male      65  23.5 Yes          Saudi
6 Case  Female    72  25.9 Yes          Saudi
```

Output:
![image](https://user-images.githubusercontent.com/66582523/207889929-b6ebebb9-ce2e-4c13-8337-6236f141292b.png)

---
title: "R Notebook"
output: html_notebook
---

# Libraries
```{r}
library(dplyr)
library(gtsummary)
library(stringr)
```

# Set working directory
```{r}

setwd("/Users/tesscherlin/Library/CloudStorage/Box-Box/MI")

```

# Read in the data
```{r}
Australian = read.csv("Australian_phenotype.csv")
Australian = Australian[,c(1:25)]
Saudi = read.table("Saudi_phenotype.txt", header = TRUE, sep="\t")

dim(Australian)
dim(Saudi)

```


# Collect and Demographics of interest
```{r}
# Age
# Sex
# BMI
# HDL
# LDL
# TSC

Australian_sub = Australian[,c(1,2,3,16,18,19,7,24, 7)]
colnames(Australian_sub) <- c("ID", "Sex", "Age", "TotalCholesterol", "HDL", "LDL", "Hypertension", "BMI", "MI")
Saudi_sub  = Saudi[,c(1,5,8,12,13,14,15, 11, 6)]
colnames(Saudi_sub) <- c("ID", "Sex", "Age", "TotalCholesterol", "HDL", "LDL", "Hypertension", "BMI", "MI")

```


# Reformat data a bit more because there were some inconsistencies in entries
```{r}
Saudi_sub_complete = as_tibble(Saudi_sub)
Saudi_sub_complete$MI = ifelse(Saudi_sub_complete$MI==1, "Case", ifelse(Saudi_sub_complete$MI==0, "Control", ""))
Saudi_sub_complete$Hypertension = ifelse(Saudi_sub_complete$Hypertension=="NO", "No", ifelse(Saudi_sub_complete$Hypertension=="Yes", "Yes", ifelse(Saudi_sub_complete$Hypertension=="No", "No", "NA"))) 

Australian_sub_complete = as_tibble(Australian_sub)
Australian_sub_complete$Sex = ifelse(Australian_sub_complete$Sex==1, "Male", ifelse(Australian_sub_complete$Sex==2, "Female", ""))
Australian_sub_complete$MI = ifelse(Australian_sub_complete$MI==1, "Case", ifelse(Australian_sub_complete$MI==0, "Control", ""))
Australian_sub_complete$Hypertension = ifelse(Australian_sub_complete$Hypertension==1, "Yes", ifelse(Australian_sub_complete$Hypertension==0, "No", NA))
```

# Make table
```{r}

# Select the variables (or columns) you want to use
dat1 = Saudi_sub_complete %>% select(MI, Sex, Age, BMI, Hypertension)
dat1$Group = "Saudi"
dat2 = Australian_sub_complete %>% select(MI, Sex, Age, BMI, Hypertension)
dat2$Group = "Aus"
datTotal = rbind(dat1, dat2)
datTotal$BMI = as.numeric(unlist(datTotal$BMI))

# Make your table
table <- datTotal %>% tbl_summary(by = Group, # split table by group
                                  missing = "no", # don't list missing data separately
                                  type = c("Age", "BMI") ~ "continuous",
                                  digits = all_continuous() ~ 1, # round to one decimal point for continuous data
                                  statistic = list(all_continuous() ~ "{mean} ({sd})")) %>%
                                  # This is the add-on section
                                  #add_n() %>% # add column with total number of non-missing observations
                                  #add_p() %>% # test for a difference between groups
                                  modify_header(label = "**Demographics**") %>% # update the column header
                                  bold_labels() 


table
                              

```


