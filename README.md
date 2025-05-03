Motivation

This project serves as an application of quantitative methods, specifically portoflio optimisation, to the realm of Fantasy Premier League (FPL). Exploring player selection through the lens of optimisation, this project not only demonstrates my undertanding of financial concepts but also integrates skills such as data cleaning, statistical analysis, model delveopment and evaluation.
Additionally, the inherent randomess of football provides a unqiue challenge in applying these methods. As a football enthuasist, this project combines my professional interests with my passion for the psort, making it an engaging exercise. 


Data Sources & Cleaning

The data source for this project is a collection of dataframes from GitHub user "vaastav", who has compiled comprehensive FPL datasets for each season. These datasets contain detailed gameweek statistics for every player, including total points, goals, assists, minutes played, and more. I use data from the 2017/18 season through to 2023/24.
To tailor the data for my analysis, several cleaning and preprocessing steps were required. Key challenges included:
- Inconsistent player names across seasons, e.g. name changes "Heung-Min Son" --> "Son Heung-min" makes it hard to track player perfomance over time
- The dataframes from earlier seasons did not have team names, only opponent team number, this required considerable remapping

Exploratory Data Analysis

Although the end goal is to construct an optimal FPL team, the path to doing so isn’t immediately obvious. Exploratory Data Analysis (EDA) is therefore a crucial step, it allows me to investigate patterns in player performance, understand key features such as point distributions by position, and evaluate predictor variables. This stage informs model development and helps guide how to approach the optimisation process.

Similar to financial assets, football players exhibit both returns (FPL points) and volatility (variation in weekly points). Understanding this risk/return relationship is essential for constructing a balanced team, just as in portfolio theory, selecting players with high average returns and manageable volatility should improve model performance.

The first plot highlights a positive convex relationship between the standard deviation of weekly FPL points and total points over a season. The correlation coefficient across seasons is around 0.9, indicating that higher total points tend to be associated with greater volatility.

This mirrors the risk/return tradeoff seen in financial markets; achieving higher returns typically requires accepting more risk. However, in finance, the relationship is often concave (i.e. marginal returns diminish with added risk), whereas here we observe convexity (greater volatility appears to amplify total returns).

