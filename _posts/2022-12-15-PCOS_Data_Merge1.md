This is the code I used for the first batch of data merging for the PCOS Center. I ended up with different files for each category. 
This will need to be changed for the final product.



<img width="370" alt="Screen Shot 2022-12-15 at 11 33 04 AM" src="https://user-images.githubusercontent.com/66582523/207916209-fa9470b3-843b-4adc-aa19-01c37d5003a7.png">



```
library(dplyr)
library(stringr)
```

# Use Box working directory
```
setwd("~/Library/CloudStorage/Box-Box/pcosCenter/")
```

# Read in PCOS Center
```
PCOS_center = read.csv("~/Library/CloudStorage/Box-Box/pcosCenter/PCOS_patients_472.TC.17Oct2022.csv")
PCOS_center = PCOS_center[,1:91]
#sample = read.csv("Sample_6551998.csv", header = FALSE)
#sample = sample[,1:91]
#colnames(sample) <- colnames(PCOS_center)
```

# Read in Penn Medicine files
```

demographics = read.csv("~/Library/CloudStorage/Box-Box/pcosCenter/anuja_shared_demographics_data.csv")
colnames(demographics) <- c("MRN", "First_name", "Middle_name", "Last_name", "DOB", "LIVING", "RACE", "MARITAL STATUS", "USE OF MPM", 
                                "GRAVIDA", "PARA", "TERM", "PRETERM", "ABORTIONS", "SDI_score", "Current_PCP", "PCP_Start", "PCP_last")
substances = read.csv("~/Library/CloudStorage/Box-Box/pcosCenter/anuja_shared_demographics_substances.csv")
colnames(substances) <- c("MRN", "SMOKING.HX", "ALCOHOL.HX")
labs = read.csv("~/Library/CloudStorage/Box-Box/pcosCenter/anuja_shared_labs.csv")
colnames(labs) <- c("MRN", "CLINICAL_VISIT_DATE", "ALK.PHOS", "ALT", "AMH", "AST", "HDL", "LDL", "TOTAL.CHOLESTEROL", "CRP", "hsCRP", "DHEAS", "GLUCOSE", "HBA1C","INSULIN", "X17OHP", "PRL", "SHBG", "FREE.TESTOSTERONE", "TOTAL.TESTOSTERONE", "TRIGLYCERIDES", "TSH")
lab_units = read.csv("~/Library/CloudStorage/Box-Box/pcosCenter/anuja_shared_labs_units.csv")
meds = read.csv("~/Library/CloudStorage/Box-Box/pcosCenter/anuja_shared_meds_all.csv")
colnames(meds) <- c("MRN", "Start_date", "MED", "DOSE", "DOSE_UNIT", "DAYS_SUPPLY", "FREQUENCY", "REFILLS")
PMH = read.csv("~/Library/CloudStorage/Box-Box/pcosCenter/anuja_shared_PMHx.csv")
colnames(PMH) <- c("MRN", "DX_NAME", "ICD10", "ICD9", "COMMENTS_SHORT", "COMMENTS_LONG", "ANNOTATION", "HISTORY_SOURCE")
nutrition = read.csv("~/Library/CloudStorage/Box-Box/pcosCenter/anuja_shared_visits_nutrition.csv")
colnames(nutrition) <- c("MRN", "NUMBER.OF.VISITS.WITH.NUTRITION")
PCOS_visits = read.csv("~/Library/CloudStorage/Box-Box/pcosCenter/anuja_shared_visits_PCOS.csv")
colnames(PCOS_visits) <- c("MRN", "PCOS DX", "Number_PCOS_visits", "Number_total_visits", "first_PCOS_visit", "last_PCOS_visit")

### Need to update labels
BP = read.csv("~/Library/CloudStorage/Box-Box/pcosCenter/anuja_shared_vitals_BP.csv")
colnames(BP) <- c("MRN", "CLINICAL_VISIT_DATE", "BP_SYSTOLIC", "BP_DIASTOLIC")
vitals = read.csv("~/Library/CloudStorage/Box-Box/pcosCenter/anuja_shared_vitals_data.csv")
colnames(vitals) <- c("MRN", "CLINICAL_VISIT_DATE", "WEIGHT_LBS", "WEIGHT_KGS", "HEIGHT", "BMI", "BMI_CALC", "HR")
```

