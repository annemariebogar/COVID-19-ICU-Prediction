# **Brazil COVID-19 Clinical Data and Diagnosis Prediction, 2020**
**Author:** Anne Marie Bogar<br/>
**Active Project Dates:** July 14-27, 2024<br/>
<br/>
## Summary
During the COVID-19 Pandemic, many hospitals around the world were overwhelmed with patients in their intensive care units. This resulted in many patients not being able to receive the care that they needed in a safe amount of time, leading to millions of deaths worldwide. The goal of this project is to design a Machine Learning model to predict which patients will and will not eventually require admission to the ICU. The prediction will allow for hospitals to prepare for incoming patients as well as discharge patients that don't need intensive care earlier on.
<br/>
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

## ADASYN
Because only the rows in which the target value equaled 0 were kept, the modeling dataset heavily favored the non-ICU patients. In order to strengthen the recall of the models and address the imbalance, Adaptive Synthetic Sampling was used to balance the dataset.

## Hyperparameter Tuning
A Grid Search was used to tune hyperparameters to optimize the performance of the models. Unfortunately, the grid search overfitted the models, leading to less accuracy in the overall performance of the models. To combat this, the most important hyperparameters were chosen and manually tuned. In all, each models’ accuracy increased by a couple percentage points.

## Modeling
One model was created to perform two tasks: the first was to predict, with the data given, which patients would wind up in the ICU; the second was to predict, from the first window of 0-2 hours of admission to the hospital, which patients should stay at the hospital for further evaluation and which could go home with remote checkups.<br/><br/>
For the first task, all the data available (of the rows in which the target value did not equal 1) was used for both training the model and testing the model. The data was split 70/30 for the training and testing sets. Of the three types of models used – Random Forest, Logistic Regression, and Decision Tree – the Random Forest was the most accurate. To measure the models, both the accuracy and the F1 score were used. The base level of the model, which consisted of a basic categorical encoding and dropping all rows with missing values, resulted in 86.79% accuracy with an F1 score of 0.88. After preprocessing, feature selection, ADASYN and hyperparameter tuning, the Random Forest model had an accuracy of 94.59% and an F1 score of 0.95. The next best model was the Decision Tree, with a base accuracy of 84.91% and F1 score of 0.87, and a final accuracy of 89.38 % and F1 score of 0.89. Lastly, the Logistic Regression model had a base accuracy of 84.91% with an F1 score of 0.87 and a final accuracy of 78.76% with an F1 score of 0.78.<br/><br/>
For the second task, the data was split into two datasets: one consisting of only the rows with 0-2 hour windows (the First 2 Hours dataset), and the other with the rest of the rows. In order to train the model, the First 2 Hours dataset was split 20/80 and the 20% training data was added to the other dataset. The remaining 80% was used for testing the model. The WINDOW feature was also excluded, as the idea would be that input is always from the first 2 hours of admission. Of the three models used, Random Forest was again the more accurate. The base level of the model, which consisted of a basic categorical encoding and replacing all missing values with 1 (the rows could not be dropped as there would be hardly any data to use) resulted in 67.49% accuracy and an F1 score of 0.59. After preprocessing, feature selection, ADASYN and hyperparameter tuning, the Random Forest model had an accuracy of 88.29% and an F1 score of 0.87. The Decision Tree model was second best, with an initial accuracy 69.26% and an F1 score of 0.63, and a final accuracy of 82.91% and an F1 score of 0.82. Finally, the Logistic Regression model had a base accuracy of 68.91% and an F1 score of 0.63, and a final accuracy of 72.15% with an F1 score of 0.72.

## Conclusion
While these models cannot perfectly predict whether a patient will eventually need intensive care, the accuracy rate is high enough to have helped the overwhelmed hospitals in both preparing room for eventual patients and clearing out the hospital of less serious patients. According to the dataset, only 30% of cases in which the patient eventually needed the ICU were caught in the first 2 hours, and only 50% of cases were caught within the first 4 hours. With use of the Random Forest model, the hospitals will be able to now catch 88% of these cases within the first 2 hours, allowing for more time to prepare recourses for the patients. In addition, according to the dataset, all the patients stayed for over 12 hours in the hospital even though roughly half of them never needed intensive care. With the Random Forest model, the hospitals will be able to predict with an 84% accuracy which patients they can safely send home and monitor from afar, freeing up space in the hospital for more pressing cases.
