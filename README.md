# Introduction
The project is to provide insights on the Integrated Single Electricity Market (I-SEM) based on Day-Ahead Market (DAM). The relevant variables are provided in an Excel Spreadsheet, which consists of commodity prices, Wind Index, and few other key variables. 

Hence, the objective of this project is:

 1. Findings on the variables that drive the DAM Price
 2. Developing trading strategies to try and profits across the different electricity prices

# Datasets
A total of 6 csv files are provided which consist of DAM Price, Relative Commodity Price, Generator Price / Availability and Wind Demand and Output. The data has been extracted from the trading platform and has been cleaned and index set into similar timeframe. 

## Feature Constructing using Principal Component Analysis
Generator price and availability tend to have 18 variables each. If all the datasets are to be merged, the total number of independent variables will be more than 40. 
Hence, to resolve this issue, the price and availability data for the generator are merged, creating one dataset. Applying Principal Component Analysis will reduce the dimension of the dataset accordingly depending on the variance value tagged with the number of indexes.

The total variance for N = 5 is equal to 83.67%, which is sufficient enough for this project. 

## Train and Test Dataset

The data will be split between the Train/Test with the following manner:

Train : 2018 - 2020

Test : 2021

Snapshot of the complete dataset is as follows
![data](https://i.ibb.co/gWK2bDw/image13.png)
The first column 'DAM Price' will be the targeted variable and the remaining will be the independent variables. 

# Variability Importance Analysis

This chapter is to study the importance of the independent variables towards affecting the DAM Price. Recently with the price of DAM Price is surging to a yearly high. It is an excellent opportunity to study the market landscape and differentiate the changes that are currently happening on other commodity indexes. 


## Correlation Analysis

Firstly, a correlation matrix is plotted to study the relationship between the targeted variable against other variables

![correlation matrix](https://i.ibb.co/8z6JJLx/image12.png)

From the correlation matrix, a strong correlation can be observed between DAM Price against GAS Price, Continuous Wind Forecast(CWF), Actual Wind Output (AWO) and  Actual Net Demand Output (ANDO). However, most of the wind index has strong correlation between each other (see CWF and AWO).

Based from the result, a further study/method are to be conducted using the following method:

1. Regression Method
2. Multilayer Perceptron Regressor (MLPR)

## Regression Method

![RM](https://i.ibb.co/2cvLxZ1/image14.png)

Based from the result of regression analysis

R^2 = 68% <br>

Based from the R-Squared score, the model tend to explained 68% of variance for 2021 test data. 

**Variable Importance Analysis Result**<br>

The plot showed that CWF, AWO and Gas price tend has the highest coefficient values. CWF is negatively correlated to DAM Price and GAS Price and AWO is positively correlated to DAM Price for this Regression model.  However, since AWO and CWF are strongly correlated, further analysis on the relationship on DAM Price against Continuos Wind Forecast and Gas Price only. 

## Multi Layer Perceptron Regressor

Next, the data is pass into Multi Layer Perceptron Regressor. This model manage to improve the R^2 score by 3% to 71%.
However, machine learning method doesnt provide much information on the variable importance. (Black Box Problem). The model will be used to predict DAM Price during the implementation of trading strategy later in this project. 

## Result Visualisation

For this part, All the result from the analysis will be visualised. The focus will be on GAS Price and CWF against DAM Price. 

![graf1](https://i.ibb.co/ys4wGty/image15.png)

From the plot above, the graph proved that the movement of monthly average price of DAM is against the direction of Continuous Wind Forecast. The trend surfaced more clearly during end of 2019 to early 2020 and second quarter of 2021. 

![graf2](https://i.ibb.co/THJ6bVv/image16.png)

The plot showcased a strong positive correlation between the average monthly price of DAM and Average Monthly Price of Gas. 

Based from these 2 plots, it can be conclude that the movement of DAM Price is highly affected by these 2 variables. Hence, The surge on DAM Price in 2021 may due to the combination of Increased Gas Price and Low Wind Output

# Development of Trading Strategy and Algorithm

In this section, it will mainly focus on the development of trading algorithm based on the information gathered from the previous analysis. The trading strategies will be used on the trading data, which is on half-hour row inputs. The trading data consist of DAM Price, IDA1, IDA2, IDA3, and BM Price. 

A similar data-wrangling method will be applied to the trading data. However, the gas price, which is in daily frequency, requires some adjustment to fit the half-hour rows. Thus, the forward fill method will be applied after spreading the DateTime index. The dataset can be observed by running code line 130. 

![td](https://i.ibb.co/QDyJHZZ/image17.png)

## Trade Strategy 1

For the first strategy, the data showed that DAM Price is higher than BM for avery half hour timeline by 58.42%.

Hence, a simple trading rule is applied on the trading data which is to sell all DAM Price for every half hour from 2019 to 2021

![profitgraf](https://i.ibb.co/4JXpNGp/image18.png)

Based on the result from the chart which plots the value of cumulative profit. The profit gain for 3 years is at EUR34,258. The trading frequency is 100% for all the 3 years. Some losses are recorded during 2020, but it stabilised back in 2021. 

## Trade Strategy 2

Trading strategy 2 will trade base on the value of CWF and Gas Price. 
The strategy is to sell the t+1 DAM Price and buy back BM Price if the value of t index satisfy the following rule

1. value of Cont WF is more than 1,000
2. Value of Gas price is less than 35

![profitgraf2](https://i.ibb.co/6ZkXxy9/image19.png)

The Above is the example of Strat 2 Profits on 3 half-hour periods. Notice that for the first row, the Cont WF is above 1,000 and Gas price is below 35, thus, the algo will short DAM Price for the next period (Sold at 48.505) and Buy back at 35.07 gaining 13.43 Euros.
Same goes for the next period where 14.17 Euros was gained.

the total profit gain in 3 years for this strategy is EUR50,670 

![profitgraf3](https://i.ibb.co/mbNpjLJ/image20.png)

Based from the plot, Trade strategy 2 performs extremely well in 2019 and moderately in 2020. However in 2021, the strategy didnt trade any period as the Gas Price did not satisfy the threshold 
which was set in the algorithm. The gas price for 2021 has been above 35 until June 2021. Thus, it has 0% period traded in 2021. 

## Trade Strategy 3 (using Prediction values)

Trading strategy 3 will trade base on the value of Cont WF, Gas Price and using the value of Pred DAM which is value simulate using MLP Regressor.
Sample of Pred DAM is seen below

The strategy is to sell the IDA1 Price and buy back BM Price at index t, if the value of t index satisfy the following rule

1. Value of Continuous Wind Forecast more than 700
2. Value of Gas price is less than 55
3. DAM Price > DAM Pred for that day

In the contrary, situation 1,2,3 is reversed then <br>

1. Value of Continuous Wind Forecast less than 700
2. Value of Gas price is more than 55
3. DAM Price < DAM Pred for that day

Buy the IDA1 Price and sell back BM Price at index t <br>

![pg4](https://i.ibb.co/W3bj7Lv/image21.png)

To simulate an example lets take value from above table. On 19th April 2019, DAM Price > DAM Pred.

![gp4](https://i.ibb.co/S00sznY/image22.png)

Next, observing values from the above rows: <br>

Cont WF > 700 and GAS Price < 55. Thus the decision is to sell IDA1 Price and Buy BM Price.
as all these 3 conditions are satisfied for all the 3 rows at hour 6:30, 7:00 and 7:30. the profits recorded are EUR7.464, EUR5.167 and EUR 13.13 respectively. 

![gp5](https://i.ibb.co/vd7KrF5/image23.png)

The plot showcased the cumulative profit gained from Trading Strategy 3 which sum up to a total of EUR 67,539 in the span of 3 years. The strategy has a trading period of 30% which has win ratio of above 55%. 

Thus, based on all the 3 trading strategy, this strategy tend to provide the best result with reliable consistency on the trades. 

# Final Result and Evaluation

![gp](https://i.ibb.co/ygDXgDf/image24.png)

![tableprofit](https://i.ibb.co/jJ9XscS/TablePNL.png)

Based on the plot, The total profits made from 2019 to 2021 for the 3 strategies are as follows: <br>

**Strategy 1** : EUR 34,258 <br>

**Strategy 2** : EUR 50,670<br>

**Strategy 3** : EUR 67,539<br>

There are also other key findings, such as : <br>

- For 2019, All the strategy tend to provide good results, However, the market landscape change drastically for 2020 as all strategies recorded lost Year on Year. 
- Strategy 1 has traded all the period, however, for 2020, the landscape has reversed which caused the 12k loss.
- Strategy 2 didn’t execute any trade for 2021, this is due to the value of gas price is above the threshold set in the algorithm.
- Strategy 3 only trade on average 30% of total period, however, the win ratio is above 55% which then provide a good consistency for the cumulative profit across 3 years. 
- Based from the plot, Strategy 3 showcased the best result in term of consistency and profit. 
- Hence we can conclude that, the fundamental and technical analysis approach for the algorithm suits the market landscape.

# Conclusion

- In this project, the main focus of the analysis was based on the variable Continuous Wind Forecast and Gas Price
- However, there is still other information that is still reliable to be extracted from the data.
- For Instance, the Generator Price and Availability were not significant in the regression analysis
- On the other hand, if a tree-based decision is to be used, the generator availability index might provide better insights.
- Using that information into the trading algorithm can provide better results as the price of Integrated Single Electricity Market (I-SEM) is highly based on the cost of generator in the case where the wind output is low

However, proper new features need to be developed before passing the data into the model to use this approach.

In terms of the trading algorithm, it is shown that the trading accuracy (win ratio) is improved by increasing the trading rules. In short, these are what are called trading procedures. The increase in the number of the trading rule will reduce the total period traded. But it can improve the win ratio if a proper direction is developed. It is also highly dependance on the fund manager/ Company to decide the trading Risk/Reward ratio in order to determine the best result outcome. 