# STEP 1 -- TRAITS
### Substances
### Demographics
### Nutrition
### PCOS Visits
```


#################################################################################
# SUBSTANCES
#################################################################################
# Substances needs to be combined
substances_cleaned = data.frame(MRN = unique(substances$MRN),
                  Numer_Substances_Entries = NA,
                  SMOKING_HX = NA,
                  ALCOHOL_HX = NA)
smoke = NULL
drink = NULL

for (i in 1: length(unique(substances$MRN))) {
# for (i in 1:5) { 
  print(i)
  MRN = unique(substances$MRN)[i]
  print(MRN)
  sub_history = substances[which(substances$MRN %in% MRN), ]
  
  for (j in 1:nrow(sub_history)){
    
  print(j)
  smoke  = paste(smoke, sub_history[j,2], sep = ",")
  drink = paste(drink, sub_history[j,3], sep = ",")

  }
  
  substances_cleaned[i,2] <- nrow(sub_history)
  substances_cleaned[i,3] <- smoke
  substances_cleaned[i,4] <- drink
  
  smoke = NULL
  drink = NULL
}

  substances_cleaned$SMOKING_HX = substr(substances_cleaned$SMOKING_HX, 2, nchar(substances_cleaned$SMOKING_HX))
  substances_cleaned$ALCOHOL_HX = substr(substances_cleaned$ALCOHOL_HX, 2, nchar(substances_cleaned$ALCOHOL_HX))
  
  colnames(substances_cleaned) <- c("MRN", "Substances_Visits", "SMOKING_HX", "ALCOHOL_HX")
  
###############################################################################
  # Demographics
##############################################################################

#duplicates = demographics[which(duplicated(demographics$MRN)),1]
#dem_dup = filter(demographics, grepl(paste(duplicates, collapse="|"), MRN))  
#write.csv(dem_dup, "PCOS_Center_duplicate_demographic_entries.csv")

  

# First, keep only one copy of each women
#demographics_unique = demographics[grepl(paste(unique(demographics$MRN), collapse="|"), MRN), ]

#demographics_unique = demographics %>%
#    distinct(MRN, .keep_all = TRUE)

demographics_cleaned = demographics[!duplicated(demographics$MRN),]
demographics_cleaned$RACE <- NA 

race = data.frame(MRN = unique(demographics$MRN),
                  RACE = NA)

for (i in 1: length(unique(demographics$MRN))) {
# for (i in 1:1) { 
  print(i)
  MRN = unique(demographics$MRN)[i]
  print(MRN)
  sub_race = demographics[which(demographics$MRN %in% MRN), ]
  
  for (j in 1:nrow(sub_race)){
    
  print(j)
  race[i,2]  = paste(race[i,2], sub_race[j,7], sep = ",")

  }
  
  demographics_cleaned[i,7] <- race[i,2]

}

  demographics_cleaned$RACE = substr(demographics_cleaned$RACE, 4, nchar(demographics_cleaned$RACE))





###############################################################################
  # MERGE TRAITS
##############################################################################
dim(demographics_cleaned)
dim(substances_cleaned)
dim(nutrition)
dim(PCOS_visits)

# First, merge the 4 datasets based on MRN 

traits1 = merge(demographics_cleaned, substances_cleaned, by = "MRN")
traits2 = left_join(traits1, nutrition, by = "MRN")
traits3 = left_join(traits2, PCOS_visits, by = "MRN")


Step1_Traits = traits3
  

```

