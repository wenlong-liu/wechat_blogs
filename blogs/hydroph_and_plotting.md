hydrograph
================
Wenlong
4/15/2018

对于水文工作者来说，降雨-水文过程线图是最基础的数据展示方式，能够制作一张符合要求的水文过程线图(hydrograph)是一项必备技能。笔者今天简要介绍一下降雨-水文过程线图，然后着重介绍几种常用的图表制作软件和制作过程。

## 1\. 降雨水文过程线图介绍

现在科学的核心假定之一是质量守恒与能量守恒定律 (mass balance).
在水文学中，质量守恒的体现形式之一就是水循环，包括全球大循环与区域小循环。对于全球大循环，海洋的中通过蒸发进入大气层，然后运移到大陆形成降水；降水汇集到河道形成径流，最后在流入海洋。当我们研究河流时，一般认为水文过程是降水驱动的。因此，我们经常讲降雨和水文过程画在一起，形成降雨-水文过程线图。

![](http://map.ps123.net/world/UploadFile/201503/2015031023542662.jpg)
Fig.1 全球水循环示意图

Fig.2 是一张典型的降雨(precipitation) -
水文过程线图(Hydrograph)。习惯上，我们将x轴设置为时间轴，例如小时，天数或者月份等等；y轴设置为径流量，单位为立方米每秒，有时候也会用深度的概念，即所有径流量铺在相应面积上的水层厚度来表示水量。需要注意的是，很多水文学工作者倾向于将降水设置为柱状图，倒过来放在水文过程的上方。这种构图首先展示了该水文过程是相应降水驱动的，其次要能够表征一些参数，例如洪峰滞时(Fig.
2中的Lag)既是降雨中心至洪峰出现的时间。

![](http://4.bp.blogspot.com/-52kbcyXSmpI/T4iEqMFMoQI/AAAAAAAACIo/HK-p4Orgy3U/s1600/storm_1.gif)
Fig. 2 降雨水文过程示意图

## 2\. 降水-水文过程线图制作方法

降水-水文过程线图的制作方法其实特别简单，只需要以下步骤即可：

  - 整理降水数据和径流数据，保证两者之间时间轴相同；
  - 将径流数据设置成折线图或者散点图；
  - 将降水数据设置成柱状图，投影到y轴副坐标轴，并反转坐标轴；
  - 调整坐标轴的数据展示范围，以保证降水和径流数据图没有重叠。

以下笔者将展示不同软件的做法， 包括MS Excel, ggplot2 in R, matplotlib in Python, etc.

### 2.1 MS Excel

MS Excel
是大家非常熟悉的数据分析与可视化软件，其功能强大，操作也较为直观和简单。用Excel制作水文过程线图的关键点是将降雨放置在副坐标轴，同时翻转副坐标轴。具体的步骤如下所示：

1.  把降雨数据和流量数据放入同一张图表；
2.  将降雨数据设置在y轴的副坐标轴，并翻转坐标轴；
3.  设置降雨数据为柱状图，流量数据为散点图(或者折线图)；
4.  如果两组数据之间没有对齐，检查x轴的起始与结束时间是否正确；
5.  调整两个y轴的最大值，保证两组数据之间没有重合。

最后的结果如Fig.
3所示，详细的数据及制作方法可以从[这里](https://github.com/wenlong-liu/wechat_blogs/blob/master/data/sample_data.xlsx)下载.

![](https://github.com/wenlong-liu/wechat_blogs/blob/master/materials/hydrograph.png?raw=true)

Fig. 3 Excel 制作水文过程线图示例 (Data source: USGS)

### 2.2 highcharter in R

highcharter
提供了Highcharts的R接口。Highchart是一款基于JavaScript的绘图库，可以用来制作多种类型的图表。由于图表渲染较慢，本文中将15min的数据综合为逐日数据。具体代码如下所示。

``` r
# load required libraries and data.
require(highcharter)
require(tidyverse)
# for local data.
hydrology_data = read_csv("../data/sample_data.csv")
# down scaling to daily data.
hydrology_daily = hydrology_data %>% 
  mutate(date = as.Date(format(dateTime, '%Y-%m-%d'))) %>% 
  select(-dateTime) %>% 
  group_by(date) %>% 
  summarise(precipitation = sum(precipitation, na.rm = TRUE),
            discharge = mean(discharge, na.rm = TRUE)) 

#plot hydrograph and precipitation.  
hydrograph <- highchart() %>% 
hc_yAxis_multiples(list(title = list(text = "Precipitation (cm)"), reversed = TRUE, opposite = TRUE, max = 10), 
                   list(title = list(text = "Discharge  (m3/s)"), max = 200)) %>% 
hc_add_series(name = "Precipitation", data = hydrology_daily$precipitation, type = "column" ) %>% 
hc_add_series(name = "Stream discharge", data = hydrology_daily$discharge, type = "spline", yAxis = 1) %>%
hc_xAxis(categories = hydrology_daily$date, title = list(text = "Date"))

htmltools::tagList(hydrograph)
```

<!--html_preserve-->

<div id="htmlwidget-4fc9c894226fed5893a3" class="highchart html-widget" style="width:100%;height:500px;">

</div>

<script type="application/json" data-for="htmlwidget-4fc9c894226fed5893a3">{"x":{"hc_opts":{"title":{"text":null},"yAxis":[{"title":{"text":"Precipitation (cm)"},"reversed":true,"opposite":true,"max":10},{"title":{"text":"Discharge  (m3/s)"},"max":200}],"credits":{"enabled":false},"exporting":{"enabled":false},"plotOptions":{"series":{"turboThreshold":0},"treemap":{"layoutAlgorithm":"squarified"},"bubble":{"minSize":5,"maxSize":25}},"annotationsOptions":{"enabledButtons":false},"tooltip":{"delayForDisplay":10},"series":[{"data":[0,0,0,0,0,0,0,0.5842,0,0,0.5842,2.0066,0.1016,0,0.1524,0.3556,0,0,0,0,0,0.3556,0.4318,0.0762,0,0,0.2286,0.127,0.0508,0.1016,0,0,0,0,0.4318,0.0762,0.1778,0.889,0,0.127,0.381,0.1778,0,0,0.3302,0.3048,1.8796,0.0254,0.1778,2.0828,0,0.2794,1.2954,0.1524,1.397,2.3876,0,0,0.0508,3.3782,0.5334,0,0,0,0,0.1524,0,0.1016,0,0,0,0.1524,0,0,0,0,0,0,0,0.9144,0,0,0,0,0,1.8796,0.7366,0.762,0.762,0,0],"name":"Precipitation","type":"column"},{"data":[9.44526315789474,8.31260416666667,6.755,5.38020833333333,5.1103125,4.55177083333333,4.45052083333333,4.4903125,4.57041666666667,4.63791666666667,5.691875,21.5569791666667,39.7054166666667,36.6523958333333,38.551875,41.7442708333333,50.3858333333333,33.0717708333333,28.4908333333333,26.8109375,24.4348958333333,28.0595833333333,60.7155208333333,68.6380208333333,58.9430208333333,51.934375,50.12,51.8173958333333,49.8828125,44.2003125,29.2721875,22.4416666666667,20.8223958333333,15.6967708333333,15.1811458333333,14.2994791666667,11.2642708333333,7.54572916666667,6.26395833333333,5.74729166666667,6.3996875,7.24583333333333,13.845,26.2139583333333,26.8626041666667,45.1158333333333,83.4403125,89.27125,45.1621875,55.0616666666667,103.804895833333,78.9329166666667,62.4857291666667,79.7672916666667,57.7101041666667,93.9996875,109.591770833333,74.63875,67.4063541666667,69.628125,133.504375,127.8740625,57.4858333333333,53.7951041666667,65.5496875,67.27875,66.3245833333333,65.1778125,59.9466666666667,58.0120833333333,57.5190625,57.0595833333333,53.2477083333333,31.9467708333333,19.2436458333333,11.7709375,9.44229166666667,8.79364583333333,7.4325,7.12854166666667,7.18395833333333,7.58177083333333,7.92770833333333,7.56864583333333,6.97916666666667,8.59145833333333,31.8352083333333,35.531875,45.3479166666667,51.33375,31.005625],"name":"Stream discharge","type":"spline","yAxis":1}],"xAxis":{"categories":["2018-01-01","2018-01-02","2018-01-03","2018-01-04","2018-01-05","2018-01-06","2018-01-07","2018-01-08","2018-01-09","2018-01-10","2018-01-11","2018-01-12","2018-01-13","2018-01-14","2018-01-15","2018-01-16","2018-01-17","2018-01-18","2018-01-19","2018-01-20","2018-01-21","2018-01-22","2018-01-23","2018-01-24","2018-01-25","2018-01-26","2018-01-27","2018-01-28","2018-01-29","2018-01-30","2018-01-31","2018-02-01","2018-02-02","2018-02-03","2018-02-04","2018-02-05","2018-02-06","2018-02-07","2018-02-08","2018-02-09","2018-02-10","2018-02-11","2018-02-12","2018-02-13","2018-02-14","2018-02-15","2018-02-16","2018-02-17","2018-02-18","2018-02-19","2018-02-20","2018-02-21","2018-02-22","2018-02-23","2018-02-24","2018-02-25","2018-02-26","2018-02-27","2018-02-28","2018-03-01","2018-03-02","2018-03-03","2018-03-04","2018-03-05","2018-03-06","2018-03-07","2018-03-08","2018-03-09","2018-03-10","2018-03-11","2018-03-12","2018-03-13","2018-03-14","2018-03-15","2018-03-16","2018-03-17","2018-03-18","2018-03-19","2018-03-20","2018-03-21","2018-03-22","2018-03-23","2018-03-24","2018-03-25","2018-03-26","2018-03-27","2018-03-28","2018-03-29","2018-03-30","2018-03-31","2018-04-01"],"title":{"text":"Date"}}},"theme":{"chart":{"backgroundColor":"transparent"}},"conf_opts":{"global":{"Date":null,"VMLRadialGradientURL":"http =//code.highcharts.com/list(version)/gfx/vml-radial-gradient.png","canvasToolsURL":"http =//code.highcharts.com/list(version)/modules/canvas-tools.js","getTimezoneOffset":null,"timezoneOffset":0,"useUTC":true},"lang":{"contextButtonTitle":"Chart context menu","decimalPoint":".","downloadJPEG":"Download JPEG image","downloadPDF":"Download PDF document","downloadPNG":"Download PNG image","downloadSVG":"Download SVG vector image","drillUpText":"Back to {series.name}","invalidDate":null,"loading":"Loading...","months":["January","February","March","April","May","June","July","August","September","October","November","December"],"noData":"No data to display","numericSymbols":["k","M","G","T","P","E"],"printChart":"Print chart","resetZoom":"Reset zoom","resetZoomTitle":"Reset zoom level 1:1","shortMonths":["Jan","Feb","Mar","Apr","May","Jun","Jul","Aug","Sep","Oct","Nov","Dec"],"thousandsSep":" ","weekdays":["Sunday","Monday","Tuesday","Wednesday","Thursday","Friday","Saturday"]}},"type":"chart","fonts":[],"debug":false},"evals":[],"jsHooks":[]}</script>

<!--/html_preserve-->

Fig.4 使用highcharter绘制水文过程线图

### 2.3 ggplot2 in R

ggplot2 是R中一款非常流行且强大的数据可视化工具。
ggplot2常年在所有R包的下载榜中名列前茅。对于笔者来说，ggplot2最大的用处就治愈了笔者的直男审美(汗。)。鉴于ggplot2的开发团队不太喜欢次坐标轴的做法，我们可以用facet
plots来制作水文过程线图。

由于ggplot2中所有的facet需要统一的图表格式，为了满足section 1中的要求，我们需要稍微变化一下图表的制作方式，具体内容如下：

  - 利用dplyr::filter 的方法来制作不同的facet, 这样就可以在不同的facet中画出不同的图表；
  - 利用facet的标签来替代y坐标轴标题；
  - 将数据格式从宽型（wide) 转换成长型(long)；
  - 将参数因子化并设置为不同的因素水平(level),这样即可对facet进行排序。

详细代码如下：

``` r
require(tidyverse)

# for local data.
hydrology_data = read_csv("../data/sample_data.csv")

# change wide data to long data.
hydrology_data = hydrology_data %>% 
  mutate( "precipiatation (cm)" = precipitation,
          "discharge (m3/s)" = discharge) %>% 
  gather(variable, values, c("precipiatation (cm)", "discharge (m3/s)")) %>% 
  mutate(variable = factor(variable, levels = c("precipiatation (cm)", "discharge (m3/s)")))

# plot data.
# plot separately using filter to change different figure format.
f1 = ggplot(hydrology_data, aes(x = dateTime, y = values, color=variable))+
facet_wrap(~variable, scale='free_y', nrow = 2, strip.position = "left")
f2 = f1 + geom_step(data  = filter(hydrology_data, variable == "precipiatation (cm)"))
f3 = f2 +  geom_line(data = filter(hydrology_data, variable == "discharge (m3/s)"))

# update graph format.
f3+theme_bw()+
  theme(legend.position = "none")+
  ylab(NULL)+
  theme(strip.background = element_blank(),
           strip.placement = "outside")+
  scale_x_datetime(name = "Date", date_breaks = "15 day")
```

![](hydroph_and_plotting_files/figure-gfm/hydrograph-1.png)<!-- -->

Fig.5 使用ggplot2绘制水文过程线图

# 3 Take-home points

  - 水文过程线图是水文学研究的基础，需要熟练掌握相应制图技术；
  - 本文中介绍了数种制作科技论文配图的方法；
  - 本文中的用R制图的代码均为reproducible output, 读者可以登录[my
    rep](https://github.com/wenlong-liu/wechat_blogs)查看和调试代码。

The end. Thanks for reading.
