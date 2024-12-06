# Predicting the Perfect Recipe

by Finley Gordon (fgordon@umich.edu)

---

## Introduction 

This analysis is centered around the Recipes and Ratings dataset, which contains information on recipes and ratings from food.com. The dataset contains 234,429 rows and 12 columns with information about 83,782 unique recipes. There are a larger number of rows than unique recipes in the dataset because each row is a review of a recipe, and some recipes have many reviews. 

My motivation for working with this dataset stems from my love of food - I frequently cook for myself, and often when I want to try something new I use a recipe from a website like food.com. I've certainly cooked some un-reviewed recipes that I wasn't satisfied with before, though, despite how appetizing the picture looked (and I promise I'm a good cook!). Recounting these experiences caused me to wonder: what are the characteristics of an above average recipe? Or, in other words, **can we predict whether or not a recipe will be above average based on its features?** It would be useful to know which characteristics of recipes predict a better than average recipe, and to what degree these characteristics are able to predict this outcome. Such predictive abilities would allow us to have some insight on how a recipe will taste when no one has reviewed it yet. Creating a model that tries to make predictions about recipes in this fashion is the central aim of this analysis. 

 To accomplish this aims, I will first clean the dataset and perform some exploratory data analysis before framing my prediction problem more specifically, creating a baseline model, and improving upon this model to create a final model. In order to get us started, I will summarize some of the most relevant columns in the initial Recipes and Ratings dataset that I plan to use.

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

### Imputation

---

## Framing a Prediction Problem

---

## Baseline Model

---

## Final Model

