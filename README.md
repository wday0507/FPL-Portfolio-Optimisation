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


The next plot shows the relationship between a players point this season, and next season. Across all seasons, the average r^2 is around 0.5 for goalkeepers, midfielders and forwards, whereas it as c0.3 for defenders. 

It wasnt obvious to me that the hardest position to predict next seasons points would be defenders, if i ahd to guess i would say its because 

Modeling Expected Points


Team Optimisation & Strategy


Baseline Teams (for Comparison)


Results


Next Steps
