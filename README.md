# tralingstop
tralingstop
//@version=5
//
// ▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒ 
//  -----------------------------------------------------------------------------
//  Copyright 2022 Iason Nikolas | jason5480
//  Template Strategy script may be freely distributed under the MIT license.
//
//  Permission is hereby granted, free of charge,
//  to any person obtaining a copy of this software and associated documentation files (the "Software"),
//  to deal in the Software without restriction, including without limitation the rights to use, copy, modify, merge,
//  publish, distribute, sublicense, and/or sell copies of the Software, and to permit persons to whom the Software is furnished to do so,
//  subject to the following conditions:
//
//  The above copyright notice and this permission notice shall be included in all copies or substantial portions of the Software.
//
//  THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND,
//  EXPRESS OR IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY,
//  FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM,
//  DAMAGES OR OTHER LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM,
//  OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE SOFTWARE.
//
//  -----------------------------------------------------------------------------
//
//  Authors:  @jason5480
//  Revision: v0.3.2
//  Date:     22-Jun-2022
//
//  Description
//  =============================================================================
//  This script is designed to be used as a template for building new strategies.
//  The framework provide you with a configurable implementation of the entry, exit,
//  stop loss and take profit trailing logic. The user of this script has to copy
//  it and replace the openLongPosition, openShortPosition, closeLongPosition and
//  closeShortPosition variables in the STRATEGY module according to his needs!
//  
//  -----------------------------------------------------------------------------
//  Disclaimer:
//    1. I am not licensed financial advisors or broker dealer. I do not tell you
//       when or what to buy or sell. I developed this software which enables you
//       execute manual or automated trades using TradingView. The
//       software allows you to set the criteria you want for entering and exiting
//       trades.
//    2. Do not trade with money you cannot afford to lose.
//    3. I do not guarantee consistent profits or that anyone can make money with no
//       effort. And I am not selling the holy grail.
//    4. Every system can have winning and losing streaks.
//    5. Money management plays a large role in the results of your trading. For
//       example: lot size, account size, broker leverage, and broker margin call
//       rules all have an effect on results. Also, your Take Profit and Stop Loss
//       settings for individual pair trades and for overall account equity have a
//       major impact on results. If you are new to trading and do not understand
//       these items, then I recommend you seek education materials to further your
//       knowledge.
//
//    YOU NEED TO FIND AND USE THE TRADING SYSTEM THAT WORKS BEST FOR YOU AND YOUR
//    TRADING TOLERANCE.
//
//    I HAVE PROVIDED NOTHING MORE THAN A TOOL WITH OPTIONS FOR YOU TO TRADE WITH THIS PROGRAM ON TRADINGVIEW.
//    
//    I accept suggestions to improve the script.
//    If you encounter any problems I will be happy to share with me.
//  -----------------------------------------------------------------------------
//
// ▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒
// SETUP ============================================================================================================
strategy(title = 'Template Trailing Strategy',
         shorttitle = 'TTS',
         overlay = true,
         pyramiding = 0,
         default_qty_type = strategy.percent_of_equity,
         default_qty_value = 100,
         initial_capital = 100000)

import jason5480/external_input_utils/1 as exiu

//
// ▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒
// 📆📈 FILTERS =====================================================================================================
// Description: Module responsible for filtering out long and short open signals that do not meet user defined rules
// Dependencies: NONE
// Results: longFiltersApproval, shortFiltersApproval

// INPUT ============================================================================================================
usefromDate = input.bool(defval = true, title = 'From', inline = 'From Date', group = '📆 Filters - Time')
fromDate = input.time(defval = timestamp('01 Jan 2021 00:00 UTC'), title = '', inline = 'From Date', group = '📆 Filters - Time')
usetoDate = input.bool(defval = false, title = 'To ', inline = 'To Date', group = '📆 Filters - Time')
toDate = input.time(defval = timestamp('31 Dec 2121 23:59 UTC'), title = '', inline = 'To Date', group = '📆 Filters - Time')

useSessionStart = input.bool(defval = false, title = 'Session Start', inline = 'Session Start', group = '📆 Filters - Time')
sessionStartHour = input.int(defval = 12, title = '', minval = 0, maxval = 23, step = 1, inline = 'Session Start', group = '📆 Filters - Time')
sessionStartMinute = input.int(defval = 00, title = ':', minval = 0, maxval = 59, step = 1, tooltip = 'Start time of the session.', inline = 'Session Start', group = '📆 Filters - Time')
useSessionEnd = input.bool(defval = false, title = 'Session End', inline = 'Session End', group = '📆 Filters - Time')
sessionEndHour = input.int(defval = 20, title = '', minval = 0, maxval = 23, step = 1, inline = 'Session End', group = '📆 Filters - Time')
sessionEndMinute = input.int(defval = 00, title = ':', minval = 0, maxval = 59, step = 1, tooltip = 'End time of the session.', inline = 'Session End', group = '📆 Filters - Time')
useSessionDay = input.bool(defval = false, title = 'Session Days', inline = 'Session Days', group = '📆 Filters - Time')
mon = input.bool(defval = true, title = 'Mon', inline = 'Session Days', group = '📆 Filters - Time')
tue = input.bool(defval = true, title = 'Tue', inline = 'Session Days', group = '📆 Filters - Time')
wed = input.bool(defval = true, title = 'Wed', inline = 'Session Days', group = '📆 Filters - Time')
thu = input.bool(defval = true, title = 'Thu', inline = 'Session Days', group = '📆 Filters - Time')
fri = input.bool(defval = true, title = 'Fri', inline = 'Session Days', group = '📆 Filters - Time')
sat = input.bool(defval = false, title = 'Sat', inline = 'Session Days', group = '📆 Filters - Time')
sun = input.bool(defval = false, title = 'Sun', inline = 'Session Days', group = '📆 Filters - Time')

longTradesEnabled = input.bool(defval = true, title = 'Long Trades', inline = 'Trades', group = '📈 Filters - Trend')
shortTradesEnabled = input.bool(defval = true, title = 'Short Trades', tooltip = 'Enable long/short trades.', inline = 'Trades', group = '📈 Filters - Trend')

emaFilterEnabled = input.bool(defval = true, title = 'EMA Filter', tooltip = 'Enable long/short trades based on EMA.', group = '📈 Filters - Trend')
emaTimeframe = input.timeframe(defval = 'D', title = '  EMA Res/Len/Src', inline = 'EMA Filter', group = '📈 Filters - Trend')
emaLength = input.int(defval = 200, title = '', inline = 'EMA Filter', group = '📈 Filters - Trend')
emaSrc = exiu.str_to_src(input.string(defval = 'close', title = '', options = ['open', 'high', 'low', 'close', 'hl2', 'hlc3', 'ohlc4', 'hlcc4'], tooltip = 'The timeframe, period and source for the EMA calculation.', inline = 'EMA Filter', group = '📈 Filters - Trend'))
emaAtrBandEnabled = input.bool(defval = true, title = 'EMA ATR Band', tooltip = 'Enable ATR band for EMA filter.', group = '📈 Filters - Trend')
filterAtrLength = input.int(defval = 5, title = '  EMA ATR Len/Mul', minval = 1, inline = 'EMA ATR', group = '📈 Filters - Trend')
filterAtrMul = input.float(defval = 1.0, title = '', tooltip = 'ATR length and multiplier to be used for the ATR calculation that will be added on top of the EMA filter.', minval = 0.1, step = 0.1, inline = 'EMA ATR', group = '📈 Filters - Trend')

// LOGIC ============================================================================================================
bool dateFilterApproval = (usefromDate ? time_close >= fromDate : true) and (usetoDate ? time_close < toDate : true)

isInSession() =>
    var time_start_session = timestamp(year(time_close), month(time_close), dayofmonth(time_close), sessionStartHour, sessionStartMinute, second(time_close))
    var time_end_session = timestamp(year(time_close), month(time_close), dayofmonth(time_close), sessionEndHour, sessionEndMinute, second(time_close))
    var one_day = 86400000
    var time_day = time_close
    if (useSessionStart and useSessionEnd and time_start_session >= time_end_session)
        time_end_session :=  time_end_session + one_day
        time_day := time_day + one_day
    if ((useSessionStart and useSessionEnd and time_close >= time_end_session) or ((not useSessionStart or not useSessionEnd) and ta.change(dayofmonth(time_close))))
        time_start_session := time_start_session + one_day
        time_end_session := time_start_session >= time_end_session ? time_end_session + one_day : time_end_session
        time_day := time_day + one_day
    isSessionDay = switch dayofweek(time_day)
        dayofweek.monday => mon
        dayofweek.tuesday => tue
        dayofweek.wednesday => wed
        dayofweek.thursday => thu
        dayofweek.friday => fri
        dayofweek.saturday => sat
        dayofweek.sunday => sun
        => false
    (useSessionDay ? isSessionDay : true) and (useSessionStart ? time_close >= time_start_session : true) and (useSessionEnd ? time_close < time_end_session : true)

