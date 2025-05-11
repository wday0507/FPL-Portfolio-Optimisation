***
# Motivation #
This project investigates the potential of applying portfolio optimisation techniques to Fantasy Premier League (FPL) team selection. The goal is to assess whether these quantitative methods, commonly used to optimise asset allocation, can improve decision making in FPL by balancing risk and reward when selecting players. The randomness and unpredictability of football present a unique challenge when applying these methods.

This project also aims to demonstrate my understanding of financial concepts while showcasing skills in data cleaning, statistical analysis, and model development.
<br>
<br>
***

# Data Sources & Cleaning #

The data source for this project is a collection of dataframes from GitHub user "vaastav", who has compiled comprehensive FPL datasets for each season. These datasets contain detailed gameweek statistics for every player, including total points, goals, assists, minutes played, and more. The dataset from each season contains c500 players for 38 gameweeks (c.20,000 rows total). I use data from the 2017/18 season through to 2023/24.
To tailor the data for my analysis, several cleaning and preprocessing steps were required. Key challenges included:
- Inconsistent player names across seasons, e.g. name changes "Heung-Min Son" --> "Son Heung-min" makes it hard to track player perfomance over time
- The dataframes from earlier seasons did not have team names, only opponent team number, this required considerable remapping
<br>
<br>

***

# Feature Engineering #

I try and extract meaningful predictors that are not immediately obvious in the dataset. This aims to improve prediction accuracy. For example:
- Goals per 90: normalises goals to per 90 mins player to help identify efficient goal scorers
- Form trend: measures change in performance, indicates player improvements that may not be priced in
- Minutes momentum: tracks whether a player is increasing or decreasing game time
<br>
<br>

***

# Exploratory Data Analysis #

Although the end goal is to construct an optimal FPL team, how we will do it is not immediately obvious. Exploratory Data Analysis (EDA) is therefore a crucial step, it allows me to investigate patterns in player performance, understand key features such as point distributions by position, and evaluate predictor variables.

Similar to financial assets, football players exhibit both returns (FPL points) and volatility (variation in weekly points). Understanding this risk/return relationship is essential for constructing a balanced team, just as in portfolio theory, selecting players with high average returns and manageable volatility should improve model performance.

The first plot highlights a positive convex relationship between the standard deviation of weekly FPL points and total points over a season. The correlation coefficient across seasons is around 0.9, indicating that higher total points tend to be associated with greater volatility.

This mirrors the risk/return tradeoff seen in financial markets; achieving higher returns typically requires accepting more risk. However, in finance, the relationship is often concave (i.e. marginal returns diminish with added risk), whereas here we observe convexity (greater volatility appears to amplify total returns).