This is likely due to the scoring patterns of 'high value' players like Erling Haaland, who may score few points in week 1 but a hatrick in week 2. In contrast, defenders tend to accumulate points more steadily, leading to lower volatility and lower overall upside.
![image](https://github.com/user-attachments/assets/f03d6954-2ee4-4d4e-96da-c22228861353)


The plot below shows the scores of players, segmented by position, alongside their respective prices. The first  observation is that top performing forwards and midfielders tend to score significantly higher than the average player, particularly when compared to goalkeepers and defenders. This suggests that the higher priced players may justify their premium. However, there is considerable variation in performance at each price point. While more expensive players generally perform better, higher costs do not always guarantee higher scores, this holds true with the exception of the most expensive players.

![image](https://github.com/user-attachments/assets/523b9f80-40a4-4573-a41f-d718ca448613)


The next plot explores the relationship between a player’s points in one season and their points in the following season. On average, the r^2 value is around 0.5 for goalkeepers, midfielders, and forwards, and approximately 0.3 for defenders.


*** this has changes now as i nonly include players with season minutes > 1000 ***
It wasn’t immediately obvious to me that defenders would be the hardest position to predict. If I had to guess, I’d attribute it to the higher volatility in their scoring often dependent on clean sheets, which are more team dependent and less consistent than goal contributions.

An r^2 of 0.5 already indicates a fairly strong relationship, suggesting that current season performance is a decent predictor of next season’s output. Next, I’ll explore whether there are any position-specific patterns or relationships that can help refine this further.

![image](https://github.com/user-attachments/assets/90fbb930-9e5f-4cda-8bb6-70a011588d22)


Modeling Expected Points

The next step involves modelling a player’s expected FPL points. The goal is to understand how each input contributes to a player's total output and to estimate expected future performance using interpretable linear models. These models will be trained on historical data, and I will evaluate their fit across different positions to account for role specific dynamics in point scoring.

Before performing any regression analysis, it's important to assess multicollinearity between the predictor variables. High multicollinearity can distort coefficient estimates and reduce model interpretability.

I use Varianec Infation Facotor (VIF) to measure multicollineairty, VIF quantifies how much a variable's coeffiecint variance is inflated due to multicollinearity with other predictors
The formula is VIF(X) = 1 / (1 - R^2) where R^2 is the R^2 from regressing X on all the other predictors. If VIF for a given variable is high, that variable can be explained by the other variables


As expected, the total points variable exhibits extremely high multicollinearity with other features like goals, assists, and minutes, this makes sense given that total points is largely a function of those underlying stats. For this reason, total points should not be included as an independent variable in any regression. Other variables such as goals, assists, and minutes display moderate multicollinearity, which is acceptable for regression purposes.

**output here**


The table below presents the average adjusted R² values of linear regression models across multiple seasons, segmented by player position and based on different combinations of predictor variables. The results clearly indicate that attempting to decompose total points into position-specific performance metrics does not lead to more accurate predictions of future points. In every case, the most effective model remains the simplest: a player’s total points in season one is the best predictor of their total points in season two.

Total points already encapsulates the key performance metrics such as goals, assists, minutes played and clean sheets, along with other contributing factors. Attempting to reconstruct it from its components introduces noise without improving predictive power. The challenge is analogous to forecasting stock returns, it is very difficult, and identifying predictive signals often requires proprietary, high-quality data. In the context of FPL, this suggests that simple historical performance may contain more predictive value than trying to model points from isolated metrics.

![image](https://github.com/user-attachments/assets/6bd4d002-710c-4964-9ba6-fc3ac86845ef)


Team Optimisation & Strategy

### Objective:

The goal of the team optimization section is to select an optimal fantasy football squad for the next season. We aim to maximize the expected points from the starting 11 players while incorporating risk (variance in points) for the starting players. The objective function balances the points scored by the starting and bench players, while adjusting the impact of risk based on the **lambda\_risk** parameter.

### What Are We Trying to Optimize and Why?

We are trying to optimize the composition of the fantasy football squad to achieve the highest possible expected performance in the next season. Specifically, we focus on:

* **Maximizing the expected points** of the starting 11 players and the bench.
* **Minimizing risk** associated with the volatility of points for the starting players.

The optimization takes into account both the points a player is expected to score and the variability (risk) of those points. This ensures that the team is composed of players who can consistently perform well, without overexposing to high risk.

### Constraints:

The optimization problem is subject to several constraints to ensure the squad is realistic and meets the requirements of the fantasy football format:

1. **Squad Size:** The total number of players in the squad must be 15.
2. **Starting Lineup:** Exactly 11 players must be in the starting lineup.
3. **Position Constraints:**

   * 2 goalkeepers (GK)
   * 5 defenders (DEF)
   * 5 midfielders (MID)
   * 3 forwards (FWD)
4. **Starting Lineup Formation:**
   * 1 goalkeeper must be in the starting lineup.
   * At least 3 defenders, 2 midfielders, and 1 forward must be in the starting lineup.
   * Exactly 10 outfield players (DEF, MID, FWD) must be in the starting lineup.
5. **Team Limitations:** A maximum of 3 players from any single team can be included in the squad.
6. **Player Value:** The total value of players in the squad must not exceed a specified budget (e.g., the sum of their next season’s initial value should be ≤ 100).

High-Level Algorithm:

This problem is solved using **convex optimization**, where we define the optimization variables (whether a player is in the squad and whether they are in the starting lineup) and set up an objective function to maximize the expected points while minimizing risk. The **constraints** ensure that the team composition adheres to the rules of the fantasy football league.

The problem is formulated as a **linear program** and solved using the **SCIPY solver** with the highs method for optimization, allowing us to find the optimal squad composition.

Below is the optimal team choice for the 23/24 season based on 22/23 data. The algorithm clearly prioritised high-scoring forwards, particularly expensive, star talent like Haaland, which suggests a bias toward raw point totals rather than positional value. Most players underperformed in the following season, which is expected due to natural mean reversion — the model selects last season’s top performers, many of whom regress. Interestingly, the optimal squad often includes multiple players from the same club (e.g. Brentford, Arsenal), which could indicate a lack of diversification. While this might be coincidental, it also reflects the model’s tendency to stack players from strong-performing teams without accounting for correlated risk.

![image](https://github.com/user-attachments/assets/b1f94429-82a1-4293-818e-80e938ab4bd2)


Baseline Teams (for Comparison)

To fairly benchmark my optimal team, I need a baseline to compare against. I simulate this by generating thousands of random teams using a simple greedy/random logic that mimics how a casual player might build their squad. The idea is: a typical user sees a list of players, grabs one or two expensive stars, then realises they’re running out of budget and fills the rest with mid-range or cheap options, often resulting in a few strong picks and a bunch of weak fillers. The algorithm follows this pattern by randomly choosing quotas of expensive, medium, and cheap players, then filling a squad under budget while satisfying basic position and club constraints. This gives me a realistic distribution of “naive” team performances to compare against helping to show whether my optimised solution actually adds value over random or common-sense strategies.


Results
Results show that generally my optimal team outperforms the average 'baseline team'.

![image](https://github.com/user-attachments/assets/71573da9-7523-4b47-a607-bd5efad8fa47)

Next Steps