bool sessionFilterApproval = isInSession()
bool timeFilterApproval = dateFilterApproval and sessionFilterApproval

emaLine = request.security(syminfo.tickerid, emaTimeframe, ta.ema(emaSrc, emaLength))
emaAtr = ta.atr(filterAtrLength)
emaUpperBand = emaLine + filterAtrMul * emaAtr
emaLowerBand = emaLine - filterAtrMul * emaAtr
bool emaLongApproval = emaFilterEnabled ? close > (emaAtrBandEnabled ? emaUpperBand : emaLine) and open > (emaAtrBandEnabled ? emaUpperBand : emaLine) : true
bool emaShortApproval = emaFilterEnabled ? close < (emaAtrBandEnabled ? emaLowerBand : emaLine) and open < (emaAtrBandEnabled ? emaLowerBand : emaLine) : true

bool longFiltersApproval = longTradesEnabled and emaLongApproval and timeFilterApproval
bool shortFiltersApproval = shortTradesEnabled and emaShortApproval and timeFilterApproval

// PLOT =============================================================================================================
bgcolor(color = timeFilterApproval ? color.new(color.gray, 90) : na, offset = 1, title = 'Period')

showEma = input.bool(defval = true, title = 'Show EMA Line', inline = 'EMA Show', group = '📊 Plot')
showEmaBand = input.bool(defval = false, title = 'Show EMA Band', tooltip = 'Show the EMA line/band.', inline = 'EMA Show', group = '📊 Plot')

emaLineColor = emaLongApproval ? color.teal : emaShortApproval ? color.maroon : color.gray
plot(series = emaFilterEnabled and showEma ? emaLine : na, color = emaLineColor, style = plot.style_line, linewidth = 2, title = 'EMA Line')
emaUpperBandPlot = plot(series = emaUpperBand, color = na, style = plot.style_line, linewidth = 1, title = 'EMA Upper Band')
emaLowerBandPlot = plot(series = emaLowerBand, color = na, style = plot.style_line, linewidth = 1, title = 'EMA Lower Band')
emaBandFillColor = emaFilterEnabled and emaAtrBandEnabled and showEmaBand ? color.new(emaLineColor, 95) : na
fill(plot1 = emaUpperBandPlot, plot2 = emaLowerBandPlot, color = emaBandFillColor, title = 'EMA Band')

//
// ▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒
// 🛠️ STRATEGY ====================================================================================================
// Description: Module responsible for the open and close position logic. This is implemented based on deal conditions defined internally (in this script) or externaly (based on conditions that take as input other indicator)
// Dependencies: FILTERS
// Results: openLongPosition, openShortPosition, closeLongPosition, closeShortPosition

import HeWhoMustNotBeNamed/enhanced_ta/14 as eta

// INPUT ============================================================================================================
dealMode = input.string(defval = '🔧Internal', title = 'Deal Conditions Mode', options = ['🔧Internal', '🔨External'], tooltip = 'Use the "Internal" MA cross over/under logic to start and end your deals. Or use an "External" indicator to construct your own start and end deal conditions.', group = '🛠️ Strategy')
ignoreEndDeals = input.bool(defval = true, title = 'Ignore End Deals', tooltip = 'A close signal will not be emited when the end deal condition is met, thus you will not exit your existing position based on the strategy logic. If this option is checked, you will exit only when the stop loss or the take profit are triggered.', group = '🛠️ Strategy')
closeAtSessionEnd = input.bool(defval = false, title = 'Close at Session End', tooltip = 'Close all positions at the market price at the end of each session or the end of time window.', group = '🛠️ Strategy')

fastMAType = str.lower(input.string(defval = 'SMA', title = 'Fast MA Type/Len', options = ['SMA', 'EMA', 'HMA', 'RMA', 'WMA', 'VWMA', 'SWMA', 'LINREG', 'MEDIAN'], inline = 'Fast MA', group = '🔧 Strategy - Internal'))
fastMALen = input.int(defval = 21, title = '', tooltip = 'The type and the length of the fast MA.', inline = 'Fast MA', group = '🔧 Strategy - Internal')
slowMAType = str.lower(input.string(defval = 'SMA', title = 'Slow MA Type/Len', options = ['SMA', 'EMA', 'HMA', 'RMA', 'WMA', 'VWMA', 'SWMA', 'LINREG', 'MEDIAN'], inline = 'Slow MA', group = '🔧 Strategy - Internal'))
slowMALen = input.int(defval = 49, title = '', tooltip = 'The type and the length of the slow MA.', inline = 'Slow MA', group = '🔧 Strategy - Internal')

externalInput = input.source(defval = close, title = 'External Input', tooltip = 'Select input from an external indicator. The indicator should be added to the same chart with this strategy and the desired value that will take part in the condition should be ploted in the chart.', group = '🔨 Strategy - External')

startLongDealOperator = input.string(defval = '>=', title = 'Start Long Deal when input', options = ['==', '<', '>', '<=', '>=', '!=', 'crossover', 'crossunder', 'mod2', 'mod3', 'noop'], inline = 'Start Long Deal', group = '🔨 Strategy - External')
startLongDealValue = input.float(defval = 2.0, title = '', tooltip = 'Conditon to start a Long Deal based on external input.', inline = 'Start Long Deal', group = '🔨 Strategy - External')
endLongDealOperator = input.string(defval = '<=', title = 'End Long Deal when input', options = ['==', '<', '>', '<=', '>=', '!=', 'crossover', 'crossunder', 'mod2', 'mod3', 'noop'], inline = 'End Long Deal', group = '🔨 Strategy - External')
endLongDealValue = input.float(defval = 1.0, title = '', tooltip = 'Conditon to end a Long Deal based on external input.', inline = 'End Long Deal', group = '🔨 Strategy - External')

startShortDealOperator = input.string(defval = '<=', title = 'Start Short Deal when input', options = ['==', '<', '>', '<=', '>=', '!=', 'crossover', 'crossunder', 'mod2', 'mod3', 'noop'], inline = 'Start Short Deal', group = '🔨 Strategy - External')
startShortDealValue = input.float(defval = -2.0, title = '', tooltip = 'Conditon to start a Short Deal based on external input.', inline = 'Start Short Deal', group = '🔨 Strategy - External')
endShortDealOperator = input.string(defval = '>=', title = 'End Short Deal when input', options = ['==', '<', '>', '<=', '>=', '!=', 'crossover', 'crossunder', 'mod2', 'mod3', 'noop'], inline = 'End Short Deal', group = '🔨 Strategy - External')
endShortDealValue = input.float(defval = -1.0, title = '', tooltip = 'Conditon to end a Short Deal based on external input.', inline = 'End Short Deal', group = '🔨 Strategy - External')

// LOGIC ============================================================================================================
fastMA = eta.ma(close, fastMAType, fastMALen)
slowMA = eta.ma(close, slowMAType, slowMALen)

bool startLongDeal = dealMode == '🔧Internal' ? exiu.eval_cond(fastMA, 'crossover', slowMA) : exiu.eval_cond(externalInput, startLongDealOperator, startLongDealValue, false)
bool endLongDeal = dealMode == '🔧Internal' ? exiu.eval_cond(fastMA, 'crossunder', slowMA) : exiu.eval_cond(externalInput, endLongDealOperator, endLongDealValue, true)

bool startShortDeal = dealMode == '🔧Internal' ? exiu.eval_cond(fastMA, 'crossunder', slowMA) : exiu.eval_cond(externalInput, startShortDealOperator, startShortDealValue, false)
bool endShortDeal = dealMode == '🔧Internal' ? exiu.eval_cond(fastMA, 'crossover', slowMA) : exiu.eval_cond(externalInput, endShortDealOperator, endShortDealValue, true)

bool openLongPosition = longFiltersApproval and startLongDeal
bool openShortPosition = shortFiltersApproval and startShortDeal

bool closeLongPosition = ignoreEndDeals ? false : longTradesEnabled and endLongDeal
bool closeShortPosition = ignoreEndDeals ? false : shortTradesEnabled and endShortDeal

