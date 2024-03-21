// This source code is subject to the terms of the Mozilla Public License 2.0 at https://mozilla.org/MPL/2.0/
// © Munkhtur
// @version=5
//
indicator("Session High Low", shorttitle="Session" ,overlay=true)
//
timezone                 = input.string  (defval = "Europe/London", title = "Timezone", options = ["GMT", "Etc/UTC", "Asia/Tokyo", "Europe/London", "America/New_York"], inline = "settings 1", group = "Indicator settings")
timeframe                = input.int     (15,    title = "Timeframe",  inline = "settings", group = "Indicator settings")
hide_history             = input.bool    (defval = false, title = "Hide History", inline = "settings 1", group = "Indicator settings")
show_price               = input.bool    (defval = true,  title = "Show price",   inline = "settings", group = "Indicator settings")
show_timeframe           =               (timeframe.multiplier <= timeframe) and  (timeframe.isintraday)
//
Session_time             = input.session (title = "Session time", defval = "0800-1330", inline = "Session time", group = "Session settings")
Session_start            = time          (timeframe.period, Session_time, timezone)
Session_start_hour       = input.int     (title = "Hour",          defval = 08, minval = 0, maxval = 23, inline = "Session start", group = "Session settings")
Session_start_minute     = input.int     (title = "Minute",        defval = 00, minval = 0, maxval = 59, inline = "Session start", group = "Session settings")
Session_end_hour         = input.int     (title = "Hour",          defval = 16, minval = 0, maxval = 23, inline = "Session end",   group = "Session settings")
Session_end_minute       = input.int     (title = "Minute",        defval = 59, minval = 0, maxval = 59, inline = "Session end",   group = "Session settings")
Extend_line_hour         = input.int     (title = "Line extend hour",defval = 23, minval = 0, maxval = 23, inline = "line extend",  group = "High Low settings")
Extend_line_minute       = input.int     (title = "Minute",          defval = 59, minval = 0, maxval = 59, inline = "line extend",  group = "High Low settings")
Session_start_timestamp  = timestamp     (timezone, year, month, dayofmonth, Session_start_hour, Session_start_minute, 00)
Session_end_timestamp    = timestamp     (timezone, year, month, dayofmonth, Session_end_hour,   Session_end_minute,   00)
Extend_line_timestamp    = timestamp     (timezone, year, month, dayofmonth, Extend_line_hour,   Extend_line_minute,   00)

Session_start_vertical   = input.bool    (defval=true, title="Start", inline = "Session start",    group = "Session settings")
Session_end_vertical     = input.bool    (defval=true, title="End",   inline = "Session end",      group = "Session settings")
//
var float Session_high   = na
var float Session_low    = na
Session_high_color       = input.color   (title = "Session high",      defval = color.blue, inline = "line extend 1",   group = "High Low settings")
Session_low_color        = input.color   (title = "Session low",       defval = color.red,  inline = "line extend 1",   group = "High Low settings")
Vertical_start_color     = input.color   (title = "Line color",      defval = color.blue,   inline = "Session start",   group = "Session settings")
Vertical_end_color       = input.color   (title = "Line color",      defval = color.red,    inline = "Session end",     group = "Session settings")
var Session_high_line    = line.new      (x1 = na, y1 = na, x2 = na, xloc = xloc.bar_time, y2 = close, color = Session_high_color)
var Session_low_line     = line.new      (x1 = na, y1 = na, x2 = na, xloc = xloc.bar_time, y2 = close, color = Session_low_color )
var Session_high_price   = label.new     (x  = na, y  = na,          xloc = xloc.bar_time, textcolor = Session_high_color, style = label.style_label_left, size = size.small)
var Session_low_price    = label.new     (x  = na, y  = na,          xloc = xloc.bar_time, textcolor = Session_low_color,  style = label.style_label_left, size = size.small)
var Vertical_start       = line.new      (x1 = na, y1 = na, x2 = na, xloc = xloc.bar_time, y2 = close, color = Vertical_start_color)                                                        // Asian Killzone Start Vertical Line
var Vertical_end         = line.new      (x1 = na, y1 = na, x2 = na, xloc = xloc.bar_time, y2 = close, color = Vertical_end_color  )
var Session_start_time   = time
new_day                  = ta.change(dayofweek)       
//
Vertical_line_start (start, color, style, width) =>
    line.new(x1 = start, y1 = low - ta.tr, x2 = start, y2 = high + ta.tr, xloc = xloc.bar_time, extend = extend.both, color = Vertical_start_color, style = line.style_dotted, width = 1)
