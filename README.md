# TradeTheMarket - Day Trading
A few tools I've developed and back tested against live market data for personal use, enjoy :)

"This is being back tested against live stock markets, In six months I will be uploading detailed logs on this setup as well as a detailed explanation to the use of this, as well as a notion template for managing/journaling/tracking trades, see soon :)"

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

Tradingview Pine Script
########################
```
// This source code is subject to the terms of the Mozilla Public License 2.0 at https://mozilla.org/MPL/2.0/
// © MDL_Ranger

//@version=4
strategy("SIGNALS - SETTINGS (SMA)", overlay = true)
//DIDI - VOLUME
//DIDI - VOLUME
Dsrc = input(title="Source", type=input.source, defval=close, group = "DIDI - Volume")
curtaLen = input(title="Curta (Short) Length", type=input.integer, defval=3, group = "DIDI - Volume")
mediaLen = input(title="Media (Medium) Length", type=input.integer, defval=8, group = "DIDI - Volume")
longaLen = input(title="Longa (Long) Length", type=input.integer, defval=20, group = "DIDI - Volume") //20
ma_type = input(title="Didi Index MA", type=input.string, defval="SMA", options=["ALMA", "EMA", "WMA", "SMA", "SMMA", "HMA"], group = "DIDI - Volume")
alma_offset  = input(defval=0.85,type=input.float, title="* Arnaud Legoux (ALMA) Only - Offset Value", minval=0, step=0.01, group = "DIDI - Volume")
alma_sigma   = input(defval=6, title="* Arnaud Legoux (ALMA) Only - Sigma Value", minval=0, group = "DIDI - Volume")
show_filling = input(true, title="Show Filling", type=input.bool, group = "DIDI - Volume")

//QQE Trend Line
useQQE = input(title="QQE 1 or 2", type=input.integer, defval=1, options=[1,2], group = "DIDI - Volume")
signal_len = input(defval=14,type=input.integer, title="Signal Length", group = "DIDI - Volume")
smoothening = input(defval=5,type=input.integer, title="Smoothening Length", group = "DIDI - Volume")
qqe_factor = input(defval=4.236,type=input.float, title="QQE Factor", group = "DIDI - Volume")

ma(type, src, len) =>
    float result = 0
    if type=="SMA" // Simple
        result := sma(src, len)
    if type=="EMA" // Exponential
        result := ema(src, len)
    if type=="WMA" // Weighted
        result := wma(src, len)
    if type=="SMMA" // Smoothed
        w = wma(src, len)
        result := na(w[1]) ? sma(src, len) : (w[1] * (len - 1) + src) / len
    if type=="HMA" // Hull
        result := wma(2 * wma(src, len / 2) - wma(src, len), round(sqrt(len)))
    if type=="ALMA" // Arnaud Legoux
        result := alma(src, len, alma_offset, alma_sigma)
    result

//Credits to Shizaru
qqeLine(src, len, sf, qqe) =>
    wildersper = len*2-1
    rsima = ema(src,sf)

    atr = abs(rsima[1]-rsima)
    maatrrsi = ema(atr,wildersper)

    dar = ema(maatrrsi,wildersper)*qqe

    trr = 0.0
    trr := rsima>nz(trr[1])?((rsima-dar)<trr[1]?trr[1]:(rsima-dar)):((rsima+dar)>trr[1]?trr[1]:(rsima+dar))

//Credits to Glaz
qqeLine2(src, len, sf, qqe) =>
    Wilders_Period = len*2-1

    RsiMa = ema(src,sf)
    
    AtrRsi = abs(RsiMa[1] - RsiMa)
    MaAtrRsi = ema(AtrRsi, Wilders_Period)
    dar = ema(MaAtrRsi,Wilders_Period) * qqe
    
    longband = 0.0
    shortband=0.0
    trend = 0
    
    DeltaFastAtrRsi= dar
    RSIndex=RsiMa
    newshortband=  RSIndex + DeltaFastAtrRsi
    newlongband= RSIndex - DeltaFastAtrRsi
    longband := RSIndex[1] > longband[1] and RSIndex > longband[1]? max(longband[1],newlongband):newlongband
    shortband := RSIndex[1] < shortband[1] and  RSIndex < shortband[1]? min(shortband[1], newshortband):newshortband
    trend := cross(RSIndex, shortband[1])?1:cross(longband[1], RSIndex)?-1:nz(trend[1],1)
    FastAtrRsiTL = trend==1? longband: shortband
    
    FastAtrRsiTL_plot = FastAtrRsiTL

media = ma(ma_type, Dsrc, mediaLen)
curta = (ma(ma_type, Dsrc, curtaLen) - media)
longa = (ma(ma_type, Dsrc, longaLen) - media)

plot(0, title="Media", color=color.gray, transp=0)

trendColor = show_filling ? (curta > longa ? color.lime : color.orange) : na

qqe_line = useQQE == 1 ? qqeLine (curta, signal_len, smoothening, qqe_factor) : qqeLine2 (curta, signal_len, smoothening, qqe_factor)

qqeColor = qqe_line > 0 ? color.lime : color.orange
c_qqeline_cross_Long = crossover(curta, qqe_line)
c_qqeline_cross_Short = crossover(qqe_line, curta)

//Signals based on crossover
c_cross_Long = crossover(curta, longa)
c_cross_Short = crossover(longa, curta)

//Signals based on signal position
c_trend_Long = curta > longa ? 1 : 0
c_trend_Short = longa > curta ? 1 : 0

confirm_Long = c_cross_Long
confirm_Short = c_cross_Short

long_direction_check = (confirm_Long and (curta > 0  and longa > 0) and (curta > longa))
false_long_direction_check = (confirm_Long and ( not (curta > 0  and longa > 0) ) and (curta > longa))
short_direction_check = (confirm_Short and (curta < 0  and longa < 0) and (curta < longa))
false_short_direction_check = (confirm_Short and ( not (curta < 0  and longa < 0)) and (curta < longa))

//RXOL - EXIT
upv = input(color.rgb(255,0,0), title = "Positive RXOL", group = "RXOL Colors")
dnv = input(color.rgb(144,144,144), title = "Negative RXOL", group = "RXOL Colors")
linec = input(color.rgb(255,255,255,75), title = "RXOL Outline", group = "RXOL Colors")
mlen = input(defval = 21, title = "Average Length", group = "RXOL Base Settings")
th = input(defval = 1 , title = "Base Level", type = input.float, group = "RXOL Base Settings")
matype = input(defval = "SMA", options = ["SMA", "EMA"], title = "MA Type", group = "RXOL Base Settings")
smoothing = input(defval = false, title = "Smooth RXOL?", group = "Smoothing")
smoothlen = input(defval = 8, title = "Smoothing Factor", group = "Smoothing")

vol = volume
mvol = matype == "SMA"? sma(vol, mlen) :  ema(vol, mlen)
float rvol = smoothing? hma(vol/mvol, smoothlen) : (vol/mvol)

bool  bull = rvol > th


//SMA - TREND
//Global Variables
var lbl = label.new(na, na, "", color = color.lime, style = label.style_label_lower_right, size=size.tiny)
var lbs = label.new(na, na, "", color = color.red, style = label.style_label_upper_right, size=size.tiny)

ld = input(defval = 5, type=input.float, title = "Long Distance", group = "Distance from Moving Averages & Price")
sd = input(defval = 3, type=input.float,  title = "Exit Distance", group = "Distance from Moving Averages & Price")

llen = input(defval = 200, title = "MA Long", group = "TREND - Moving Average Settings")
slen = input(defval = 13, title = "MA Slow", group = "TREND - Moving Average Settings")
flen = input(defval = 8, title = "MA Slow", group = "TREND - Moving Average Settings")
fma = sma(close, flen)
sma = sma(close, slen)
lma = sma(close, llen)


//crossovers
long = crossover(fma, sma)
short = crossunder(fma, sma)

//check distance
pDifLong = (close - sma) / sma * 100
pDifShort = (close - sma)/sma * 100
pcDifLong = (close - lma) / lma * 100
pcDifShort = (close - lma)/lma * 100
exitDifLong = close < sma
enterLong = close > lma


//when price is more than 5.5% distance from SMA and is in Long/Short
if enterLong and pcDifLong >= ld 
    labelText = "Entry: " + tostring(open, format.mintick)
    tooltipText = "Offest in bars: " + tostring(13) + "\n4ow: " + tostring(low, format.mintick) + "\nHigh: " + tostring(high, format.mintick)
    // Update the label's position, text and tooltip.
    label.set_xy(lbl, bar_index, close)
    label.set_text(lbl, labelText)
    label.set_tooltip(lbl, tooltipText)
    alert("LONG: SMA --- Signal")
    strategy.entry('Long', strategy.long, 50)
if exitDifLong and pDifShort <= -sd and not bull
    labelText = "Exit: " + tostring(open, format.mintick)
    tooltipText = "Offest in bars: " + tostring(13) + "\nLow: " + tostring(low, format.mintick) + "\nHigh: " + tostring(high, format.mintick)
    // Update the label's position, text and tooltip.
    label.set_xy(lbs, bar_index, close)
    label.set_text(lbs, labelText)
    label.set_tooltip(lbs, tooltipText)
    strategy.exit('Long')


plot(fma, title = "SMA", color=color.lime, transp=75)
plot(sma, title = "SMA", color=color.red, transp=75)
plot(lma, title = "SMA", color=color.white, transp=75)
```