# STEP 2 -- Medical History
### Mainly this is lengthly as it is required to combine multiple medical history instances into each cell
```{r}

#################################################################################
# PMH Patient Medical History
#################################################################################
# PMH needs to be combined
PMH_cleaned = data.frame(MRN = unique(PMH$MRN),
                  Number_PMH_Entries = NA,
                  DX_NAME = NA,
                  ICD10 = NA,
                  ICD9 = NA,
                  COMMENTS_SHORT = NA,
                  COMMENTS_LONG = NA,
                  ANNOTATION = NA,
                  HISTORY_SOURCE = NA)
DX_NAME = PMH_cleaned[,c(1,3)]
DX_NAME[,2] <- NA
ICD10 = PMH_cleaned[,c(1,4)]
ICD10[,2] <- NA
ICD9 = PMH_cleaned[,c(1,5)]
ICD9[,2] <- NA
COMMENTS_SHORT = PMH_cleaned[,c(1,6)]
COMMENTS_SHORT[,2] <- NA
COMMENTS_LONG = PMH_cleaned[,c(1,7)]
COMMENTS_LONG[,2] <- NA
ANNOTATION = PMH_cleaned[,c(1,8)]
ANNOTATION[,2] <- NA
HISTORY_SOURCE = PMH_cleaned[,c(1,9)]
HISTORY_SOURCE[,2] <- NA

for (i in 1: length(unique(PMH$MRN))) {
# for (i in 1:1) { 
  print(i)
  MRN = unique(PMH$MRN)[i]
  print(MRN)
  sub_history = PMH[which(PMH$MRN %in% MRN), ]
  
  for (j in 1:nrow(sub_history)){
    
  print(j)
  DX_NAME[i,2]  = paste(DX_NAME[i,2], sub_history[j,2], sep = ",")
  ICD10[i,2] = paste(ICD10[i,2], sub_history[j,3], sep = ",")
  ICD9[i,2] = paste(ICD9[i,2], sub_history[j,4], sep = ",")
  COMMENTS_SHORT[i,2] = paste(COMMENTS_SHORT[i,2], sub_history[j,5], sep = ",")
  COMMENTS_LONG[i,2] = paste(COMMENTS_LONG[i,2], sub_history[j,6], sep = ",")
  ANNOTATION[i,2] = paste(ANNOTATION[i,2], sub_history[j,7], sep = ",")
  HISTORY_SOURCE[i,2] = paste(HISTORY_SOURCE[i,2], sub_history[j,8], sep = ",")

  }
  
  PMH_cleaned[i,2] <- nrow(sub_history)
  PMH_cleaned[i,3] <- DX_NAME[i,2]
  PMH_cleaned[i,4] <- ICD10[i,2]
  PMH_cleaned[i,5] <- ICD9[i,2]
  PMH_cleaned[i,6] <- COMMENTS_SHORT[i,2]
  PMH_cleaned[i,7] <- COMMENTS_LONG[i,2]
  PMH_cleaned[i,8] <- ANNOTATION[i,2]
  PMH_cleaned[i,9] <- HISTORY_SOURCE[i,2]

}

  PMH_cleaned$DX_NAME = substr(PMH_cleaned$DX_NAME, 4, nchar(PMH_cleaned$DX_NAME))
  PMH_cleaned$ICD10 = substr(PMH_cleaned$ICD10, 4, nchar(PMH_cleaned$ICD10))
  PMH_cleaned$ICD9 = substr(PMH_cleaned$ICD9, 4, nchar(PMH_cleaned$ICD9))
  PMH_cleaned$COMMENTS_SHORT = substr(PMH_cleaned$COMMENTS_SHORT, 4, nchar(PMH_cleaned$COMMENTS_SHORT))
  PMH_cleaned$COMMENTS_LONG = substr(PMH_cleaned$COMMENTS_LONG, 4, nchar(PMH_cleaned$COMMENTS_LONG))
  PMH_cleaned$ANNOTATION = substr(PMH_cleaned$ANNOTATION, 4, nchar(PMH_cleaned$ANNOTATION))
  PMH_cleaned$HISTORY_SOURCE = substr(PMH_cleaned$HISTORY_SOURCE, 4, nchar(PMH_cleaned$HISTORY_SOURCE))
  

 # New loop to remove duplicates from each cell

PMH_cleaned2 = PMH_cleaned  

for (i in 1:nrow(PMH_cleaned)) {
  
  for (j in 3:ncol(PMH_cleaned)) {
    
    d <- sapply(PMH_cleaned[i,j], function(x) paste(unique(unlist(str_split(x,","))), collapse = ", "))
    #d <- unlist(strsplit(PMH_cleaned[i,j], split=","))
    PMH_cleaned2[i,j] = d
  }
}
  

# Save data as Step 2
dim(PMH_cleaned2)
Step2_Medical_History = PMH_cleaned2

```

