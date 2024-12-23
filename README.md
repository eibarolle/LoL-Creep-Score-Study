## Introduction

### Introduction and Question Identification

League of Legends is a popular multiplayer online battle arena, or MOBA, game developed by Riot Games in 2009. With over 100 million registered players worldwide, it is one of the most influental games in esports history. To analyze competitive matches, we will be working with a professional data set developed by Oracle’s Elixir containing official League matches from 2022. This dataset includes various identifications and gameplay metrics from these matches, like player position, champion choices, kills, and earned gold. These variables are useful for analyzing gameplay viability and goals. 

While playing League of Legends, one of the main ways to get gold is by killing monsters around the map, including respawning minions and shared objectives teams fight over. The statistical metric "creep score" keeps track of every monster killed, with a general rule of thumb that every basic minion killed awards 1 creep score. With more monsters killed and thus more gold, the player can purchase stat increasing items, which could theoretically improve chances of victory and change gameplay strategies.

The central question we are interested in the relationship creep score per minute (cspm) has to other gameplay variables and metrics, including win rate. CSPM is used over creep score in general to account for varying game lengths. Using data analysis, the impact cspm and other statistics have on each other can be measured and tested. We can then utilize these given features to predict cspm. This predictive model can be used to strategize around high cspm objectives and picks, as well as improve the user's gameplay.

#### Introduction of Columns

This Oracle's Elixir dataset includes various identifications and gameplay metrics from competitive League matches in 2022. There are 150180 rows in this dataset, as well as 161 columns/variables. Here is a list of the key columns analyzed:

- gameid: Contains a unique string identifier representing an individual match.

- participantid: Contains an integer representing the participant's id. 1-5 represents a Red team player, 6-10 represents a Blue team player, 100 represents the Red team itself, and 200 represents the Blue team itself.

- result: Contains an integer representing the outcome of a match for the corresponding participant. 1 represents a win, 0 represents a loss.

- patch: Contains a string representing the game patch the match was played in.

- cspm: Contains a float representing the average monsters + minions killed per minute by a participant.

- kills: Contains an integer representing how many times a participant lands a finishing blow on an enemy champion.

- position: Contains a string represents the participant's position (top, jg, mid, adc, sup, or team itself).

- champion: Contains a string represents the champion picked by the participant.

- totalgold: Contains a int representing the overall amount of gold a player gained throughout the match from individual and shared actions.

- earnedgold: Contains a int representing the overall amount of gold a player gained throughout the match solely from individual actions.

- league: Contains a string represents the specific league tournament the match was held in, such as LEC for European matches.

## Data Cleaning and Exploratory Data Analysis
### Data Cleaning
To clean the dataset, some columns are in a string format instead of a numerical format, which makes it harder to graph and find measures of central tendency. Therefore, multiple columns, including cspm, are converted to floats or integers with astype(). In addition, to account for missing values, every empty value in the dataset is converted to np.nan using replace(). In addition, a column 'cspm_missing' is recorded as "Yes" if a "cspm" value is missing and "No" if not to further analyze its missingness. Finally, for participants, two new dataframes are created, one holding only players and the other holding only teams for further analysis on both. This is because team stats are aggregates of each of its players. Here is the head of the cleaned dataset with some relevant columns:
```py
print(df[0:5][['gameid', 'participantid', 'playername', 'teamname', 'position', 'cspm', 'cspm_missing']].to_markdown(index=False))
```
### Univariate Data Analysis
To start off, univariate data analysis is performed on the cspm column for the full cleaned dataset using a histogram.

<iframe src="plots/cspm_plot.html" width=800 height=600 frameBorder=0></iframe>

This plot shows three different normal curves with their own peaks from 0-3, 5-12, and a smaller one at 25-40 cspm, which could be because different positions (including teams) on average have different cspm. However, for the overall dataset, it has a slight right skew. 

Univariate data analysis is performed again on the earned gold column.

<iframe src="plots/earned_plot.html" width=800 height=600 frameBorder=0></iframe>

This plot shows a high normal (though arguably very slightly right skewed) distribution from 0-20k earned gold, which is followed by a smaller, spread out distribution from 20k-60k earned gold. This could be because the first distribution is oriented towards the players' earned gold, while the second could represent the team's total earned gold.