This is likely due to the scoring patterns of 'high value' players like Salah/Haaland, who may score 0 points in week 1 but a hatrick in week 2. In contrast, defenders tend to accumulate points more steadily, leading to lower volatility and lower overall upside.
![image](https://github.com/user-attachments/assets/f03d6954-2ee4-4d4e-96da-c22228861353)
<br>
<br>

The plot below shows player scores by position and price for the 22/23 season. Top performing forwards and midfielders tend to score well above the average, especially compared to goalkeepers and defenders, suggesting that premium players can justify their cost. However, performance varies considerably at each price point. While expensive players generally perform better, higher prices do not always mean higher scores, except for the very top-priced players.


![image](https://github.com/user-attachments/assets/c5c36aa5-26cc-45c1-affb-a9a1831c3f06)
<br>
<br>

The next plot explores the relationship between a player’s points in one season (21/22) and their points in the following season (22/23). Across all measured seasons, the avg. r^2 for GK's is c0.35, DEF's is c0.2, MID's is c0.35-0.40 and FWD's is c0.3. These mini regressions exlcude players who played less than 1000 minutes in 'season 1'. Without this filter, the r^2 values would be artificially inflated (around 0.5), primarily due to a large number of players who recorded 0 minutes and 0 points in both seasons.



These figures suggest moderate persistence in performance with midfielders showing the highest year to year consistency. Next, I aim to improve on this baseline by incorporatng additional predictive features. 
![image](https://github.com/user-attachments/assets/7658c285-e997-4f8c-b277-bf4520e1b6f4)
<br>
<br>

***

# Modeling Expected Points #

To build the best possible FPL team, we ideally want to predict next season’s points as accurately as possible. While using last season’s points is a reasonable baseline, this section explores whether we can improve on that with a more refined approach.
<br>
<br>


### Dealing with Multicollinearity ### 

Before running any regression, it's important to check for multicollinearity between predictors, as it can distort coefficient estimates and reduce interpretability.

I use Variance Infation Factor (VIF), which measures how much a variable's variance is inflated due to correlation with other predictors. The formula is VIF(X) = 1 / (1 - R^2) where R^2 is the R^2 from regressing X on all the other predictors. 

As expected, the total points variable shows extremely high multicollinearity with features like goals, assists, and minutes which is unsurprising given that total points is largely derived from these stats. For this reason, total points is excluded from my models.

After removing total points, we still observe notable multicollinearity between clean sheets and minutes played for goalkeepers. Since clean sheets are more volatile than minutes, I choose to exclude them. With this adjustment, all remaining VIF values fall within an acceptable range (<5). This process helps guide model selection by removing redundant predictors.


![image](https://github.com/user-attachments/assets/91caae58-4b42-47f5-95a6-4031f32d0a64)
<br>
<br>
![image](https://github.com/user-attachments/assets/be5d4ffe-2734-4cf2-b482-b259d730057e)
<br>
<br>
![image](https://github.com/user-attachments/assets/10265e44-2cf0-4905-a7d2-a61a60c4109a)




<br>
<br>

### Linear Regression ###

I run a range of linear regression models for each position, testing different combinations of predictor variables while managing multicollinearity. A full breakdown of feature sets is available in the Jupyter notebook. The chart below compares the performance of some of these models alongside a simple baseline for each position.

The results show that the naive model, which uses total points from the previous season as the sole predictor, consistently outperforms all other models in predicting total points for the following season.

Total points already embed key performance indicators like goals, assists, minutes played, and clean sheets. Trying to reconstruct it from its components introduces noise and degrades predictive performance.

The challenge here is similar to forecasting stock returns, signals are weak, and useful predictive power is hard to extract without high quality data. In the context of FPL, previous total points carry more information than breaking them into individual stats or engineering additional features. Even models using trends in form or minutes fail to outperform the baseline out of sample.

![image](https://github.com/user-attachments/assets/41053617-37df-419d-adb1-156e72a3bacc)


<br>
<br>
While more advanced models (e.g., regularisation or ensemble methods) were considered, they rely on the same weakly predictive features. Without additional information or richer data sources, there is limited scope for meaningful improvement beyond the naive baseline.

***

# Team Optimisation #

### Objective: ###

The goal of the team optimisation section is to select an optimal fantasy football squad for the next season. We aim to maximize the expected points from the starting 11 players while incorporating risk (variance in points) for the starting players. The objective function balances the points scored by the starting and bench players, while adjusting the impact of risk based on the lambda parameter.

<br>
<br>

### Constraints: ###

The optimisation problem is subject to several constraints to ensure the squad meets the requirements of the FPL format:

1. **Squad Size:** The total number of players in the squad must be 15.
2. **Starting Lineup:** Exactly 11 players must be in the starting lineup.
3. **Position Constraints:**
   * 2 goalkeepers - 1 starting
   * 5 defenders, at least 3 must start
   * 5 midfielders, at least 3 must start
   * 3 forwards, at least 1 must start
4. **Team Constraints:** A maximum of 3 players from any single team can be included in the squad.
5. **Squad Value:** The total value of players in the squad must not exceed £100m.
<br>
<br>

### High Level Algorithm: ###

This problem is a mixed integer linear program solved using a scipy solver. The decision variables are whether a player is in the squad and whether they are in the starting lineup and the objective function is to maximise the expected points while minimising risk. The constraints ensure that the team composition adheres to the rules of FPL.
<br>
<br>

### Lambda Experimentation ###

We have previously speculated that FPL team selection diverges from traditional portfolio optimisation, particularly in its treatment of volatility. In financial markets, variance is penalised because it represents drawdown risk. But in FPL, the players with the highest upside often exhibit the greatest volatility. The plot below shows optimal team performance across a range of risk aversion parameters (λ), and the results clearly indicate that the optimal λ is zero. This suggests that the best strategy is simply to maximise expected points, without penalising variance. Interestingly, risk aversion is penalised far more than risk seeking behaviour. In hindsight, this is intuitive since there is no concept of 'losses' in FPL, volatility in weekly points may not reflect meaningful “risk.” This highlights an important distinction: while both domains involve uncertainty, the nature of that uncertainty and how it should be penalised must be tailored to the specific decision context.

![image](https://github.com/user-attachments/assets/69673419-7355-4eb8-8601-b6e97811a8db)

<br>
<br>


Below is the optimal team choice for the 23/24 season based on 22/23 data. The algorithm clearly prioritised high scoring forwards, particularly expensive players like Haaland, which suggests a bias toward raw point totals rather than positional value. Most players underperformed in the following season, which is expected due to natural mean reversion. Interestingly, the optimal squad often includes multiple players from the same club (e.g. Brentford, Arsenal), which could indicate a lack of diversification. While this might be coincidental, it also reflects the model’s tendency to stack players from strong performing teams without accounting for correlated risk.

![image](https://github.com/user-attachments/assets/b1f94429-82a1-4293-818e-80e938ab4bd2)
<br>
<br>

***

### Baseline Teams (for Comparison) ###

To fairly benchmark my optimal team, I need a baseline to compare against. I simulate this by generating thousands of random teams using a simple greedy/random logic that mimics how a casual player might build their squad. The idea is: a typical user sees a list of players, grabs one or two expensive stars, then realises they’re running out of budget and fills the rest with mid range or cheap options, often resulting in a few strong picks and a bunch of weak fillers. The algorithm follows this pattern by randomly choosing quotas of expensive, medium, and cheap players, then filling a squad under budget while satisfying basic position and club constraints. This gives me a realistic distribution of “naive” team performances to compare against helping to show whether my optimised solution actually adds value over random or common sense strategies.
<br>
<br>

***

# Results #
These results demonstrate that my optimal team consistently outperforms the baseline teams across all seasons, with gains ranging from 30 to over 400 points. Importantly, each optimal team is constructed using only data from the preceding season. This avoids any hindsight bias and suggests that the model captures information that generalises well to future performance. Whilst the model is an improvement compared to the baseline teams, the variation in performance is significant and highlights the inherent randomness in football.

![image](https://github.com/user-attachments/assets/5cf2dbb9-3579-427d-bdf9-2f30341c5388)

<br>
<br>

***

# Limitations #
Single Gameweek Perspective: Teams are optimised as if selected in Gameweek 1 and held unchanged for the entire season. This excludes transfers, captain choices, chip usage, and injuries, key elements of real FPL strategy. This limits the realism and practical applicability of the results.

No Predictive Model: We do not forecast future points. Instead, we assume a player's points is their previous season’s points. While crude, this is a practical baseline in the absence of richer features.

No Train/Test Split: There is no explicit model training or validation, each season is optimised in isolation using historical data. Since the optimiser is reused generically each time, introducing complex validation would risk curve fitting.

Substitutes: Substitute players are given a flat 10% contribution to the objective function. No rotation or fixture based logic is applied.

No significance testing: Formal significance testing across seasons is not performed due to the small sample size (only 6 seasons), which makes statistical inference unreliable. Additionally, the distribution of team scores varies significantly between seasons, violating key assumptions of standard hypothesis tests. 
<br>
<br>


***

# Conclusion & Next Steps #

This project demonstrates that whilst portfolio optimisation principles cannot be directly applied to FPL team construction, a basic optimisation framework can still add value to team selection process. The optimal team, built using prior season points data, consistently outperforms simulated teams, though the margin of outperformance varies considerably. 

Next steps:

1/ Find FPL 'Alpha': explore richer player/team level features that may predict future performance better than past total points (e.g. player role changes, match load, betting odds derived signals). 

2/ Simulate performance: develop a more realistic season long simulation that reflects how the team would perform in the upcoming season, rather than just evaluating past points. This would enable more accurate comparison to baseline teams, and real world teams.
