//@version=6
indicator("Gold ICT Pro v0.1.0", shorttitle="Gold ICT Pro", overlay=true, max_labels_count=500, max_lines_count=500)

//====================================================
// VERSION
//====================================================
var string VERSION = "v0.1.0"

//====================================================
// INPUTS
//====================================================
groupGeneral = "General Settings"

showTrend = input.bool(true, "Show 1H Trend", group=groupGeneral)
showSessions = input.bool(true, "Show Sessions", group=groupGeneral)
showInfo = input.bool(true, "Show Information Table", group=groupGeneral)

//====================================================
// GOLD CHECK
//====================================================
isGold =
     str.contains(syminfo.ticker, "XAU") or
     str.contains(syminfo.ticker, "GOLD")

bgcolor(not isGold ? color.new(color.red, 85) : na)

if not isGold
    label.new(
         bar_index,
         high,
         "⚠ This indicator is designed for XAUUSD only.",
         style=label.style_label_down,
         color=color.red,
         textcolor=color.white)

//====================================================
// MULTI TIMEFRAME DATA
//====================================================
htfClose = request.security(syminfo.tickerid, "60", close)
htfEMA50 = request.security(syminfo.tickerid, "60", ta.ema(close, 50))
htfEMA200 = request.security(syminfo.tickerid, "60", ta.ema(close, 200))

//====================================================
// TREND ENGINE (Temporary)
//====================================================
trend =
     htfEMA50 > htfEMA200 ? 1 :
     htfEMA50 < htfEMA200 ? -1 : 0

trendText =
     trend == 1 ? "Bullish" :
     trend == -1 ? "Bearish" :
     "Neutral"

trendColor =
     trend == 1 ? color.lime :
     trend == -1 ? color.red :
     color.orange

//====================================================
// SESSION FILTER
//====================================================
londonSession = not na(time(timeframe.period, "0800-1100"))
newYorkSession = not na(time(timeframe.period, "1330-1630"))

sessionText =
     londonSession ? "London" :
     newYorkSession ? "New York" :
     "Off Session"

//====================================================
// INFO TABLE
//====================================================
var table info = table.new(position.top_right, 2, 5, border_width=1)

if barstate.islast and showInfo
    table.cell(info, 0, 0, "Gold ICT Pro")
    table.cell(info, 1, 0, VERSION)

    table.cell(info, 0, 1, "Trend")
    table.cell(info, 1, 1, trendText, text_color=trendColor)

    table.cell(info, 0, 2, "Session")
    table.cell(info, 1, 2, sessionText)

    table.cell(info, 0, 3, "Symbol")
    table.cell(info, 1, 3, syminfo.ticker)

    table.cell(info, 0, 4, "Status")
    table.cell(info, 1, 4, isGold ? "Ready" : "Wrong Symbol")

//====================================================
// PLOTS
//====================================================
plot(showTrend ? htfEMA50 : na, title="HTF EMA 50")
plot(showTrend ? htfEMA200 : na, title="HTF EMA 200")