# STEP 3 -- Deal with Demographics
### Combine Clarity Demographics (STEP1) with Clarity Medical History (STEP2)
### Merge Clarity Demographics with PCOS Center Demographics
### Assess how good Clarity works
```{r}
# Clarity Demographics
clarity_Demographics = left_join(Step1_Traits, Step2_Medical_History, by = "MRN")

# Make Constants
# Subject ID, MRN, Patient Name, DOB, RACE
PCOS_center_Constants = unique(PCOS_center[,c(1,3:4,6:7)])
Clarity_Constants = clarity_Demographics[,c(1,2,3,4,5,7)]

Combined_Constants = PCOS_center_Constants

for (i in 1:nrow(PCOS_center_Constants)) {
  
  MRN = PCOS_center_Constants[i,"MRN"]
  Clarity = Clarity_Constants[grep(MRN, Clarity_Constants$MRN),]
  Combined_Constants[i,6:11] <- Clarity
  
}

Combined_Constants2 = Combined_Constants[, c(1,2,6,3,7,8,9,4,10,5,11)]


# Pull out duplicate MRNs
dup_MRNs = unique(Combined_Constants2[which(duplicated(Combined_Constants2$MRN)),"MRN"])
Combined_Constants_duplicates = Combined_Constants2[Combined_Constants2$MRN %in% dup_MRNs,]

write.csv(Combined_Constants_duplicates, "~/Library/CloudStorage/Box-Box/Longitudinal PCOS study (1)/Merging_Files/PCOS_Center_Duplicate_MRNs.csv")

######################################
# DOB Discrepancies
#####################################
# Read in the formatted
new_Constants = read.csv("PCOS_Center_Duplicate_MRNs.csv")
new_Constants = new_Constants[,-1]
new_Constants_uniq = unique(new_Constants)
new_dup_MRN = unique(new_Constants_uniq[which(duplicated(new_Constants_uniq$MRN)),"MRN"])

new_Constants_dup = new_Constants[new_Constants$MRN %in% new_dup_MRN,]

# Final DOB discrepancies
write.csv(new_Constants_dup, "PCOS_Center_DOB_discrepancies.csv")

#######################################################################
# RACE Discrepancies
######################################################################
#Combined_Constants2_race = unique(Combined_Constants2[,c(1:7,10:11)])

#dup_Race = unique(Combined_Constants2_race[which(duplicated(Combined_Constants2_race$MRN)),"MRN"])
#Combined_Race_duplicates = Combined_Constants2_race[Combined_Constants2_race$MRN %in% dup_Race,]

#write.csv(Combined_Race_duplicates, "PCOS_center_Race_Discrepancies.csv")

#new_Race = read.csv("PCOS_center_Race_Discrepancies.csv")
#new_Race = new_Race[,-1]
#new_Race_wDOB = merge(new_Race, Combined_Constants2, by = "MRN")
#new_Race_wDOB = new_Race_wDOB[,c(2,1,3:7,16,17,8,9)]
#colnames(new_Race_wDOB) <- colnames(Combined_Constants2)
#new_Race_uniq = unique(new_Race_wDOB)


##############################################3
#  
Combined_Constantsx = unique(Combined_Constants2[,-c(3,8)])

# This code maintains only the complete rows
Combined_Constantsxx = Combined_Constantsx[-which(Combined_Constantsx$RACE == ""), ]

##############################################################
# Clean up the data
# DOB
# Race
##############################################################
dim(Combined_Constantsxx)
dim(PCOS_center_Constants)

# Slim the data 
# Remove second MRN and PCOS Center DOB
# Keep only unique rows (any DOB discrepencies)
# Remove Race duplicates
#Combined_Constants3 = unique(Combined_Constantsxx[,c(1,2,4:7,9,10:11)])

#new_Race_uniq_MRN = new_Race_uniq$MRN
#Combined_Constants4 = Combined_Constants3[!Combined_Constants3$MRN %in% new_Race_uniq_MRN, ]

#new_Race_uniq2 = new_Race_uniq[,c(1,2,4:7,9,10:11)]
#Combined_Constants5 = rbind(Combined_Constants4, new_Race_uniq2)
#Combined_Constants6 = Combined_Constants5[order(Combined_Constants5$SUBJECT.ID.),]


# These are the 475 PCOS Patients with corrected DOB and RACE Variables
colnames(Combined_Constantsxx) <- c("SUBJECT.ID.", "MRN", "PATIENT_NAME", "First_name", "Middle_name", "Last_name", "DOB", "RACE_PCOS_Center", 
                                   "RACE_Clarity")

Combined_Constants6 = Combined_Constantsxx
```