RXOL - Exit Indicator
######################
```
osc// This source code is subject to the terms of the Mozilla Public License 2.0 at https://mozilla.org/MPL/2.0/
// © FractalTrade15

//@version=4

// ———————————————————— Functions
// {

// ————— Function returning `_color` with `_transp` transparency.
f_colorNew(_color, _transp) =>
    _r = color.r(_color)
    _g = color.g(_color)
    _b = color.b(_color)
    color _return = color.rgb(_r, _g, _b, _transp)

// —————————————————————————————————————————
// ————— Advance/Decline Gradient
// ————————————————————————————————————————— 
// ————— Function returning one of the bull/bear colors in a transparency proportional to the current qty of advances/declines 
//       relative to the historical max qty of adv/dec for this `_source`.
f_c_gradientAdvDec(_source, _center, _c_bear, _c_bull) =>
    // float _source: input signal.
    // float _center: centerline used to determine if signal is bull/bear.
    // color _c_bear: most bearish color.
    // color _c_bull: most bullish color.
    // Dependency: `f_colorNew()`
    var float _maxAdvDec = 0.
    var float _qtyAdvDec = 0.
    bool  _xUp     = crossover(_source, _center)
    bool  _xDn     = crossunder(_source, _center)
    float _chg     = change(_source)
    bool  _up      = _chg > 0
    bool  _dn      = _chg < 0
    bool  _srcBull = _source > _center
    bool  _srcBear = _source < _center
    _qtyAdvDec := 
      _srcBull ? _xUp ? 1 : _up ? _qtyAdvDec + 1 : _dn ? max(1, _qtyAdvDec - 1) : _qtyAdvDec :
      _srcBear ? _xDn ? 1 : _dn ? _qtyAdvDec + 1 : _up ? max(1, _qtyAdvDec - 1) : _qtyAdvDec : _qtyAdvDec
    // Keep track of max qty of advances/declines.
    _maxAdvDec := max(_maxAdvDec, _qtyAdvDec)
    // Calculate transparency from the current qty of advances/declines relative to the historical max qty of adv/dec for this `_source`.
    float _transp = 100 - (_qtyAdvDec * 100 / _maxAdvDec)
    var color _return = na
    _return := _srcBull ? f_colorNew(_c_bull, _transp) : _srcBear ? f_colorNew(_c_bear, _transp) : _return
    



study("RXOL - EXIT")

upv = input(color.rgb(255,0,0), title = "Positive RVOL", group = "RVOL Colors")
dnv = input(color.rgb(144,144,144), title = "Negative RVOL", group = "RVOL Colors")
linec = input(color.rgb(255,255,255,75), title = "RVOL Outline", group = "RVOL Colors")
mlen = input(defval = 21, title = "Average Length", group = "RVOL Base Settings")
th = input(defval = 1 , title = "Base Level", type = input.float, group = "RVOL Base Settings")
matype = input(defval = "SMA", options = ["SMA", "EMA"], title = "MA Type", group = "RVOL Base Settings")
smoothing = input(defval = false, title = "Smooth RVOL?", group = "Smoothing")
smoothlen = input(defval = 8, title = "Smoothing Factor", group = "Smoothing")

vol = volume
mvol = matype == "SMA"? sma(vol, mlen) :  ema(vol, mlen)
float rvol = smoothing? hma(vol/mvol, smoothlen) : (vol/mvol)

bool  bull = rvol > th

if not bull
    alert("EXIT: RXOL --- Signal")

rvcol = f_c_gradientAdvDec(rvol, th, dnv, upv) 
pvol = plot(rvol, style = plot.style_line, color = linec, linewidth = 2)
pth = plot(th, color = color.rgb(255,255,255,70))
fill(pvol,pth, color = rvcol)
```

