# Are 'Best' Recipes Really the Best?

Written by Aaron Rasin as part of a project for DSC80 at the University of California San Diego

### Introduction

Cooking recipes have been around for thousands of years, passed down from generations, and an important part of all cultures worldwide. Almost a quarter into the 21st century, there are so many recipes on the internet it's now impossible to decide what to use. Without the famous chefs' cookbooks of the past, most people are left to their own devices to investigate a recipe before trying it. **But how do we know which recipes to trust?**

Ratings are clearly designed to indicate quality of a recipe on most websites, but that doesn't mean there aren't any other important factors. People often use things like the number of steps, cooktime, and title of a recipe to gauge the quality of a recipe as well. While the quality of a recipe is subjective, we want to investigate the relationship between these different methods of detecting quality. Specifically, assuming ratings are a trusted measure of public approval, we would like to know if recipes that claim to be high quality using their title, are also enjoyed by the public more often. Is a recipe called 'Best Chocolate Cake Ever' likely to have a better rating than a recipe just called 'Chocolate Cake'?

Using <a href="https://www.kaggle.com/datasets/shuyangli94/food-com-recipes-and-user-interactions">kaggle.com</a> datasets originally sourced from <a href="https://www.food.com/">food.com</a>, we will investigate how indications of high quality (IHQ) in a recipe title, affect the ratings. We found two datasets: one for recipes and one for reviews. The recipes dataset has a row for every recipe (83,782 recipes) posted between 2008 and 2018. There's information such as recipe name, recipe ID, preparation time, contributor ID, submission date, tags, nutrition information, number of steps, steps text, and description for every recipe. The ratings dataset has a row for every comment under a recipe (731,927 comments) from 2008 to 2018 and contains user ID, recipe ID, date of interaction, rating, and review text.

We're interested in classifying recipes containing IHQ in their titles to determine if this has a correlation with recipe ratings. Within this investigation we hope to discover some kind of relationship between IHQ and other recipe features. That is, does including superlative and self-praising language in a recipe title tell you anything about the recipe itself? In order to research this question and the more general relationship surrounding it, we'll first take a look at the ratings. Once we understand the distribution of ratings and its relationship with other features such as number of steps and ingredients, cooktime and nutrition, we can compare how IHQ fits into the equation.



### Cleaning and Exploratory Data Analysis

##### Data Wrangling
We first clean the data and extract the relevant information. This involves merging the recipe dataset with the rating dataset (left merge) so that no rating/comment is without a recipe. Then, we rename a few columns to clarify their meaning. Next, we have to deal with comments that give a rating of zero. This is actually not technically a rating, but just a comment. Ratings are on a scale of 1 to 5, so 0 represents absence of a rating, and thus shouldn't be counted as a rating of 0 out of 5, but should be counted as missing (NaN). Now, we can calculate the average rating of each recipe. This operation creates duplicate columns, which we immediately remove. Let's take a look at the columns of interest of our dataset so far.

| name                                  |   minutes | recipe_date   | nutrition                                    |   n_steps |   n_ingredients |   rating |
|:--------------------------------------|----------:|:--------------|:---------------------------------------------|----------:|----------------:|---------:|
| impossible macaroni and cheese pie    |        50 | 2008-01-01    | [386.1, 34.0, 7.0, 24.0, 41.0, 62.0, 8.0]    |        11 |               7 |        3 |
| impossible rhubarb pie                |        55 | 2008-01-01    | [377.1, 18.0, 208.0, 13.0, 13.0, 30.0, 20.0] |         6 |               8 |        3 |
| impossible seafood pie                |        45 | 2008-01-01    | [326.6, 30.0, 12.0, 27.0, 37.0, 51.0, 5.0]   |         7 |               9 |        3 |
| paula deen s caramel apple cheesecake |        45 | 2008-01-01    | [577.7, 53.0, 149.0, 19.0, 14.0, 67.0, 21.0] |        11 |               9 |        5 |
| midori poached pears                  |        25 | 2008-01-01    | [386.9, 0.0, 347.0, 0.0, 1.0, 0.0, 33.0]     |         8 |               9 |        5 |

##### Data Extraction
Now that we have our single DataFrame, let's make the features of interest easily accessible. First, we notice that the dates in the date column are strings, which is tedious to perform operations on. Instead, we convert all of the dates from string to datetime which makes them much easier to deal with later. Second, we also notice that the nutrition column is a list of unlabeled nutrition facts. This is also quite bothersome to use in data analysis. Since we know we'll want to investigate nutrition later, we separate each item in the list of nutrition facts into its own column, where each value is a float. Let's look at what the nutrition columns look like after extraction.

|   calories |   total fat (PDV) |   sugar (PDV) |   sodium (PDV) |   protein (PDV) |   saturated fat (PDV) |   carbohydrates (PDV) |
|-----------:|------------------:|--------------:|---------------:|----------------:|----------------------:|----------------------:|
|      386.1 |                34 |             7 |             24 |              41 |                    62 |                     8 |
|      377.1 |                18 |           208 |             13 |              13 |                    30 |                    20 |
|      326.6 |                30 |            12 |             27 |              37 |                    51 |                     5 |
|      577.7 |                53 |           149 |             19 |              14 |                    67 |                    21 |
|      386.9 |                 0 |           347 |              0 |               1 |                     0 |                    33 |


