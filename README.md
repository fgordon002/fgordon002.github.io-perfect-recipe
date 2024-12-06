# Predicting the Perfect Recipe

by Finley Gordon (fgordon@umich.edu)

---

## Introduction 

This analysis is centered around the Recipes and Ratings dataset, which contains information on recipes and ratings from food.com. The dataset contains 234,429 rows and 12 columns with information about 83,782 unique recipes. There are a larger number of rows than unique recipes in the dataset because each row is a review of a recipe, and some recipes have many reviews. 

My motivation for working with this dataset stems from my love of food - I frequently cook for myself, and often when I want to try something new I use a recipe from a website like food.com. I've certainly cooked some un-reviewed recipes that I wasn't satisfied with before, though, despite how appetizing the picture looked (and I promise I'm a good cook!). Recounting these experiences caused me to wonder: what are the characteristics of an above average recipe? Or, in other words, **can we predict an above average recipe based on some of its known features?** It would be useful to know which characteristics of recipes predict a better than average recipe, and to what degree these characteristics are able to predict this outcome. Such predictive abilities would allow us to have some insight on how a recipe will taste when no one has reviewed it yet. Creating a model that tries to make predictions about recipes in this fashion is the central aim of this analysis. 

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
        <td>['60-minutes-or-less', 'time-to-make', 'course', 'main-ingredient', 'preparation', 'for-large-groups', 'desserts', 'lunch', 'snacks', 'cookies-and-brownies', 'chocolate', 'bar-cookies', 'brownies', 'number-of-servings']</td>
        <td>10</td>
        <td>['heat the oven to 350f and arrange the rack in the middle', 'line an 8-by-8-inch glass baking dish with aluminum foil', 'combine chocolate and butter in a medium saucepan and cook over medium-low heat', 'stirring frequently', 'until evenly melted', 'remove from heat and let cool to room temperature', 'combine eggs', 'sugar', 'cocoa powder', 'vanilla extract', 'espresso', 'and salt in a large bowl and briefly stir until just evenly incorporated', 'add cooled chocolate and mix until uniform in color', 'add flour and stir until just incorporated', 'transfer batter to the prepared baking dish', 'bake until a tester inserted in the center of the brownies comes out clean', 'about 25 to 30 minutes', 'remove from the oven and cool completely before cutting']</td>
        <td>9</td>
        <td>['bittersweet chocolate', 'unsalted butter', 'eggs', 'granulated sugar', 'unsweetened cocoa powder', 'vanilla extract', 'brewed espresso', 'kosher salt', 'all-purpose flour']</td>
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
        <td>['60-minutes-or-less', 'time-to-make', 'cuisine', 'preparation', 'north-american', 'for-large-groups', 'canadian', 'british-columbian', 'number-of-servings']</td>
        <td>12</td>
        <td>['pre-heat oven the 350 degrees f', 'in a mixing bowl', 'sift together the flours and baking powder', 'set aside', 'in another mixing bowl', 'blend together the sugars', 'margarine', 'and salt until light and fluffy', 'add the eggs', 'water', 'and vanilla to the margarine / sugar mixture and mix together until well combined', 'add in the flour mixture to the wet ingredients and blend until combined', 'scrape down the sides of the bowl and add the chocolate chips', 'mix until combined', 'scrape down the sides to the bowl again', 'using an ice cream scoop', 'scoop evenly rounded balls of dough and place of cookie sheet about 1 - 2 inches apart to allow for spreading during baking', 'bake for 10 - 15 minutes or until golden brown on the outside and soft & chewy in the center', 'serve hot and enjoy!']</td>
        <td>11</td>
        <td>['white sugar', 'brown sugar', 'salt', 'margarine', 'eggs', 'vanilla', 'water', 'all-purpose flour', 'whole wheat flour', 'baking soda', 'chocolate chips']</td>
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
        <td>['60-minutes-or-less', 'time-to-make', 'course', 'main-ingredient', 'preparation', 'side-dishes', 'vegetables', 'easy', 'beginner-cook', 'broccoli']</td>
        <td>6</td>
        <td>['preheat oven to 350 degrees', 'spray a 2 quart baking dish with cooking spray', 'set aside', 'in a large bowl mix together broccoli', 'soup', 'one cup of cheese', 'garlic powder', 'pepper', 'salt', 'milk', '1 cup of french onions', 'and soy sauce', 'pour into baking dish', 'sprinkle remaining cheese over top', 'bake for 25 minutes or until cheese is lightly browned', 'sprinkle with rest of french fried onions and bake until onions are browned and cheese is bubbly', 'about 10 more minutes']</td>
        <td>9</td>
        <td>['frozen broccoli cuts', 'cream of chicken soup', 'sharp cheddar cheese', 'garlic powder', 'ground black pepper', 'salt', 'milk', 'soy sauce', 'french-fried onions']</td>
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

### Bivariate Analysis

### Interesting Aggregates

### Imputation

---

## Framing a Prediction Problem

---

## Baseline Model

---

## Final Model

