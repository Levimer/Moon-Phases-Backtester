# Moon-Phases-Strategy
Let's you test buy and sell strategies using the 8 major moon phases


// @version=5
INITIAL_CAPITAL = 10000
MAKER_FEE = 0.00
TAKER_FEE = 0.01
COMMISION_VALUE = (MAKER_FEE + TAKER_FEE)/2  // You cannot directly set maker and taker fees, forced to use a commision fee on buys and sells
strategy("Moon Phases Strategy", overlay = true, initial_capital = INITIAL_CAPITAL, commission_value = COMMISION_VALUE)
if timeframe.ismonthly   
    runtime.error("Unsupported timeframe.")




//////////////////////////// Moon Inputs
// Full Moon Inputs
enableFullMoonIcon = input.bool(defval = true, title = "     🌕", group = "Full Moon")
enableFullMoonBackground = input.bool(defval = true, title = "", group = "Full Moon", inline = "full moon background")
fullMoonBackgroundColor = input.color(color.white, title = "", group = "Full Moon", inline = "full moon background")

// First Quarter Inputs
enableFirstQuarterIcon = input.bool(defval = false, title = "     🌓", group = "First Quarter")
enableFirstQuarterBackground = input.bool(defval = true, title = "", group = "First Quarter", inline = "first quarter background")
firstQuarterBackgroundColor = input.color(color.blue, title = "", group = "First Quarter", inline = "first quarter background")

// New Moon Inputs
enableNewMoonIcon = input.bool(defval = true, title = "     🌑", group = "New Moon")
enableNewMoonBackground = input.bool(defval = true, title = "", group = "New Moon", inline = "new moon background")
newMoonBackgroundColor = input.color(color.blue, title = "", group = "New Moon", inline = "new moon background")

// Third Quarter Inputs
enableThirdQuarterIcon = input.bool(defval = false, title = "     🌗", group = "Third Quarter")
enableThirdQuarterBackground = input.bool(defval = true, title = "", group = "Third Quarter", inline = "third quarter background")
thirdQuarterBackgroundColor = input.color(color.white, title = "", group = "Third Quarter", inline = "third quarter background")

// Waxing Crescent Inputs
enableWaxingCrescentIcon = input.bool(defval = false, title = "     🌒", group = "Waxing Crescent")
enableWaxingCrescentBackground = input.bool(defval = true, title = "", group = "Waxing Crescent", inline = "waxing crescent background")
waxingCrescentBackgroundColor = input.color(color.purple, title = "", group = "Waxing Crescent", inline = "waxing crescent background")

// Waxing Gibbous Inputs
enableWaxingGibbousIcon = input.bool(defval = false, title = "     🌔", group = "Waxing Gibbous")
enableWaxingGibbousBackground = input.bool(defval = true, title = "", group = "Waxing Gibbous", inline = "waxing gibbous background")
waxingGibbousBackgroundColor = input.color(color.orange, title = "", group = "Waxing Gibbous", inline = "waxing gibbous background")

// Waning Gibbous Inputs
enableWaningGibbousIcon = input.bool(defval = false, title = "     🌖", group = "Waning Gibbous")
enableWaningGibbousBackground = input.bool(defval = true, title = "", group = "Waning Gibbous", inline = "waning gibbous background")
waningGibbousBackgroundColor = input.color(color.yellow, title = "", group = "Waning Gibbous", inline = "waning gibbous background")

// Waning Crescent Inputs
enableWaningCrescentIcon = input.bool(defval = false, title = "     🌘", group = "Waning Crescent")
enableWaningCrescentBackground = input.bool(defval = true, title = "", group = "Waning Crescent", inline = "waning crescent background")
waningCrescentBackgroundColor = input.color(color.green, title = "", group = "Waning Crescent", inline = "waning crescent background")



//////////////////////////// Strategy Inputs
// Inputs for strategy
startDate = input.time(timestamp("2020-01-01 00:00"), title="Start Date", group="Strategy")
endDate = input.time(timestamp("2024-07-07 00:00"), title="End Date", group="Strategy")
positionInput = input.int(1000, title="Position Amount", group="Strategy")
leverageRatioInput = input.int(10, title="Leverage Ratio", group="Strategy")
targetPriceRatioInput = input.float(1.08, title="Target Price Ratio", group="Strategy", step=0.01, minval=1.00001)
stopLossPriceRatioInput = input.float(0.92, title="Stop Loss Price Ratio", group="Strategy", step=0.01, maxval=0.99999)
buyMoonPhaseInput = input.string(defval="🌕", title="Buy Moon Phase", options=["🌑", "🌒", "🌓", "🌔", "🌕", "🌖", "🌗", "🌘"], group="Strategy")
sellMoonPhaseInput = input.string(defval="🌑", title="Sell Moon Phase", options=["🌑", "🌒", "🌓", "🌔", "🌕", "🌖", "🌗", "🌘"], group="Strategy")


