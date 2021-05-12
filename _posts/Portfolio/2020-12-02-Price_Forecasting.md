---
title: "Price Forecasting and Automated Trading"
tags: [Time Series]
categories:
  - Portfolio
date: 2020-12-02
use_math: true
---

### 1. Machine learning forecaster and auto-trader overview   
![Overview](/assets/img_price/AIPE2.png)    
I designed and trained an algorithm on the historical price of Bitcoin to forecast its price movements 30 minutes into the future. I programmed the forecast to be delivered to users as a Telegram message or in our mobile app, AIBitBip, every five minutes.  
Here is an example of how my forecasting model would be used. The auto-trader would buy via the BitMEX API when the price was expected to rise and sell when it was expected to fall. Note that BitMEX is a margin trading market like the Forex trading market in that both permitted leveraged trading. Therefore, if my forecast model was successful, we could earn significant returns with a relatively small investment. However, if it was not, we could lose a significant amount of money quickly.
   
### 2. Data and model platform   
![Overall](/assets/img_price/Overall_dataLake.png)     
This diagram shows the data pipeline and model inference platform used for BitMEX trading. The auto-trader algorithm inside the API server was built using a Django REST framework. Market price data was acquired via BitMEX and Binance API calls and stored in an AWS Aurora MySQL database([Github repository](https://github.com/Seonwhee-Finance/BitMEX_API){:target="_blank"}). Unstructured data was scraped and stored as as-is JSONs and stored on an AWS S3 server.  
<img src="/assets/img_price/AWS_EMR4.png" alt="datalake" width="450" />       
Historical price data is the most influential factor in predicting price movements and volatility. Stock markets record market prices at any given time as open, high, low, or close. Thus, I stored market price data in a relational database.   
I scraped news articles and stored them in JSON format in an AWS S3 Bucket. These JSONs were mined using the AWS EMR SparkAPI.   

### 3. Using machine learning on time-series data   
#### (1) Exploratory data analysis   

![decompose](/assets/img_price/Decomposed_series.png)
<br>  
The plot above is a time series decomposition plot of Bitcoin data.
Time series analysis is conducted to identify gradual movements over a long period of time. The trend line in the plot above indicates the general course or tendency of the data over time. It is a centered moving average of the data and moves between seasonal peaks and valleys. This line is considered to be de-seasonalized.  
Although we can see that Bitcoin prices generally decreased, creating a downtrend in the time series, it is hard to say that the general trend was decreasing.  
A time series that exhibits a repeating pattern at fixed intervals within a certain period is said to have seasonality. The time series above does not appear to have a seasonal pattern, but it does appear to have a cyclical pattern.  
The remainder is the error in the model that reflects the difference between an observed value and the trendline estimate. It is unaccounted for by combining the seasonal change and the trend.  
    
#### (2) Price forecast regression    
##### 1) The assumption of stationarity   
The one of challenges that makes time series regression results less compelling is that real-world financial time series data does not obey the assumption of stationarity.   
A stationary time series is one in which the mean and variance are constant over time. A stationary time series is relatively easy to predict because the mean and variance will be the same in the future as they have been in the past.   
<img src="/assets/img_price/historical.png" alt="historical" width="300" /><img src="/assets/img_price/recent.png" alt="recent" width="300" />   
Due to the fact that it was not stationary, recent data(Right, 8/1/2019 – 8/31/2019) and historical data(Left, 1/1/2018 – 7/31/2019) had different variance distributions.  
Differencing is a method of making a non-stationary time series stationary. It is an important part of preparing data for use in an autoregressive integrated moving average (ARIMA) model (Box & Jenkins, 1990).  
  
  
##### 2) Autocorrelation   
Autocorrelation is how correlated a time series is with its past values.
Autocorrelation function (ACF) plots show a time series correlated with itself by certain lengths of time.<br>
![time_series](/assets/img_price/TimeSeries.png)
<br>
This plot shows correlations between points in time up to and including the lag duration.   
The ACF plot of the Bitcoin series shows the correlation coefficient on the vertical axis and the lag duration on the horizontal axis. The lag duration will show how far out the time series is correlated with itself. This series is undifferenced and has a slow decay, meaning that current values are much more correlated with recent values than with values further in the past. This result suggests that the time series is not stationary and so will need to be differenced.   
The partial correlation function (PACF) describes the correlation between two variables while controlling for the values of another set of variables. Let's take a look at the PACF plot above for our Bitcoin dataset.    
It has a significant spike only at Lag 1, meaning that all autocorrelation is effectively explained by Lag 1. The PACF will show how many AR terms I need to use to explain the autocorrelation pattern in a time series. The partial autocorrelation drops off at Lag 2, which is indicative of an AR(2) model.   

   
##### 3) ARIMA model  
In pure AR(p) models, the autoregressive terms in ARIMAs look like a linear regression where the predictive variables are a certain number of previous periods which are represented by p.   
When p = 2, we use the two previous periods of the time series in the autoregressive portion of the calculation to help adjust the line fitted to forecast the series.   
The differencing term, I(d), of an ARIMA refers to the process we use to transform a time series into a stationary one that does not have any trends or seasonality.   
d is the number of transformations used in the process.   
The moving average, MA(q), is the lag of the error component, which is the part of the time series not explained by trends or seasonality.
MA models look like linear regression models in which the predictive variables are the previous q periods of errors.   
Putting all of these variables together, non-seasonal ARIMA models are expressed as ARIMA(p, d, q) where p, d, and q are all lad period durations used to calculate the ARIMA.   

##### 4) Regime-Switching model
Besides stationarity, another significant obstacle to modeling time series data is non-linearity. Many models use hybridized ARIMAs to deal with non-linearity, such as SVM (Pai & Lin, 2005).  
Another type of statistical time series models is regime-switching models, also known as Markov switching models. These models describe the tendency of many economic variables to behave quite differently during economic downturns, such as financial crises, than they do in other times.   
I assumed that the Bitcoin market’s characteristics could be classified into two states according to its volatility: high and low.  
  
For $t=0,1,2,3,...t_0$   
<center>$ y_t = c_{s_{t}=1} + \phi y_{t-1} + \varepsilon_{t}, \quad where \; \varepsilon_{t} \sim N(0, \sigma^2)$</center><br>   
For $t=t_{0}+1, t_{0}+2, ...$   
<center>$ y_t = c_{s_{t}=2} + \phi y_{t-1} + \varepsilon_{t}, \quad where \; \varepsilon_{t} \sim N(0, \sigma^2)$</center><br>        
where $s_t$ is a random variable. We can define a probabilistic model of what caused the change from $s_{t} = 1$ to $s_{t} = 2$ as a two-state Markov chain with:<br>   
<center>$Pr(s_{t}=j|s_{t-1}=i,s_{t-2}=k,...,y_{t-1},y_{t-2},...) = Pr(s_{t}=j|s_{t-1}=i)=P_{ij}$</center><br>
The equation above assumed that the probability of state transition depends on the value of the most recent state.  
Here is the Bitcoin price data after being log-differenced to be made stationary:  

<img src="/assets/img_price/log-diff.png" alt="logdiff" width="300" /><img src="/assets/img_price/regime1.png" alt="regime" width="300" />    

This data can be decomposed into high-variance and stable states, indicating that this time series data could be modeled using a regime-switching model. [<i class="fab fa-github"></i>](https://github.com/Seonwhee-Finance/Time-Series-Models){:target="_blank"}



#### (2) Classification   
Many researchers have used deep neural networks to analyze financial time series. My first deep learning-based time series forecasting model was motivated by the work of Bao et al. (2017) who used the LSTM model with a stacked autoencoder and wavelet transformation to analyze price data. My forecasting model was unsuccessful at producing an accurate regression for some reason. I withdrew the neural network regressor and dove deep into conventional time series models, like ARIMA. However, I found this neural network resolved the classification problem to some extent.  

The effective classification models were based on the random forest (Breiman, 2001), LightGBM (Ke *et al*., 2017), and XGBoost (Chen & Guestrin, 2016) techniques. Given how volatile the recent data was, the model was re-trained on it and its hyperparameters were optimized.
   
  
#### (3) Different approaches to forecasting  
##### 1) Different variables   
To identify variables that affect price movements, researchers have tried different approaches to analyzing time series data. Bitcoin is a cryptocurrency based on blockchain technology, so blockchain information, such as hash rate and average block size, was used to model Bitcoin price predictions (Jang & Lee, 2018).<br><br>
##### 2) Learning from images rather than numerical data  
Trendlines or candlestick formations are used to determine whether a current trend is likely to continue or not. Certain formations indicate entry points for short-term investors. The figure above (FOREX training group, n.d.) shows how investors can interpret the candlestick chart.<br>
This technical analysis motivated me to run convolutional neural networks (CNN) on candlestick charts rather than analyze numeric sequences from OHLCV.<br>
I hypothesized that patterns in the candlestick chart that represent future price movements could be extracted by a CNN. My Github repo[<i class="fab fa-github"></i>](https://github.com/Seonwhee-Finance/Candlestick_pattern_CNN){:target="_blank"} has an example of one such implementation.<br>
The idea was simple to implement. First, candlestick charts of randomly selected time intervals were created and saved as images. Then the images were annotated according to whether the price was rising or falling. A 5-minute candlestick chart of a stock’s price from 8–10 AM on 2/24/2019 saved as an image file could be annotated to show that the price rose when it was higher at 10:05 AM than at 10:00 AM.  

 
    
-------------   
Breiman, L. (2001). Random Forests. *Machine Learning, 45*(1), 5–32. https://doi.org/10.1023/A:1010933404324 <br><br>
Chen, T. & Guestrin, C. (2016). XGBoost: A scalable tree boosting system. *Proceedings of the ACM SIGKDD International Conference on Knowledge Discovery and Data Mining*, 785–794. https://doi.org/10.1145/2939672.2939785   
  
Ke, G., Meng, Q., Finley, T., Wang, T., Chen, W., Ma, W., Ye, Q., & Liu, T. (2017). LightGBM: A Highly Efficient Gradient-Boosting Decision Tree. In G. Isabelle, U. Von Luxberg, S. Bengio, H. Wallach, R. Fergus, S. V. N. Vishwanathan, & R. Garnett (Eds.), *Advances in Neural Information Processing Systems 30 (NIPS 2017)* (pp. 3149–3157). https://doi.org/10.1145/1731903.1731925 <br>  
FOREX Training group. (n.d.). [How to Trade Forex with Japanese Candlestick Patterns]. Retrieved April 12, 2020, from https://forextraininggroup.com/how-to-trade-forex-with-japanese-candlestick-patterns/ <br><br>
Velay, M., & Daniel, F. (2018). Stock Chart Pattern recognition with Deep Learning. https://arxiv.org/abs/1808.00418 <br><br>
Bao, W., Yue, J., & Rao, Y. (2017). A deep learning framework for financial time series using stacked autoencoders and long-short term memory. *PLOS ONE, 12*(7). https://doi.org/10.1371/journal.pone.0180944 <br><br>
Box, G. E. P., & Jenkins, G. (1990). *Time Series Analysis, Forecasting and Control*. Holden-Day, Inc. <br><br>
Jang, H., & Lee, J. (2018). An Empirical Study on Modeling and Prediction of Bitcoin Prices With Bayesian Neural Networks Based on Blockchain Information. *IEEE Access, 6*. https://doi.org/10.1109/ACCESS.2017.2779181 <br><br>
Kim, H., Roh, T., & Choi, T. (2018). A Study of Bayesian Markov-Switching ARMA(p,q)-GARCH(r,s) Model with Hamilton Filter. *The Korean Data Analysis Society, 20*(4). https://doi.org/10.37727/jkdas.2018.20.4.1801 <br><br>
Pai, P.-F., & Lin, C.-S. (2005). A hybrid ARIMA and support vector machines model in stock price forecasting. *Omega, 33*(6). https://doi.org/10.1016/j.omega.2004.07.024 <br><br>
Bao, W., Yue, J., & Rao, Y. (2017). A deep learning framework for financial time series using stacked autoencoders and long-short term memory. *PLoS ONE, 12*(7), 1–24. https://doi.org/10.6084/m9.figshare.5028110