### Univariate Analysis

##### Ratings Histogram
Here we made a histogram of ratings to show the distribution. As you can see, it's quite skewed (left) since most recipes have only 5 star reviews. We'll be careful in the future, remembering that this distribution is far from uniform or even normal.
<iframe src="plots/ratings_histogram.html" width=800 height=600 frameBorder=0></iframe>

##### Minutes Histogram
Originally, it would've been insightful to analyze a boxchart instead of histogram of recipe minutes since boxcharts display outliers more clearly. Unfortunately, some outliers are so large (over a million minutes!) that this ruins the visualization. Instead, we've removed times over 12 hours (less than 1% of recipes) in order to better understand the distribution of recipe times. Now that there are fewer outliers, it makes more sense to draw a histogram. This histogram is still quite skewed, so it seems most recipes take less than an hour to make, with a peak at around 20-40 minutes. This distribution is quite the opposite from the ratings histogram in that it is right skewed, so most recipes take less than an hour. 
<iframe src="plots/minutes_histogram.html" width=800 height=600 frameBorder=0></iframe>



### Bivariate Analysis

##### Rating vs Number of Steps in Recipe
 Generally, recipes are rated pretty highly (between 4 and 5). This makes drawing conclusions difficult, but not impossible. Although there is no obvious relationship between number of steps in a recipe and its rating, there is a subtle upward correlation between the two features. Especially among recipes with more than a few ratings (not on the integer intervals) recipes with more steps tend to be rated higher. However, this is by no means conclusive, and likely a result of the imbalance of data. We can also see many recipes along the integer intervals of the x axis since many recipes only have one or a couple reviews that are all the same. Most of the recipes with 60 or more steps are rated five stars.

<iframe src="plots/scatter_nsteps_rating.html" width=800 height=600 frameBorder=0></iframe>

##### Rating vs Number of Calories in Recipe
As we begin to look at how ratings and nutrition are related, we notice essentially the same patterns as from the scatterplot of number of steps and rating. Any slight upward trends in the data that might indicate that recipes with more calories are more highly rated are most likely a result of data imbalance. In general, this is something to be aware of: there are far more recipes with high ratings than low, therefore it's easy to notice patterns that can't be substantiated with further investigation. 

<iframe src="plots/scatter_calories_rating.html" width=800 height=600 frameBorder=0></iframe>



### Analysis of Aggregated Data

##### Recipe Mintues over Time
Let's see how ratings have changed over the years the get a better sense of any time-related trends. Before analyzing the minutes per recipe, we first get rid of outliers. We define outliers as recipes that take more than 24 hours to make (less than 1% of all recipes). We initially look at a scatterplot of recipe minutes over time. This plot reveals not only that the number of recipes has decreased over time, but it looks as if the average minutes of the recipes also decreases over time. Notice the horizontal lines in the scatterplot, which is due to recipes rounding the number of minutes of their recipe to the nearest group of 10 or 30 minutes. The table below the plot shows that the number of recipes per year in fact decreases dramatically from 2008 to 2018.
<iframe src="plots/scatter_minutes_date.html" width=800 height=600 frameBorder=0></iframe>

|   Year |   Minutes |   Recipes per Year |
|-------:|----------:|-------------------:|
|   2008 |  103.785  |              30745 |
|   2009 |   92.8825 |              22547 |
|   2010 |  126.525  |              11902 |
|   2011 |  235.983  |               7573 |
|   2012 |   99.6509 |               5187 |
|   2013 |   79.1229 |               3792 |
|   2014 |  112.65   |               1049 |
|   2015 |   99.6013 |                306 |
|   2016 |  199.51   |                204 |
|   2017 |  101.913  |                288 |
|   2018 |  125.894  |                189 |

##### Average Recipe Minutes per Year
print(aggregated.to_markdown(index=True))
Again, as you can see, the number of recipes per year in the dataset has greatly decreased since 2008. This is a little odd because overall production of content on the internet has grown over time. Perhaps the data somehow doesn't include all of the recipes. Before we investigate missingness, let's continue with aggregation investigation. When we aggregate the data by year, then a very different pattern emerges. Instead of a slow descent of recipe times over the years, we see most years have an average recipe time of 100 minutes except two years. This interesting aggregation shows that 2011 and 2016 were years with much higher average recipe times than any of their neighbors. Needless to say, the patterns of recipe times are hard to describe and even more difficult to predict.
<iframe src="plots/line_minutes_year.html" width=800 height=600 frameBorder=0></iframe>