// Map buy/sell inputs into moonType integers
buyMoonPhase = buyMoonPhaseInput == "🌕" ? -1 : buyMoonPhaseInput == "🌓" ? +2 : buyMoonPhaseInput == "🌑" ? +1 : -2
sellMoonPhase = sellMoonPhaseInput == "🌕" ? -1 : sellMoonPhaseInput == "🌓" ? +2 : sellMoonPhaseInput == "🌑" ? +1 : -2


// Function definitions for moon phase calculations
dayofyear(_time) =>
    // Extract year from timestamp
    _year = year(_time)

    // True if leap year, else False
    leapYear = (_year % 400 == 0) or (_year % 4 == 0 and _year % 100 != 0) 

    // Array where each index is month of the year, and value is days in that month
    dayCount = array.from(31, leapYear ? 29 : 28, 31, 30, 31, 30, 31, 31, 30, 31, 30, 31) 

    // Extract month from timestamp
    timeMonth = month(_time)

    // Count number of days before the current month
    dayofyear = 0
    if timeMonth > 1
        for i = 0 to timeMonth - 2
            dayofyear += dayCount.get(i)

    // Add day of the current month to get the day of the year
    dayofyear + dayofmonth(_time)



getUNIXTimeFromJD(julianDay) =>
    z = math.floor(julianDay + 0.5)
    f = (julianDay + 0.5) % 1
    alpha = math.floor((z - 1867216.25) / 36524.25)
    a = (z < 2299161) ? z : (z + 1 + alpha - math.floor(alpha / 4.0))
    b = a + 1524
    c = math.floor((b - 122.1) / 365.25)
    d = math.floor(365.25 * c)
    e = math.floor((b - d) / 30.6001)
    dayFloat = b - d - math.floor(30.6001 * e) + f
    _day = int(dayFloat)
    _month = (e < 13.5) ? int(e - 1) : int(e - 13)
    _year = (_month > 2.5) ? int(c - 4716) : int(c - 4715)
    secondTotal = int((dayFloat % 1) * 60 * 60 * 24)
    _hour = secondTotal / (60 * 60)
    _minute = (secondTotal - (_hour * 60 * 60)) / 60
    _second = secondTotal % 60
    timestamp(_year, _month, _day, _hour, _minute, _second)



getLastNewMoon(_time) =>
    _year = year(_time)
    dayOfYear = dayofyear(_time)
    k = (dayOfYear / 364.25 + _year - 1900) * 12.3685
    int moonPhase = na

    // Determine moon phase
    if ((k % 1.0) < 0.25)
        k := math.floor(k)
        moonPhase := +1  // New moon
    else if ((k % 1.0) < 0.5)
        k := math.floor(k) + 0.25
        moonPhase := +2  // First quarter
    else if ((k % 1.0) < 0.75)
        k := math.floor(k) + 0.5
        moonPhase := -1  // Full moon
    else
        k := math.floor(k) + 0.75
        moonPhase := -2  // Third quarter

    t = k / 1236.85
    
    // mean anomaly of the sun at the time julianDate
    anomalySun = math.toradians(359.2242 + (29.10535608 * k) - (0.0000333 * math.pow(t, 2)) - (0.00000347 * math.pow(t, 3)))
    // mean anomaly of the moon at the time julianDate
    anomalyMoon = math.toradians(306.0253 + (385.81691806 * k) + (0.0107306 * math.pow(t, 2)) + (0.00001236 * math.pow(t, 3)))
    // argument of latitude of the moon
    f = math.toradians(21.2964 + ((390.67050646 * k) - (0.0016528 * math.pow(t, 2)) - (0.00000239 * math.pow(t, 3))))
    dev = (0.1734 - (0.000393 * t)) * math.sin(anomalySun)
          + 0.0021 * math.sin(2 * anomalySun)
          - 0.4068 * math.sin(anomalyMoon)
          + 0.0161 * math.sin(2 * anomalyMoon)
          - 0.0004 * math.sin(3 * anomalyMoon)
          + 0.0104 * math.sin(2 * f)
          - 0.0051 * math.sin(anomalySun + anomalyMoon)
          - 0.0074 * math.sin(anomalySun - anomalyMoon)
          + 0.0004 * math.sin(2 * f + anomalySun)
          - 0.0004 * math.sin(2 * f - anomalySun)
          - 0.0006 * math.sin(2 * f + anomalyMoon)
          + 0.0010 * math.sin(2 * f - anomalyMoon)
          + 0.0005 * math.sin(anomalySun + 2 * anomalyMoon)

    julianDate = 2415020.75933 + (29.53058868 * k) + (0.0001178 * math.pow(t, 2)) - (0.000000155 * math.pow(t, 3)) +
          (0.00033 * math.sin(math.toradians(166.56) + (math.toradians(132.87) * t) - (math.toradians(0.009173) * math.pow(t, 2)))) + dev
    timeLastMoonPhase = getUNIXTimeFromJD(julianDate)
    timeLastMoonPhase := timeLastMoonPhase >= _time ? timeLastMoonPhase[1] : timeLastMoonPhase

    var int moonPhaseAdjusted = moonPhase
    moonPhaseAdjusted := barstate.isfirst or ta.change(timeLastMoonPhase) != 0 ? moonPhase : moonPhaseAdjusted[1]
    moonType = ta.change(moonPhaseAdjusted) ? moonPhase : na
    [moonPhaseAdjusted, moonType]


