# TradeTheMarket - Day Trading
A few tools I've developed and back tested against live market data for personal use, enjoy :)

# Setup Trading View Indicators
This has been built on a past stock trading strategy I have used for quite the while.

# About
The focus towards as a day trader, this setup would be Identifying the trend, volume and volitily of the current market.
Thus we will always be focusing on Long positions, as a golden rule "Don't trade against the trend", 
therefor this is made to identify long positions only. This is effective towards timeframes from 1hr to higher, with a 
100% accuracy backtest on Trading view.

When it comes to managing a trade there are a few set of rules we need to follow before entering, which this setup does for you
```
Checklist:
-Check the trend
-Check if Moving averages cross due to trend changes, identying the end of a trend (Long)
  If I were to work with the Daily timeframe, I would have a main Moving Average of 200 days, 
  therefor checking the trend throught the year, next would be to have Moving Average for 
  2 weeks (12) and one for a week (7), allowing us to keep track of our trend based on a 2 week
  change.
-Check if DIDI - Volume is showing buy volume
-Checks if price has closed below 200 Moving average for buy validation
-Checks if a certain distance is matched from the closed price and the 200 Moving average for buy validation
-Once checks have met, it would be recommended to follow the rules below, which impacts the accuracy of the trade


Note:
-RXOL is only built on identifying exists for your trades
```
 
What to do when you are in a trade?
```
Rules:
-Keep a look out for Buy Indication 
-Enter buy order and set a trailing stoploss of 30% from entry price, therefor allowing space for the trade to move,
  getting the most out of the trade.
-Why the trailing stop loss? 
  It's important that trade management is in place, thus minimizing the amount of losses, for example;
  If I had entered a trade and price has moved up by 30$, thus setting the trailing stop loss at break even, if
  something were to go wrong in the trade, we are only getting a loss 0f $0.00, therefor protecting our trades increases
  the success in the market.
 -This is built to avoid from sitting infront of the charts all day, but to receive alerts from tradingview, with
  long order entries and the exists for your trades.
```

