# To Tag or Not to Tag?

---

## Introduction

This will be an analysis of '30-minute' recipes from [food.com](https://www.food.com/). While all recipes from the dataset are accompanied
by a listed preparation time, not all recipes with a listed preparation time of 30 minutes or less are tagged with
the '30-minutes-or-less' tag. As such, this project will endeavor to determine whether there is a significant
difference in rating among tagged and untagged recipes with sub-30-minute listed preparation times.

### Dataset Information

Number of rows: 83781

Relevant Columns: 

1. tags: the tags associated with each recipe, initially stored as a string 

2. rating: the rating assigned to each recipe, taken as the average rating from the accompanying 'interactions' dataset (float)  

3. minutes: the number of minutes of preparation time listed for each recipe (int)  

Additional Columns:  

4. thirty_min_tag: indicates whether each recipe has the '30-minutes-or-less' tag (bool)  

5. thirty_min_listed: indicates whether each recipe has a listed preparation time of <= 30 minutes

---

## Cleaning and EDA

The main steps that were taken to clean the dataset are as follows:

1. Merged the recipe dataset with the accompanying interactions dataset in order to add rating information to the recipes dataset.  

   > Given that the rating system is on a scale of 1-5, ratings of 0 were not counted, since they likely represent missing ratings.

2. Reformatted the 'tags' column to a list instead of a string in order to more easily access relevant tags.

3. Added additional columns that indicate whether a given recipe is listed with a sub-30-minute preparation time and/or has the '30-minutes-or-less' tag in order to facilitate analysis.

   > Although there are other columns that could have been cleaned, such as the nutrition column, they were left alone since they were irrelevant to the analysis.
  

An excerpt of the relevant collumns of the cleaned dataset can be viewed here:  

| tags                                                                                                                                                                                                                                                                                               |   rating |   minutes | thirty_min_tag   | thirty_min_listed   |
|:---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|---------:|----------:|:-----------------|:--------------------|
| ['60-minutes-or-less', 'time-to-make', 'course', 'main-ingredient', 'preparation', 'for-large-groups', 'desserts', 'lunch', 'snacks', 'cookies-and-brownies', 'chocolate', 'bar-cookies', 'brownies', 'number-of-servings']                                                                        |        4 |        40 | False            | False               |
| ['60-minutes-or-less', 'time-to-make', 'cuisine', 'preparation', 'north-american', 'for-large-groups', 'canadian', 'british-columbian', 'number-of-servings']                                                                                                                                      |        5 |        45 | False            | False               |
| ['60-minutes-or-less', 'time-to-make', 'course', 'main-ingredient', 'preparation', 'side-dishes', 'vegetables', 'easy', 'beginner-cook', 'broccoli']                                                                                                                                               |        5 |        40 | False            | False               |
| ['time-to-make', 'course', 'cuisine', 'preparation', 'occasion', 'north-american', 'desserts', 'american', 'southern-united-states', 'dinner-party', 'holiday-event', 'cakes', 'dietary', 'christmas', 'thanksgiving', 'low-sodium', 'low-in-something', 'taste-mood', 'sweet', '4-hours-or-less'] |        5 |       120 | False            | False               |
| ['time-to-make', 'course', 'main-ingredient', 'preparation', 'main-dish', 'potatoes', 'vegetables', '4-hours-or-less', 'meatloaf', 'simply-potatoes2']                                                                                                                                             |        5 |        90 | False            | False               |       

### Univariate Analysis
<iframe src="ua_fig1.html" width=800 height=600 frameBorder=0></iframe>  

The most relevant column to analyze is, of course, that of the ratings of the recipes, since that is the primary metric by which the recipes will be judged.
A particularly eye-catching feature of this graph is the heavy bias towards higher ratings, which could indicate that users are more likely to leave reviews 
with ratings when they are especially satisfied with a recipe. Additionally, the spike at a four-star rating is probably due to a general tendency towards
'round' numbers, as opposed to more granular ratings.

### Bivariate Analysis
<iframe src="ba_fig1.html" width=800 height=600 frameBorder=0></iframe>  

Featuring a side-by-side comparison of ratings across the categorical variables of listed and tagged rows, it is immediately apparent that each distribution, 
including the baseline, contains a large amount of outliers. This is due to the heavy tendency of users to gravitate towards higher ratings, as alluded to in 
the univariate analysis. Notably, while the distribution of the 'listed' boxplot is almost identical to that of the larger dataset, the distribution of the 
'tagged' boxplot is noticeably weighted downwards, with a first quartile approximately .07 points lower than the other two distributions. This reflects the 
apparent tendency of tagged recipes to have worse performance, which was the driver behind this analysis.

### Interesting Aggregates

|                |   mean_rating |   count |
|:---------------|--------------:|--------:|
| (False, False) |       4.60959 |   46454 |
| (False, True)  |       4.85747 |      15 |
| (True, False)  |       4.67161 |   16696 |
| (True, True)   |       4.62278 |   20616 |

Displayed is a table of nested groups, wherein the first boolean indicates whether a recipe is listed with a <=30min preparation time and the second boolean 
indicating whether that recipe was also tagged with the '30-minutes-or-less' tag. Most notably, there are very few recipes that tagged as taking 30 minutes or less
despite having a listed preparation time of more than 30 minutes. Noticeably, however, among recipes that are listed as taking 30 minutes or less to prepare, there
is a sizeable gap in mean rating between tagged and untagged recipes, with untagged recipes being generally rated higher than their untagged counterparts. This 
could potentially indicate some sort of negative bias against tagged recipes.

---

## Assessment of Missingness

### NMAR Analysis

It is likely that the 'rating' column in the merged (i.e. post-cleaning) dataset is NMAR, since the missingness of a rating for a given recipe is dependent upon 
whether that recipe received any reviews with ratings. This is consistent with exploration of the interactions dataset, in which various recipe IDs with missing 
ratings in the primary dataset were found to either have no reviews or only reviews with ratings of zero, which were nonrated reviews. As such, in order to explain
the missingness of values in the rating column, a new column containing the number of rated reviews for each recipe could be added.

### Missingness Dependency

<iframe src="md_fig1.html" width=800 height=600 frameBorder=0></iframe>

<iframe src="md_fig1.html" width=800 height=600 frameBorder=0></iframe>

The above two graphs compare the distributions of the 'minutes' and 'n_steps' columns when recipes woth missing descriptions are present and absent. After running a permutation test using the difference between column means including and not kncluding the description-less recipes, it appears that the 'n_steps' column can be used to predict recipe missingness (approximate p-value of .01), which would indicate that the 'description' column is MAR.

---

## Hypothesis Testing

### Test Parameters

**Test Statistic (T):** mean of listed, nontagged (LN) recipes - mean of listed, tagged (LT) recipes
**Significance Level:** 95%

**Null Hypothesis:** T = 0; there is no difference between the average ratings of LN or LT recipes
**Alternative Hypothesis:** T > 0; the average rating of LN recipes is higher than that of LT recipes

The test statistic and subsequent hypotheses were chosen because the analysis aims to directly compare the directional effect of tagging recipes, which is why the test
statistic does not incorporate an absolute value. The significance level was chosen because a 95% signficance level is conventionally considered to be the standard burden
of proof.  

The analysis will be performed via a permutation test, taking a DataFrame of only recipes with listed preparation times of 30 minutes or less and shuffling which of those 
recipes will be given the '30-minutes-or-less' tag. After 10000 permutations, the p-value will be calculated as the number of times a test statistic was observed to be greater
than or equal to the observed statistic, divided by the total amount of trials.

### Conclusion

Even after conducting the test several times with varying amounts of permutations, the test consistently produced a p-value of zero, meaning that not one of the many permutations
resulted in a higher difference in average ratings in comparison to the observed statistic. This would suggest that untagged thirty-minute recipes are significantly more likely 
to receive higher average ratings relative to their untagged counterparts.  

As such, the answer to the question of 'to tag or not to tag?' appears to be convincingly in favor of the latter; while it is difficult to determine from the data the exact reason 
why untagged thirty-minute recipes enjoy such a superiority, it would seem that recipe-posters are better off leaving the tags behind.

 *by Nicholas Jumaoas*

---
