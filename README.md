### Motivation ###
This project investigates the potential of applying portfolio optimisation techniques to Fantasy Premier League (FPL) team selection. The goal is to assess whether these quantitative methods, commonly used to optimise asset allocation, can improve decision making in FPL by balancing risk and reward when selecting players. The randomness and unpredictability of football present a unique challenge when applying these methods.

This project also aims to demonstrate my understanding of financial concepts while showcasing skills in data cleaning, statistical analysis, and model development.
<br>
<br>
***

### Data Sources & Cleaning ###

The data source for this project is a collection of dataframes from GitHub user "vaastav", who has compiled comprehensive FPL datasets for each season. These datasets contain detailed gameweek statistics for every player, including total points, goals, assists, minutes played, and more. I use data from the 2017/18 season through to 2023/24.
To tailor the data for my analysis, several cleaning and preprocessing steps were required. Key challenges included:
- Inconsistent player names across seasons, e.g. name changes "Heung-Min Son" --> "Son Heung-min" makes it hard to track player perfomance over time
- The dataframes from earlier seasons did not have team names, only opponent team number, this required considerable remapping
<br>
<br>

***

### Feature Engineering ###

I try and extract meaningful predictors that are not immediately obvious in the dataset. This aims to improve prediction accuracy. For example:
- Goals per 90: noramlises goals to per 90 mins player to help identify efficietn goal scorers
- Form trend: measures change in performance, indicates player improvements that may not be priced in
- Minutes momentum: tracks whether a player is increasing or decreasing game time
<br>
<br>

***

### Exploratory Data Analysis ###

Although the end goal is to construct an optimal FPL team, how we will do it is not immediately obvious. Exploratory Data Analysis (EDA) is therefore a crucial step, it allows me to investigate patterns in player performance, understand key features such as point distributions by position, and evaluate predictor variables.

Similar to financial assets, football players exhibit both returns (FPL points) and volatility (variation in weekly points). Understanding this risk/return relationship is essential for constructing a balanced team, just as in portfolio theory, selecting players with high average returns and manageable volatility should improve model performance.

The first plot highlights a positive convex relationship between the standard deviation of weekly FPL points and total points over a season. The correlation coefficient across seasons is around 0.9, indicating that higher total points tend to be associated with greater volatility.

This mirrors the risk/return tradeoff seen in financial markets; achieving higher returns typically requires accepting more risk. However, in finance, the relationship is often concave (i.e. marginal returns diminish with added risk), whereas here we observe convexity (greater volatility appears to amplify total returns).

