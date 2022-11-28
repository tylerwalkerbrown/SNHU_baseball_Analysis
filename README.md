# SNHU_baseball_Analysis![output_35_1](https://user-images.githubusercontent.com/94020684/201807206-fcd326a1-fe44-4e5b-a33b-9ce178378548.png)

# Box Plot for Pitcher Statistics
Looking at our data for pitchers over the past four full seasons you notice games with some extreme outliers. The most consistent statistic for pitchers is the amount of time they see per game which is expected in baseball because every game is limited to 9 innings. You can see there was a game that went into the 14th inning which is very rare in the world of baseball. But you can see the complete overall domination by SNHU baseball over the past 4 years with the combination of low walk averages (lower than 1 per game) and very high strikout average (around 8 per game). 
![output_27_1](https://user-images.githubusercontent.com/94020684/203548188-d707d057-e741-4ec2-b9c0-76ba8724ec60.png)
![output_28_1](https://user-images.githubusercontent.com/94020684/203548191-37c79d14-4fd8-46e2-8ddc-f7abe7457d5b.png)
![output_29_1](https://user-images.githubusercontent.com/94020684/203548193-b6b91797-a27e-4d50-b692-3d15f494b454.png)
![output_30_1](https://user-images.githubusercontent.com/94020684/203548194-6c0da5ba-4966-4c1e-bd45-c95c08f30133.png)
![output_31_1](https://user-images.githubusercontent.com/94020684/203548195-33eb126b-1cf7-42a1-bdfe-a715cb2a96df.png)
![output_32_1](https://user-images.githubusercontent.com/94020684/203548198-7b044b6a-5e68-4c09-89b7-95e9f192d0d4.png)
![output_35_0](https://user-images.githubusercontent.com/94020684/203548200-cb9e7b3e-dbf1-408d-bc6f-4a17bdd51f28.png)
## Correlated Variables
The correlation matrix took in every pitching and hitting statistic that SNHU website has to offer. After reviewing the R^2 values the ones that returned the highest correlations were:
- '3B''HR', 'BB', 'SB','SF','PO','E','oBB','o2B','oHR','oH','oR', 'ER','ERA', 'opps', 'penman', 'AVG', 'AB', 'R','H', 'RBI'
- penman (Penman runs scored)
- opps (Opponent runs scored)
- R squared variables with > .70:
   - penman, AVG, AB, R, H, RBI, ERA, opps, ER, oR (opponent runs), oH (Opponent hits)
These statistics are very strong indicators of the teams success. 
![output_38_0](https://user-images.githubusercontent.com/94020684/203548204-fc84d9ab-578c-41f6-b4c4-c881b0c2825f.png)
# Distributions
Looking at the different distributions of the statistics that were captured in the analysis you can notice the majority have a normal distribution. Some contained exponential distributions such as:
- SF, HR, 3B, SB, E, ER, ERA, oHR
Base on balls for the SNHU followed closely to a gamma distribution (skewing to the right side of the distribution). The central tenancy was close to 8 walks per game which is very high in the game of baseball (based off professional level its about double).

![output_39_1](https://user-images.githubusercontent.com/94020684/203548206-063c9ea2-026f-43c9-b56a-7c225691aee8.png)
![output_42_0](https://user-images.githubusercontent.com/94020684/203548210-b5cca937-9dd1-4d61-9154-3b21fda545e8.png)
- Runs, RBI, Average are the most correlated variables to winning for SNHU Penman Baseball.

![output_44_0](https://user-images.githubusercontent.com/94020684/203548211-54e05706-c530-4cc2-97ba-37e3a2bb11ec.png)
- Runs, ERA, ER are the most correlated variables to winning for SNHU Penman Baseball.
# Home Field Advantage ?
![output_47_0](https://user-images.githubusercontent.com/94020684/203548212-ad95c0fb-130c-4ebd-ab4e-fdfebeb71b84.png)

Looking at the average era of each of the different locations. This holds the common understanding of home field advantage showing a lower era while the Penman play at home. Neutral settings are actually holding the lowest era out of the three options. The reason that neutral is an option is becuase in college baseball you have times that fields may flood, snow (turf complex) and then playoffs are played at neutral settings if the host team is kicked out of the tournament.
![output_49_0](https://user-images.githubusercontent.com/94020684/203548214-ec8459f0-d9c1-4593-ab25-ee3b287bb684.png)
Home and away does not sem to have a huge impact for SNHU hitters. Neutral settings are higher on average compared to both home and away.
![output_97_0](https://user-images.githubusercontent.com/94020684/203548215-9d405db0-2bd0-4cb5-92b2-ad3818584399.png)
![output_104_0](https://user-images.githubusercontent.com/94020684/203548217-bd52b430-7a24-4da3-a465-64ae9a195959.png)
![output_109_0](https://user-images.githubusercontent.com/94020684/203548220-1a78a306-819d-4a68-a190-64dd26862f17.png)
