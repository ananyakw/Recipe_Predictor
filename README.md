# 🥗 Analyzing The Effect of Healthiness and Complexity of a Recipe on its Rating
A Final Project for DSC80
By Ananya Anand Wasker

## Introduction

This project takes an analytical look into the 'Recipes and Ratings' data in order to understand the factors that influence Internet recipes' average ratings. With more and more food and cooking content becoming widely accessible through online mediums like websites, blogs, TV shows, social media etc., I'm deeply curious about what kind of recipes get high ratings, and more importantly, can these ratings be predicted with knowledge about the recipe itself?

After looking through the two datasets and perusing the source website, I was interested particularly in two features: the nutritional content of the ingredients used in the recipe and the complexity of the recipe instructions. This is not only because healthy/unhealthy recipes or simpler/difficult recipes may be rated in a particular way by audiences, but because these features have a nuanced effect *on each other*. For example, does a healthy recipe being highly challenging to make get a lower rating than a healthy recipe that's much simpler? What about the ratings of an unhealthy recipe that's quick and easy to make?

Therefore, through data exploration, data analysis, hypothesis testing, and predictive modeling, I ultimately aim to answer the research question: **To what extent do nutritional healthiness and complexity of instructions influence recipe ratings, and can I build a model to predict recipe ratings based on the interaction between these two factors?** Working towards this research question can also aid in understanding real-world implications of society's changing attitudes to health, personalized recipe recommendations based on complexity preferences, and many other such use cases.

Let's introduce our data! We originally have two datasets, one with recipe information and one with rating information. While the recipes dataset has 83782 rows × 12 columns, the ratings dataset has 731927 rows × 5 columns. A description of the relevant columns is below.

**Recipes Dataset**
| Column        | Data Type           | Description                      |
| ------------- |:-------------:|-------------------------|
| ```name```      | object |      Recipe name               |
| ```id```     | int64      | Recipe ID             |
| ```minutes``` | int64      |  Minutes to prepare recipe                     |
| ```tags``` | int64      |  Food.com tags for recipe                     |
| ```nutrition```      | object | Nutrition information        |
| ```n_steps```     | int64      | Number of steps in recipe          |
| ```steps``` | object      |  Text for recipe steps, in order                   |
| ```n_ingredients``` | int64           | Number of ingredients                      |


**Ratings Dataset**
| Column        | Data Type           | Description                      |
| ------------- |:-------------:|-------------------------|
| ```recipe_id```     | int64      | Recipe ID             |
| ```rating``` | int64           | Rating given                      |


## Data Cleaning and Exploratory Data Analysis

### Mandatory Merging Step
In order to get information about the average ratings of each recipe, we left merge both datasets on id and recipe_id. We also drop duplicated rows and add average rating for each recipe.  

### Data Inspection

We inspected the data with ```.info()```, ```.describe()```, ```isnull()```. 

### Replacing Invalid Data Points With NaN

Now that we've looked at the different aspects and qualities of the data, we replaced any invalid values with ```NaN``` values. For example, the ```minutes``` column should not have ```0``` values as the recipe can't take 0 minutes to prepare. We did this similar checking by brainstorming other kinds of possible invalid values in other columns and replace NaN. For each column we inspected the ten shortest length values and check their validity, then implement replacements to NaN. We see that for the columns ```tags``` and ```steps```, the first row is invalid and any instances of these values in those columns needs to be replaced with NaN. Any instances of non-alphabetical strings in ```description``` or ```review``` should also be changed to NaN. For numeric columns, any cases of 0 must be replaced with NaN. Even if a data point might not make sense, like the ```minutes``` column having a ```1``` minute value, removing it would introduce selection bias as we don't have any gaurantee that the data point is invalid. Thus, we keep this kind of data.

### Creating New Columns

We see that our ```nutrition``` column is a string representation of a list. We split it in into different columns so we can analyze recipes' healthiness. These new columns are 'total_calories', 'fat_pdv', 'sugar_pdv', 'sodium_pdv',
'protein_pdv', 'sat_fat_pdv', 'carbs_pdv'. 

### Outlier Check

We used the InterQuartile-Range method to check for outliers in each numeric column. We inspected them to check that none of them are invalid values (negatives that go past the lower bound). From analysis, we see that the outliers here are not an issue. For example, even though the lower bound for the ```ratings``` column is 3.75, it is completely plausible that recipes get lower ratings than that. For the rest of the columns, we see from the above DataFrame that no values ever go past the lower bound. Since extremely high values are plausible outliers from the dataset, we can conclude that all invalid outliers have been removed. In this context, it does not make sense to apply transformations to the data to lower outlier counts, as they are plausible values that provide meaningful insights into the recipes and ratings data. Therefore, we conclude our outlier check here.

### Final DataFrame

Here is the cleaned ```merged``` dataset with relevant columns.

