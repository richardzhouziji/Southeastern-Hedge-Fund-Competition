# Southeastern Hedge Fund Competition

## Calculating Trading Signals

There are two parts of the quantitative strategy. The first is the LSTM volatility prediction model, and the second is the LSTM price movement prediction model. The strategy will first predict if absolute price percentage change exceeds 0.5% on day two (today as day one), and if it is the case, the price movement prediction model will predict the direction of the price change on day two, i.e. whether price percentage change is positive or negative. The trading decision will then be made accordingly. 

We started with an ambitious goal, trying to use a deep learning algorithm to predict the percentage change of price. However, this proved too difficult a task to accomplish. As a result, we broke the process into two procedures. Volatility is much easier to predict than price, and because the price move prediction model is trained only on trading days with price percentage change greater than 0.5%, it captures features that cause the change of stock prices better. 

Specifically, we chose the S&P 500 Index data from 2014 to 2020 as our whole data set (we assume that this data set is the same as the SPY’s data). As mentioned, in order to better capture features that cause the change of stock prices, we selected days that have price percentage change greater than 0.5% in the first half of the data as our training set, and the second half as our testing set. For feature engineering, we produced 161 technical analysis indicators with Python TA-Lib, and imported external data includes agricultural data, bond index data, currency data, and energy and metals data. Principal Component Analysis was used to reduce dimensions and decrease model complexity, which resulted in 60 principal components that represented more than 95% variance of the original data set. In view of LSTM's good property for processing time series data, we used it as the last part of the model to predict the percentage change of price and took previously produced principal components as input. 

In the fitting model, dimension reduction and dropout neural network layers are used to prevent overfitting. As a result, the model is not trying to perfectly fit the training set but to capturesignificant fluctuations during a period. Thus, we can see in Figure 1 that the model doesn't fit the training set very well but has a good generalization on the testing set in Figure 2.

<center>
    <img style="border-radius: 0.3125em;
    box-shadow: 0 2px 4px 0 rgba(34,36,38,.12),0 2px 10px 0 rgba(34,36,38,.08);" 
    src="/Image/train_pct.png">
    <br>
    <div style="color:orange; border-bottom: 1px solid #d9d9d9;
    display: inline-block;
    color: #999;
    padding: 2px;">Figure 1: Predicted Percentage Change in Price (Training Set)</div>
</center>

<center>
    <img style="border-radius: 0.3125em;
    box-shadow: 0 2px 4px 0 rgba(34,36,38,.12),0 2px 10px 0 rgba(34,36,38,.08);" 
    src="/Image/test_pct.png">
    <br>
    <div style="color:orange; border-bottom: 1px solid #d9d9d9;
    display: inline-block;
    color: #999;
    padding: 2px;">Figure 2: Predicted Percentage Change in Price (Testing Set)</div>
</center>

For our strategy, we buy a fixed amount of S&P 500 when the model predicts the price will go up in the next three days and sell all if the model predicts the price will go down in the next three days. Based on this, we ran a backtest from June 21, 2018, to January 10, 2020. The comparison of annualized returns between our trading strategy and holding the S&P 500 for the long term is shown in Figure 3.

<center>
    <img style="border-radius: 0.3125em;
    box-shadow: 0 2px 4px 0 rgba(34,36,38,.12),0 2px 10px 0 rgba(34,36,38,.08);" 
    src="/Image/bt_ori.png">
    <br>
    <div style="color:orange; border-bottom: 1px solid #d9d9d9;
    display: inline-block;
    color: #999;
    padding: 2px;">Figure 3: Back Test (ignoring volatility)</div>
</center>

