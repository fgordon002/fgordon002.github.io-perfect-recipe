# Predicting the Perfect Recipe

by Finley Gordon (fgordon@umich.edu)

---

## Introduction 

This analysis is centered around the Recipes and Ratings dataset, which contains information on recipes and ratings from food.com. The dataset contains 234,429 rows and 12 columns with information about 83,782 unique recipes. There are a larger number of rows than unique recipes in the dataset because each row is a review of a recipe, and some recipes have many reviews. 

My motivation for working with this dataset stems from my love of food - I frequently cook for myself, and often when I want to try something new I use a recipe from a website like food.com. I've certainly cooked some un-reviewed recipes that I wasn't satisfied with before, though, despite how appetizing the picture looked (and I promise I'm a good cook!). Recounting these experiences caused me to wonder: what are the characteristics of a "good" recipe? Or, in other words, **can we predict the rating of a review based on characteristics of that review's recipe?** It would be useful to know which recipe characteristics predict good reviews, and to what degree these characteristics are able to predict this outcome. Such predictive abilities would allow us to have some insight on how a recipe might be reviewed in the future (and how it might taste in the present) even if no one has reviewed it yet. Creating a model that tries to make predictions about ratings in this fashion is the central aim of this analysis. 

 To accomplish these aims, I will first clean the dataset and perform some exploratory data analysis before framing my prediction problem more specifically, creating a baseline model, and improving upon this model to create a final model. In order to get us started, I will summarize some of the most relevant columns in the initial Recipes and Ratings dataset that I plan to use.

|**Column**    | **Description** |
| --- | --- | --- |
|`'name'`      |Recipe name |
|`'id'`        |Recipe ID |
|`'minutes'`   |Minutes to prepare recipe |
|`'tags'`      |Food.com tags for recipe |
|`'nutrition'` |Nutrition information in the form [calories (#), total fat (PDV), sugar (PDV), sodium (PDV), protein (PDV), saturated fat (PDV), carbohydrates (PDV)]; PDV stands for “percentage of daily value” |
|`'n_steps'`   |Number of steps in recipe |
|`'steps'`     |Text for steps in recipe, in order |
|`'n_ingredients'`   |Number of recipe ingredients |
|`'ingredients'`|List of recipe ingredients |
|`'rating'`    |Rating given |

---

## Data Cleaning and Exploratory Data Analysis

### Data Cleaning

There were multiple data cleaning steps I took for this analysis. Initially, the information for recipes and ratings were stored in two seperate datasets, so as a first step I merged them together to have one complete recipes and ratings dataset. After merging, I replaced all ratings of score "0" with NA, as it is impossible to rate a recipe 0 stars, meaning that all 0 ratings indicate that the recipe wasn't rated at all. It is therefore more appropriate to deem these ratings as missing values. My next cleaning step was to create a new `'avg_rating'` column which stored the average rating for each unique recipe. Next, I split the `'nutrition'` column into seven columns, each one corresponding to one of the values in the list stored in the `'nutrition'` column. This created new columns which were named `'calories_number'`, `'total_fat_PDV'`, `'sugar_PDV'`, `'sodium_PDV'`, `'protein_PDV'`, `'saturated_fat_PDV'`, `'carbohydrates_PDV'`. All of the text columns (`'tags'`, `'steps'`, and `'ingredients'`) appeared as lists but were actually stored as strings, so as a final cleaning step I converted these strings to lists with each element being one word. 

The effects of these cleaning steps were to create several new numerical columns for analysis (from `'nutrition'`), make the text data more workable, create an aggregate summary statistic for each unique recipe (`'avg_rating'`), and ultimately functionally exclude some recipes from analysis by marking their ratings as NA. I selected the relevant columns and saved the result as a final cleaned dataframe, which can be seen below:

<div style="overflow-x:auto;">
  <table>
    <thead>
      <tr>
        <th>name</th>
        <th>id</th>
        <th>minutes</th>
        <th>tags</th>
        <th>n_steps</th>
        <th>steps</th>
        <th>n_ingredients</th>
        <th>ingredients</th>
        <th>calories_number</th>
        <th>total_fat_PDV</th>
        <th>sugar_PDV</th>
        <th>sodium_PDV</th>
        <th>protein_PDV</th>
        <th>saturated_fat_PDV</th>
        <th>carbohydrates_PDV</th>
        <th>rating</th>
        <th>avg_rating</th>
      </tr>
    </thead>
    <tbody>
      <tr>
        <td>1 brownies in the world best ever</td>
        <td>333281</td>
        <td>40</td>
        <td>['60-minutes-or-less', ' time-to-make', ' cour...]</td>
        <td>10</td>
        <td>['heat the oven to 350f and arrange the rack i...]</td>
        <td>9</td>
        <td>['bittersweet chocolate', ' unsalted butter', ...]</td>
        <td>138.4</td>
        <td>10</td>
        <td>50</td>
        <td>3</td>
        <td>3</td>
        <td>19</td>
        <td>6</td>
        <td>4</td>
        <td>4</td>
      </tr>
      <tr>
        <td>1 in canada chocolate chip cookies</td>
        <td>453467</td>
        <td>45</td>
        <td>['60-minutes-or-less', ' time-to-make', ' cuis...]</td>
        <td>12</td>
        <td>['pre-heat oven the 350 degrees f', ' in a mix...]</td>
        <td>11</td>
        <td>['white sugar', ' brown sugar', ' salt', ' mar...]</td>
        <td>595.1</td>
        <td>46</td>
        <td>211</td>
        <td>22</td>
        <td>13</td>
        <td>51</td>
        <td>26</td>
        <td>5</td>
        <td>5</td>
      </tr>
      <tr>
        <td>412 broccoli casserole</td>
        <td>306168</td>
        <td>40</td>
        <td>['60-minutes-or-less', ' time-to-make', ' cour...]</td>
        <td>6</td>
        <td>['preheat oven to 350 degrees', ' spray a 2 qu...]</td>
        <td>9</td>
        <td>['frozen broccoli cuts', ' cream of chicken so...]</td>
        <td>194.8</td>
        <td>20</td>
        <td>6</td>
        <td>32</td>
        <td>22</td>
        <td>36</td>
        <td>3</td>
        <td>5</td>
        <td>5</td>
      </tr>
    </tbody>
  </table>
</div>

### Univariate Analysis

#### Distribution of ratings

We can look into the ratings column to see how it is distributed: 

<iframe src="assets/ratings_histogram.html" width=800 height=600 frameBorder=0></iframe>

It seems that the vast majority of ratings are 5 star ratings. This is not super surprising as many online recipes end up being good, and many people who review recipes do not review them with an intense amount of scrutiny. This may make predicting a "good" recipe more difficult - we may have to come up with different definitons of what it means to be a "good" recipe. There still are a small amount of reviews that are below 5 stars, though, meaning that it might be possible to make predictions solely off of these ratings.

#### Distribution of nutritional information

Many of the new columns that were derived from the original `'nutrition'` column share the same units of percent daily value (PDV). Given this, it makes sense to visualize the distributions of these columns in a side by side box plot. 

<iframe src="assets/nutrition_boxplots.html" width=800 height=600 frameBorder=0></iframe>

There is an immense amount of variation in nutrient PDV values. In order to make a more interpretable boxplot, I plotted the y-axis of PDV values on the log scale. This spread will need to be accounted for in subsequent model building.

### Bivariate Analysis

We can also conduct a multivariate analysis by looking at the relationship between two variables and coloring by a third. I will look at the relationship between calorie number and average rating, coloring by recipe duration. 

<iframe src="assets/recipe_avg_cal_num_scatterplot.html" width=800 height=600 frameBorder=0></iframe>

There doesnt seem to be an incredibly strong relationship between any of these three variables. There might be a slightly positive relationship between calorie number and recipe rating, so that could be used as a predictor in an initial model. 

### Interesting Aggregates

In order to better understand how the numerical features in the dataset are represented in different rating classes, we can group by rating and summarize each numerical column by displaying its mean. The following table displays information pertaining to the mean of each numerical column for each distinct rating class (1-5).

<div style="overflow-x:auto;">
    <table>
      <thead>
        <tr>
          <th>rating</th>
          <th>n_recipes</th>
          <th>avg_minutes</th>
          <th>avg_steps</th>
          <th>avg_ingredients</th>
          <th>avg_cals</th>
          <th>avg_fat_PDV</th>
          <th>avg_sugar_PDV</th>
          <th>avg_sodium_PDV</th>
          <th>avg_protein_PDV</th>
          <th>avg_carb_PDV</th>
        </tr>
      </thead>
      <tbody>
        <tr>
          <td>1</td>
          <td>2869</td>
          <td>92.6699</td>
          <td>10.6281</td>
          <td>8.9146</td>
          <td>486.511</td>
          <td>37.0683</td>
          <td>88.0394</td>
          <td>44.351</td>
          <td>34.0561</td>
          <td>16.3845</td>
        </tr>
        <tr>
          <td>2</td>
          <td>2367</td>
          <td>91.9793</td>
          <td>10.6958</td>
          <td>9.2294</td>
          <td>446.478</td>
          <td>32.7486</td>
          <td>75.1842</td>
          <td>29.4719</td>
          <td>34.2738</td>
          <td>15.0452</td>
        </tr>
        <tr>
          <td>3</td>
          <td>7169</td>
          <td>78.088</td>
          <td>9.99205</td>
          <td>9.19961</td>
          <td>425.84</td>
          <td>31.6414</td>
          <td>65.5727</td>
          <td>27.9231</td>
          <td>34.8625</td>
          <td>13.6846</td>
        </tr>
        <tr>
          <td>4</td>
          <td>37288</td>
          <td>70.3754</td>
          <td>9.5752</td>
          <td>9.10124</td>
          <td>405.072</td>
          <td>29.9471</td>
          <td>56.7858</td>
          <td>26.9398</td>
          <td>34.0496</td>
          <td>12.8305</td>
        </tr>
        <tr>
          <td>5</td>
          <td>169503</td>
          <td>71.7583</td>
          <td>9.98192</td>
          <td>9.051</td>
          <td>415.042</td>
          <td>31.8165</td>
          <td>63.0646</td>
          <td>29.1251</td>
          <td>32.6483</td>
          <td>13.0159</td>
        </tr>
      </tbody>
    </table>
</div>

From this summary table, we can see that there exist some small differences between different rating classes in terms of the numerical columns of that dataset. Counter to what I had thought from the scatterplot, it seems that average calorie number actually decreases as rating increases. The same rough trend is true for average sugar and average carb for each rating category, which makes sense as the most sugary and carb heavy recipies would likely have more calories. It might be important to keep this in mind for potential instances of multicollinearity among these columns; if such multicollinearity were present, this may bias the interpretation of model coefficients once we begin building models. Average minutes is also another interesting column that sharply decreases as rating increases. An important thing to note is the sample size for these different categories is quite different, as we saw earlier.

### Imputation

In terms of imputation, I did not replace any missing values.

---

## Framing a Prediction Problem

This analysis is centered around the question: **can we predict a high rating?** This problem can be interpreted as a **classification** problem, and more specifically a **multiclass classification problem**. The response variable is the rating, and we can evaluate our classification models in a variety of ways; mainly we can use accuracy, precision, and recall to describe the quality of the model. 

When making binary classification models, the resulting predictions can fall into one of four categories:

- **True Positive**: model correctly predicts the positive class
- **False Positive**: model incorrectly predicts the positive class
- **True Negative**: model correctly predicts the negative class
- **False negative** model incorrectly predicts the negative class

These categories only exist for binary classification models, but as we'll see in a second, we can formulate a multiclass classification problem in terms of a set of binary classification models. The previous quality metrics are defined as such:

<head>
<link rel="stylesheet" href="https://cdn.jsdelivr.net/npm/katex@0.10.2/dist/katex.min.css" integrity="sha384-yFRtMMDnQtDRO8rLpMIKrtPCD5jdktao2TV19YiZYWMDkUR5GQZR/NOVTdquEx1j" crossorigin="anonymous">
<script defer src="https://cdn.jsdelivr.net/npm/katex@0.10.2/dist/katex.min.js" integrity="sha384-9Nhn55MVVN0/4OFx7EE5kpFBPsEMZxKTCnA+4fqDmg12eCTqGi6+BB2LjY8brQxJ" crossorigin="anonymous"></script>
<script defer src="https://cdn.jsdelivr.net/npm/katex@0.10.2/dist/contrib/auto-render.min.js" integrity="sha384-kWPLUVMOks5AQFrykwIup5lo0m3iMkkHrD0uJ4H5cjeGihAutqP0yW0J6dpFiVkI" crossorigin="anonymous" onload="renderMathInElement(document.body);"></script>
<style>
.katex-display > .katex {
  display: inline-block;
  white-space: nowrap;
  max-width: 100%;
  overflow-x: scroll;
  text-align: initial;
}
.katex {
  font: normal 1.21em KaTeX_Main, Times New Roman, serif;
  line-height: 1.2;
  white-space: normal;
  text-indent: 0;
}
</style>
</head>

$$\text{accuracy } = \frac{TP + TN}{TP + FP + FN + TN}$$

$$\text{recall } = \frac{TP}{TP + FN}$$

$$\text{precision } = \frac{TP}{TP + FP}$$

Combining these three metrics will help us understand what the best possible model is.

---

## Baseline Model

### One Versus Rest Model

The first model I will use for this prediction problem is a **one versus rest logistic regression model** in which there are five seperate logistic regression models created, each one being a binary logistic regression model for each rating category. More specifically, this model will contain five sub-models, the first predicting whether or not a review gives a recipe a rating of 1 or not, the second predicting whether or not a review gives a recipe a rating of 2, etc. Based on the exploratory data analysis above, I decided to use four features as predictors in this initial model: `'minutes'`, `'calories_number'`, `'total_fat_PDV'`, and `'sodium_PDV'`. I performed a train test split, using 75% of my data to train the model and the remaining 25% to test it. After model training, I computed the model accuracy, recall, and precision on the test data. Since this was a multi class classification model, I couldn't compute recall and precision directly; instead I averaged it over the 5 models. The model had these metrics:

-**accuracy**: 0.7726
-**average recall**: 0.2
-**average precision**: 0.1525

Despite having high accuracy, the model displayed poor average recall and precision. Since these are averages across the five models, this might indicate that the one versus rest classifier is severely over-fitting one sub model and underfitting the rest. We can see if this is the case by plotting a confusion matrix:

<head>
    <title>Confusion Matrix</title>
</head>
<body>
    <h1>Confusion Matrix for Baseline One Vs Rest Model</h1>
    <img src="assets\confusion_matrix_m1.png" alt="Confusion Matrix" width="600">
</body>

Per the confusion matrix, the one versus rest classifier is classifying everything as rated a 5! This is not entirely surprising given the extreme bias in 5 ratings as seen before. This underlying distribution might not be condusive to a good model; we might need to reframe the prediction question.

### Binary Logistic Regression model

---

## Final Model

### Standard Scaling

### Dropping and adding features

### TF-IDF 

### Cross Validation

## Conclusion