# STEP 4 -- Medical Events
### Need to merge PCOS Center Data with Clarity Data
### Medical History
### Labs
### Medications
### Blood Pressure
### Vitals

### Medical History
```{r}

CD_sub = clarity_Demographics[,c(1,6,8,9,16:18,23:35)]

PCOS_center_sub = PCOS_center[,c(3,49:54)]
PCOS_center_sub = unique(PCOS_center_sub)
PCOS_center_sub_type = PCOS_center_sub[complete.cases(PCOS_center_sub[,3:7]), ]

CD_PC = merge(clarity_Demographics, PCOS_center_sub_type, by = "MRN")
CD_PC2 = CD_PC[,c(1,6,23:27, 36:41, 10:15, 8:9,16:22,28:35)]

MedicalHistory = merge(Combined_Constants6, CD_PC2, by = "MRN")
MedicalHistory = MedicalHistory[,c(2,1,3:44)]
MedicalHistory = MedicalHistory[order(MedicalHistory$SUBJECT.ID.),]

write.table(MedicalHistory, "PCOS_Center_Clarity_Merged_Medical_History.csv")

```


```{r}
dim(labs)
dim(meds)
dim(vitals)
dim(BP)

# Apply for all
PCOS_center$dataset <- "pcosCenter"
```

# Blood Pressure
```{r}

BP$dataset <- "clarity"

PCOS_center_BP = PCOS_center[,c(3,5, 16:17,92)]
#PCOS_center_BP$dataset <- "pcosCenter"

# Combine datasets
BP_combined = rbind(PCOS_center_BP, BP)
BP_combined = BP_combined[order(BP_combined$MRN, BP_combined$CLINICAL_VISIT_DATE), ]

# Merge with Constants
BP_new = BP_combined

for (i in 1:nrow(BP_combined)) {
  
  MRN = BP_combined[i,"MRN"]
  data = Combined_Constants6[grep(MRN, Combined_Constants6$MRN),]
  BP_new[i,6:14] <- data
  
}

BP_FINAL = BP_new[,c(6,1,8:14,2:5)]
BP_FINAL = BP_FINAL[order(BP_FINAL$SUBJECT.ID.),]

write.table(BP_FINAL, "PCOS_Center_Clarity_Merged_Blood_Pressure.csv")

```

