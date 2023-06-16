# To Niche or Not to Niche?

---

## Framing the Problem

Building off of the exploratory data analysis from Recipe-Data-Analysis, this will take the cleaned dataset and use
it to create a regression model designed to predict the rating of recipes. The response variable of rating was chosen 
because it is widely considered to be an indication of the quality of the rated recipe. The overarching goal of this 
model is to determine whether the 'niche-ness' of a recipe influences its rating. To that end, each tag present in the 
dataset is assigned a 'niche score,' which is calculated as the inverse of the tag's frequency.

Since the developed models engineer features to perform linear regression in order to predict recipe rating, the metrics 
chosen to evaluate the models are their RMSE and R-squared values, with the former used to provide an easily understood
measure of average model error and the latter to provide a standardized measure of goodness-of-fit. Using two metrics 
facilitates a more thorough assessment of the models, since using only one metric might fail to capture important
aspects of model performance. Furthermore, these metrics were chosen because they are widely used and thus more easily
accessible and interpretable.

---

## Baseline Model

The baseline model sought to use raw niche score data to form a rough idea of the effect of niche-ness on recipe rating. 
However, this niche-ness is not readily apparent from the taglist present in the original dataset, so a minor amount of 
engineering was conducted in order to produce some basic criteria by which to judge each recipe. The first feature to 
be created was niche_rating, which was the sum of the niche scores of all the tags in the recipe's taglist, with the 
second feature, max_niche, being the maximum niche score found in the recipe's taglist; since these are derived from the 
inverse term frequencies of the tags, both of these features are naturally quantitative.

As might be expected of a baseline model, its performance was quite bad. Its test-set RMSE of .642 indicated an incredible 
amount of average error, considering that the response variable was measured on a scale of one to five. Its R-squared value 
didn't redeem it either, with a test-set value of less than one hundred-thousandth, indicating that practically none of the 
variance can be explained by the model. With such results, it was fairly inarguable that the performance of the base model 
was, to be frank, terrible.

---

## Final Model

Clearly, there were fundamental problems with the baseline model, but it was too late to turn back; as such, the final model 
sought to address some fundamental problems with the chosen features. For one thing, the niche_rating variable was easily biased 
towards recipes with longer taglists, so the niche_average feature was created in order to offset that bias by using the average
niche score instead of the sum. Furthermore, the max_niche feature didn't quite sufficiently capture the magnitude of niche-ness
of each recipe, so the niche_range feature was derived as the difference between the maximum and minimum niche scores of the tags 
in the recipe taglists.

These updated features were created with the intention of iterating upon the baseline model, so linear regression was once again 
used as the backbone of the modeling algorithm. There weren't really many hyperparameters to adjust when dealing with linear 
regression, so using GridSearchCV to tune the hyperparameters pretty much only revealed that fit_intercept should be set to True. 
Additionally, the feature values were standardized during preprocessing, which wasn't present in the first model.

Despite this, the final model's RMSE was only negligibly different from that of the baseline model, indicating that the average 
error of the model did not meaningfully change. However, the R-squared of the model increased by a factor of ten, indicating that 
the new features allowed the model to explain significantly more of the variance in the dataset. 

---

## Fairness Analysis

As a callback to the exploratory data analysis performed in Recipe-Data-Analysis, the fairness analysis will determine whether the 
model performs differently depending on whether the recipes being analyzed have the '30-minutes-or-less' tag. 

**Test Statistic (T):** R-squared of tagged model - R-squared of nontagged model
**Significance Level:** 95%

**Null Hypothesis:** T = 0; tagging does not influence model performance
**Alternative Hypothesis:** T > 0; tagging causes model performance to increase

Performing a permutation test over 1000 trials produced a p-value of .672, suggesting that there is not enough evidence to reject the 
null hypothesis. This suggests that there is no significant change in model performance among different populations, indicating that 
the model seems to be fair, at least.

---
