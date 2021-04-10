# Price Forecasting and Automated Trading  
### 1. Machine learning forecaster and auto-trader overview   
![Overview](/assets/img/AIPE2.png)    
I designed and trained an algorithm on the historical price of Bitcoin to forecast its price movements 30 minutes into the future. I programmed the forecast to be delivered to users as a Telegram message or in our mobile app, AIBitBip, every five minutes.  
Here is an example of how my forecasting model would be used. The auto-trader would buy via the BitMEX API when the price was expected to rise and sell when it was expected to fall. Note that BitMEX is a margin trading market like the Forex trading market in that both permitted leveraged trading. Therefore, if my forecast model was successful, we could earn significant returns with a relatively small investment. However, if it was not, we could lose a significant amount of money quickly.
   
### 2. Data and model platform   
![Overall](/assets/img/Overall_dataLake.png)     
This diagram shows the data pipeline and model inference platform used for BitMEX trading. The auto-trader algorithm inside the API server was built using a Django REST framework. Market price data was acquired via BitMEX and Binance API calls and stored in an AWS Aurora MySQL database([Github repository](https://github.com/Seonwhee-Finance/BitMEX_API){:target="_blank"}). Unstructured data was scraped and stored as as-is JSONs and stored on an AWS S3 server.  
<img src="/assets/img/AWS_EMR4.png" alt="datalake" width="450" />       
Historical price data is the most influential factor in predicting price movements and volatility. Stock markets record market prices at any given time as open, high, low, or close. Thus, I stored market price data in a relational database.   
I scraped news articles and stored them in JSON format in an AWS S3 Bucket. These JSONs were mined using the AWS EMR SparkAPI.   

### 3. Time-series machine learning  
#### (1) Analysis of Bitcoin price time series data   
The difference in prices of Bitcoin was non-stationary time series data that included white noise, trends, and a seasonal component.   
<img src="/assets/img/log-diff.png" alt="logdiff" width="450" /><img src="/assets/img/regime1.png" alt="regime" width="450" />    
  
It could be decomposed into a regime with higher variance and one that was relatively stable. This characteristic indicated that this time series data could be modeled using the regime-switching model([Github repository](https://github.com/Seonwhee-Finance/Time-Series-Models){:target="_blank"}).    
<img src="/assets/img/historical.png" alt="historical" width="300" /><img src="/assets/img/recent.png" alt="recent" width="300" />  
Due to the fact that it was not stationary, recent data(Right, 8/1/2019 – 8/31/2019) and historical data(Left, 1/1/2018 – 7/31/2019) had different variance distributions.   

#### (2) Classification   
The effective classification models were based on the random forest (Breiman, 2001), LightGBM (Ke *et al.*, 2017), and XGBoost (Chen & Guestrin, 2016) techniques. Given how volatile the recent data was, the model was re-trained on it and its hyperparameters were optimized.   
  
#### (3) Investment   
The returns produced by the classifier-based investing algorithm.   
<img src="/assets/img/invest1.png" alt="invest1" width="200" /><img src="/assets/img/invest2.png" alt="invest2" width="200" /><img src="/assets/img/invest3.png" alt="invest3" width="200" />    
The classifier-based algorithm had greater returns than the algorithm that made trades less frequently. There was no correlation between the precision of the classifier and the returns it produced.   
    
-------------   
Breiman, L. (2001). Random Forests. *Machine Learning, 45*(1), 5–32. https://doi.org/10.1023/A:1010933404324   
Chen, T. & Guestrin, C. (2016). XGBoost: A scalable tree boosting system. *Proceedings of the ACM SIGKDD International Conference on Knowledge Discovery and Data Mining*, 785–794. https://doi.org/10.1145/2939672.2939785   
  
Ke, G., Meng, Q., Finley, T., Wang, T., Chen, W., Ma, W., Ye, Q., & Liu, T. (2017). LightGBM: A Highly Efficient Gradient-Boosting Decision Tree. In G. Isabelle, U. Von Luxberg, S. Bengio, H. Wallach, R. Fergus, S. V. N. Vishwanathan, & R. Garnett (Eds.), *Advances in Neural Information Processing Systems 30 (NIPS 2017)* (pp. 3149–3157). https://doi.org/10.1145/1731903.1731925


