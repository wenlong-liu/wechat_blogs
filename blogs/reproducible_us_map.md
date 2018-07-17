如何制作一个‘可重现’美国施肥量地图
================
Wenlong Liu
7/17/2018

## 背景介绍

近年来，研究者、环境工程师和相关部门对农业流域的营养元素（主要是氮磷）非常关注。美国地质调查局（USGS）的专家花费很多时间和精力制作了[全美国各郡的施肥量数据](https://wenlong-liu.github.io/usfertilizer/reference/us_fertilizer_county.html)。
基于USGS提供的数据，笔者开发了一款R拓展包，[*ggfertilizer*](https://github.com/wenlong-liu/ggfertilizer)，可以用来提取和处理美国各郡施肥量，以及制作相关图表。

本文将简要介绍*ggfertilizer*的基本功能，同时提供了清晰明了的可重现施肥量地图的制作流程。本文的目标读者为R的初阶使用者；对于R的进阶用户，笔者在*ggfertilizer*拓展包的[主页](https://wenlong-liu.github.io/ggfertilizer/)也提供了更详细的介绍。

## 为什么要强调‘可重现’？

一般说来，学术出版物应该具有可重现性，即在相同的前提条件下，其他科研人员能够重复此项研究并得到相同的结论。然而，《自然》杂志(Nature)在2016年的一份报告显示
[1,500 scientists lift the lid on
reproducibility](https://www.nature.com/news/1-500-scientists-lift-the-lid-on-reproducibility-1.19970)：

> More than 70% of researchers have tried and failed to reproduce
> another scientist’s experiments, and more than half have failed to
> reproduce their own experiments.

> 超过70%的科研人员尝试过重现某一科研人员的实验，但是没能成功；甚至超过一半的科研人员没能重现他们自己的研究。

这场所谓的“重现性危机”(reprocibility crisis)
已经引起了各方的重视。因此，为了增加自身研究成果的可信度，科研人员可以考虑提供可重现性研究成果，即研究成果中包含了数据、结果、图表和相关代码。

## 地图制作过程

### 前期准备

本文中所有内容都由[**R**](https://www.r-project.org/)及相关拓展包自动生成。**R**
是一款开源、免费的科学计算软件，读者通过[这个网页链](https://cloud.r-project.org/)接下载安装对应的软件版本。

在开始运行代码之前，读者需要安装一些拓展包。如果读者电脑上没有这些拓展包，运行以下代码将自动安装。

``` r
install.packages("ggplot2")
install.packages("usfertilizer")
install.packages("ggsn")

# check if devtools installed.
# 检查是否安装 devtools
if(!require(devtools, character.only = TRUE)){
  install.packages("devtools")
}

# install packages from my github repo.
# 通过github 安装 ggfertilizer
devtools::install_github("wenlong-liu/ggfertilizer")
```

拓展包安装成功后，我们需要运行一下代码来导入拓展包。

``` r
require(ggfertilizer)
require(ggplot2)
require(ggsn)
# import pre-packed dataset
# 导入相关数据。
data("us_fertilizer_county")
```

### 确定参数

鉴于美国各郡的施肥量估算数据的数据量很大，笔者已经将相关数据整理完毕(相关的数据来源、整理过程和包括的范围见[*usfertilizer*](https://wenlong-liu.github.io/usfertilizer/articles/Data_sources_and_cleaning.html))。
首先我们来看一下这个数据集。

``` r
str(us_fertilizer_county)
```

    ## Classes 'tbl_df', 'tbl' and 'data.frame':    625580 obs. of  12 variables:
    ##  $ FIPS      : chr  "01001" "01003" "01005" "01007" ...
    ##  $ State     : chr  "AL" "AL" "AL" "AL" ...
    ##  $ County    : chr  "Autauga" "Baldwin" "Barbour" "Bibb" ...
    ##  $ ALAND     : num  1.54e+09 4.12e+09 2.29e+09 1.61e+09 1.67e+09 ...
    ##  $ AWATER    : num  2.58e+07 1.13e+09 5.09e+07 9.29e+06 1.52e+07 ...
    ##  $ INTPTLAT  : num  32.5 30.7 31.9 33 34 ...
    ##  $ INTPTLONG : num  -86.6 -87.7 -85.4 -87.1 -86.6 ...
    ##  $ Quantity  : num  1580225 6524369 2412372 304592 1825118 ...
    ##  $ Year      : chr  "1987" "1987" "1987" "1987" ...
    ##  $ Nutrient  : chr  "N" "N" "N" "N" ...
    ##  $ Farm.Type : chr  "farm" "farm" "farm" "farm" ...
    ##  $ Input.Type: chr  "Fertilizer" "Fertilizer" "Fertilizer" "Fertilizer" ...

全体数据集包括了 625,580 行和 12 列数据，使用者需要确定感兴趣的参数，包括数据年、营养元素种类、是否输入到农场、营养元素来源等。
在这里，笔者生成列一系列等参数来举例说明如何制作地图。在下述等章节中，我们根据这些参数来生成地图。

``` r
Year <-  2001
Nutrient <- "N"

# nutrient comes from synthetic fertilizer.
# 营养元素来自化学合成肥料。
Input_Type <- "fertilizer" 

# nutrient applied to farms.
# 营养元素输入到农场。
Farm_Type <- "farm" 
```

### 制作基图

*ggfertilizer* 拓展包内置了
[**map\_us\_fertilizer()**](https://wenlong-liu.github.io/ggfertilizer/reference/map_us_fertilizer.html)
来自动画图。现在我们将上一章节确定等参数输入函数中来画基图。

``` r
# draw the map
# 制作基图
us_plot <- map_us_fertilizer(data = us_fertilizer_county, Year = Year, Nutrient = Nutrient,
                             Farm_Type = Farm_Type, Input_Type = Input_Type, 
                             add_north = TRUE) # add_north will be used in further sections.
us_plot
```

![](reproducible_us_map_files/figure-gfm/unnamed-chunk-5-1.png)<!-- -->

### 设置地图标题

实际上，画出来等地图是一个ggplot2
实例，即用户可以自行根据ggplot2的语法来修改地图。例如，我们可以根据输入参数来给地图加一个标题。

``` r
map_title <- paste(Nutrient,  " from ", Input_Type, " input to ", Farm_Type, " in the year of ",Year,
                     " \nat a county level",sep = "")
# add the title.
# 添加地图标题
us_plot <- us_plot +
      ggtitle(map_title)
us_plot
```

![](reproducible_us_map_files/figure-gfm/unnamed-chunk-6-1.png)<!-- -->

### 添加指北针和坐标尺

根据地理信息系统(Geographic Information System, GIS)的理论,
地图需要自带指北针和坐标尺。我们可以将这两项加入到地图中。

``` r
# add north symbol and scale bar.
# 添加指北针和坐标尺。
us_plot <- us_plot +
  north(us_plot$states_shape, scale = 0.15, anchor = c(x = -68, y = 50) ) +
  scalebar(us_plot$states_shape, dist = 500, dd2km = TRUE, model = 'WGS84', st.size = 2)

us_plot
```

![](reproducible_us_map_files/figure-gfm/unnamed-chunk-7-1.png)<!-- -->

### 保存地图

添加完所有元素之后，我们还可以将地图保存下来，以备不时之需。这张地图可以保存为不同的格式，包括.jpg, .pdf, .svg, 或者
.png。 在这里，我们保存为一个长度为 4 inch， 宽度为 6
inch的jpg图片。

``` r
ggsave(filename = "us_fertilizer_map_2001.jpg", width = 6, height = 4, scale = 1.5, units = "in")
```

## 总结

本文简要介绍了如何制作一个可重现的美国施肥量地图。 所有的代码和相关材料都可以在我的 [Github
Repo](https://github.com/wenlong-liu/wechat_blogs/tree/master/blogs)
里查看。

*ggfertilizer*
还处于开发中，笔者还在添加并测试不同的功能。为了笔者计划将这个扩展包发布到CRAN，更方便大家下载和安装。如果有什么疑问，欢迎大家留言～

The end.

## R session

``` r
sessionInfo()
```

    ## R version 3.5.0 (2018-04-23)
    ## Platform: x86_64-apple-darwin15.6.0 (64-bit)
    ## Running under: macOS High Sierra 10.13.5
    ## 
    ## Matrix products: default
    ## BLAS: /Library/Frameworks/R.framework/Versions/3.5/Resources/lib/libRblas.0.dylib
    ## LAPACK: /Library/Frameworks/R.framework/Versions/3.5/Resources/lib/libRlapack.dylib
    ## 
    ## locale:
    ## [1] en_US.UTF-8/en_US.UTF-8/en_US.UTF-8/C/en_US.UTF-8/en_US.UTF-8
    ## 
    ## attached base packages:
    ## [1] stats     graphics  grDevices utils     datasets  methods   base     
    ## 
    ## other attached packages:
    ## [1] bindrcpp_0.2.2     maps_3.3.0         ggsn_0.4.0        
    ## [4] ggplot2_3.0.0      ggfertilizer_0.0.4 usfertilizer_0.1.5
    ## 
    ## loaded via a namespace (and not attached):
    ##  [1] Rcpp_0.12.17      pillar_1.2.3      compiler_3.5.0   
    ##  [4] plyr_1.8.4        bindr_0.1.1       viridis_0.5.1    
    ##  [7] tools_3.5.0       digest_0.6.15     lattice_0.20-35  
    ## [10] evaluate_0.10.1   tibble_1.4.2      gtable_0.2.0     
    ## [13] viridisLite_0.3.0 png_0.1-7         pkgconfig_2.0.1  
    ## [16] rlang_0.2.1       mapproj_1.2.6     yaml_2.1.19      
    ## [19] gridExtra_2.3     withr_2.1.2       dplyr_0.7.6      
    ## [22] stringr_1.3.1     knitr_1.20        rprojroot_1.3-2  
    ## [25] grid_3.5.0        tidyselect_0.2.4  glue_1.2.0       
    ## [28] R6_2.2.2          foreign_0.8-70    rmarkdown_1.10   
    ## [31] sp_1.3-1          purrr_0.2.5       magrittr_1.5     
    ## [34] maptools_0.9-2    backports_1.1.2   scales_0.5.0     
    ## [37] htmltools_0.3.6   assertthat_0.2.0  colorspace_1.3-2 
    ## [40] labeling_0.3      stringi_1.2.3     lazyeval_0.2.1   
    ## [43] munsell_0.5.0