Vertical_line_end   (start, color, style, width) =>
    line.new(x1 = start, y1 = low - ta.tr, x2 = start, y2 = high + ta.tr, xloc = xloc.bar_time, extend = extend.both, color = Vertical_end_color, style = line.style_dotted, width = 1)
Session_start(sess) =>
    t = time("", sess , timezone)
    show_timeframe and (not barstate.isfirst) and na(t[1]) and not na(t)
//
if Session_start(Session_time) and show_timeframe
    Session_start_timestamp := time
    Session_high            := high
    Session_low             := low
    if Extend_line_timestamp < Session_start_timestamp
        Extend_line_timestamp := Extend_line_timestamp + 86400000
	if hide_history
		line.delete   (Session_high_line   [1])
		line.delete   (Session_low_line    [1])
		label.delete  (Session_high_price  [1])
		label.delete  (Session_low_price   [1])
	Session_high_line  := line.new  (x1=Session_start_timestamp, y1=Session_high, x2=Extend_line_timestamp, xloc=xloc.bar_time, y2=Session_high, color=Session_high_color, width=1, style=line.style_solid)
	Session_low_line   := line.new  (x1=Session_start_timestamp, y1=Session_low,  x2=Extend_line_timestamp, xloc=xloc.bar_time, y2=Session_low,  color=Session_low_color,  width=1, style=line.style_solid)
	Session_high_price := label.new (x= Session_start_timestamp, y =Session_high, xloc=xloc.bar_time, color=color.new(color.gray, 100), textcolor=Session_high_color, style=label.style_label_left, size=size.small)
	Session_low_price  := label.new (x= Session_start_timestamp, y =Session_low,  xloc=xloc.bar_time, color=color.new(color.gray, 100), textcolor=Session_low_color,  style=label.style_label_left, size=size.small)
	label.set_text     (Session_high_price, str.tostring(Session_high))
	label.set_text     (Session_low_price,  str.tostring(Session_low ))
//
else if (time > Session_start_timestamp and time < Session_end_timestamp)
    if Extend_line_timestamp < Session_start_timestamp
        Extend_line_timestamp := Extend_line_timestamp + 86400000
    Session_high := math.max(high, Session_high)
    Session_low  := math.min(low,  Session_low )
    if (Session_high > Session_high[1])
        line.set_xy1          (id = Session_high_line,  x = time,                  y = Session_high)
        line.set_xy2          (id = Session_high_line,  x = Extend_line_timestamp, y = Session_high)
        if show_price
            label.set_xy      (id = Session_high_price, x = Extend_line_timestamp, y = Session_high)
            label.set_xy      (id = Session_low_price,   x = Extend_line_timestamp, y = Session_low )
            label.set_text    (Session_high_price, str.tostring (Session_high))
            label.set_text    (Session_low_price,  str.tostring (Session_low ))    
        if not show_price
            label.delete  (Session_low_price)
            label.delete  (Session_high_price)
    if (Session_low  < Session_low [1])
        line.set_xy1          (id = Session_low_line,    x = time,                  y = Session_low )
        line.set_xy2          (id = Session_low_line,    x = Extend_line_timestamp, y = Session_low )
        if show_price
            label.set_xy      (id = Session_high_price,   x = Extend_line_timestamp, y = Session_high)
            label.set_xy      (id = Session_low_price,   x = Extend_line_timestamp, y = Session_low )
            label.set_text    (Session_high_price, str.tostring (Session_high))
            label.set_text    (Session_low_price,  str.tostring (Session_low ))
        if not show_price
            label.delete  (Session_low_price )
            label.delete  (Session_high_price)
//
if new_day and show_timeframe
    if Session_start_vertical and hide_history
        line.delete(Vertical_start [1])
    Vertical_start := Vertical_line_start(Session_start_timestamp, Vertical_start_color, line.style_dotted, 1)
    if not Session_start_vertical
        line.delete(Vertical_start)
        
    if Session_end_vertical and hide_history
        line.delete(Vertical_end   [1])
    Vertical_end   := Vertical_line_end(Session_end_timestamp, Vertical_end_color, line.style_dotted, 1) 
    if not Session_end_vertical
        line.delete(Vertical_end)
//End