This is likely due to the scoring patterns of 'high value' players like Salah/Haaland, who may score few points in week 1 but a hatrick in week 2. In contrast, defenders tend to accumulate points more steadily, leading to lower volatility and lower overall upside.
![image](https://github.com/user-attachments/assets/f03d6954-2ee4-4d4e-96da-c22228861353)
<br>
<br>

The plot below shows the scores of players, segmented by position, alongside their respective prices for the 22/23 season. The first  observation is that top performing forwards and midfielders tend to score significantly higher than the average player, particularly when compared to goalkeepers and defenders. This suggests that the higher priced players may justify their premium. However, there is considerable variation in performance at each price point. While more expensive players generally perform better, higher costs do not always guarantee higher scores, this holds true with the exception of the most expensive players.

![image](https://github.com/user-attachments/assets/c5c36aa5-26cc-45c1-affb-a9a1831c3f06)
<br>
<br>

The next plot explores the relationship between a player’s points in one season (21/22) and their points in the following season (22/23). Across all measured seasons, the avg. r^2 for GK's is c0.35, DEF's is c0.2, MID's is c0.35-0.40 and FWD's is c0.3. These mini regressions exlcude players who played less than 1000 minutes in 'season 1'. Without this filter, the r^2 values would be artificially inflated (around 0.5), primarily due to a large number of players who recorded 0 minutes and 0 points in both seasons.



These figures suggest moderate persistence in performancem with midfielders showing the highest year to year consistency. Next, I aim to improve on this baseline by incorporatng additional predictive features. 
![image](https://github.com/user-attachments/assets/7658c285-e997-4f8c-b277-bf4520e1b6f4)
<br>
<br>

***

### Modeling Expected Points ###

The goal is to understand how each input contributes to a player's total output and to estimate expected future performance using interpretable linear models. These models will be trained on historical data, and I will evaluate their fit across different positions to account for role specific dynamics in point scoring.

Before performing any regression analysis, it's important to assess multicollinearity between the predictor variables. High multicollinearity can distort coefficient estimates and reduce model interpretability.

I use Variance Infation Factor (VIF) to measure multicollineairty, VIF quantifies how much a variable's coeffiecint variance is inflated due to multicollinearity with other predictors
The formula is VIF(X) = 1 / (1 - R^2) where R^2 is the R^2 from regressing X on all the other predictors. If VIF for a given variable is high, that variable can be explained by the other variables


As expected, the total points variable exhibits extremely high multicollinearity with other features like goals, assists, and minutes, this makes sense given that total points is largely a function of those underlying stats. For this reason, total points should not be included as an independent variable in any regression. Notably, features such as minutes for GK's and goals scored for FWD's contain high multicollinearity as well, I suspect that this is because a large portion of the difference in points scored between GK's comes from differences in minutes (ditto for FWD's and goals scored). For this reason, the 2nd VIF graph below shows the VIF for each predictor without the total points variable. After this change, we can see the predictor contain moderate but accpetbale multicollinearity. 


![image](https://github.com/user-attachments/assets/6d308fee-4b83-441b-b935-140102056d0e)


![image](https://github.com/user-attachments/assets/32a24c9d-a625-42eb-b810-fe786e641f40)


<br>
<br>

The table below presents the average adjusted R² values of linear regression models across multiple seasons, segmented by player position and based on different combinations of predictor variables. The results clearly indicate that attempting to decompose total points into position specific performance metrics does not lead to more accurate predictions of future points. In every case, the most effective model remains the simplest: a player’s total points in season one is the best predictor of their total points in season two.

Total points already encapsulates the key performance metrics such as goals, assists, minutes played and clean sheets, along with other contributing factors. Attempting to reconstruct it from its components introduces noise without improving predictive power. The challenge is analogous to forecasting stock returns, it is very difficult, and identifying predictive signals often requires proprietary, high-quality data. In the context of FPL, this suggests that simple historical performance may contain more predictive value than trying to model points from isolated metrics.

![image](https://github.com/user-attachments/assets/6bd4d002-710c-4964-9ba6-fc3ac86845ef)
<br>
<br>

***

### Team Optimisation & Strategy ###

## Objective: ##

The goal of the team optimization section is to select an optimal fantasy football squad for the next season. We aim to maximize the expected points from the starting 11 players while incorporating risk (variance in points) for the starting players. The objective function balances the points scored by the starting and bench players, while adjusting the impact of risk based on the **lambda\_risk** parameter.
<br>
<br>

## What Are We Trying to Optimise and Why? ##

We are trying to optimise the composition of the fantasy football squad to achieve the highest possible expected performance in the next season. Specifically, we focus on:

* **Maximizing the expected points** of the starting 11 players and the bench.
* **Minimizing risk** associated with the volatility of points for the starting players.

The optimization takes into account both the points a player is expected to score and the variability (risk) of those points. This ensures that the team is composed of players who can consistently perform well, without overexposing to high risk.
<br>
<br>

## Constraints: ##

The optimisation problem is subject to several constraints to ensure the squad meets the requirements of the FPL format:

1. **Squad Size:** The total number of players in the squad must be 15.
2. **Starting Lineup:** Exactly 11 players must be in the starting lineup.
3. **Position Constraints:**
   * 2 goalkeepers - 1 starting
   * 5 defenders, at least 3 must start
   * 5 midfielders, st least 3 must start
   * 3 forwards, at least 1 must start
4. **Team Constraints:** A maximum of 3 players from any single team can be included in the squad.
5. **Squad Value:** The total value of players in the squad must not exceed £100m.
<br>
<br>

## High-Level Algorithm: ##

This problem is a mixed integer linear program solved using a scipy solver. The decision variables are whether a player is in the squad and whether they are in the starting lineup and the objective function is to maximise the expected points while minimising risk. The constraints ensure that the team composition adheres to the rules of FPL.

## Lambda Experimentation ##

We have previously speculated that FPL team selection diverges from traditional portfolio optimisation, particularly in its treatment of volatility. In financial markets, variance is penalised because it represents drawdown risk. But in FPL, the players with the highest upside often exhibit the greatest volatility. The plot below shows optimal team performance across a range of risk aversion parameters (λ), and the results clearly indicate that the optimal λ is zero. This suggests that the best strategy is simply to maximise expected points, without penalising variance. Interestingly, risk aversion is penalised far more thanr isk seeking behaviour. In hindsight, this is intuitive since there is no concept of 'losses' in FPL, volatility in weekly points may not reflect meaningful “risk.” This highlights an important distinction: while both domains involve uncertainty, the nature of that uncertainty and how it should be penalised must be tailored to the specific decision context.

![image](https://github.com/user-attachments/assets/69673419-7355-4eb8-8601-b6e97811a8db)

<br>
<br>


Below is the optimal team choice for the 23/24 season based on 22/23 data. The algorithm clearly prioritised high-scoring forwards, particularly expensive, star talent like Haaland, which suggests a bias toward raw point totals rather than positional value. Most players underperformed in the following season, which is expected due to natural mean reversion the model selects last season’s top performers, many of whom regress. Interestingly, the optimal squad often includes multiple players from the same club (e.g. Brentford, Arsenal), which could indicate a lack of diversification. While this might be coincidental, it also reflects the model’s tendency to stack players from strong-performing teams without accounting for correlated risk.

![image](https://github.com/user-attachments/assets/b1f94429-82a1-4293-818e-80e938ab4bd2)
<br>
<br>
***


### Baseline Teams (for Comparison) ###

To fairly benchmark my optimal team, I need a baseline to compare against. I simulate this by generating thousands of random teams using a simple greedy/random logic that mimics how a casual player might build their squad. The idea is: a typical user sees a list of players, grabs one or two expensive stars, then realises they’re running out of budget and fills the rest with mid-range or cheap options, often resulting in a few strong picks and a bunch of weak fillers. The algorithm follows this pattern by randomly choosing quotas of expensive, medium, and cheap players, then filling a squad under budget while satisfying basic position and club constraints. This gives me a realistic distribution of “naive” team performances to compare against helping to show whether my optimised solution actually adds value over random or common-sense strategies.
***

### Results ###
Results show that generally my optimal team outperforms the average 'baseline team'.

![image](https://github.com/user-attachments/assets/71573da9-7523-4b47-a607-bd5efad8fa47)
<br>
<br>

***

### Limitations ###
Single Gameweek Perspective: Teams are optimised as if picked for Gameweek 1 and held for the full season. This ignores transfers, injuries, captain choices, and chip usage, all of which affect real FPL outcomes.

No Predictive Model: We do not forecast future points. Instead, we assume a player's points is their previous season’s points. While crude, this is a practical baseline in the absence of richer features.

No Train/Test Split: There is no explicit model training or validation — each season is optimised in isolation using historical data. Since the optimiser is reused generically each time, introducing complex validation would risk curve fitting.

Limited Seasons: Only 6 seasons (2017/18 to 2022/23) are used. This may limit generalisability, especially across rule changes or shifting meta strategies.

Starters + Substitutes: Substitute players are given a flat 10% contribution to the objective function. No rotation or fixture based logic is applied.

Simplified Constraints: While position and team constraints are respected, other FPL specific nuances (e.g. fixture difficulty, double gameweeks, price changes) are not considered.
***

Conclusion & Next Steps