### Bivariate Data Analysis
To further analyze the relationship between cspm and earned gold, a scatterplot is formed between them for bivariate analysis.
<iframe src="plots/cspm_gold_scatter.html" width=800 height=600 frameBorder=0></iframe>
In this scatterplot, cspm and earned gold have a positive correlation, and the data is organized into two clusters: one from 0-15 cspm, and the other from 20-50 cspm.
Another scatterplot is formed on the relationship between cspm and the participant's kills.
<iframe src="plots/cspm_kills_scatter.html" width=800 height=600 frameBorder=0></iframe>
In this scatterplot, the data is again organized into two clusters: one from 0-15 cspm, and the other from 20-50 cspm. The latter cluster has a greater average kill count compared to the 0-15 cspm cluster.
From these findings, we learned that cspm has a positive correlation with earned gold and kills, which directly result in a greater win percentage.

### Interesting Aggregates
For aggregates, we grouped cspm by position and analyzed the mean of each group.
```py
print(pivot_position.to_markdown(index=False))
```
From the pivot table, we can observe that the means are significantly different for each position. Even accounting for team being the sum of each participant there, mid and bot have the highest mean cspm while sup(port) has by far the lowest. Thus, we should keep in mind position's impact when trying to predict cspm.
## Assessment of Missingness
### NMAR Analysis
The column believed to be NMAR (Not Missing At Random) is the column 'teamid'. This is because if a competitive match was played against a newer or exhibition team, they might not have official documentation and thus wouldn't have a teamid. Therefore, this column being missing would depend upon itself and the team circumstances, so it would count as NMAR. In addition, the 'teamid' column being missing doesn't follow any trends from other columns (in other words, no dependency). For additional data to test whether 'teamid' is NMAR or MAR (Missing At Random), it could be useful to record another column 'uniqueteam' on whether each participant has played for the same team throughout all of 2022 or not. This will make it easier to analyze whether the missingness is due to the column itself, the players, or a sampling error.
### Missingness Dependency
The missingness of our focus column, 'cspm', will be tested on against the columns "league" and "side" using permutations. The significance level chosen is 0.1 for leniency, and the test statistic will be TVD because both other columns are categorical. For "league", the hypotheses are as follows:
##### Null Hypothesis: The distribution of league when cspm is missing is the same as the distribution of league when cspm isn't missing. In other words, cspm isn't MAR based off league.
##### Alternative Hypothesis: The distribution of league when cspm is missing is not the same as the distribution of league when cspm isn't missing. In other words, cspm is MAR based off league. 
For the test, the tvd for the observation data is recorded by taking the value counts of each league's CSPM missingness. After that, 1000 permutations are conducted on the 'cspm_missing' column, and each permutation's tvd is recorded. Once the permutations are done, the p-value is recorded by checking the proportion of simulated tvds that are at least as large as the observed tvd. After the permutation tests, the observed statistic was recorded as 24.318279569892475, and the p-value was recorded as **0.0**. The empirical distribution of the TVDs is graphed below.
<iframe src="plots/league_tvdf.html" width=800 height=600 frameBorder=0></iframe>
From the permutation test's resulting p-value of 0.0 being smaller than the significance level, and no simulated tvd being at least as large as the observed tvd, we can reject the null hypothesis that the distribution of league when cspm is missing is the same as the distribution of league when cspm isn't missing. In other words, the missingness of cspm depends on the league column.
A similar permutation test is conducted on cspm's missingness in relation to team side with the same test statistic and significance level.

