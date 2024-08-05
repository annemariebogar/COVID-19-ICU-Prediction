# **Brazil COVID-19 Clinical Data and Diagnosis Prediction, 2020**
**Author:** Anne Marie Bogar<br/>
**Active Project Dates:** July 14-27, 2024<br/>
<br/>
## Summary
During the COVID-19 Pandemic, many hospitals around the world were overwhelmed with patients in their intensive care units. This resulted in many patients not being able to receive the care that they needed in a safe amount of time, leading to millions of deaths worldwide. The goal of this project is to design a Machine Learning model to predict which patients will and will not eventually require admission to the ICU. The prediction will allow for hospitals to prepare for incoming patients as well as discharge patients that don't need intensive care earlier on.
<br/
## Data
The [data](https://www.kaggle.com/datasets/S%C3%ADrio-Libanes/covid19) is a CSV file containing COVID-19 patient information from three hospitals in Brazil.<br/><br/>
Patient Demographic Information
- PATIENT_VISIT_IDENTIFIER: (numerical) patient identification number
- AGE_ABOVE65: (categorical) 0 for no, 1 for yes
- AGE_PERCENTIL: (categorical)
- GENDER: (categorical) 0 for man, 1 for woman

<br/>Patient Previous Grouped Diseases (categorical, 0 for no and 1 for yes)

| | | |
| ----------- | ----------- | ----------- |
| DISEASE GROUPING 1 | DISEASE GROUPING 2 | DISEASE GROUPING 3 |
| DISEASE GROUPING 4 | DISEASE GROUPING 5 | DISEASE GROUPING 6 |
| HTN (hypertension) | IMMUNOCOMPROMISED | OTHER |

<br/>Blood Results (median, mean, min, max, diff) (numerical)

| | | | | | |
| ----------- | ----------- | ----------- |----------- |----------- |----------- |
| ALBUMIN | BE_ARTERIAL | BE_VENOUS |BIC_ARTERIAL | BIC_VENOUS | BILLIRUBIN |
| BLAST | CALCIUM | CREATININ | FFA | GGT | GLUCOSE |
| HEMATOCRITE | HEMOGRLOBIN | INR | LACTATE | LEUKOCYTES | LINFOCITOS |
| NEUTROPHILES | PO2_ARTERIAL | PO2_VENOUS | PCO2_ARTERIAL | PCO2_VENOUS | PCR |
| PH_ARTERIAL | PH_VENOUS | PLATELETS | POTASSIUM | SATO2_ARTERIAL | SATO2_VENOUS |
| SODIUM | TGO | TGP | TTPA | UREA | DIMER |

<br/>Vital Signs (median, mean, min, max, diff, relative diff) (numerical)

| | | |
| ----------- | ----------- | ----------- |
| BLOODPRESSURE_DIASTOLIC | BLOODPRESSURE_SISTOLIC | HEART_RATE |
| RESPIRATORY_RATE | TEMPERATURE | OXYGEN_SATURATION |

<br/>

- WINDOW: (categorical) the time intervals at which each Blood Result/Vital Signs were taken
- ICU: (cateogrical) target, whether the patient was admitted to the ICU based on their vitals and blood results during that window; 0 for no, 1 for yes

_Note: Each row where the target value = 1 was omitted from the dataset for modeling because the medical staff knew at that point that the patient needed intensive care. 
For the rows remaining for each patient, the ICU value was changed to whether or not the patient ultimately wound up in the ICU._

## Preprocessing
Two of the categorical features, AGE_PERCENTIL and WINDOW, needed to be encoded. AGE_PERCENTIL was converted into the actual percentile number (i.e. 90th to 90). 
To preserve the hour interval, WINDOW was encoded to the median of each interval (i.e. 2-4 to 3). To keep patient anonymity, the numerical data in the dataset was 
scaled according to the Min Max Scaler. As not to disproportionately favor any features, AGE_PERCENTIL and WINDOW were also min max scaled. 
<br/><br/>About 43% of the data in the dataset was missing, presumably because while vitals are checked every hour, blood labs are usually only done once a day. 
However, it can be assumed that the missing values were relatively similar to the values in the preceding or former windows. Therefore, the missing values were backfilled, 
and frontfilled when necessary, for each patient. This was accomplished by grouping by patient and then backfilling for each patient individually. 

## Feature Selection
A correlation matrix showed that the min, max, mean and median for each feature had a correlation of over .95, so all but the mean features were dropped. 
On a similar note, the diff and relative diff features also had a correlation of over .95 and therefore the diff was dropped for each of the vital sign features. 
A further look into the correlation matrix between the ICU target and all the numerical features revealed that about half the features did not even have a neutral relationship 
with the outcome, and these features were dropped as well. As none of the relative diff features had any strong relationship with the ICU values, they were also dropped. 
Scatterplots used for exploratory data analysis confirmed this lack of correlation. After feature selection was complete, the numerical features were brought down from 216 to 23.