// Plotting moons
[moonPhase, moonType] = getLastNewMoon(time_close)
 

// Plot icons for moon phases
plotchar(time >= startDate and time <= endDate and enableNewMoonIcon and moonType == +1, title="New Moon", char="🌑", location=location.abovebar, size=size.small, display=display.pane)  
plotchar(time >= startDate and time <= endDate and enableFirstQuarterIcon and moonType == +2, title="First Quarter", char="🌓", location=location.abovebar, size=size.small, display=display.pane)
plotchar(time >= startDate and time <= endDate and enableFullMoonIcon and moonType == -1, title="Full Moon", char="🌕", location=location.abovebar, size=size.small, display=display.pane)  
plotchar(time >= startDate and time <= endDate and enableThirdQuarterIcon and moonType == -2, title="Third Quarter", char="🌗", location=location.abovebar, size=size.small, display=display.pane)

// Plot background colors for moon phases
bgcolor(time >= startDate and time <= endDate and enableNewMoonBackground and moonPhase == +1 ? color.new(newMoonBackgroundColor, 95) : time >= startDate and time <= endDate and enableFullMoonBackground and moonPhase == -1 ? color.new(fullMoonBackgroundColor, 95) : time >= startDate and time <= endDate and enableFirstQuarterBackground and moonPhase == +2 ? color.new(firstQuarterBackgroundColor, 95) : time >= startDate and time <= endDate and enableThirdQuarterBackground and moonPhase == -2 ? color.new(thirdQuarterBackgroundColor, 95) : na)



/////////////////////////////////////////////////////////////////////////////////////////////
// Strategy logic
var float buyPrice = na
var float targetPrice = na
var float stopLossPrice = na
var bool inTrade = false
inDateRange = (time >= startDate) and (time <= endDate)


// Buy Condition
if inTrade == false and inDateRange and buyMoonPhase == moonType

    // Calculate amount of BTC 
    buyPrice := close
    targetPrice := buyPrice * targetPriceRatioInput
    stopLossPrice := buyPrice * stopLossPriceRatioInput
    leveragedPositionDollarAmount = (positionInput * leverageRatioInput)
    buy_btc_qty = leveragedPositionDollarAmount / buyPrice
    

    // Enter order
    strategy.entry("Buy", strategy.long, qty=buy_btc_qty, comment = "targetPrice " + str.tostring(targetPrice) + " stopLossPrice " + str.tostring(stopLossPrice))
    inTrade := true


// Sell condition
if inTrade == true and inDateRange
    if sellMoonPhase == moonType
        strategy.close("Buy", comment="Ran out of time, closing trade. -> " + str.tostring(close))
        inTrade := false

    else if close <= stopLossPrice
        strategy.close("Buy", comment="Hit stop loss target")
        inTrade := false

    else if close >= targetPrice
        strategy.close("Buy", comment="Hit target price")
        inTrade := false 


// Display strategy results on the chart
winRate = (strategy.wintrades / strategy.closedtrades) * 100
netProfit = strategy.netprofit
totalClosedTrades = strategy.closedtrades
grossProfit = strategy.grossprofit
grossLoss = strategy.grossloss
profitFactor = grossProfit / math.abs(grossLoss)
avgTrade = strategy.netprofit / strategy.closedtrades

if (time == endDate)
    var label infoLabel = na
    label.delete(infoLabel)
    infoLabel := label.new(x=bar_index, y=high, text="Win Rate: " + str.tostring(winRate, "#.##") + "%\nNet Profit: " + str.tostring(netProfit, "#.##") + "\nTotal Closed Trades: " + str.tostring(totalClosedTrades) + "\nProfit Factor: " + str.tostring(profitFactor, "#.###") + "\nAvg. Trade: " + str.tostring(avgTrade, "#.##"), style=label.style_none, color=color.blue, textcolor=color.white, size=size.normal)