##### Null Hypothesis: The distribution of team side when cspm is missing is the same as the distribution of team side when cspm isn't missing. In other words, cspm isn't MAR based off team side.
##### Alternative Hypothesis: The distribution of team side when cspm is missing is not the same as the distribution of team side when cspm isn't missing. In other words, cspm is MAR based off team side. 
For the test, the tvd for the observation data is recorded by taking the value counts of each side's CSPM missingness. After that, 1000 permutations are conducted on the 'cspm_missing' column, and each permutation's tvd is recorded. Once the permutations are done, the p-value is recorded by checking the proportion of simulated tvds that are at least as large as the observed tvd. After the permutation tests, the observed statistic was recorded as 0.0, and the p-value was recorded as **1.0**. The empirical distribution of the TVDs is graphed below.
<iframe src="plots/side_tvdf.html" width=800 height=600 frameBorder=0></iframe>
From the p-value equaling 1.0 and thus being greater than the significance level, we fail to reject the null hypothesis that the distribution of team side when cspm is missing is the same as the distribution of team side when cspm isn't missing. In other words, the missingness of cspm isn't MAR based on the team side column.

## Hypothesis Testing
To further investigate cspm's impact on winning League games, a permutation test can be conducted testing whether there is a significant difference in the cspm distribution of losing and winning teams. This investigation is beneficial for observing how much cspm affecting a participant's chances at winning.
#### Null Hypothesis: 
The proportion of "winning" results among teams with a higher creep score/minute (cspm) from the 2022 League of Legends Dataframe is equal to 0.5. This null hypothesis is chosen because the team rows are from matches against each other, meaning in all cases, one team must win, and the other must lose. Thus, the proportion of "winning" results is ${1}/{2}$, or 0.5.
#### Alternative Hypothesis: 
The proportion of "winning" results among teams with a higher creep score/minute from the 2022 League of Legends Dataframe is greater than 0.5. This alternative hypothesis is chosen because we hypothesize that with a larger creep score, teams will be able to accumulate more gold for better stats.
#### Test statistic: 
The win rate proportion for teams with a higher creep score/minute. This is chosen because the data can be cleaned and grouped into both results and the team with the higher cspm.
#### Significance Level: 
$\alpha$ = 0.05. This significance level is chosen because of its balance between finding significant patterns and reducing statistical errors.
From the permutation test, the observed test statistic (proportion of higher cspm teams who won) equals 0.777, and the p-value for the test equals **0.0**. The empirical distribution of the proportions is graphed below.
<iframe src="plots/q4.html" width=800 height=600 frameBorder=0></iframe>
Because the p-value for the permutation test was lower than 0.05, we reject the null hypothesis that the proportion of "winning" results among teams with a higher creep score/minute (cspm) from the 2022 League of Legends Dataframe is equal to 0.5. This result represents how having a higher cspm is positively correlated with winning and thus both beneficial and sought after.

## Framing a Prediction Problem
### Problem Identification
From the last part, we've learned that cspm has a considerable impact on winning results for competitive League of Legends matches. To prioritize this statistic for better games, creating a prediction model based off other columns/variables could be useful, such as league or champion. For this problem, we can utilize **regression** to predict cspm, since it's a numerical variable. So, a prediction problem now arises: **Can we predict a player's creep score/minute based off of other game statistics?**

As stated earlier, creep score/minute will be utilized as the response variable, since it's a reliable measure for creep score that accounts for varying game lengths. To evaluate our model, we will be using the numerical regression metric RMSE (Root Mean Squared Error) for interpretability compared to similar counterparts like MSE and balancing outliers.
## Baseline Model
### Model Description

For our baseline model, we aimed to predict a player’s creep score per minute (cspm) using two initial features:

1. `position` (categorical, nominal): The role the player occupies (e.g. top, jng, mid, bot, sup)

2. `totalgold` (quantitative): The total amount of gold a player accumulated during the match.

We applied a OneHotEncoder to the position column. This transformation converts the nominal categorical variable into multiple binary indicator variables, one for each possible position, ensuring that our linear model can properly handle categorical data.
We used a StandardScaler on the totalgold column. Scaling numerical features ensures that differences in the scale of various predictors do not disproportionately influence the model’s coefficients.

### Model Implementation

We implemented these transformations and our model (a LinearRegression model) in a single sklearn Pipeline. The pipeline ensures that data processing and model fitting occur seamlessly in a single workflow. We split the dataset into training and test sets, trained the model on the training set, and evaluated it on the test set to measure its generalization performance.

### Performance

Using Root Mean Squared Error (RMSE) as our metric, the baseline model achieved a value of **1.2038**.