# LABS
```{r}

labs$dataset <- "clarity"


# Order datasets to be merged
PCOS_center_labs = PCOS_center[,which(colnames(PCOS_center) %in% colnames(labs))]
PCOS_center_labs_ordered = PCOS_center_labs %>% select(names(labs))

# Combine datasets
Labs_combined = rbind(PCOS_center_labs, labs)
Labs_combined = Labs_combined[order(Labs_combined$MRN, Labs_combined$CLINICAL_VISIT_DATE), ]

# Merge with Constants
Labs_new = Labs_combined

for (i in 1:nrow(Labs_combined)) {
  
  MRN = Labs_combined[i,"MRN"]
  data = Combined_Constants6[grep(MRN, Combined_Constants6$MRN),]
  Labs_new[i,24:32] <- data
  
}

Labs_FINAL = Labs_new[,c(24,1,26:32,2:23)]
Labs_FINAL = Labs_FINAL[order(Labs_FINAL$SUBJECT.ID.),]
#Labs_FINAL = merge(Labs_FINAL, BP_FINAL[,c(2,10,11)], by = "MRN", all = TRUE)


write.table(Labs_FINAL, "PCOS_Center_Clarity_Merged_Labs.csv")

```

# Vitals
```{r}
vitals$dataset <- "clarity"
vitals2 = vitals

vitals2$BMI.30..waist.non.asian..88cm..waist.asian..80cm = NA
vitals2$TG.150 = NA
vitals2$HDL.50 = NA
vitals2$BP.130.80.or.HTN = NA
vitals2$gluc.100.or.HbA1c..5.7.or.diabetes = NA

vitals2 = vitals2[,c(1:8,10:14,9)]

# Order datasets to be merged
PCOS_center_vitals = PCOS_center[,which(colnames(PCOS_center) %in% colnames(vitals2))]
#PCOS_center_vitals = cbind(PCOS_center_vitals_ordered, PCOS_center[,c(55:59)])
#PCOS_center_vitals = PCOS_center_vitals[,c(1:8,10:14,9)]

#PCOS_center_vitals_ordered = PCOS_center_vitals %>% select(names(vitals))


# Combine datasets
Vitals_combined = rbind(PCOS_center_vitals, vitals2)
Vitals_combined = Vitals_combined[order(Vitals_combined$MRN, Vitals_combined$CLINICAL_VISIT_DATE), ]


# Merge with Constants
Vitals_new = Vitals_combined

for (i in 1:nrow(Vitals_combined)) {
  
  MRN = Vitals_combined[i,"MRN"]
  data = Combined_Constants6[grep(MRN, Combined_Constants6$MRN),]
  Vitals_new[i,15:23] <- data
  
}

Vitals_FINAL = Vitals_new[,c(15,1,16:23,2:14)]
Vitals_FINAL = Vitals_FINAL[order(Vitals_FINAL$SUBJECT.ID.),]

write.table(Vitals_FINAL, "PCOS_Center_Clarity_Merged_Vitals.csv")

```


# Medications
```{r}
#########################################################################################
meds$dataset <- "clarity"

# No information from PCOS Center

# Merge with Constants
Medications_new = meds

for (i in 1:nrow(meds)) {
  
  MRN = meds[i,"MRN"]
  data = Combined_Constants6[grep(MRN, Combined_Constants6$MRN),]
  Medications_new[i,10:18] <- data
  
}

Medications_FINAL = Medications_new[,c(10,1,12:18,2:9)]
Medications_FINAL = Medications_FINAL[order(Medications_FINAL$SUBJECT.ID.),]

write.table(Medications_FINAL, "PCOS_Center_Clarity_Merged_Medications.csv")

```


# PCOS Center Data
```{r}

PCOS_data = PCOS_center[,c(3,5,31:33,38:43,46:48,60:63,69:71)]

# Merge with Constants
PCOS_Misc_new = PCOS_data

for (i in 1:nrow(PCOS_data)) {
  
  MRN = PCOS_data[i,"MRN"]
  data = Combined_Constants6[grep(MRN, Combined_Constants6$MRN),]
  PCOS_Misc_new[i,22:30] <- data
  
}

PCOS_Misc_FINAL = PCOS_Misc_new[,c(22,1,24:28,2:21)]
PCOS_Misc_FINAL = PCOS_Misc_FINAL[order(PCOS_Misc_FINAL$SUBJECT.ID.),]

write.table(PCOS_Misc_FINAL, "PCOS_Center_Clarity_Merged_PCOS_Misc.csv")

```