DIDI - Volume
```
//@version=4
//Credits to Glaz and Shizaru for their QQE indicator code.

study("DIDI - VOLUME")
src = input(title="Source", type=input.source, defval=close)

curtaLen = input(title="Curta (Short) Length", type=input.integer, defval=3)
mediaLen = input(title="Media (Medium) Length", type=input.integer, defval=8)
longaLen = input(title="Longa (Long) Length", type=input.integer, defval=20) //20

ma_type = input(title="Didi Index MA", type=input.string, defval="SMA", options=["ALMA", "EMA", "WMA", "SMA", "SMMA", "HMA"])
alma_offset  = input(defval=0.85,type=input.float, title="* Arnaud Legoux (ALMA) Only - Offset Value", minval=0, step=0.01)
alma_sigma   = input(defval=6, title="* Arnaud Legoux (ALMA) Only - Sigma Value", minval=0)

show_filling = input(true, title="Show Filling", type=input.bool)

//QQE Trend Line
useQQE = input(title="QQE 1 or 2", type=input.integer, defval=1, options=[1,2])
signal_len = input(defval=14,type=input.integer, title="Signal Length")
smoothening = input(defval=5,type=input.integer, title="Smoothening Length")
qqe_factor = input(defval=4.236,type=input.float, title="QQE Factor")

ma(type, src, len) =>
    float result = 0
    if type=="SMA" // Simple
        result := sma(src, len)
    if type=="EMA" // Exponential
        result := ema(src, len)
    if type=="WMA" // Weighted
        result := wma(src, len)
    if type=="SMMA" // Smoothed
        w = wma(src, len)
        result := na(w[1]) ? sma(src, len) : (w[1] * (len - 1) + src) / len
    if type=="HMA" // Hull
        result := wma(2 * wma(src, len / 2) - wma(src, len), round(sqrt(len)))
    if type=="ALMA" // Arnaud Legoux
        result := alma(src, len, alma_offset, alma_sigma)
    result

//Credits to Shizaru
qqeLine(src, len, sf, qqe) =>
    wildersper = len*2-1
    rsima = ema(src,sf)

    atr = abs(rsima[1]-rsima)
    maatrrsi = ema(atr,wildersper)

    dar = ema(maatrrsi,wildersper)*qqe

    trr = 0.0
    trr := rsima>nz(trr[1])?((rsima-dar)<trr[1]?trr[1]:(rsima-dar)):((rsima+dar)>trr[1]?trr[1]:(rsima+dar))

//Credits to Glaz
qqeLine2(src, len, sf, qqe) =>
    Wilders_Period = len*2-1

    RsiMa = ema(src,sf)
    
    AtrRsi = abs(RsiMa[1] - RsiMa)
    MaAtrRsi = ema(AtrRsi, Wilders_Period)
    dar = ema(MaAtrRsi,Wilders_Period) * qqe
    
    longband = 0.0
    shortband=0.0
    trend = 0
    
    DeltaFastAtrRsi= dar
    RSIndex=RsiMa
    newshortband=  RSIndex + DeltaFastAtrRsi
    newlongband= RSIndex - DeltaFastAtrRsi
    longband := RSIndex[1] > longband[1] and RSIndex > longband[1]? max(longband[1],newlongband):newlongband
    shortband := RSIndex[1] < shortband[1] and  RSIndex < shortband[1]? min(shortband[1], newshortband):newshortband
    trend := cross(RSIndex, shortband[1])?1:cross(longband[1], RSIndex)?-1:nz(trend[1],1)
    FastAtrRsiTL = trend==1? longband: shortband
    
    FastAtrRsiTL_plot = FastAtrRsiTL

media = ma(ma_type, src, mediaLen)
curta = (ma(ma_type,src, curtaLen) - media)
longa = (ma(ma_type, src, longaLen) - media)

plot(0, title="Media", color=color.gray, transp=0)
A = plot(curta, title="Curta", color=color.green, transp=0, linewidth=1)
B = plot(longa, title="Longa", color=color.red, transp=0, linewidth=1)

trendColor = show_filling ? (curta > longa ? color.lime : color.orange) : na
fill(A, B, color=trendColor, transp=80)

qqe_line = useQQE == 1 ? qqeLine (curta, signal_len, smoothening, qqe_factor) : qqeLine2 (curta, signal_len, smoothening, qqe_factor)

qqeColor = qqe_line > 0 ? color.lime : color.orange

plot(qqe_line, color=qqeColor, linewidth=2, transp=20, style=plot.style_cross)

c_qqeline_cross_Long = crossover(curta, qqe_line)
c_qqeline_cross_Short = crossover(qqe_line, curta)

//Signals based on crossover
c_cross_Long = crossover(curta, longa)
c_cross_Short = crossover(longa, curta)

//Signals based on signal position
c_trend_Long = curta > longa ? 1 : 0
c_trend_Short = longa > curta ? 1 : 0

confirm_Long = c_cross_Long
confirm_Short = c_cross_Short

long_direction_check = (confirm_Long and (curta > 0  and longa > 0) and (curta > longa))
plotshape(long_direction_check, color = color.green, style=shape.triangleup, location=location.top, transp=65)
    
false_long_direction_check = (confirm_Long and ( not (curta > 0  and longa > 0) ) and (curta > longa))
plotshape(false_long_direction_check, color = color.green, style=shape.xcross, location=location.top, transp=65)

short_direction_check = (confirm_Short and (curta < 0  and longa < 0) and (curta < longa))
plotshape(short_direction_check, color = color.red, style=shape.triangledown, location=location.top, transp=65)

false_short_direction_check = (confirm_Short and ( not (curta < 0  and longa < 0)) and (curta < longa))
plotshape(false_short_direction_check, color = color.red, style=shape.xcross, location=location.top, transp=65)

if long_direction_check
    alert("VOLUME: Long - RVOL")

plotshape(c_qqeline_cross_Long and c_trend_Long, color = color.green, style=shape.triangleup, location=location.bottom, transp=65)
plotshape(c_qqeline_cross_Short and c_trend_Short, color = color.red, style=shape.triangledown, location=location.bottom, transp=65)

plotshape(c_qqeline_cross_Long and c_trend_Short, color = color.green, style=shape.xcross, location=location.bottom, transp=65)
plotshape(c_qqeline_cross_Short and c_trend_Long, color = color.red, style=shape.xcross, location=location.bottom, transp=65)
```
