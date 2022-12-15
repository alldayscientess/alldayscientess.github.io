---
layout: post
title: gtSummary_demo
date: '2022-12-15'
categories: snippet
---


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
# Age
# Sex
# BMI
# HDL
# LDL
# TSC
```{r}
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


