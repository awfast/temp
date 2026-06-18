study(title="Main indicator", overlay=true)

out1 = sma(close, 20)
plot(out1, title="SMA 20", color=blue, linewidth = 1)

out2 = sma(close, 50)
plot(out2, title="SMA 50", color=#923a9d, linewidth = 3, style=3)


out3 = sma(close, 200)
plot(out3, title="SMA 200", color=red, linewidth = 4)

plot(ema(close,12),color = purple, title = '12 EMA', linewidth=1)
plot(ema(close,26),color = orange, title = '26 EMA', linewidth=2)



//plot(ema(close,50),color = black, title = '50 EMA', linewidth=1)
plot(ema(close,200),color = gray, title = '200 EMA', linewidth=4)

//Ichimoku Cloud lines
conversionPeriods = input(9, minval=1, title="Conversion Line Periods"),
basePeriods = input(26, minval=1, title="Base Line Periods")

donchian(len) => avg(lowest(len), highest(len))

conversionLine = donchian(conversionPeriods)
baseLine = donchian(basePeriods)

length1 = input(20, minval=1, title="CCI")
ma_length = input(26, minval=1, title="EMA")
src = input(close, title="Source")

ma1 = sma(src, length1)
cci1 = (src - ma1) / (0.015 * dev(src, length1))
plotColour = (cci1 > ema(cci1, ma_length)) ? green : red

Check0 = conversionLine < baseLine
Check1 = conversionLine > baseLine
Check2 = (conversionLine > baseLine) and cci1 > 90
Check4 = cci1 < -100
Check5 = cci1 < -200
Check6 = cci1 > 180
Check7 = cci1 <= -120 and plotColour != green

Crossdown = Check0
Crossup = Check1
CrossupCareful = Check2
CrossupBuyTheDip = Check4
CrossupBestBuy = Check5
ShortHighCCI = Check6
LongConsideration = Check7

IchimokuCrossOverDown = crossover(Crossdown, 0.5)
Longtrigger = crossover(Crossup, 0.5)
LongCarefultrigger = Longtrigger and crossover(CrossupCareful, 0.1)

LongBestBuy = offset(crossover(CrossupBestBuy, 0.5), 1)
LongBuyTheDip = offset(crossover(CrossupBuyTheDip, 0.5), 1)

ShortOverbought = ShortHighCCI

plotshape(IchimokuCrossOverDown, title= "Short Entry", location=location.abovebar, color=red, transp=1, style=shape.triangledown, text="Ichimoku")
plotshape(Longtrigger, title= "Long Entry", location=location.belowbar, color=green, transp=0, style=shape.triangleup, text="Ichimoku")
plotshape(LongCarefultrigger, title= "Long Entry", location=location.abovebar, color=orange, transp=0, style=shape.xcross, text="X")
plotshape(LongBuyTheDip, title= "Long Entry", location=location.abovebar, color=orange, transp=0, style=shape.triangledown, text="BUY THE DIP")
plotshape(LongBestBuy, title= "Long Entry", location=location.abovebar, color=black, transp=1, style=shape.triangledown, text="BEST BUY")
plotshape(LongConsideration, title= "Long Entry", location=location.belowbar, color=gray, transp=1, style=shape.flag, text="")

plotshape(ShortOverbought, title= "Short Entry", location=location.abovebar, color=red, transp=0, style=shape.triangledown, text="^")

//plot(conversionLine, color=#0496ff, title="Conversion Line", linewidth=2)
//plot(baseLine, color=#991515, title="Base Line", linewidth=2)

//TD Sequential

//---- User inputs ---------------------------
PriceSource = input(title="Price: Source", type=source, defval=close)
SetupBars = input(title="Setup: Bars", type=integer, defval=9, minval=4, maxval=31)
SetupLookback = input(title="Setup: Lookback Bars", type=integer, defval=4, minval=1, maxval=14)
SetupEqualEnable = input(title="Setup: Include Equal Price", type=bool, defval=false)
SetupPerfLookback = input(title="Setup: Perfected Lookback", type=integer, defval=3, minval=1, maxval=14)
SetupShowCount = input(title="Setup: Show Count", type=bool, defval=true)
SetupTrendExtend = input(title="Setup Trend: Extend", type=bool, defval=false)
SetupTrendShow = input(title="Setup Trend: Show", type=bool, defval=true)
CntdwnBars = input(title="Countdown: Bars", type=integer, defval=13, minval=3, maxval=31)
CntdwnLookback = input(title="Countdown: Lookback Bars", type=integer, defval=2, minval=1, maxval=30)
CntdwnQualBar = input(title="Countdown: Qualifier Bar", type=integer, defval=8, minval=3, maxval=30)
CntdwnAggressive = input(title="Countdown: Aggressive", type=bool, defval=false)
CntdwnShowCount = input(title="Countdown: Show Count", type=bool, defval=true)
RiskLevelShow = input(title="Risk Level: Show", type=bool, defval=false)
Transp = input(title="Transparency", type=integer, defval=0, minval=0, maxval=100)

PlotEditEnable = false  // show/hide some of the plots from Format window in the user interface.


//---- TD Price Flips ---------------------------
// Plot parameters
// Price Equal plot (pep). Uses plotshape()
pepColor=gray, pepLoc=location.belowbar, pepSize=size.auto, pepStyle=shape.circle

// Create impulse series of price action. Compare where price is greater/less/equal than prior price.
setupPriceUp = (PriceSource > PriceSource[SetupLookback])
setupPriceDown = (PriceSource < PriceSource[SetupLookback])
setupPriceEqual = (PriceSource == PriceSource[SetupLookback])


//---- TD Setups ---------------------------
// Plot parameters
// Setup Count plot (scp)
scpShowLast=144   // Plot char/shapes for previous # of bars, from right edge.
// Setup Count Up plot (scup), Setup Count Down plot (scdp). Uses plotchar()
scupColor=green, scupLoc=location.abovebar, scupSize=size.auto
scdpColor=red,   scdpLoc=location.abovebar, scdpSize=size.auto
// Setup Count Up/Down > Last plot (scuglp/scdglp). Uses plotshape()
scuglpColor=#00900080, scuglpLoc=location.abovebar, scuglpSize=size.auto, scuglpStyle=shape.diamond, scuglpText=""
scdglpColor=#F0000080, scdglpLoc=location.abovebar, scdglpSize=size.auto, scdglpStyle=shape.diamond, scdglpText=""
// Setup Sell plot (ssp), Setup Buy plot (sbp). Uses plotshape()
sspColor=#FF3333, sspLoc=location.abovebar, sspSize=size.normal, sspStyle=shape.triangledown, sspText=""
sbpColor=#33FF33, sbpLoc=location.abovebar, sbpSize=size.normal, sbpStyle=shape.triangleup,   sbpText=""
// Setup Sell/Buy Perfected plot (sspp/sbpp). Uses plotshape()
ssppColor=#FFFF0080, ssppLoc=location.abovebar, ssppSize=size.small, ssppStyle=shape.diamond, ssppText=""
sbppColor=#FFFF0080, sbppLoc=location.abovebar, sbppSize=size.small, sbppStyle=shape.diamond, sbppText=""

// Look for the establishment of momentum by counting consecutive up/down price moves.
//   Up/down counters are mutually exclusive; only one is actively counting, while the other is in reset.
// Equal price ticks are captured separately so that up/down ticks aren't active on the
//   same bar. If equal price enabled,
//     Then include equal price in the present up or down count
//     Else ignore equal price and reset count when present

setupCountUp = SetupEqualEnable ?
   (setupPriceUp or (setupCountUp[1] and setupPriceEqual)) ? nz(setupCountUp[1])+1 : 0
   :
   setupPriceUp ? nz(setupCountUp[1])+1 : 0
setupCountDown = SetupEqualEnable ?
   (setupPriceDown or (setupCountDown[1] and setupPriceEqual)) ? nz(setupCountDown[1])+1 : 0
   :
   setupPriceDown ? nz(setupCountDown[1])+1 : 0

// Error check: make sure Setup Up and Down counts don't count on the same bar
// setupCountErrorCheck = setupCountUp and setupCountDown
// plotshape(setupCountErrorCheck, text="Setup Count Error", style=shape.flag, color=yellow, location=location.abovebar, size=size.normal) // debug

// A Setup event is when up/down count == SetupBars
// Sell Setups are defined by up counts, Buy Setups by down counts.

setupSell = setupCountUp==SetupBars ? valuewhen(setupCountUp==SetupBars, PriceSource, 0) : na

setupBuy = setupCountDown==SetupBars ? valuewhen(setupCountDown==SetupBars, PriceSource, 0) : na

// Count bars between setups... used by other indicators
setupSellCount = barssince(setupSell)
setupBuyCount = barssince(setupBuy)

// Perfected Setups
//   For each sell/buy setup, an additional evaluation is performed to determine if it is "perfected".
//   This consists of looking back a few bars and determining if the setup event's price is 
//   higher(sell)/lower(buy) than the lookback bars price. If not, a retest of the lookback
//   bars high/low price is likely.
// SetupPerfLookback (user input) defines which bars to use for perfection evaluation. 
//   Two bars are included in the evaluation: SetupPerfLookback AND SetupPerfLookback+1
// The evalution adheres to DeMark's original definiton where the bar before the setup event
//   also qualifies a perfected setup, even if the setup event bar doesn't qualify.
// Example) Traditional settings: SetupBars=9, SetupPerfLookback=3, 
//   then a sell setup is perfected when 
//       ( (high(8) >= high(6)) and (high(8) >= high(7)) ) or   // start evaluation
//       ( (high(9) >= high(6)) and (high(9) >= high(7)) ) or 
//       ...
//       ( (high(9+n) >= high(6)) and (high(9+n) >= high(7)) )  // continued eval
//   where n counts past the setup event, on setupCountUp bars. The evalution
//   continues until the logic evaluates true, or cancelled.
// Cancelation rules for perfection evaluation aren't clear...(?) So here's a liberal approach:
//   - If a Setup event in the same direction appears, re-start
//   - If a Setup event in the opposite direction appears, cancel

// To evaluate perfected setups, create additional series: 
//   - A perfect price series, used for comparision to the setup event price or beyond
//   - A mask series which holds the decision logic of "perfected" or "deffered"
//     After the mask is created, it overlays setup count series to plot visual indicators.
// For convenience, define const integer variables to translate the meaning of perfected/deffered
setupIsPerfected = 2 
setupIsDeferred = 1 

// Perfected Sell Setup events
// Get the price for which Sell Setup perfection is evaluated. Stair-step series.

setupSellPerfPrice = setupCountUp==SetupBars ?
   ((valuewhen(setupCountUp==(SetupBars-SetupPerfLookback), high, 0) >= 
      valuewhen(setupCountUp==(SetupBars-SetupPerfLookback+1), high, 0)) ? 
         valuewhen(setupCountUp==(SetupBars-SetupPerfLookback), high, 0 ) :
         valuewhen(setupCountUp==(SetupBars-SetupPerfLookback+1), high, 0 )
   ) : nz(setupSellPerfPrice[1])
//plot(setupSellPerfPrice, color=yellow, linewidth=2)  // debug

// Create mask to hold "perfected" decisions. This is like a state-machine, where new inputs
//   determine what to do next. The logic:
// First, cancellation
//   - If a perfected event found, cancel (done)
//   - If a Buy Setup event occurs, cancel. This Sell Setup trend is over.
// Second, start (or re-start) evaluation
//   - If a new Setup Sell event is present, start. Compare SetupBars and (SetupBars-1) to perf price.
//       If one of these bars passes, then set mask to perfected. 
//       Else, set mask to deferred and continue evaluaton.
// Third, continue evaluation
//   - If mask is deffered, check any bar (count up or down) for perfection, until cancelation. 
//   - If a perfected Sell Setup event NOT found, then seamlessly roll into the next Sell Setup event.

setupSellPerfMask = 
   ((nz(setupSellPerfMask[1])>=setupIsPerfected) or (not na(setupBuy))) ? na : 
      setupCountUp==SetupBars ? 
         ((valuewhen(setupCountUp==(SetupBars-1), high, 0) >= setupSellPerfPrice) or 
          (valuewhen(setupCountUp==SetupBars, high, 0) >= setupSellPerfPrice)) ?
             setupIsPerfected : setupIsDeferred 
         : 
         na(setupSellPerfMask[1]) ? na : 
           high>=setupSellPerfPrice ? setupIsPerfected : setupSellPerfMask[1]

// Get the perfected bar for plotting later
setupSellPerf = setupSellPerfMask==setupIsPerfected ? PriceSource : na

// Perfected Buy Setup events

setupBuyPerfPrice = setupCountDown==SetupBars ?
   ((valuewhen(setupCountDown==(SetupBars-SetupPerfLookback), low, 0) <= 
      valuewhen(setupCountDown==(SetupBars-SetupPerfLookback+1), low, 0)) ? 
         valuewhen(setupCountDown==(SetupBars-SetupPerfLookback), low, 0 ) :
         valuewhen(setupCountDown==(SetupBars-SetupPerfLookback+1), low, 0 )
   ) : nz(setupBuyPerfPrice[1])
//plot(setupBuyPerfPrice, color=yellow, linewidth=2)  // debug


setupBuyPerfMask = 
   ((nz(setupBuyPerfMask[1])>=setupIsPerfected) or (not na(setupSell))) ? na : 
      setupCountDown==SetupBars ? 
         ((valuewhen(setupCountDown==(SetupBars-1), low, 0) <= setupBuyPerfPrice) or 
          (valuewhen(setupCountDown==SetupBars, low, 0) <= setupBuyPerfPrice)) ?
             setupIsPerfected : setupIsDeferred 
         : 
         na(setupBuyPerfMask[1]) ? na : 
           low<=setupBuyPerfPrice ? setupIsPerfected : setupBuyPerfMask[1]

setupBuyPerf = setupBuyPerfMask==setupIsPerfected ? PriceSource : na


//---- TD Setup Trend (TDST) ---------------------------
// Plot parameters
// Setup Trend Support/Resistance plot (stsp/strp). Uses plot()
stspColor=green, stspStyle=circles, stspOffset=0 //=(1-SetupBars)
strpColor=#FF9999, strpStyle=circles, strpOffset=0 //=(1-SetupBars)
// Shading between support/resistance lines. Uses fill()
stpColorNormal=#00000000   // Support is below resistance (#00000000 = no color and transparent)
stpColorFlip  =#20202040   // Support is above resistance, flipped

// TDSTs are support/resistance lines, i.e. the lowest/highest price since: 
//   1) The beginning of this Setup (price flip, count==1), or
//   2) The previous Setup of the same trend, e.g. if a Sell Setup, then low(support) from previous Sell Setup
//      This option is enabled by user input "Setup Trend: Extend"
// Support is established at the beginning of (or prior to) a up/sell trend.
// Resistance is established at the beginning of (or prior to) a down/buy trend.

// Limitations of Extended Setup Trend:
//   Cludgy coding... unable to properly extract setupSellCount from seris and hand it to
//     the lowest() function as integer.
//   if/else logic only allows for steps of "SetupBars" to look for lowest/highest,
//     thus resolution is "chuncky". 
//   Stop at 10*SetupBars to avoid script consuming too much server juice... and rejected.
// TODO: Is there a better way to code extended trend lines?

setupTrendSupport = setupSell ? 
   (SetupTrendExtend ? (
      setupSellCount[1] <=   SetupBars ? lowest(SetupBars) : 
      setupSellCount[1] <= 2*SetupBars ? lowest(2*SetupBars) :
      setupSellCount[1] <= 3*SetupBars ? lowest(3*SetupBars) :
      setupSellCount[1] <= 4*SetupBars ? lowest(4*SetupBars) :
      setupSellCount[1] <= 5*SetupBars ? lowest(5*SetupBars) :
      setupSellCount[1] <= 6*SetupBars ? lowest(6*SetupBars) :
      setupSellCount[1] <= 7*SetupBars ? lowest(7*SetupBars) :
      setupSellCount[1] <= 8*SetupBars ? lowest(8*SetupBars) :
      setupSellCount[1] <= 9*SetupBars ? lowest(9*SetupBars) : lowest(10*SetupBars))
      : lowest(SetupBars) )
   : nz(setupTrendSupport[1])

setupTrendResist = setupBuy ? 
   (SetupTrendExtend ? (
      setupBuyCount[1] <=   SetupBars ? highest(SetupBars) : 
      setupBuyCount[1] <= 2*SetupBars ? highest(2*SetupBars) : 
      setupBuyCount[1] <= 3*SetupBars ? highest(3*SetupBars) : 
      setupBuyCount[1] <= 4*SetupBars ? highest(4*SetupBars) : 
      setupBuyCount[1] <= 5*SetupBars ? highest(5*SetupBars) : 
      setupBuyCount[1] <= 6*SetupBars ? highest(6*SetupBars) : 
      setupBuyCount[1] <= 7*SetupBars ? highest(7*SetupBars) : 
      setupBuyCount[1] <= 8*SetupBars ? highest(8*SetupBars) : 
      setupBuyCount[1] <= 9*SetupBars ? highest(9*SetupBars) : highest(10*SetupBars))
      : highest(SetupBars))
   : nz(setupTrendResist[1])


//---- Plot everything ---------------------------
// Background plots come first. Shapes that need to be on top are last.

// TDST (Support/Resistance)
//   Use plot offset to move line back to beginning of Setup count...
stsp=plot(SetupTrendShow?setupTrendSupport:na, title="TDST Support",    style=stspStyle, color=stspColor, linewidth=2, offset=stspOffset)
strp=plot(SetupTrendShow?setupTrendResist:na,  title="TDST Resistance", style=strpStyle, color=strpColor, linewidth=2, offset=strpOffset)