### Assessment of Rating Missingness
The only column with a substantial number (more than 100) of missing values is the ratings column. The rows with no rating are recipes that never got a comment/rating. Although we could easily argue that rating missingness is dependent on the number of comments, making it missing at random (MAR), we want to investigate further. Are there other features in our dataset that could influence the missingness of ratings, thus making it not missing at random (NMAR)? If we can't find any missingess dependency, this would make rating missing completely at random (MCAR). Specifically, is rating missingness dependent on the nutrition of the recipe? Perhaps recipes that are designed based on certain nutrition values are less popular. This is reasonable since people often cook without thinking about nutrition facts like protein. Let's see if missingness of the recipe ratings depends on the percent daily value of protein in a recipe. We first take a look at a few columns of the data aggregated by rating missingess.

| rating_missing   |   calories |   total fat (PDV) |   sugar (PDV) |   protein (PDV) |
|:-----------------|-----------:|------------------:|--------------:|----------------:|
| False            |    427.191 |           32.3996 |       67.8143 |         33.0933 |
| True             |    515.05  |           39.652  |       95.1138 |         34.3806 |

It's immediately apparent that calories and sugar have very different average values between recipes with and without ratings. Other nutrition facts such as total fat, however, have only slightly different values. In order to test any theories regarding rating missingness depending on the amount of protein or protein in a recipe, we will need to run a permutation test using the absolute difference in protein/fat content as the statistic, with 10,000 trials and a p-value of .05. If less than 5% of the trials in the permutation test contain test statistics equal to or greater than the observed test statistic, we can reject the null hypothesis.

##### Rating NMAR Test on Protein

Null hypothesis: the distribution of protein per recipe <u>without</u> a rating is the **same** as the distribution of the protein per recipe <u>with</u> a rating <br>

Alternative hypothesis: the distribution of protein per recipe <u>without</u> a rating is **different** from the distribution of the protein per recipe <u>with</u> a rating <br>

Test statistic: the absolute difference between average protein per recipe including rating and protein per recipe missing rating.

Result: Our test yielded a p-value of 0.1988, with which we fail to reject the null hypothesis. This means that we don't have sufficient evidence to dispute that rating missingness depends on protein in a recipe. Instead, let's look at other nutrition facts that probably vary more between recipes. We could reason that while differing amounts of protein in a recipe don't determine rating missingness, differing amounts of fat do. This might be because while high protein in a recipe doesn't deter people from trying and rating recipes, high fat might. Especially given the recent efforts in recipes to find low fat alternatives, recipes with high fat are more likely to get no ratings. 

<iframe src="plots/protein_missing_histogram.html" width=800 height=600 frameBorder=0></iframe>

##### Rating NMAR Test on Fat Content (PDV)

Null hypothesis: the distribution of the fat per recipe <u>without</u> a rating is the **same** as the distribution of the fat per recipe <u>with</u> a rating <br>

Alternative hypothesis: the distribution of the fat per recipe <u>without</u> a rating is **different** from the distribution of the fat per recipe <u>with</u> a rating <br>

Test statistic: the absolute difference between fat content (PDV) per recipe including rating and fat content per recipe missing rating.

Result: This permutation test returned a p-value of approximately 0.0, which is certainly below our threshold of 0.05 and thus small enough to reject the null hypothesis. We can now infer that whether or not a recipe has a rating (rating missingness) is dependent on the fat content of the recipe. We declare that recipe rating is NMAR, since it is dependent on percent daily value of fat in the recipe.

<iframe src="plots/fat_missing_histogram.html" width=800 height=600 frameBorder=0></iframe>



### IHQ Hypothesis Testing

##### Defining Indications of High Quality (IHQ)

Now, we'd like to return to investigate our initial question. Basically, does a recipe bragging how good it is actually tell you anything about the quality of the recipe itself? We'd like to know if recipes with words related to quality or authenticity have higher ratings. First, we have to put the recipe titles back into the dataframe since we lost them during the groupby. Then we add a boolean column 'best' with True for recipes with some superlative or indication of high quality (IHQ) in the title. Words we deemed as IHQ (or superlative) are as follows: best, most, amazing, yummy, perfect, ultimate, good, love, and authentic. 

##### Permutation Testing

Next, let's run a permutation test to see if recipes with IHQ in the title have a different rating distribution than recipes without. Our null hypothesis for this test is that there is no difference between ratings in recipes with or without IHQ in the title. Our alternative hypothesis is that recipes with such indications have higher ratings than those without. To run this test we'll use, as our test statistic, average ratings of recipes with IHQ minus average ratings of recipes without IHQ.

<iframe src="plots/best_rating_histogram.html" width=800 height=600 frameBorder=0></iframe>

##### Conclusion

When we run the permutation test (10,000 samples and significance level .05) with ratings as our feature, we get a p value of 0.316. This is not significant enough to reject the null hypothesis, therefore we can't prove recipes with IHQ have better ratings. This means that although we noticed some subtle trends of IHQ recipes having higher ratings, we can not say with any statistical significance that IHQ recipes are any different in terms of publicly judged quality than other recipes. Using this knowledge, we argue that **reading recipe titles with any IHQ in them doesn't reveal anything about the quality of the recipe itself.**