| name                                 |   minutes | tags                                                                                                                                                                                                                                                                                               | nutrition                                     |   n_steps |   n_ingredients |   avg_rating |   total_calories |   fat_pdv |   sugar_pdv |   sodium_pdv |   protein_pdv |   sat_fat_pdv |   carbs_pdv |
|:-------------------------------------|----------:|:---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|:----------------------------------------------|----------:|----------------:|-------------:|-----------------:|----------:|------------:|-------------:|--------------:|--------------:|------------:|
| 1 brownies in the world    best ever |        40 | ['60-minutes-or-less', 'time-to-make', 'course', 'main-ingredient', 'preparation', 'for-large-groups', 'desserts', 'lunch', 'snacks', 'cookies-and-brownies', 'chocolate', 'bar-cookies', 'brownies', 'number-of-servings']                                                                        | [138.4, 10.0, 50.0, 3.0, 3.0, 19.0, 6.0]      |        10 |               9 |            4 |            138.4 |        10 |          50 |            3 |             3 |            19 |           6 |
| 1 in canada chocolate chip cookies   |        45 | ['60-minutes-or-less', 'time-to-make', 'cuisine', 'preparation', 'north-american', 'for-large-groups', 'canadian', 'british-columbian', 'number-of-servings']                                                                                                                                      | [595.1, 46.0, 211.0, 22.0, 13.0, 51.0, 26.0]  |        12 |              11 |            5 |            595.1 |        46 |         211 |           22 |            13 |            51 |          26 |
| 412 broccoli casserole               |        40 | ['60-minutes-or-less', 'time-to-make', 'course', 'main-ingredient', 'preparation', 'side-dishes', 'vegetables', 'easy', 'beginner-cook', 'broccoli']                                                                                                                                               | [194.8, 20.0, 6.0, 32.0, 22.0, 36.0, 3.0]     |         6 |               9 |            5 |            194.8 |        20 |           6 |           32 |            22 |            36 |           3 |
| millionaire pound cake               |       120 | ['time-to-make', 'course', 'cuisine', 'preparation', 'occasion', 'north-american', 'desserts', 'american', 'southern-united-states', 'dinner-party', 'holiday-event', 'cakes', 'dietary', 'christmas', 'thanksgiving', 'low-sodium', 'low-in-something', 'taste-mood', 'sweet', '4-hours-or-less'] | [878.3, 63.0, 326.0, 13.0, 20.0, 123.0, 39.0] |         7 |               7 |            5 |            878.3 |        63 |         326 |           13 |            20 |           123 |          39 |
| 2000 meatloaf                        |        90 | ['time-to-make', 'course', 'main-ingredient', 'preparation', 'main-dish', 'potatoes', 'vegetables', '4-hours-or-less', 'meatloaf', 'simply-potatoes2']                                                                                                                                             | [267.0, 30.0, 12.0, 12.0, 29.0, 48.0, 2.0]    |        17 |              13 |            5 |            267   |        30 |          12 |           12 |            29 |            48 |           2 |


### Univariate Analysis

Let's look at the distributions of our relevant variables, for example, ```tags```, ```n_steps```, ```n_ingredients```, ```avg_rating```, and the nutrition columns.

<iframe
  src="top-tags/.html"
  width="800"
  height="600"
  frameborder="0"
></iframe>

From this plot, we see the most frequent tags on recipes on food.com. Notable tags include 'easy', 'low-in-something', '60-minutes-or-less', '30-minutes-or-less', '3-steps-or-less'. This indicates that people tend to make recipes for easier, simpler dishes that are healthy and delicious, contributing to the high frequency of these recipes in the merged dataset. This is an interesting insight because it informs us of people's priorities / what kinds of recipes get the most traffic on food.com! Since we are looking into complexity of recipes, this is a helpful plot!

Now let's look at the distribution of nutritional content.

<iframe
  src="/assets/log-nutritional-distributions.html"
  width="800"
  height="600"
  frameborder="0"
></iframe>

Here, we see visualizations of each of the nutritional columns. We employed the log scale because our data is heavily right-skewed due to the presence of outliers, as seen in the section before this. Doing the log-scale helps lower the effect of outliers in the visualizations. We also see that most histograms here are normally distributed, but have a uptick in 0 values for each nutrient. This high prevalance of 0 values in each column is to be expected because of the sheer variety of recipes and foods. Dessert recipes, of which there are bound to be many, are expected to have almost 0 protein and 0 carbohydrates present, while fried foods are expected to have almost 0 sugar present. These wildly different nutritional goals for each recipe are just a few examples that explain the high 0 values in each plot. Since we are looking into nutrition of foods, this is insightful information!

### Bivariate Analysis

We also investigated relationships between different variables that could help us uncover our research question.

<iframe
  src="correlation-matrix/.html"
  width="800"
  height="600"
  frameborder="0"
></iframe>

From the correlation matrix, we see lots of interesting relationships. Primarily, we see that ```total_calories``` is highly correlated with ```saturated fats```, ```fats```, ```sugars```. ```Carbohydrates``` in recipes are highly associated with ```sugar``` content in recipes. Meanwhile, we see that ```sugar``` is not highly correlated with ```protein``` content in foods, and both ```saturated fats``` and unsaturated ```fats``` are also not highly correlated with ```sodium``` in foods. From this matrix, we can extract that healthier recipes in this dataset will have higher ```protein``` content and lower ```sugar``` content, and therefore, an overall lower ```total_calories``` count. These are a few indicators that we can use later in the prediction model to map out the healthiness of recipes.


## Assessment of Missingness

Describe any missing data issues and how you handled them.

## Hypothesis Testing

Summarize the statistical tests you ran and their results.

## Framing a Prediction Problem

Explain how you formulated the prediction problem, e.g., regression target and features.

## Baseline Model

Describe your baseline model and baseline performance.

## Final Model

Explain your best model, performance metrics, and tuning.

## Fairness Analysis

Discuss any fairness considerations or bias analyses you performed.