// PLOT =============================================================================================================
var fastColor = color.new(#0056BD, 0)
plot(series = dealMode == '🔧Internal' ? fastMA : na, title = 'Fast MA', color = fastColor, linewidth = 1, style = plot.style_line)
var slowColor = color.new(#FF6A00, 0)
plot(series = dealMode == '🔧Internal' ? slowMA : na, title = 'Slow MA', color = slowColor, linewidth = 1, style = plot.style_line)

//
// ▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒
// ☭ SHARED VARIABLES 1 =============================================================================================
// Description: Module responsible for general purpose variable that needed for several other modules
// Dependencies: STRATEGY
// Results: ALL

import HeWhoMustNotBeNamed/arrayutils/20 as pa

// INPUT ============================================================================================================
atrMethod = input.string(defval = 'STATIC', title = 'ATR Method', options = ['STATIC', 'DYNAMIC', 'LADDER'], tooltip = 'The method to calculate the ATR used for the trailing.', group = '☭ Shared Variables')

atrMaType = str.lower(input.string(defval = 'RMA', title = '  ATR Smooth Type/Len', options = ['SMA', 'EMA', 'HMA', 'RMA', 'WMA'], inline = 'ATR', group = '☭ Shared Variables'))
atrLength = input.int(defval = 14, title = '', minval = 1, tooltip = 'The smoothing type and the length to be used for the ATR calculation.', inline = 'ATR', group = '☭ Shared Variables')

// LOGIC ============================================================================================================
// the open signals when not already into a position
bool validOpenLongPosition = openLongPosition and not (strategy.position_size > 0)
bool validOpenShortPosition = openShortPosition and not (strategy.position_size < 0)
bool validCloseLongPosition = closeLongPosition and strategy.position_size > 0
bool validCloseShortPosition = closeShortPosition and strategy.position_size < 0

// The close price when last valid open was triggered
float openClose = ta.valuewhen(validOpenLongPosition or validOpenShortPosition, close, 0)
// The close price when last valid close was triggered
float closeClose = ta.valuewhen(validCloseLongPosition or validCloseShortPosition, close, 0)

// count how far are the last valid open and close signals
int barsSinceValidOpenLong = nz(ta.barssince(validOpenLongPosition), 999999)
int barsSinceValidOpenShort = nz(ta.barssince(validOpenShortPosition), 999999)
int barsSinceCloseLong = nz(ta.barssince(closeLongPosition and timeFilterApproval), 999999)
int barsSinceCloseShort = nz(ta.barssince(closeShortPosition and timeFilterApproval), 999999)

// count how far are the last enter signals
var bool enterLongPosition = false
var bool enterShortPosition = false
int barsSinceEnterLong = nz(ta.barssince(enterLongPosition), 999999)
int barsSinceEnterShort = nz(ta.barssince(enterShortPosition), 999999)

// calculate atr based on method selected, static atr when last open signal was triggered, dynamic atr is the current atr that change over time, ladder atr for positive and negative bars
dynamicAtr = ta.atr(atrLength)
staticAtr = ta.valuewhen(validOpenLongPosition or validOpenShortPosition, dynamicAtr, 0)

var negativeTrs = array.new<float>()
var positiveTrs = array.new<float>()

if(open > close)
    pa.push(negativeTrs, ta.tr(true), atrLength)
else
    pa.push(positiveTrs, ta.tr(true), atrLength)

ladderLongAtr = pa.ma(negativeTrs, atrMaType, atrLength)
ladderShortAtr = pa.ma(positiveTrs, atrMaType, atrLength)

getLongAtr() =>
    switch atrMethod
        'STATIC' => staticAtr
        'DYNAMIC' => dynamicAtr
        'LADDER' => ladderLongAtr
        => na

getShortAtr() =>
    switch atrMethod
        'STATIC' => staticAtr
        'DYNAMIC' => dynamicAtr
        'LADDER' => ladderShortAtr
        => na

//
// ▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒
// ⇩ ENTRY ==========================================================================================================
// Description: Module responsible for the trailing entry logic implementation based on the deviation
// Dependencies: STRATEGY, SHARED VARIABLES 1
// Results: enterLongPosition, enterShortPosition

// INPUT ============================================================================================================
enableEntryTrailing = input.bool(defval = false, title = 'Enable Trailing', tooltip = 'Enable or disable the trailing for entry position.', group = '⇩ Entry')
devEntryMethod = input.string(defval = 'PERC', title = 'Deviation Method', options = ['PERC', 'ATR', 'LOC'], tooltip = 'The method to calculate the deviation for the trailing entry.', group = '⇩ Entry')
devEntryPerc = input.float(defval = 3.0, title = '  Deviation %', minval = 0.01, maxval = 100, step = 0.05, tooltip = 'The step to follow the price when the open position condition is met.', group = '⇩ Entry') / 100
devEntryAtrMul = input.float(defval = 0.5, title = '  Deviation ATR Mul', minval = 0.01, step = 0.05, tooltip = 'Multiplier to be used on the ATR to calculate the step for following the price, until the entry target is reached.', group = '⇩ Entry')
devEntryLen = input.int(defval = 3, title = '  Dev Loc Extrema Len/Ticks', minval = 1, inline = 'Deviation Entry Local Extrema', group = '⇩ Entry')
devEntryTicks = input.int(defval = 2, title = '', minval = 0, tooltip = 'Local extrema (minimum/maximum) within a window of length minus/plus some ticks to be used for the entry target.', inline = 'Deviation Entry Local Extrema', group = '⇩ Entry')
ctrLongEntrySrc = exiu.str_to_src(input.string(defval = 'high', title = 'Long/Short Entry Control', options = ['open', 'high', 'low', 'close', 'hl2', 'hlc3', 'ohlc4', 'hlcc4'], inline = 'Control', group = '⇩ Entry'))
ctrShortEntrySrc = exiu.str_to_src(input.string(defval = 'low', title = '', options = ['open', 'high', 'low', 'close', 'hl2', 'hlc3', 'ohlc4', 'hlcc4'], tooltip = 'The price source to check with the entry target to trigger the entry order for long/short position.', inline = 'Control', group = '⇩ Entry'))

// LOGIC ============================================================================================================
bool enterLongIsPending = barsSinceEnterLong > barsSinceValidOpenLong and strategy.position_size <= 0 // enter -> validOpen
bool tryEnterLongPosition = longFiltersApproval and enterLongIsPending

float entryHighestHigh = ta.highest(high, devEntryLen)
float openEntryHighestHigh = ta.valuewhen(validOpenLongPosition, entryHighestHigh, 0)

getLongEntryOpenBaseScr() =>
    switch devEntryMethod
        'PERC' => openClose
        'ATR' => openClose
        'LOC' => openEntryHighestHigh
        => na

getLongEntryTrailingBaseScr() =>
    switch devEntryMethod
        'PERC' => low
        'ATR' => low
        'LOC' => entryHighestHigh
        => na

getLongEntryPrice(baseSrc) =>
    switch devEntryMethod
        'PERC' => baseSrc * (1 + devEntryPerc)
        'ATR' => baseSrc + devEntryAtrMul * getLongAtr()
        'LOC' => baseSrc + devEntryTicks * syminfo.mintick
        => na

float longEntryPrice = na
bool isFirstValidOpenLongPosition = validOpenLongPosition and na(longEntryPrice[1])
longEntryPrice := if isFirstValidOpenLongPosition
    getLongEntryPrice(getLongEntryOpenBaseScr())
else if tryEnterLongPosition
    math.min(getLongEntryPrice(getLongEntryTrailingBaseScr()), nz(longEntryPrice[1], 999999))
else
    na

enterLongPosition := enableEntryTrailing ? tryEnterLongPosition and ta.crossover(isFirstValidOpenLongPosition ? close : ctrLongEntrySrc, longEntryPrice) : validOpenLongPosition

bool enterShortIsPending = barsSinceEnterShort > barsSinceValidOpenShort and strategy.position_size >= 0 // enter -> validOpen
bool tryEnterShortPosition = shortFiltersApproval and enterShortIsPending

float entryLowestLow = ta.lowest(low, devEntryLen)
float openEntryLowestLow = ta.valuewhen(validOpenShortPosition, entryLowestLow, 0)

getShortEntryOpenBaseScr() =>
    switch devEntryMethod
        'PERC' => openClose
        'ATR' => openClose
        'LOC' => openEntryLowestLow
        => na

getShortEntryTrailingBaseScr() =>
    switch devEntryMethod
        'PERC' => high
        'ATR' => high
        'LOC' => entryLowestLow
        => na

getShortEntryPrice(baseSrc) =>
    switch devEntryMethod
        'PERC' => baseSrc * (1 - devEntryPerc)
        'ATR' => baseSrc - devEntryAtrMul * getShortAtr()
        'LOC' => baseSrc - devEntryTicks * syminfo.mintick
        => na

float shortEntryPrice = na
bool isFirstValidOpenShortPosition = validOpenShortPosition and na(shortEntryPrice[1])
shortEntryPrice := if isFirstValidOpenShortPosition
    getShortEntryPrice(getShortEntryOpenBaseScr())
else if tryEnterShortPosition
    math.max(getShortEntryPrice(getShortEntryTrailingBaseScr()), nz(shortEntryPrice[1]))
else
    na

enterShortPosition := enableEntryTrailing ? tryEnterShortPosition and ta.crossunder(isFirstValidOpenShortPosition ? close : ctrShortEntrySrc, shortEntryPrice) : validOpenShortPosition

// PLOT =============================================================================================================
var buyColor = color.new(color.green, 0)
plot(series = enableEntryTrailing ? longEntryPrice : na, title = 'Long Buy Price', color = buyColor, linewidth = 1, style = plot.style_linebr)
plot(series = enableEntryTrailing ? shortEntryPrice : na, title = 'Short Sell Price', color = buyColor, linewidth = 1, style = plot.style_linebr)

//
// ▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒
// ⇧ EXIT ===========================================================================================================
// Description: Module responsible for the trailing exit logic implementation based on the deviation
// Dependencies: STRATEGY, SHARED VARIABLES 1
// Results: exitLongPosition, exitShortPosition, longIsActive, shortIsActive

// INPUT ============================================================================================================
enableExitTrailing = input.bool(defval = false, title = 'Enable Trailing', tooltip = 'Enable or disable the trailing for exit position.', group = '⇧ Exit')
devExitMethod = input.string(defval = 'PERC', title = 'Deviation Method', options = ['PERC', 'ATR', 'LOC'], tooltip = 'The method to calculate the deviation for the trailing exit.', group = '⇧ Exit')
devExitPerc = input.float(defval = 3.0, title = '  Deviation %', minval = 0.01, maxval = 100, step = 0.05, tooltip = 'The step to follow the price when the close position condition is met.', group = '⇧ Exit') / 100
devExitAtrMul = input.float(defval = 0.5, title = '  Deviation ATR Mul', minval = 0.01, step = 0.05, tooltip = 'Multiplier to be used on the ATR to calculate the step for following the price, until the exit target is reached.', group = '⇧ Exit')
devExitLen = input.int(defval = 3, title = '  Dev Loc Extrema Len/Ticks', minval = 1, inline = 'Deviation Exit Local Extrema', group = '⇧ Exit')
devExitTicks = input.int(defval = 2, title = '', minval = 0, tooltip = 'Local extrema (minimum/maximum) within a window of length minus/plus some ticks to be used for the entry target.', inline = 'Deviation Exit Local Extrema', group = '⇧ Exit')
ctrLongExitSrc = exiu.str_to_src(input.string(defval = 'low', title = 'Long/Short Exit Control', options = ['open', 'high', 'low', 'close', 'hl2', 'hlc3', 'ohlc4', 'hlcc4'], inline = 'Control', group = '⇧ Exit'))
ctrShortExitSrc = exiu.str_to_src(input.string(defval = 'high', title = '', options = ['open', 'high', 'low', 'close', 'hl2', 'hlc3', 'ohlc4', 'hlcc4'], tooltip = 'The price source to check with the entry target to trigger the entry order for long/short position.', inline = 'Control', group = '⇧ Exit'))

// LOGIC ============================================================================================================
bool exitLongIsPending = barsSinceEnterLong > barsSinceCloseLong and strategy.position_size > 0 // enter -> close
bool tryExitLongPosition = exitLongIsPending

float exitLowestLow = ta.lowest(low, devExitLen)
float closeExitLowestLow = ta.valuewhen(validCloseLongPosition, exitLowestLow, 0)

getLongExitCloseBaseScr() =>
    switch devExitMethod
        'PERC' => closeClose
        'ATR' => closeClose
        'LOC' => closeExitLowestLow
        => na

getLongExitTrailingBaseScr() =>
    switch devExitMethod
        'PERC' => high
        'ATR' => high
        'LOC' => exitLowestLow
        => na

getLongExitPrice(baseSrc) =>
    switch devExitMethod
        'PERC' => baseSrc * (1 - devExitPerc)
        'ATR' => baseSrc - devExitAtrMul * getLongAtr()
        'LOC' => baseSrc - devExitTicks * syminfo.mintick
        => na

float longExitPrice = na
bool isFirstValidCloseLongPosition = validCloseLongPosition and na(longExitPrice[1])
longExitPrice := if isFirstValidCloseLongPosition
    getLongExitPrice(getLongExitCloseBaseScr())
else if tryExitLongPosition
    math.max(getLongExitPrice(getLongExitTrailingBaseScr()), nz(longExitPrice[1], 999999))
else
    na

bool exitLongPosition = enableExitTrailing ? timeFilterApproval and ta.crossunder(isFirstValidCloseLongPosition ? close : ctrLongExitSrc, longExitPrice) : validCloseLongPosition

bool longIsActive = enterLongPosition or strategy.position_size > 0 and not exitLongPosition

bool exitShortIsPending = barsSinceEnterShort > barsSinceCloseShort and strategy.position_size < 0 // enter -> close
bool tryExitShortPosition = exitShortIsPending

float exitHighestHigh = ta.highest(high, devExitLen)
float closeExitHighestHigh = ta.valuewhen(validCloseShortPosition, exitHighestHigh, 0)

getShortExitCloseBaseScr() =>
    switch devExitMethod
        'PERC' => closeClose
        'ATR' => closeClose
        'LOC' => closeExitHighestHigh
        => na

getShortExitTrailingBaseScr() =>
    switch devExitMethod
        'PERC' => low
        'ATR' => low
        'LOC' => exitHighestHigh
        => na

getShortExitPrice(baseSrc) =>
    switch devExitMethod
        'PERC' => baseSrc * (1 + devExitPerc)
        'ATR' => baseSrc + devExitAtrMul * getShortAtr()
        'LOC' => baseSrc + devExitTicks * syminfo.mintick
        => na

float shortExitPrice = na
bool isFirstValidCloseShortPosition = validCloseShortPosition and na(shortExitPrice[1])
shortExitPrice := if isFirstValidCloseShortPosition
    getShortExitPrice(getShortExitCloseBaseScr())
else if tryExitShortPosition
    math.min(getShortExitPrice(getShortExitTrailingBaseScr()), nz(shortExitPrice[1], 999999))
else
    na

bool exitShortPosition = enableExitTrailing ? timeFilterApproval and ta.crossover(isFirstValidCloseShortPosition ? close : ctrShortExitSrc, shortExitPrice) : validCloseShortPosition

bool shortIsActive = enterShortPosition or strategy.position_size < 0 and not exitShortPosition

// PLOT =============================================================================================================
var sellColor = color.new(color.red, 0)
plot(series = enableExitTrailing ? longExitPrice : na, title = 'Long Sell Price', color = sellColor, linewidth = 1, style = plot.style_linebr)
plot(series = enableExitTrailing ? shortExitPrice : na, title = 'Short Sell Price', color = sellColor, linewidth = 1, style = plot.style_linebr)

//
// ▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒
// ☭ SHARED VARIABLES 2 =============================================================================================
// Description: Module responsible for variable that needed both for STOP LOSS and TAKE PROFIT modules
// Dependencies: SHARED VARIABLES 1
// Results: ALL

// INPUT ============================================================================================================
numOfTakeProfitTargets = input.int(defval = 1, title = 'Take Profit Targets', minval = 0, tooltip = 'The number of take profit targets to be set for each entry. The first target is the initial target and every additional target is a step target.', group = '🟢 Take Profit')

// LOGIC ============================================================================================================
// take profit has to communicate the execution of take profit targets with the stop loss logic when 'TP' mode is selected
var longTrailingTakeProfitExecuted = array.new<bool>(numOfTakeProfitTargets, false)
var shortTrailingTakeProfitExecuted = array.new<bool>(numOfTakeProfitTargets, false)

//
// ▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒
// 🔴 STOP LOSS =====================================================================================================
// Description: Module responsible for the stop loss logic implementation based on the method and the trailing mode
// Dependencies: SHARED VARIABLES 2, EXIT
// Results: longTrailingStopLossPrice, shortTrailingStopLossPrice

// INPUT ============================================================================================================
stopLossMethod = input.string(defval = 'PERC', title = 'Stop Loss Method', options = ['PERC', 'ATR', 'LOC'], tooltip = 'The method to calculate the stop loss (percentagewise, ATR based or based on local extrema).', group = '🔴 Stop Loss')
longTrailingStopLossPerc = input.float(defval = 7.5, title = '  Long/Short Stop Loss %', minval = 0.05, maxval = 100, step = 0.05, inline = 'Stop Loss Perc', group = '🔴 Stop Loss') / 100
shortTrailingStopLossPerc = input.float(defval = 7.5, title = '', minval = 0.05, maxval = 100, step = 0.05, tooltip = 'The percentage of the price decrease/increase to set the stop loss price target for long/short positions.', inline = 'Stop Loss Perc', group = '🔴 Stop Loss') / 100
longStopLossAtrMul = input.float(defval = 3.0, title = '  ATR Long/Short Mul  ', minval = 0.1, step = 0.1, inline = 'Stop Loss ATR Multiplier', group = '🔴 Stop Loss')
shortStopLossAtrMul = input.float(defval = 3.0, title = '', minval = 0.1, step = 0.1, tooltip = 'ATR multiplier to be used for the long/short stop loss.', inline = 'Stop Loss ATR Multiplier', group = '🔴 Stop Loss')
stopLossLocLen = input.int(defval = 5, title = '  Loc Extrema Len/Ticks  ', minval = 1, inline = 'Stop Loss Local Extrema', group = '🔴 Stop Loss')
stopLossLocTicks = input.int(defval = 2, title = '', minval = 0, tooltip = 'Local extrema (minimum/maximum) within a window of length minus/plus some ticks to be used for the long/short stop loss.', inline = 'Stop Loss Local Extrema', group = '🔴 Stop Loss')
useSecPercMul = input.bool(defval = false, title = 'Use Sec %/Mul', tooltip = 'After the take profit price target is reached, use the "Sec Stop Loss %" or "Sec ATR Mul".', group = '🔴 Stop Loss')
longTrailingStopLossPercSec = input.float(defval = 10.0, title = '  Sec Long/Short Stop Loss %', minval = 0.05, maxval = 100, step = 0.05, inline = 'Stop Loss Perc Sec', group = '🔴 Stop Loss') / 100
shortTrailingStopLossPercSec = input.float(defval = 10.0, title = '', minval = 0.05, maxval = 100, step = 0.05, tooltip = 'The secondary percentage of the price decrease/increase to set the stop loss price target for long/short positions after the take profit target is reached.', inline = 'Stop Loss Perc Sec', group = '🔴 Stop Loss') / 100
longStopLossAtrMulSec = input.float(defval = 4.0, title = '  Sec ATR Long/Short Mul  ', minval = 0.1, step = 0.1, inline = 'Stop Loss ATR Multiplier Sec', group = '🔴 Stop Loss')
shortStopLossAtrMulSec = input.float(defval = 4.0, title = '', minval = 0.1, step = 0.1, tooltip = 'Secondary ATR multiplier to be used for the long/short Stop Loss after the take profit target is reached.', inline = 'Stop Loss ATR Multiplier Sec', group = '🔴 Stop Loss')
stopLossLocLenSec = input.int(defval = 9, title = '  Sec Loc Extrema Len/Ticks', minval = 1, inline = 'Stop Loss Local Extrema Sec', group = '🔴 Stop Loss')
stopLossLocTicksSec = input.int(defval = 5, title = '', minval = 0, tooltip = 'Secondary Local extrema (minimum/maximum) within a window of length minus/plus some ticks to be used for the long/short stop loss after the take profit target is reached.', inline = 'Stop Loss Local Extrema Sec', group = '🔴 Stop Loss')
breakEvenEnabled = input.bool(defval = false, title = 'Break Even', tooltip = 'When take profit price target is reached, move the stop loss to the entry price (or to a more strict price defined by the "Stop Loss %" or "ATR Mul").', group = '🔴 Stop Loss')

stopLossTrailingMode = input.string(defval = 'TP', title = 'Trailing Mode', options = ['TP', 'ON', 'OFF'], tooltip = 'Enable the trailing for stop loss when initial take profit order is executed (TP) or from the start of the entry order (ON) or not at all (OFF).', group = '🔴 Stop Loss')

// LOGIC ============================================================================================================
longInitTrailingTakeProfitExecuted = array.size(longTrailingTakeProfitExecuted) > 0 ? array.get(longTrailingTakeProfitExecuted, 0) : false

float entryClose = ta.valuewhen(enterLongPosition or enterShortPosition, close, 0)

float stopLossLowestLow = ta.lowest(low, (useSecPercMul and longInitTrailingTakeProfitExecuted ? stopLossLocLenSec : stopLossLocLen))
float entryStopLossLowestLow = ta.valuewhen(enterLongPosition, stopLossLowestLow, 0)

getLongStopLossEntryBaseScr() =>
    switch stopLossMethod
        'PERC' => entryClose
        'ATR' => entryClose
        'LOC' => entryStopLossLowestLow
        => na

float entryHigh = ta.valuewhen(enterLongPosition, high, 0)
getLongStopLossTrailingBaseScr() =>
    switch stopLossMethod
        'PERC' => high
        'ATR' => high
        'LOC' => stopLossLowestLow
        => na

getLongStopLossPrice(baseSrc) =>
    switch stopLossMethod
        'PERC' => baseSrc * (1 - (useSecPercMul and longInitTrailingTakeProfitExecuted ? longTrailingStopLossPercSec : longTrailingStopLossPerc))
        'ATR' => baseSrc - (useSecPercMul and longInitTrailingTakeProfitExecuted ? longStopLossAtrMulSec : longStopLossAtrMul) * getLongAtr()
        'LOC' => baseSrc - (useSecPercMul and longInitTrailingTakeProfitExecuted ? stopLossLocTicksSec : stopLossLocTicks) * syminfo.mintick
        => na

getLongStopLossPerc() =>
    (close - getLongStopLossPrice(getLongStopLossEntryBaseScr())) / close

// trailing starts when the take profit price is reached if 'TP' mode is set or from the very begining if 'ON' mode is selected
bool enableLongStopLossTrailing = stopLossTrailingMode == 'ON' or (stopLossTrailingMode == 'TP' and longInitTrailingTakeProfitExecuted)

// calculate trailing stop loss price when enter long position and peserve its value until the position closes
float longTrailingStopLossPrice = na
longTrailingStopLossPrice := if longIsActive
    if enterLongPosition
        getLongStopLossPrice(getLongStopLossEntryBaseScr())
    else
        stopPrice = getLongStopLossPrice(enableLongStopLossTrailing ? getLongStopLossTrailingBaseScr() : getLongStopLossEntryBaseScr())
        stopPrice := breakEvenEnabled and longInitTrailingTakeProfitExecuted ? math.max(stopPrice, strategy.opentrades.entry_price(0)) : stopPrice
        math.max(stopPrice, nz(longTrailingStopLossPrice[1]))
else
    na

shortInitTrailingTakeProfitExecuted = array.size(shortTrailingTakeProfitExecuted) > 0 ? array.get(shortTrailingTakeProfitExecuted, 0) : false

float stopLossHighestHigh = ta.highest(high, (useSecPercMul and shortInitTrailingTakeProfitExecuted ? stopLossLocLenSec : stopLossLocLen))
float entryStopLossHighestHigh = ta.valuewhen(enterShortPosition, stopLossHighestHigh, 0)

getShortStopLossEntryBaseScr() =>
    switch stopLossMethod
        'PERC' => entryClose
        'ATR' => entryClose
        'LOC' => entryStopLossHighestHigh

getShortStopLossTrailingBaseScr() =>
    switch stopLossMethod
        'PERC' => low
        'ATR' => low
        'LOC' => stopLossHighestHigh

getShortStopLossPrice(baseSrc) =>
    switch stopLossMethod
        'PERC' => baseSrc * (1 + (useSecPercMul and shortInitTrailingTakeProfitExecuted ? shortTrailingStopLossPercSec : shortTrailingStopLossPerc))
        'ATR' => baseSrc + (useSecPercMul and shortInitTrailingTakeProfitExecuted ? shortStopLossAtrMulSec : shortStopLossAtrMul) * getShortAtr()
        'LOC' => baseSrc + (useSecPercMul and longInitTrailingTakeProfitExecuted ? stopLossLocTicksSec : stopLossLocTicks) * syminfo.mintick
        => na

getShortStopLossPerc() =>
    (getShortStopLossPrice(getShortStopLossEntryBaseScr()) - close) / close

// trailing starts when the take profit price is reached if 'TP' mode is set or from the very begining if 'ON' mode is selected
bool enableShortStopLossTrailing = stopLossTrailingMode == 'ON' or (stopLossTrailingMode == 'TP' and shortInitTrailingTakeProfitExecuted)

// calculate trailing stop loss price when enter short position and peserve its value until the position closes
float shortTrailingStopLossPrice = na
shortTrailingStopLossPrice := if shortIsActive
    if enterShortPosition
        getShortStopLossPrice(getShortStopLossEntryBaseScr())
    else
        stopPrice = getShortStopLossPrice(enableShortStopLossTrailing ? getShortStopLossTrailingBaseScr() : getShortStopLossEntryBaseScr())
        stopPrice := breakEvenEnabled and shortInitTrailingTakeProfitExecuted ? math.min(stopPrice, strategy.opentrades.entry_price(0)) : stopPrice
        math.min(stopPrice, nz(shortTrailingStopLossPrice[1], 999999.9))
else
    na

// PLOT =============================================================================================================
var stopLossColor = color.new(#e25141, 0)
plot(series = longTrailingStopLossPrice, title = 'Long Trail Stop', color = stopLossColor, linewidth = 1, style = plot.style_linebr, offset = 1)
plot(series = shortTrailingStopLossPrice, title = 'Short Trail Stop', color = stopLossColor, linewidth = 1, style = plot.style_linebr, offset = 1)

//
// ▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒
// 🟢 TAKE PROFIT ===================================================================================================
// Description: Module responsible for the take profit logic implementation based on the method and the number of step take profit targets and the trailing deviation
// Dependencies: SHARED VARIABLES 1, SHARED VARIABLES 2, EXIT
// Results: longInitTakeProfitPrice, shortInitTakeProfitPrice, longInitTrailingTakeProfitExecuted, shortInitTrailingTakeProfitExecuted

// INPUT ============================================================================================================
initTakeProfitMethod = input.string(defval = 'PERC', title = 'Init Take Profit Method', options = ['PERC', 'ATR', 'RR', 'LOC'], tooltip = 'The method to calculate the initial take profit price target.', group = '🟢 Take Profit')
longInitTakeProfitPerc = input.float(defval = 10.0, title = '  Long/Short Init Take Profit %', minval = 0.05, step = 0.05, inline = 'Init Take Profit Perc', group = '🟢 Take Profit') / 100
shortInitTakeProfitPerc = input.float(defval = 10.0, title = '', minval = 0.05, step = 0.05, tooltip = 'The percentage of the price increase/decrease to set the initial take profit price target for long/short positions.', inline = 'Init Take Profit Perc', group = '🟢 Take Profit') / 100
longInitTakeProfitAtrMul = input.float(defval = 6.0, title = '  Init ATR Long/Short Mul  ', minval = 0.1, step = 0.1, inline = 'Init Take Profit ATR Multiplier', group = '🟢 Take Profit')
shortInitTakeProfitAtrMul = input.float(defval = 6.0, title = '', minval = 0.1, step = 0.1, tooltip = 'ATR multiplier to be used for the long/short initial take profit target.', inline = 'Init Take Profit ATR Multiplier', group = '🟢 Take Profit')
longInitRiskRewardRatio = input.float(defval = 1.5, title = '  Init Long/Short RR Ratio  ', minval = 0.1, step = 0.1, inline = 'Init Risk Reward Ratio', group = '🟢 Take Profit')
shortInitRiskRewardRatio = input.float(defval = 1.5, title = '', minval = 0.1, step = 0.1, tooltip = 'The Risk/Reward Ratio to be used for the long/short initial take profit based on the initial stop loss price target.', inline = 'Init Risk Reward Ratio', group = '🟢 Take Profit')
takeProfitLocLen = input.int(defval = 14, title = '  Loc Extrema Len/Ticks  ', minval = 1, inline = 'Take Profit Local Extrema', group = '🟢 Take Profit')
takeProfitLocTicks = input.int(defval = 5, title = '', minval = 0, tooltip = 'Local extrema (minimum/maximum) within a window of length minus/plus some ticks to be used for the long/short take profit.', inline = 'Take Profit Local Extrema', group = '🟢 Take Profit')

stepTakeProfitMethod = input.string(defval = 'PERC', title = 'Step Take Profit Method', options = ['PERC', 'ATR', 'RR'], tooltip = 'The method to calculate the step take profit price targets.', group = '🟢 Take Profit')
longStepTakeProfitPerc = input.float(defval = 5.0, title = '  Long/Short Step Take Profit %', minval = 0.05, step = 0.05, inline = 'Step Take Profit Perc', group = '🟢 Take Profit') / 100
shortStepTakeProfitPerc = input.float(defval = 5.0, title = '', minval = 0.05, step = 0.05, tooltip = 'The percentage of the price increase/decrease to be added to the initial percentage for the step take profit price targets for long/short positions.', inline = 'Step Take Profit Perc', group = '🟢 Take Profit') / 100
longStepTakeProfitAtrMul = input.float(defval = 1.0, title = '  Step ATR Long/Short Mul  ', minval = 0.1, step = 0.1, inline = 'Step Take Profit ATR Multiplier', group = '🟢 Take Profit')
shortStepTakeProfitAtrMul = input.float(defval = 1.0, title = '', minval = 0.1, step = 0.1, tooltip = 'ATR multiplier to be added to the initial ATR multiplier for the long/short step take profit price targets.', inline = 'Step Take Profit ATR Multiplier', group = '🟢 Take Profit')
longStepRiskRewardRatio = input.float(defval = 1.0, title = '  Step Long/Short RR Ratio  ', minval = 0.1, step = 0.1, inline = 'Step Risk Reward Ratio', group = '🟢 Take Profit')
shortStepRiskRewardRatio = input.float(defval = 1.0, title = '', minval = 0.1, step = 0.1, tooltip = 'The Risk/Reward Ratio to be added to the initial Risk/Reward Ratio for the long/short step take profit price targets based on the initial stop loss price target.', inline = 'Step Risk Reward Ratio', group = '🟢 Take Profit')

enableTakeProfitTrailing = input.bool(defval = false, title = 'Enable Trailing', tooltip = 'Enable or disable the trailing for take profit. WARNING! This feature will repaint.', group = '🟢 Take Profit')
devTakeProfitMethod = input.string(defval = 'PERC', title = '  Deviation Method', options = ['PERC', 'ATR'], tooltip = 'The method to calculate the deviation for the trailing take profit.', group = '🟢 Take Profit')
devTakeProfitPerc = input.float(defval = 1.0, title = '  Deviation %', minval = 0.01, maxval = 100, step = 0.05, tooltip = 'The percentage wise step to be used for following the price, when the take profit target is reached.', group = '🟢 Take Profit') / 100
devTakeProfitAtrMul = input.float(defval = 1.0, title = '  Deviation ATR Mul', minval = 0.01, step = 0.05, tooltip = 'Multiplier to be used on the ATR to calculate the step for following the price, when the take profit target is reached.', group = '🟢 Take Profit')

// LOGIC ============================================================================================================
float takeProfitHighestHigh = ta.highest(high, takeProfitLocLen)
float openTakeProfitHighestHigh = ta.valuewhen(validOpenLongPosition, takeProfitHighestHigh, 0)

getLongInitTakeProfitBaseScr() =>
    switch initTakeProfitMethod
        'PERC' => openClose
        'ATR' => openClose
        'RR' => openClose
        'LOC' => math.max(openTakeProfitHighestHigh, high)
        => na

getLongTakeProfitPrice(baseSrc, takeProfitMethod, takeProfitPerc, takeProfitAtrMul, riskRewardRatio) =>
    switch takeProfitMethod
        'PERC' => baseSrc * (1 + takeProfitPerc)
        'ATR' => baseSrc + takeProfitAtrMul * getLongAtr()
        'RR' => baseSrc + riskRewardRatio * (baseSrc - getLongStopLossPrice(getLongStopLossEntryBaseScr()))
        'LOC' => baseSrc - takeProfitLocTicks * syminfo.mintick
        => na

getLongTakeProfitPerc(baseSrc, takeProfitMethod, takeProfitPerc, takeProfitAtrMul, riskRewardRatio) =>
    (baseSrc - getLongTakeProfitPrice(baseSrc, takeProfitMethod, takeProfitPerc, takeProfitAtrMul, riskRewardRatio)) / baseSrc
    
getTrailingOffsetTicks(takeProfitPrice, atr) =>
    switch devTakeProfitMethod
        'PERC' => takeProfitPrice * devTakeProfitPerc / syminfo.mintick
        'ATR' => devTakeProfitAtrMul * atr / syminfo.mintick
        => na

// calculate take profit prices when enter long position and peserve their values until the entire position closes
var longTakeProfitPrices = array.new<float>(numOfTakeProfitTargets, na)
for [i, takeProfitPrice] in longTakeProfitPrices
    if longIsActive and not array.get(longTrailingTakeProfitExecuted, i)
        longTakeProfitPerc = i * longStepTakeProfitPerc
        longTakeProfitAtrMul = i * longStepTakeProfitAtrMul
        longRiskRewardRatio = i * longStepRiskRewardRatio
        if enterLongPosition
            array.set(longTakeProfitPrices, i, getLongTakeProfitPrice(getLongTakeProfitPrice(getLongInitTakeProfitBaseScr(), initTakeProfitMethod, longInitTakeProfitPerc, longInitTakeProfitAtrMul, longInitRiskRewardRatio), stepTakeProfitMethod, longTakeProfitPerc, longTakeProfitAtrMul, longRiskRewardRatio))
        else
            array.set(longTakeProfitPrices, i, nz(takeProfitPrice, getLongTakeProfitPrice(getLongTakeProfitPrice(getLongInitTakeProfitBaseScr(), initTakeProfitMethod, longInitTakeProfitPerc, longInitTakeProfitAtrMul, longInitRiskRewardRatio), stepTakeProfitMethod, longTakeProfitPerc, longTakeProfitAtrMul, longRiskRewardRatio)))
    else
        array.set(longTakeProfitPrices, i, na)

for [i, takeProfitPrice] in longTakeProfitPrices
    executed = strategy.position_size > 0 and (array.get(longTrailingTakeProfitExecuted, i) or ((strategy.position_size < strategy.position_size[1] or strategy.position_size[1] == 0) and high >= takeProfitPrice))
    array.set(longTrailingTakeProfitExecuted, i, executed)

var longTrailingTakeProfitOffsetTicks = array.new<float>(numOfTakeProfitTargets, na)
if (enterLongPosition)
    for [i, takeProfitPrice] in longTakeProfitPrices
        array.set(longTrailingTakeProfitOffsetTicks, i, getTrailingOffsetTicks(takeProfitPrice, getLongAtr()))

float takeProfitLowestLow = ta.lowest(low, takeProfitLocLen)
float openTakeProfitLowestLow = ta.valuewhen(validOpenShortPosition, takeProfitLowestLow, 0)

getShortInitTakeProfitBaseScr() =>
    switch initTakeProfitMethod
        'PERC' => openClose
        'ATR' => openClose
        'RR' => openClose
        'LOC' => math.min(openTakeProfitLowestLow, low)
        => na

getShortTakeProfitPrice(baseSrc, takeProfitMethod, takeProfitPerc, takeProfitAtrMul, riskRewardRatio) =>
    switch takeProfitMethod
        'PERC' => baseSrc * (1 - takeProfitPerc)
        'ATR' => baseSrc - takeProfitAtrMul * getShortAtr()
        'RR' => baseSrc - riskRewardRatio * (getShortStopLossPrice(getShortStopLossEntryBaseScr()) - baseSrc)
        'LOC' => baseSrc + takeProfitLocTicks * syminfo.mintick
        => na

getShortTakeProfitPerc(baseSrc, takeProfitMethod, shortTakeProfitPerc, shortTakeProfitAtrMul, shortRiskRewardRatio) =>
    (getShortTakeProfitPrice(baseSrc, takeProfitMethod, shortTakeProfitPerc, shortTakeProfitAtrMul, shortRiskRewardRatio) - baseSrc) / baseSrc

// calculate take profit prices when enter short position and peserve their values until the entire position closes
var shortTakeProfitPrices = array.new<float>(numOfTakeProfitTargets, na)
for [i, takeProfitPrice] in shortTakeProfitPrices
    if shortIsActive and not array.get(shortTrailingTakeProfitExecuted, i)
        shortTakeProfitPerc = i * shortStepTakeProfitPerc
        shortTakeProfitAtrMul = i * shortStepTakeProfitAtrMul
        shortRiskRewardRatio = i * shortStepRiskRewardRatio
        if enterShortPosition
            array.set(shortTakeProfitPrices, i, getShortTakeProfitPrice(getShortTakeProfitPrice(getShortInitTakeProfitBaseScr(), initTakeProfitMethod, shortInitTakeProfitPerc, shortInitTakeProfitAtrMul, shortInitRiskRewardRatio), stepTakeProfitMethod, shortTakeProfitPerc, shortTakeProfitAtrMul, shortRiskRewardRatio))
        else
            array.set(shortTakeProfitPrices, i, nz(takeProfitPrice, getShortTakeProfitPrice(getShortTakeProfitPrice(getShortInitTakeProfitBaseScr(), initTakeProfitMethod, shortInitTakeProfitPerc, shortInitTakeProfitAtrMul, shortInitRiskRewardRatio), stepTakeProfitMethod, shortTakeProfitPerc, shortTakeProfitAtrMul, shortRiskRewardRatio)))
    else
        array.set(shortTakeProfitPrices, i, na)

for [i, takeProfitPrice] in shortTakeProfitPrices
    executed = strategy.position_size < 0 and (array.get(shortTrailingTakeProfitExecuted, i) or ((strategy.position_size > strategy.position_size[1] or strategy.position_size[1] == 0) and low <= takeProfitPrice))
    array.set(shortTrailingTakeProfitExecuted, i, executed)

var shortTrailingTakeProfitOffsetTicks = array.new<float>(numOfTakeProfitTargets, na)
if (enterShortPosition)
    for [i, takeProfitPrice] in shortTakeProfitPrices
        array.set(shortTrailingTakeProfitOffsetTicks, i, getTrailingOffsetTicks(takeProfitPrice, getShortAtr()))

// PLOT =============================================================================================================
var takeProfitColor = color.new(#419388, 0)

// close price when the valid enter signal was triggered
int enterBar = ta.valuewhen(enterLongPosition or enterShortPosition, bar_index, 0)

updateLines(lines, prices) =>
    for [i, ln] in lines
        price = array.get(prices, i)
        if (not na(price))
            line.set_y1(ln, price)
            line.set_y2(ln, price)
            line.set_x2(ln, bar_index + 1)

moveAllElements(fromArr, toArr) =>
    len = array.size(fromArr) - 1
    for i = 0 to len >=0 ? len : na
        array.push(toArr, array.pop(fromArr))

var allLongTakeProfitLines = array.new<line>()
var currentLongTakeProfitLines = array.new<line>()

if (longIsActive)
    if (enterLongPosition)
        moveAllElements(currentLongTakeProfitLines, allLongTakeProfitLines)
        for [i, takeProfitPrice] in longTakeProfitPrices
            pa.push(currentLongTakeProfitLines, line.new(x1 = enterBar + 1, y1 = takeProfitPrice, x2 = bar_index + 1, y2 = takeProfitPrice, xloc = xloc.bar_index, extend = extend.none, color = color.from_gradient(i, 0, numOfTakeProfitTargets, takeProfitColor, color.lime), style = line.style_solid, width = 1), numOfTakeProfitTargets)
    updateLines(currentLongTakeProfitLines, longTakeProfitPrices)

var allShortTakeProfitLines = array.new<line>()
var currentShortTakeProfitLines = array.new<line>()

if (shortIsActive)
    if (enterShortPosition)
        moveAllElements(currentShortTakeProfitLines, allShortTakeProfitLines)
        for [i, takeProfitPrice] in shortTakeProfitPrices
            pa.push(currentShortTakeProfitLines, line.new(x1 = enterBar + 1, y1 = takeProfitPrice, x2 = bar_index + 1, y2 = takeProfitPrice, xloc = xloc.bar_index, extend = extend.none, color = color.from_gradient(i, 0, numOfTakeProfitTargets, takeProfitColor, color.lime), style = line.style_solid, width = 1), numOfTakeProfitTargets)
    updateLines(currentShortTakeProfitLines, shortTakeProfitPrices)

//
// ▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒
// 💰 QUANTITY/RISK MANAGEMENT ======================================================================================
// Description: Module responsible for the calculation of the quantity percentage that will be used on each entry
// Dependencies: SHARED VARIABLES 1, ENTRY, EXIT
// Results: longEntryBaseQuantity, shortEntryBaseQuantity

import jason5480/math_utils/2 as mu

// INPUT ============================================================================================================
takeProfitQuantityPerc = input.float(defval = 80, title = 'Take Profit Quantity %', minval = 0.0, maxval = 100, step = 1.0, tooltip = 'The percentage of the position that will be withdrawn when ALL the take profit price targets are reached. If more than one take profit target is set, then ALL targets will share equally this amount and exit accordingly.', group = '💰 Quantity/Risk Management') / 100

quantityMethod = input.string(defval = 'RISK', title = 'Quantity Method', options = ['RISK', 'EQUITY'], tooltip = 'The method to calculate the quantity to enter each new position.', group = '💰 Quantity/Risk Management')
riskPerc = input.float(defval = 2.0, title = '  Capital at Risk %', minval = 1, maxval = 100, tooltip = 'The maximum percentage of the equity to risk in every trade when no leverage is used.', group = '💰 Quantity/Risk Management') / 100
equityPerc = input.float(defval = 20.0, title = '  Equity %        ', minval = 1, maxval = 100, tooltip = 'The percentage of the equity to enter in every trade when no leverage is used.', group = '💰 Quantity/Risk Management') / 100
minTrade = input.int(defval = 10, title = 'Minimum Trade Price', minval = 1, tooltip = 'The minimum trade price in quote currency that is allowed in the exchange for a valid new position.', group = '💰 Quantity/Risk Management')
longLeverage = input.int(defval = 1, title = 'Leverage Long/Short ', minval = 1, inline = 'Leverage', group = '💰 Quantity/Risk Management')
shortLeverage = input.int(defval = 1, title = '', minval = 1, tooltip = 'Leverage factor used to multiply the initial risk quantity of each trade (by borrowing the remaining amount). Thus, the profits and losses are multiplied respectivelly.', inline = 'Leverage', group = '💰 Quantity/Risk Management')

// LOGIC ============================================================================================================
var int quoteDecimalDigits = mu.num_of_decimal_digits(syminfo.mintick * syminfo.pointvalue)

getLongRiskQuoteQuantity() =>
    mu.clamp(strategy.equity * riskPerc * longLeverage / getLongStopLossPerc(), minTrade, strategy.equity * longLeverage, quoteDecimalDigits)

getLongEquityQuoteQuantity() =>
    mu.clamp(strategy.equity * equityPerc * longLeverage, minTrade, strategy.equity * longLeverage, quoteDecimalDigits)

getLongQuoteQuantity() =>
    switch quantityMethod
        'RISK' => getLongRiskQuoteQuantity()
        'EQUITY' => getLongEquityQuoteQuantity()
        => na

getLongQuoteQuantityPerc() =>
    getLongQuoteQuantity() / strategy.equity

getLongBaseQuantity() =>
    getLongQuoteQuantity() / close

float longEntryBaseQuantity = na
longEntryBaseQuantity := if longIsActive
    if validOpenLongPosition
        getLongBaseQuantity()
    else
        nz(longEntryBaseQuantity[1], getLongBaseQuantity())
else
    na

getShortRiskQuoteQuantity() =>
    mu.clamp(strategy.equity * riskPerc * shortLeverage / getShortStopLossPerc(), minTrade, strategy.equity * shortLeverage, quoteDecimalDigits)

getShortEquityQuoteQuantity() =>
    mu.clamp(strategy.equity * equityPerc * shortLeverage, minTrade, strategy.equity * shortLeverage, quoteDecimalDigits)

getShortQuoteQuantity() =>
    switch quantityMethod
        'RISK' => getShortRiskQuoteQuantity()
        'EQUITY' => getShortEquityQuoteQuantity()
        => na

getShortQuoteQuantityPerc() =>
    getShortQuoteQuantity() / strategy.equity

getShortBaseQuantity() =>
    getShortQuoteQuantity() / close

float shortEntryBaseQuantity = na
shortEntryBaseQuantity := if shortIsActive
    if validOpenShortPosition
        getShortBaseQuantity()
    else
        nz(shortEntryBaseQuantity[1], getShortBaseQuantity())
else
    na

// PLOT =============================================================================================================
if (validOpenLongPosition)
    label.new(x = bar_index, y = na, text = 'Buy\n' + str.tostring(100 * getLongQuoteQuantityPerc(), '#.##') + '%', yloc = yloc.belowbar, color = buyColor, style = label.style_label_up, textcolor = color.new(color.white, 0))
if (validOpenShortPosition)
    label.new(x = bar_index, y = na, text = 'Sell\n' + str.tostring(100 * getShortQuoteQuantityPerc(), '#.##') + '%', yloc = yloc.abovebar, color = sellColor, style = label.style_label_down, textcolor = color.new(color.white, 0))
if (validCloseShortPosition)    
    label.new(x = bar_index, y = na, text = 'Buy', yloc = yloc.belowbar, color = buyColor, style = label.style_label_up, textcolor = color.new(color.white, 0))
if (validCloseLongPosition)
    label.new(x = bar_index, y = na, text = 'Sell', yloc = yloc.abovebar, color = sellColor, style = label.style_label_down, textcolor = color.new(color.white, 0))

//
// ▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒
// 🔔 ALERT MESSAGES ================================================================================================
// Description: Module responsible for the message outputs when strategy orders are being executed
// Dependencies: NONE
// Results: ALL

// INPUT ============================================================================================================
msgOpenLong = input.text_area(defval = 'Long: Started', title = 'Open Long', tooltip = 'Alert messages emited when open long position.', group = '🔔 Alert Messages')
msgOpenShort = input.text_area(defval = 'Short: Started', title = 'Open Short', tooltip = 'Alert messages emited when open short position.', group = '🔔 Alert Messages')
msgCloseLong = input.text_area(defval = 'Long: Closed at market price', title = 'Close Long', tooltip = 'Alert messages emited when close long position.', group = '🔔 Alert Messages')
msgCloseShort = input.text_area(defval = 'Short: Closed at market price', title = 'Close Short', tooltip = 'Alert messages emited when close short position.', group = '🔔 Alert Messages')
msgTPSLLong = input.text_area(defval = 'Long: Take Profit or Stop Loss executed', title = 'TP/SL Long', tooltip = 'Alert message emited when the first quantity target (take profit or stop loss) for long position is reached.', group = '🔔 Alert Messages')
msgTPSLShort = input.text_area(defval = 'Short: Take Profit or Stop Loss executed', title = 'TP/SL Short', tooltip = 'Alert message emited when the first quantity target (take profit or stop loss) for short position is reached.', group = '🔔 Alert Messages')
msgSLLong = input.text_area(defval = 'Long: Stop Loss executed', title = 'SL Long', tooltip = 'Alert message emited when the second quantity stop loss target for long position is reached.', group = '🔔 Alert Messages')
msgSLShort = input.text_area(defval = 'Short: Stop Loss executed', title = 'SL Short', tooltip = 'Alert message emited when the second quantity stop loss target for short position is reached.', group = '🔔 Alert Messages')
msgCloseAll = input.text_area(defval = 'Closed all positions at market price', title = 'Close All', tooltip = 'Alert messages emited when close all positions.', group = '🔔 Alert Messages')
msgMaxDrawdown= input.text_area(defval = 'Death is the new beginning', title = 'Max Drawdown', tooltip = 'Alert message emited when the max drawdown limit is reached.', group = '🔔 Alert Messages')

//
// ▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒
// 🗲 POSITION ORDERS ===============================================================================================
// Description: Module responsible for the actual execution of the strategy orders
// Dependencies: STRATEGY, ENTRY, EXIT, STOP LOSS, TAKE PROFIT, QUANTITY MANAGEMENT, ALERT MESSAGES
// Results: NONE

// INPUT ============================================================================================================
maxDrawdown = input.int(defval = 25, title = 'Max Drawdown %', minval = 1, maxval = 100, tooltip = 'The maximum drawdown to stop trading.', group = '💰 Quantity/Risk Management')

highlighting = input.bool(defval = false, title = 'Show Position Highlighter', tooltip = 'Highlight winning/lossing position.', group = '📊 Plot')

// LOGIC ============================================================================================================
// close all positions at the end of the session
strategy.close_all(when = closeAtSessionEnd and not timeFilterApproval, comment = 'Close All', alert_message = msgCloseAll)

// close on trend reversal
strategy.close(id = 'Long Entry', when = exitLongPosition, comment = 'Close Long', alert_message = msgCloseLong)

// close on trend reversal
strategy.close(id = 'Short Entry', when = exitShortPosition, comment = 'Close Short', alert_message = msgCloseShort)

// getting into LONG position
strategy.entry(id = 'Long Entry', direction = strategy.long, qty = longEntryBaseQuantity, when = enterLongPosition, alert_message = msgOpenLong)
// submit exit order for trailing take profit price also set the stop loss for the take profit percentage in case that stop loss is reached first
for [i, longTakeProfitPrice] in longTakeProfitPrices
    strategy.exit(id = 'Long Take Profit ' + str.tostring(i + 1) + ' / Stop Loss', from_entry = 'Long Entry', qty = longEntryBaseQuantity * takeProfitQuantityPerc / numOfTakeProfitTargets, limit = enableTakeProfitTrailing ? na : longTakeProfitPrice, stop = longTrailingStopLossPrice, trail_price = enableTakeProfitTrailing ? longTakeProfitPrice : na, trail_offset = enableTakeProfitTrailing ? array.get(longTrailingTakeProfitOffsetTicks, i) : na, when = longIsActive, alert_message = msgTPSLLong)
// submit exit order for trailing stop loss price for the remaining percent of the quantity not reserved by the take profit order
strategy.exit(id = 'Long Stop Loss', from_entry = 'Long Entry', stop = longTrailingStopLossPrice, when = longIsActive, alert_message = msgSLLong)

// getting into SHORT position
strategy.entry(id = 'Short Entry', direction = strategy.short, qty = shortEntryBaseQuantity, when = enterShortPosition, alert_message = msgOpenShort)
// submit exit order for trailing take profit price also set the stop loss for the take profit percentage in case that stop loss is reached first
for [i, shortTakeProfitPrice] in shortTakeProfitPrices
    strategy.exit(id = 'Short Take Profit ' + str.tostring(i + 1) + ' / Stop Loss', from_entry = 'Short Entry', qty = shortEntryBaseQuantity * takeProfitQuantityPerc / numOfTakeProfitTargets, limit = enableTakeProfitTrailing ? na : shortTakeProfitPrice, stop = shortTrailingStopLossPrice, trail_price = enableTakeProfitTrailing ? shortTakeProfitPrice : na, trail_offset = enableTakeProfitTrailing ? array.get(shortTrailingTakeProfitOffsetTicks, i) : na, when = shortIsActive, alert_message = msgTPSLShort)
// submit exit order for trailing stop loss price for the remaining percent of the quantity not reserved by the take profit order
strategy.exit(id = 'Short Stop Loss', from_entry = 'Short Entry', stop = shortTrailingStopLossPrice, when = shortIsActive, alert_message = msgSLShort)

// limit the maximum drawdown
strategy.risk.max_drawdown(value = maxDrawdown, type = strategy.percent_of_equity, alert_message = msgMaxDrawdown)

// PLOT =============================================================================================================
lowHighPrice = high > strategy.position_avg_price and low < strategy.position_avg_price ? longIsActive ? high : shortIsActive ? low : na
             : high > strategy.position_avg_price ? high
             : low < strategy.position_avg_price ? low
             : na

pricePlot = plot(series = lowHighPrice, title = 'Price', color = na, linewidth = 1, style = plot.style_linebr)
var posColor = color.new(color.white, 0)
posPlot = plot(series = strategy.position_avg_price, title = 'Position', color = posColor, linewidth = 1, style = plot.style_linebr)

highlightColor = lowHighPrice > strategy.position_avg_price and longIsActive or lowHighPrice < strategy.position_avg_price and shortIsActive ? takeProfitColor
               : lowHighPrice < strategy.position_avg_price and longIsActive or lowHighPrice > strategy.position_avg_price and shortIsActive ? stopLossColor
               : na

fill(plot1 = posPlot, plot2 = pricePlot, color = highlighting ? color.new(highlightColor, 90) : na, title = 'Highlight trades')

// ==================================================================================================================