In Figure 3, we see that our model is very sensitive to volatility, especially price bouncing back, but performs not very well when the change in price is relatively stable. We think it is because we selected data with a price change percentage greater than 0.5% as the training set, which allowed the model to detect price percentage change. 
Meanwhile, it is commonly believed that predicting volatility is a much easier task to accomplish than predicting price. Volatility tends to cluster, i.e. high volatile trading days are usually followed by high volatile trading days as well, whereas price movement is a much more stochastic process. However, there are multiple ways to denote volatility in financial literature; in our case, since our goal is to capture trading signals on days when absolute price change percentage is greater than 0.5%, we chose the absolute value of day return to denote volatility. Thus the predicted result of the volatility model is absolute price change percentage and can be used directly as signals to determine whether it makes sense to use a price prediction model to forecast the price movement tomorrow. We used 14-day rolling standard deviation as input data for the LSTM model to capture the lag effect of time series data. The prediction on test data is shown in Figure 4. Though the model does not make an exact prediction of volatility, it captures the general shape of volatility movement. Moreover, it matches with real volatility well when a threshold line of 0.5% is put on the plot. The model predicts most trading days accurately that have more than 0.5% volatility and thus enables the price prediction model to signal the direction of the market these days.

<center>
    <img style="border-radius: 0.3125em;
    box-shadow: 0 2px 4px 0 rgba(34,36,38,.12),0 2px 10px 0 rgba(34,36,38,.08);" 
    src="/Image/vol.png">
    <br>
    <div style="color:orange; border-bottom: 1px solid #d9d9d9;
    display: inline-block;
    color: #999;
    padding: 2px;">Figure 4: Volatility (Predicted v.s. Actual)</div>
</center>

By combining these two models, we filter out days with low volatility and choose the rest to decide whether to trade or not based on our strategy, which helps fully use the advantage of our model. The performance of this model will be discussed in the next section. 
In summary, our final model for calculating trading signals includes two parts. One is LSTM-Volatility to predict volatility; the other is the PCA-LSTM model to predict price change. And the schematic diagram is shown in Figure 5.

<center>
    <img style="border-radius: 0.3125em;
    box-shadow: 0 2px 4px 0 rgba(34,36,38,.12),0 2px 10px 0 rgba(34,36,38,.08);" 
    src="/Image/sum.png">
    <br>
    <div style="color:orange; border-bottom: 1px solid #d9d9d9;
    display: inline-block;
    color: #999;
    padding: 2px;">Figure 5: Schematic Diagram</div>
</center>

## Analysis of Strategy Prospects

We simulated our active equity investing strategy (excluded the Treasury Note and cash) on the testing period from June 21, 2018, to January 10, 2020. 

<center>
    <img style="border-radius: 0.3125em;
    box-shadow: 0 2px 4px 0 rgba(34,36,38,.12),0 2px 10px 0 rgba(34,36,38,.08);" 
    src="/Image/bt_adj.png">
    <br>
    <div style="color:orange; border-bottom: 1px solid #d9d9d9;
    display: inline-block;
    color: #999;
    padding: 2px;">Figure 6: Back Test (adjusted by volatility)</div>
</center>

As Figure 6 showed, the active equity trading strategy would deliver an annualized return of 43.88%, while the benchmark, the S&P 500 Index, produced an annualized return of 9.82%. In terms of risk measurements, the annualized standard deviation of the active SPY investment is 0.425, compared to the benchmark’s annualized standard deviation of 0.123. The return skewness is -0.478, and the kurtosis is 4.655, which means the return is negatively skewed and does not have a normal distribution. It is inappropriate to use the Sharpe ratio on an abnormal distributed return pattern. Alternatively, we measured the Sortino ratio, which accounts for only the downside deviation. The Sortino ratio is 1.463. One unit of downside risk would be compensated by 1.463 units of an excess return over the benchmark. We also implemented several methods to evaluate the risks during extreme events. The return over the maximum drawdown is 0.969, indicating that 1% of the maximum drawdown would be rewarded by 0.97% of the return. The 5% VaR is -4.14%, denoting that there is a 5% probability that the SPY investment would lose more than 4.14%. Expected shortfall, or CVaR, is -5.86%, with a 99% confidence interval of [-8.41%, -4.69%]. If the loss exceeds the VaR threshold, the average loss would be 5.86%. 

Total Portfolio: The total return of the SPY investment would be 58.89% between June 21, 2018, and January 10, 2020. We assume that the 2-Year Treasury Note would generate a total return of 2%. At the end of the invested period, the weights of the SPY, Treasury Note, and cash in the portfolio would be around 78.5%, 18%, and 3.5%. Our portfolio’s weighted average total return would be 46.6%, compared with the benchmark’s total return of 12.7%. The alpha would be 33.9%.