An RMSE of 1.2 means that, on average, our predictions of cspm deviate from the true values by about 1.2 units. Without additional context, it’s difficult to say if this performance is “good” or “bad.” However, this gives us a starting point. Since cspm values often range roughly between 0 and 10 in professional play, an error of around 1.2 represents a considerable but not major deviation. Considering this is our baseline model with minimal feature engineering and no hyperparameter tuning, there is room for improvement.

## Final Model

### New Features Added

1. `earnedgold` (quantitative): Indicates how much gold a player has earned through individual actions, directly reflecting their farming and objective control abilities. Since higher earned gold often correlates with better overall game performance, it may improve predictions of cspm.

2. `kills` (quantitative): The number of kills a player secures can indirectly affect their opportunity to farm creeps. More kills can yield “map control,” allowing players to spend more time safely farming, potentially increasing cspm.

By adding these parameters, we focused on relevant gameplay metrics that logically impact a player’s ability to farm more effectively.

### Feature Transformations

1. **Categorical Encoding**: We continued using `OneHotEncoder` for the player’s position, turning the nominal categorical variable into appropriate numerical inputs for the model.

2. **Scaling**: All numeric features (totalgold, earnedgold, kills) were standardized using `StandardScaler`. This ensures that features are on a similar scale, preventing any single variable from dominating the model due to its magnitude.

2. **Polynomial Features**: We introduced polynomial features to capture non-linear relationships. For instance, interactions between earnedgold and kills might influence cspm in ways not captured by a purely linear relationship.

### Hyperparameter Tuning and Model Choice

We employed a LinearRegression model and performed hyperparameter tuning using `GridSearchCV` to decide whether to include polynomial features (degree=1, meaning no additional complexity, or degree=2, allowing for squared and interaction terms). This search allowed us to systematically evaluate if non-linear transformations of features improved prediction quality. The best model used a polynomial degree of 2, which indicates that capturing non-linearities improved our predictive performance.

### Performance and Comparison

- Baseline Model RMSE: **1.2038**
- Final Model RMSE: **0.8606**

This model provided a substantial improvement from the baseline, indicating that the addition of the `earnedgold` and `kills` features and the polynomial transformations greatly enhanced our ability to predict `cspm`.

## Fairness Analysis

#### Groups and Justification

For the fairness analysis, we examined whether our final model’s predictive performance (in terms of RMSE) differs between two distinct player groups defined by their in-game role. Specifically, we focused on:

1. **Group X:** Top-lane players (`position == 'top'`)

2. **Group Y:** Mid-lane players (`position == 'mid'`)

Both top and mid players frequently farm lane minions, making them ideal groups to compare. If our model is to be accurate, it is important to ensure that it does not favor one lane over another in its predictions of creep score per minute (`cspm`).

#### Choice of Evaluation Metric

Since we built a regression model to predict `cspm`, we used RMSE (Root Mean Squared Error) as our performance metric. RMSE measures the average prediction error magnitude, providing an intuitive sense of how far off our predictions are from the true values.

#### Hypotheses

1. **Null Hypothesis (H0):** The model is fair. The difference in RMSE between top-lane and mid-lane players is due to random chance. Formally, there is no systematic difference in performance between the two groups.

2. **Alternative Hypothesis (H1):** The model is unfair. The difference in RMSE is not due to chance, indicating that the model performs systematically differently for top-lane players compared to mid-lane players.

#### Test Statistic and Procedure

The statistical proccess we employed during this analysis is as follows:

1. **Observe Difference:** Compute RMSE separately for top-lane and mid-lane subsets of the test data. Find the absolute difference in their RMSEs.

2. **Permutation Test:** Combine both groups, shuffle their “group labels” many times, and recompute the RMSE difference for each shuffled scenario. This simulates the distribution of differences we would expect under the null hypothesis.

3. **Compute p-value:** The calculated p-value from our permutation test was **0.752**.

At a p-value of 0.752, we reject fail to reject the null hypothesis. We do not have sufficient statistical evidence to conclude that our model treats these two groups differently, and the observed difference in performance between top-lane and mid-lane players could easily occur by random chance.

