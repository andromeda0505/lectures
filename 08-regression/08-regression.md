---
title: "Data Science for Economists"
subtitle: "Lecture 8: Regression analysis in R"
author:
  name: Grant R. McDermott
  affiliation: University of Oregon | [EC 607](https://github.com/uo-ec607/lectures)
# date: Lecture 6  #"04 August 2020"
output: 
  html_document:
    theme: flatly
    highlight: haddock 
    # code_folding: show
    toc: yes
    toc_depth: 4
    toc_float: yes
    keep_md: true
---



Today's lecture is about the bread-and-butter tool of applied econometrics and data science: regression analysis. My goal is to give you a whirlwind tour of the key functions and packages. I'm going to assume that you already know all of the necessary theoretical background on causal inference, asymptotics, etc. This lecture will *not* cover any of theoretical concepts or seek to justify a particular statistical model. Indeed, most of the models that we're going to run today are pretty silly. We also won't be able to cover some important topics. For example, I'll only provide the briefest example of a Bayesian regression model and I won't touch times series analysis at all. (Although, I will provide links for further reading at the bottom of this document.) These disclaimers aside, let's proceed...

## Software requirements

### R packages 

It's important to note that "base" R already provides all of the tools we need for basic regression analysis. However, we'll be using several external packages today, because they will make our lives easier and offer increased power for some more sophisticated analyses.

- New: **broom**, **estimatr**, **fixest**, **sandwich**, **lmtest**, **AER**, **lfe**, **margins**, **modelsummary**, **vt**
- Already used: **tidyverse**, **hrbrthemes**, **listviewer**

The **broom** package was bundled with the rest of tidyverse and **sandwich** should get installed as a dependency of several of the above packages. Still, a convenient way to install (if necessary) and load everything is by running the below code chunk. 


```r
## Load and install the packages that we'll be using today
if (!require("pacman")) install.packages("pacman")
pacman::p_load(tidyverse, broom, hrbrthemes, estimatr, fixest, sandwich, lmtest, AER, lfe, margins, modelsummary, vtable)
## My preferred ggplot2 plotting theme (optional)
theme_set(hrbrthemes::theme_ipsum())
```

While we've already loaded all of the required packages for today, I'll try to be as explicit about where a particular function is coming from, whenever I use it below. 

Something else that I want to mention up front is that we'll mostly be working with the `starwars` data frame that we've already seen from previous lectures. Here's a quick reminder of what it looks like to refresh your memory.


```r
starwars
```

```
## # A tibble: 87 x 14
##    name  height  mass hair_color skin_color eye_color birth_year sex   gender
##    <chr>  <int> <dbl> <chr>      <chr>      <chr>          <dbl> <chr> <chr> 
##  1 Luke…    172    77 blond      fair       blue            19   male  mascu…
##  2 C-3PO    167    75 <NA>       gold       yellow         112   none  mascu…
##  3 R2-D2     96    32 <NA>       white, bl… red             33   none  mascu…
##  4 Dart…    202   136 none       white      yellow          41.9 male  mascu…
##  5 Leia…    150    49 brown      light      brown           19   fema… femin…
##  6 Owen…    178   120 brown, gr… light      blue            52   male  mascu…
##  7 Beru…    165    75 brown      light      blue            47   fema… femin…
##  8 R5-D4     97    32 <NA>       white, red red             NA   none  mascu…
##  9 Bigg…    183    84 black      light      brown           24   male  mascu…
## 10 Obi-…    182    77 auburn, w… fair       blue-gray       57   male  mascu…
## # … with 77 more rows, and 5 more variables: homeworld <chr>, species <chr>,
## #   films <list>, vehicles <list>, starships <list>
```


## Regression basics

### The `lm()` function

R's workhorse command for running regression models is the built-in `lm()` function. The "**lm**" stands for "**l**inear **m**odels" and the syntax is very intuitive.^[Indeed, all other regression packages in R that I'm aware of --- including those that allow for much more advanced and flexible models --- closely follow the `lm()` syntax.] 

```r
lm(y ~ x1 + x2 + x3 + ..., data = df)
```

You'll note that the `lm()` call includes a reference to the data source (in this case, a hypothetical data frame called `df`). We covered this in our earlier lecture on R language basics and object-orientated programming, but the reason is that many objects (e.g. data frames) can exist in your R environment at the same time. So we need to be specific about where our regression variables are coming from --- even if `df` is the only data frame in our global environment at the time. Another option would be to use indexing, but I find it a bit verbose:

```r
lm(df$y ~ df$x1 + df$x2 + df$x3 + ...)
```

Let's run a simple bivariate regression of starwars characters' mass on height.


```r
ols1 = lm(mass ~ height, data = starwars)
# ols1 = lm(starwars$mass ~ starwars$height) ## Also works
ols1
```

```
## 
## Call:
## lm(formula = mass ~ height, data = starwars)
## 
## Coefficients:
## (Intercept)       height  
##    -13.8103       0.6386
```

The resulting object is pretty terse, but that's only because it buries most of its valuable information --- of which there is a lot --- within its internal list structure. You can use the `str()` function to view this structure. Or, if you want to be fancy, the interactive `listviewer::jsonedit()` function that we saw in the previous lecture is a nice option.


```r
# str(ols1) ## Static option
listviewer::jsonedit(ols1, mode="view") ## Interactive option
```

<!--html_preserve--><div id="htmlwidget-8ef7ddf9d26030504f3e" style="width:100%;height:10%;" class="jsonedit html-widget"></div>
<script type="application/json" data-for="htmlwidget-8ef7ddf9d26030504f3e">{"x":{"data":{"coefficients":{"(Intercept)":-13.8103136287302,"height":0.638571004587035},"residuals":{"1":-19.0238991602397,"2":-17.8310441373045,"3":-15.4925028116251,"4":20.8189707021492,"5":-32.9753370593249,"6":20.1446748122381,"7":-16.5539021281305,"8":-16.1310738162121,"9":-19.0481802106971,"10":-25.4096092061101,"11":-22.2410352336323,"13":-19.7838754171137,"14":-21.132467196936,"15":-22.6624701648267,"16":1260.060387826,"17":-17.7467571510656,"18":8.86753280306401,"19":-11.335372674014,"20":-19.7467571510656,"21":-24.8481802106971,"22":26.0961127113233,"23":5.48182275719367,"24":-20.2167541831749,"25":-18.9396121740008,"26":-18.132467196936,"29":-22.3839347749288,"30":-20.3610471051953,"31":-20.4338902565674,"32":-18.1567482473934,"34":-45.3496032703285,"35":-47.2295913987655,"39":-17.7096388850176,"42":-17.9396121740008,"44":-44.8553251877619,"45":-1.21536080245099,"47":-25.2767601189564,"48":-22.2410352336323,"49":-30.6267452795026,"50":-24.3496032703285,"52":-53.6867512152841,"55":-26.2410352336323,"57":-19.3253222198712,"60":-23.0481802106971,"61":-38.5467571510656,"62":-42.1924731327175,"64":-29.4338902565674,"66":-24.0481802106971,"67":-38.4696151418916,"68":-10.6267452795026,"69":-44.4224464217007,"72":-21.6367957336455,"74":-61.4338902565674,"76":-42.8553251877619,"77":34.8789766379308,"78":0.384698555364137,"79":-27.2410352336323,"80":-51.8553251877619,"81":-37.7353133161989,"87":-46.5539021281305},"effects":{"(Intercept)":-747.466613505302,"height":172.783889465672,"3":-8.91507473191358,"4":21.4194000157428,"5":-29.4427951434848,"6":22.0983868653301,"7":-13.8671619244768,"8":-9.61003251731305,"9":-17.3764020616673,"10":-23.6814442762678,"11":-20.8511909886646,"12":-20.6495024046433,"13":-19.2915287054689,"14":-20.4268242076726,"15":1262.18326022153,"16":-15.3419508514742,"17":10.7084712945311,"18":-3.06634116992954,"19":-17.3419508514742,"20":-23.1764020616673,"21":26.8093155865418,"22":6.75889344053646,"23":-18.2066553492705,"24":-16.8167397784715,"25":-16.2915287054689,"26":-15.3554124487178,"27":-17.3923729974795,"28":-19.3259799156619,"29":-16.936064344863,"30":-44.4108532718603,"31":-47.8696712630454,"32":-12.0343992983051,"33":-15.8167397784715,"34":-42.9016131346699,"35":5.47484083888535,"36":-22.4772463536779,"37":-20.8511909886646,"38":-29.8007688426593,"39":-23.4108532718603,"40":-52.0713598470667,"41":-24.8511909886646,"42":-17.7663176324662,"43":-21.3764020616673,"44":-36.1419508514742,"45":-39.5621197098763,"46":-28.3259799156619,"47":-22.3764020616673,"48":-35.9520352806753,"49":-9.80076884265928,"50":-45.3444601900428,"51":-14.1007923801226,"52":-60.3259799156619,"53":-40.9016131346699,"54":34.6899910201503,"55":-0.819249117040115,"56":-25.8511909886646,"57":-49.9016131346699,"58":-37.360431125855,"59":-43.8671619244768},"rank":2,"fitted.values":{"1":96.0238991602397,"2":92.8310441373045,"3":47.4925028116251,"4":115.181029297851,"5":81.9753370593249,"6":99.8553251877619,"7":91.5539021281305,"8":48.1310738162121,"9":103.048180210697,"10":102.40960920611,"11":106.241035233632,"13":131.783875417114,"14":101.132467196936,"15":96.6624701648267,"16":97.939612174001,"17":94.7467571510656,"18":101.132467196936,"19":28.335372674014,"20":94.7467571510656,"21":103.048180210697,"22":113.903887288677,"23":107.518177242806,"24":99.2167541831749,"25":97.9396121740008,"26":101.132467196936,"29":42.3839347749288,"30":88.3610471051953,"31":109.433890256567,"32":108.156748247393,"34":111.349603270329,"35":129.229591398766,"39":57.7096388850176,"42":97.9396121740008,"44":99.8553251877619,"45":46.215360802451,"47":90.2767601189564,"48":106.241035233632,"49":112.626745279503,"50":111.349603270329,"52":103.686751215284,"55":106.241035233632,"57":104.325322219871,"60":103.048180210697,"61":94.7467571510656,"62":92.1924731327175,"64":109.433890256567,"66":103.048180210697,"67":93.4696151418916,"68":112.626745279503,"69":132.422446421701,"72":36.6367957336455,"74":109.433890256567,"76":99.8553251877619,"77":124.121023362069,"78":135.615301444636,"79":106.241035233632,"80":99.8553251877619,"81":117.735313316199,"87":91.5539021281305},"assign":[0,1],"qr":{"qr":[[-7.68114574786861,-1336.64954904012],[0.130188910980824,270.578977473948],[0.130188910980824,0.287474707506683],[0.130188910980824,-0.104277826225195],[0.130188910980824,0.0879026620206323],[0.130188910980824,-0.0155791393425052],[0.130188910980824,0.0324659827189515],[0.130188910980824,0.283778928886571],[0.130188910980824,-0.0340580324430655],[0.130188910980824,-0.0303622538229534],[0.130188910980824,-0.0525369255436258],[0.130188910980824,-0.200368070348108],[0.130188910980824,-0.0229706965827293],[0.130188910980824,0.00289975375805507],[0.130188910980824,-0.00449180348216904],[0.130188910980824,0.0139870896183912],[0.130188910980824,-0.0229706965827293],[0.130188910980824,0.398348066110045],[0.130188910980824,0.0139870896183912],[0.130188910980824,-0.0340580324430655],[0.130188910980824,-0.0968862689849704],[0.130188910980824,-0.0599284827838499],[0.130188910980824,-0.0118833607223932],[0.130188910980824,-0.00449180348216904],[0.130188910980824,-0.0229706965827293],[0.130188910980824,0.31704093646758],[0.130188910980824,0.0509448758195118],[0.130188910980824,-0.071015818644186],[0.130188910980824,-0.0636242614039619],[0.130188910980824,-0.0821031545045222],[0.130188910980824,-0.18558495586766],[0.130188910980824,0.22834224958489],[0.130188910980824,-0.00449180348216904],[0.130188910980824,-0.0155791393425052],[0.130188910980824,0.294866264746907],[0.130188910980824,0.0398575399591756],[0.130188910980824,-0.0525369255436258],[0.130188910980824,-0.0894947117447463],[0.130188910980824,-0.0821031545045222],[0.130188910980824,-0.0377538110631775],[0.130188910980824,-0.0525369255436258],[0.130188910980824,-0.0414495896832896],[0.130188910980824,-0.0340580324430655],[0.130188910980824,0.0139870896183912],[0.130188910980824,0.0287702040988395],[0.130188910980824,-0.071015818644186],[0.130188910980824,-0.0340580324430655],[0.130188910980824,0.0213786468586153],[0.130188910980824,-0.0894947117447463],[0.130188910980824,-0.20406384896822],[0.130188910980824,0.350302944048588],[0.130188910980824,-0.071015818644186],[0.130188910980824,-0.0155791393425052],[0.130188910980824,-0.156018726906763],[0.130188910980824,-0.22254274206878],[0.130188910980824,-0.0525369255436258],[0.130188910980824,-0.0155791393425052],[0.130188910980824,-0.119060940705643],[0.130188910980824,0.0324659827189515]],"qraux":[1.13018891098082,1.02507442547873],"pivot":[1,2],"tol":1e-07,"rank":2},"df.residual":57,"na.action":{},"xlevels":{},"call":{},"terms":{},"model":{"mass":[77,75,32,136,49,120,75,32,84,77,84,112,80,74,1358,77,110,17,75,78.2,140,113,79,79,83,20,68,89,90,66,82,40,80,55,45,65,84,82,87,50,80,85,80,56.2,50,80,79,55,102,88,15,48,57,159,136,79,48,80,45],"height":[172,167,96,202,150,178,165,97,183,182,188,228,180,173,175,170,180,66,170,183,200,190,177,175,180,88,160,193,191,196,224,112,175,178,94,163,188,198,196,184,188,185,183,170,166,193,183,168,198,229,79,193,178,216,234,188,178,206,165]}},"options":{"mode":"view","modes":["code","form","text","tree","view"]}},"evals":[],"jsHooks":[]}</script><!--/html_preserve-->

As we can see, this `ols1` object has a bunch of important slots... containing everything from the regression coefficients, to vectors of the residuals and fitted (i.e. predicted) values, to the rank of the design matrix, to the input data, etc. etc. To summarise the key pieces of information, we can use the --- *wait for it* --- generic `summary()` function. This will look pretty similar to the default regression output from Stata that many of you will be used to.


```r
summary(ols1)
```

```
## 
## Call:
## lm(formula = mass ~ height, data = starwars)
## 
## Residuals:
##     Min      1Q  Median      3Q     Max 
##  -61.43  -30.03  -21.13  -17.73 1260.06 
## 
## Coefficients:
##             Estimate Std. Error t value Pr(>|t|)
## (Intercept) -13.8103   111.1545  -0.124    0.902
## height        0.6386     0.6261   1.020    0.312
## 
## Residual standard error: 169.4 on 57 degrees of freedom
##   (28 observations deleted due to missingness)
## Multiple R-squared:  0.01792,	Adjusted R-squared:  0.0006956 
## F-statistic:  1.04 on 1 and 57 DF,  p-value: 0.312
```

We can then dig down further by extracting a summary of the regression coefficients:


```r
summary(ols1)$coefficients
```

```
##               Estimate  Std. Error    t value  Pr(>|t|)
## (Intercept) -13.810314 111.1545260 -0.1242443 0.9015590
## height        0.638571   0.6260583  1.0199865 0.3120447
```

### Get "tidy" regression coefficients with the `broom` package

While it's easy to extract regression coefficients via the `summary()` function, in practice I always use the **broom** package ([link ](https://broom.tidyverse.org/)) to do so. This package has a bunch of neat features to convert regression (and other statistical) objects into "tidy" data frames. This is especially useful because regression output is so often used as an input to something else, e.g. a plot of coefficients or marginal effects. Here, I'll use `broom::tidy(..., conf.int = TRUE)` to coerce the `ols1` regression object into a tidy data frame of coefficient values and key statistics.


```r
library(broom)

tidy(ols1, conf.int = TRUE)
```

```
## # A tibble: 2 x 7
##   term        estimate std.error statistic p.value conf.low conf.high
##   <chr>          <dbl>     <dbl>     <dbl>   <dbl>    <dbl>     <dbl>
## 1 (Intercept)  -13.8     111.       -0.124   0.902 -236.       209.  
## 2 height         0.639     0.626     1.02    0.312   -0.615      1.89
```

Again, I could now pipe this tidied coefficients data frame to a **ggplot2** call, using saying `geom_pointrange()` to plot the error bars. Feel free to practice doing this yourself now, but we'll get to some explicit examples further below.

A related and also useful function is `broom::glance()`, which summarises the model "meta" data (R<sup>2</sup>, AIC, etc.) in a data frame.


```r
glance(ols1)
```

```
## # A tibble: 1 x 12
##   r.squared adj.r.squared sigma statistic p.value    df logLik   AIC   BIC
##       <dbl>         <dbl> <dbl>     <dbl>   <dbl> <dbl>  <dbl> <dbl> <dbl>
## 1    0.0179      0.000696  169.      1.04   0.312     1  -386.  777.  783.
## # … with 3 more variables: deviance <dbl>, df.residual <int>, nobs <int>
```

(BTW, If you're wondering how to export regression results to other formats (e.g. LaTeX tables), don't worry: We'll get to that at the very end of the lecture.)

### Regressing on subsetted or different data

Our simple model isn't particularly good; our R<sup>2</sup> is only 0.018. Different species and homeworlds aside, we may have an extreme outlier in our midst...


```r
starwars %>%
  ggplot(aes(x=height, y=mass)) +
  geom_point(alpha=0.5) +
  geom_point(
    data = starwars %>% filter(mass==max(mass, na.rm=T)), 
    col="red"
    ) +
  geom_text(
    aes(label=name),
    data = starwars %>% filter(mass==max(mass, na.rm=T)), 
    col="red", vjust = 0, nudge_y = 25
    ) +
  labs(
    title = "Spot the outlier...",
    caption = "Aside: Always plot your data!"
    )
```

```
## Warning: Removed 28 rows containing missing values (geom_point).
```

![](08-regression_files/figure-html/jabba-1.png)<!-- -->

Maybe we should exclude Jabba from our regression? You can do this in two ways: 1) Create a new data frame and then regress, or 2) Subset the original data frame directly in the `lm()` call.

#### 1) Create a new data frame

Recall that we can keep multiple objects in memory in R. So we can easily create a new data frame that excludes Jabba using `dplyr::filter()`.


```r
starwars2 =
  starwars %>% 
  filter(name != "Jabba Desilijic Tiure")
  # filter(!(grepl("Jabba", name))) ## Regular expressions also work

ols2 = lm(mass ~ height, data = starwars2)
summary(ols2)
```

```
## 
## Call:
## lm(formula = mass ~ height, data = starwars2)
## 
## Residuals:
##     Min      1Q  Median      3Q     Max 
## -39.382  -8.212   0.211   3.846  57.327 
## 
## Coefficients:
##              Estimate Std. Error t value Pr(>|t|)    
## (Intercept) -32.54076   12.56053  -2.591   0.0122 *  
## height        0.62136    0.07073   8.785 4.02e-12 ***
## ---
## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
## 
## Residual standard error: 19.14 on 56 degrees of freedom
##   (28 observations deleted due to missingness)
## Multiple R-squared:  0.5795,	Adjusted R-squared:  0.572 
## F-statistic: 77.18 on 1 and 56 DF,  p-value: 4.018e-12
```

#### 2) Subset directly in the `lm()` call

Running a regression directly on a subsetted data frame is equally easy.


```r
ols2a = lm(mass ~ height, data = starwars %>% filter(!(grepl("Jabba", name))))
summary(ols2a)
```

```
## 
## Call:
## lm(formula = mass ~ height, data = starwars %>% filter(!(grepl("Jabba", 
##     name))))
## 
## Residuals:
##     Min      1Q  Median      3Q     Max 
## -39.382  -8.212   0.211   3.846  57.327 
## 
## Coefficients:
##              Estimate Std. Error t value Pr(>|t|)    
## (Intercept) -32.54076   12.56053  -2.591   0.0122 *  
## height        0.62136    0.07073   8.785 4.02e-12 ***
## ---
## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
## 
## Residual standard error: 19.14 on 56 degrees of freedom
##   (28 observations deleted due to missingness)
## Multiple R-squared:  0.5795,	Adjusted R-squared:  0.572 
## F-statistic: 77.18 on 1 and 56 DF,  p-value: 4.018e-12
```

The overall model fit is much improved by the exclusion of this outlier, with R<sup>2</sup> increasing to 0.58. Still, we should be cautious about throwing out data. Another approach is to handle or account for outliers with statistical methods. Which provides a nice segue to robust and clustered standard errors.

## Robust and clustered standard errors

Dealing with statistical irregularities (heteroskedasticity, clustering, etc.) is a fact of life for empirical researchers. However, it says something about the economics profession that a random stranger could walk uninvited into a live seminar and ask, "How did you cluster your standard errors?", and it would likely draw approving nods from audience members. 

The good news is that there are *lots* of ways to get robust and clustered standard errors in R. For many years, these have been based on the excellent **sandwich** package ([link](https://cran.r-project.org/web/packages/sandwich/index.html)). However, my preferred way these days is to use the **estimatr** package ([link](https://declaredesign.org/r/estimatr/articles/getting-started.html)), which is both fast and provides convenient aliases for the standard regression functions. For example, you can obtain robust standard errors using `estimatr::lm_robust()`. Let's illustrate by implementing a robust version of the `ols1` regression that we ran earlier.


```r
# library(estimatr) ## Already loaded
ols1_robust = lm_robust(mass ~ height, data = starwars)
tidy(ols1_robust, conf.int = TRUE)
```

```
##          term   estimate   std.error  statistic      p.value    conf.low
## 1 (Intercept) -13.810314 23.45557632 -0.5887859 5.583311e-01 -60.7792950
## 2      height   0.638571  0.08791977  7.2631109 1.159161e-09   0.4625147
##    conf.high df outcome
## 1 33.1586678 57    mass
## 2  0.8146273 57    mass
```

The package defaults to using Eicker-Huber-White robust standard errors, commonly referred to as "HC2" standard errors. You can easily specify alternate methods using the `se_type = ` argument.^[See the [package documentation](https://declaredesign.org/r/estimatr/articles/mathematical-notes.html#lm_robust-notes) for a full list of options.] For example, you can specify Stata robust standard errors if you want to replicate code or results from that language. (See [here](https://declaredesign.org/r/estimatr/articles/stata-wls-hat.html) for more details on why this isn't the default and why Stata's robust standard errors differ from those in R and Python.)


```r
ols1_robust_stata = lm_robust(mass ~ height, data = starwars, se_type = "stata")
tidy(ols1_robust_stata, conf.int = TRUE)
```

```
##          term   estimate   std.error  statistic      p.value    conf.low
## 1 (Intercept) -13.810314 23.36219608 -0.5911394 5.567641e-01 -60.5923043
## 2      height   0.638571  0.08616105  7.4113649 6.561046e-10   0.4660365
##    conf.high df outcome
## 1 32.9716771 57    mass
## 2  0.8111055 57    mass
```

**estimatr** also supports (robust) instrumental variable regression and clustered standard errors. I'm actually going to hold off discussing these issues until we get to the relevant sections below. But here's a quick example of the latter just to illustrate:


```r
ols1_robust_clustered = lm_robust(mass ~ height, data = starwars, clusters = homeworld)
```

```
## Warning in eval(quote({: Some observations have missingness in the cluster
## variable(s) but not in the outcome or covariates. These observations have been
## dropped.
```

```r
tidy(ols1_robust_clustered, conf.int = TRUE)
```

```
##          term   estimate   std.error  statistic      p.value    conf.low
## 1 (Intercept) -9.3014938 28.84436408 -0.3224718 0.7559158751 -76.6200628
## 2      height  0.6134058  0.09911832  6.1886211 0.0002378887   0.3857824
##    conf.high       df outcome
## 1 58.0170751 7.486034    mass
## 2  0.8410291 8.195141    mass
```

### Aside on HAC (Newey-West) standard errors

On thing I want to flag is that **estimatr** does not yet offer support for HAC (i.e. heteroskedasticity and autocorrelation consistent) standard errors *a la* [Newey-West](https://en.wikipedia.org/wiki/Newey%E2%80%93West_estimator). I've submitted a [feature request](https://github.com/DeclareDesign/estimatr/issues/272) on GitHub --- vote up if you would like to see it added sooner! --- but you can still obtain these pretty easily using the aforementioned **sandwich** package. For example, we can use `sandwich::NeweyWest()` on our existing `ols1` object to obtain HAC SEs for it.


```r
# library(sandwich) ## Already loaded
NeweyWest(ols1) ## Print the HAC VCOV
```

```
##             (Intercept)       height
## (Intercept) 452.3879311 -0.505227090
## height       -0.5052271  0.005994863
```

```r
sqrt(diag(NeweyWest(ols1))) ## Print the HAC SEs
```

```
## (Intercept)      height 
##  21.2694130   0.0774265
```

If you plan to use HAC SEs for inference, then I recommend converting the model object with `lmtest::coeftest()`. This function provides a convenient way to do hypothesis testing with a wide variety of alternate variance-covariance (VCOV) matrices. Of course, these alternate VCOV matrices could extended way beyond HAC, but here's how it would work for the present case:


```r
# library(lmtest) ## Already loaded
ols1_hac = lmtest::coeftest(ols1, vcov=NeweyWest)
ols1_hac
```

```
## 
## t test of coefficients:
## 
##               Estimate Std. Error t value  Pr(>|t|)    
## (Intercept) -13.810314  21.269413 -0.6493    0.5187    
## height        0.638571   0.077427  8.2474 2.672e-11 ***
## ---
## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
```

Note that its easy to convert `coeftest()`-adjusted models to tidied **broom** objects too. 


```r
tidy(ols1_hac, conf.int = TRUE)
```

```
## # A tibble: 2 x 7
##   term        estimate std.error statistic  p.value conf.low conf.high
##   <chr>          <dbl>     <dbl>     <dbl>    <dbl>    <dbl>     <dbl>
## 1 (Intercept)  -13.8     21.3       -0.649 5.19e- 1  -56.4      28.8  
## 2 height         0.639    0.0774     8.25  2.67e-11    0.484     0.794
```

## Dummy variables and interaction terms

### Dummy variables as *factors*

Dummy variables are a core component of many regression models. However, these can be a pain to create in many statistical languages, since you first have to tabulate a whole new matrix of binary variables and then append it to the original data frame. In contrast, R has a much more convenient framework for creating and evaluating dummy variables in a regression. You simply specify the variable of interest as a [factor](https://r4ds.had.co.nz/factors.html).^[Factors are variables that have distinct qualitative levels, e.g. "male", "female", "hermaphrodite", etc.]

For this next section, it will be convenient to demonstrate using a subsample of the data that comprises only humans. I'll first create this `humans` data frame and then demonstrate the dummy-variables-as-factors approach.
 

```r
humans = 
  starwars %>% 
  filter(species=="Human") %>%
  mutate(gender_factored = as.factor(gender)) %>% ## create factored version of "gender"
  select(contains("gender"), everything())
humans
```

```
## # A tibble: 35 x 15
##    gender gender_factored name  height  mass hair_color skin_color eye_color
##    <chr>  <fct>           <chr>  <int> <dbl> <chr>      <chr>      <chr>    
##  1 mascu… masculine       Luke…    172    77 blond      fair       blue     
##  2 mascu… masculine       Dart…    202   136 none       white      yellow   
##  3 femin… feminine        Leia…    150    49 brown      light      brown    
##  4 mascu… masculine       Owen…    178   120 brown, gr… light      blue     
##  5 femin… feminine        Beru…    165    75 brown      light      blue     
##  6 mascu… masculine       Bigg…    183    84 black      light      brown    
##  7 mascu… masculine       Obi-…    182    77 auburn, w… fair       blue-gray
##  8 mascu… masculine       Anak…    188    84 blond      fair       blue     
##  9 mascu… masculine       Wilh…    180    NA auburn, g… fair       blue     
## 10 mascu… masculine       Han …    180    80 brown      fair       brown    
## # … with 25 more rows, and 7 more variables: birth_year <dbl>, sex <chr>,
## #   homeworld <chr>, species <chr>, films <list>, vehicles <list>,
## #   starships <list>
```

```r
ols_dv = lm(mass ~ height + gender_factored, data = humans)
summary(ols_dv)
```

```
## 
## Call:
## lm(formula = mass ~ height + gender_factored, data = humans)
## 
## Residuals:
##     Min      1Q  Median      3Q     Max 
## -16.068  -8.130  -3.660   0.702  37.112 
## 
## Coefficients:
##                          Estimate Std. Error t value Pr(>|t|)  
## (Intercept)              -84.2520    65.7856  -1.281   0.2157  
## height                     0.8787     0.4075   2.156   0.0441 *
## gender_factoredmasculine  10.7391    13.1968   0.814   0.4259  
## ---
## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
## 
## Residual standard error: 15.19 on 19 degrees of freedom
##   (13 observations deleted due to missingness)
## Multiple R-squared:  0.444,	Adjusted R-squared:  0.3855 
## F-statistic: 7.587 on 2 and 19 DF,  p-value: 0.003784
```


In fact, I'm even making things more complicated than they need to be. R is "friendly" and tries to help whenever it thinks you have misspecified a function or variable. While this is something to be [aware of](https://rawgit.com/grantmcdermott/R-intro/master/rIntro.html#r_tries_to_guess_what_you_meant), it normally just works<sup>TM</sup>. A case in point is that we don't actually *need* to specify a qualitative or character variable as a factor in a regression. R will automatically do this for you regardless, since that's the only sensible way to include string variables in a regression.


```r
## Use the non-factored "gender" variable instead
ols_dv2 = lm(mass ~ height + gender, data = humans)
summary(ols_dv2)
```

```
## 
## Call:
## lm(formula = mass ~ height + gender, data = humans)
## 
## Residuals:
##     Min      1Q  Median      3Q     Max 
## -16.068  -8.130  -3.660   0.702  37.112 
## 
## Coefficients:
##                 Estimate Std. Error t value Pr(>|t|)  
## (Intercept)     -84.2520    65.7856  -1.281   0.2157  
## height            0.8787     0.4075   2.156   0.0441 *
## gendermasculine  10.7391    13.1968   0.814   0.4259  
## ---
## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
## 
## Residual standard error: 15.19 on 19 degrees of freedom
##   (13 observations deleted due to missingness)
## Multiple R-squared:  0.444,	Adjusted R-squared:  0.3855 
## F-statistic: 7.587 on 2 and 19 DF,  p-value: 0.003784
```


### Interaction effects

Like dummy variables, R provides a convenient syntax for specifying interaction terms directly in the regression model without having to create them manually beforehand.^[Although there are very good reasons that you might want to modify your parent variables before doing so (e.g. centering them). As it happens, I'm [on record](https://twitter.com/grant_mcdermott/status/903691491414917122) as stating that interaction effects are most widely misunderstood and misapplied concept in econometrics. However, that's a topic for another day. (Read the paper in the link!)] You can just use `x1:x2` (to include only the interaction term) or `x1*x2` (to include the parent terms and interaction terms). Generally speaking, you are best advised to include the parent terms alongside an interaction term. This makes the `*` option a good default. 

For example, we might wonder whether the relationship between a person's body mass and their height is modulated by their gender. That is, we want to run a regression of the form

$$Mass = \beta_0 + \beta_1 D_{Male} + \beta_2 Height + \beta_3 D_{Male} \times Height$$

To implement this in R, we simply run the following


```r
ols_ie = lm(mass ~ gender * height, data = humans)
summary(ols_ie)
```

```
## 
## Call:
## lm(formula = mass ~ gender * height, data = humans)
## 
## Residuals:
##     Min      1Q  Median      3Q     Max 
## -16.250  -8.158  -3.684  -0.107  37.193 
## 
## Coefficients:
##                        Estimate Std. Error t value Pr(>|t|)
## (Intercept)            -61.0000   204.0565  -0.299    0.768
## gendermasculine        -15.7224   219.5440  -0.072    0.944
## height                   0.7333     1.2741   0.576    0.572
## gendermasculine:height   0.1629     1.3489   0.121    0.905
## 
## Residual standard error: 15.6 on 18 degrees of freedom
##   (13 observations deleted due to missingness)
## Multiple R-squared:  0.4445,	Adjusted R-squared:  0.3519 
## F-statistic: 4.801 on 3 and 18 DF,  p-value: 0.01254
```


## Panel models

### Fixed effects with the **fixest** package

The simplest (and least efficient) way to include fixed effects in a regression model is, of course, to use dummy variables. However, it isn't very efficient or scalable. What's the point learning all that stuff about the Frisch-Waugh-Lovell theorem, within-group transformations, etcetera, etcetera if we can't use them in our software routines? Again, there are several options to choose from here. For example, many of you are probably familiar with the excellent **lfe** package ([link](https://cran.r-project.org/web/packages/lfe/index.html)), which offers near-identical functionality to the popular Stata library, **reghdfe** ([link](http://scorreia.com/software/reghdfe/)). However, for fixed effects models in R, I am going to advocate that you take a look at the **fixest** package ([link](https://github.com/lrberge/fixest)).

**fixest** is relatively new on the scene and has quickly become one of my packages in the entire R catalogue. It has a boatload of functionality built in to it: support for nonlinear models, high-dimensional fixed effects, multiway clustering, etc. It is also insanely fast... as in, up to *orders of magnitude* faster than **lfe** or **reghdfe**. (See: [benchmarks](https://github.com/lrberge/fixest#benchmarking.)) I won't be able to cover all of **fixest**'s features in depth here --- see the [introductory vignette](https://cran.r-project.org/web/packages/fixest/vignettes/fixest_walkthrough.html) for a thorough walkthrough --- but I hope to least give you a sense of why I am so enthusiastic about it. Let's start off with a simple example before moving on to something a little more demanding.

#### Simple FE model

The package's main function is `fixest::feols()`, which is used for estimating linear fixed effects models. The syntax is such that you first specify the regression model as per normal, and then list the fixed effect(s) after a `|`. An example may help to illustrate. Let's say that we again want to run our simple regression of mass on height, but this time control for species-level fixed effects.^[Since we specify "species" in the fixed effects slot below, `feols()` will automatically coerce it to a factor variable even though we didn't explicitly tell it to.]


```r
# library(fixest) ## Already loaded

ols_fe = feols(mass ~ height | species, data = starwars) ## Fixed effect(s) go after the "|"
ols_fe
```

```
## OLS estimation, Dep. Var.: mass
## Observations: 58 
## Fixed-effects: species: 31
## Standard-errors: Clustered (species) 
##        Estimate Std. Error t value  Pr(>|t|)    
## height 0.974876   0.044291   22.01 < 2.2e-16 ***
## ---
## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
## Log-likelihood: -214.02   Adj. R2: 0.99282 
##                         R2-Within: 0.66249
```

Note that the resulting model object has automatically clustered the standard errors by the fixed effect variable (i.e. species). We'll explore some more options for adjusting standard errors in **fixest** objects shortly, but you can specify vanilla standard errors simply by calling the `se` argument in `summary.fixest()` as follows.


```r
summary(ols_fe, se = 'standard')
```

```
## OLS estimation, Dep. Var.: mass
## Observations: 58 
## Fixed-effects: species: 31
## Standard-errors: Standard 
##        Estimate Std. Error t value Pr(>|t|)    
## height 0.974876   0.136463  7.1439 1.38e-07 ***
## ---
## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
## Log-likelihood: -214.02   Adj. R2: 0.99282 
##                         R2-Within: 0.66249
```

Before continuing, let's quickly save a "tidied" data frame of the coefficients for later use. I'll use vanilla standard errors again, if only to show you that the `broom::tidy()` method for `fixest` objects also accepts an `se` argument. This basically just provides another convenient way for you to adjust standard errors for your models on the fly.


```r
# coefs_fe = tidy(summary(ols_fe, se = 'standard'), conf.int = TRUE) ## yields same result as below
coefs_fe = tidy(ols_fe, se = 'standard', conf.int = TRUE)
```

#### High dimensional FEs and multiway clustering

As I already mentioned above, **fixest** supports (arbitrarily) high-dimensional fixed effects and (up to fourway) multiway clustering. To see this in action, let's add "homeworld" as an additional fixed effect to the model.


```r
## We now have two fixed effects: species and homeworld
ols_hdfe = feols(mass ~ height |  species + homeworld, data = starwars)
```

```
## NOTE: 32 observations removed because of NA values (Breakup: LHS: 28, RHS: 6, Fixed-effects: 13).
```

Easy enough, but the standard errors of the above model are automatically clustered by species, i.e. the first fixed effect variable. (Print the model to screen to prove this for yourself.) Let's go a step further and cluster by both "species" and "homeworld". ^[I most definitely am not claiming that this is a particularly good or sensible clustering strategy, but just go with it.] We can do this using either the `se` or `cluster` arguments of `summary.fixest()`. I'll (re)assign the model to the same `ols_hdfe` object, but you could of course create a new object if you so wished.


```r
## Cluster by both species and homeworld
# ols_hdfe = summary(ols_hdfe, se = 'twoway') ## Same effect as the next line
ols_hdfe = summary(ols_hdfe, cluster = c('species', 'homeworld'))
ols_hdfe
```

```
## OLS estimation, Dep. Var.: mass
## Observations: 55 
## Fixed-effects: species: 30,  homeworld: 38
## Standard-errors: Two-way (species & homeworld) 
##        Estimate Std. Error t value Pr(>|t|)    
## height 0.755844   0.117257   6.446  0.09798 .  
## ---
## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
## Log-likelihood: -188.55   Adj. R2: 1.00768 
##                         R2-Within: 0.48723
```

#### Comparing our model coefficients

**fixest** provides an inbuilt `coefplot()` function for plotting estimation results. This is especially useful for tracing the evolution of treatment effects over time. (Take a look [here](https://cran.r-project.org/web/packages/fixest/vignettes/fixest_walkthrough.html#23_adding_interactions:_yearly_treatment_effect.) When it comes to comparing coefficients across models, however, I personally prefer to do this "manually" with **ggplot2**. Consider the below example, which leverages the fact that we have saved (or can save) regression models as data frames with `broom::tidy()`.


```r
## First get tidied output of the ols_hdfe object
coefs_hdfe = tidy(ols_hdfe, conf.int = TRUE)

bind_rows(
  coefs_fe %>% mutate(reg = "Model 1\nFE and no clustering"),
  coefs_hdfe %>% mutate(reg = "Model 2\nHDFE and twoway clustering")
  ) %>%
  ggplot(aes(x=reg, y=estimate, ymin=conf.low, ymax=conf.high)) +
  geom_pointrange() +
  labs(Title = "Marginal effect of height on mass") +
  geom_hline(yintercept = 0, col = "orange") +
  ylim(-0.5, NA) +
  labs(
    title = "'Effect' of height on mass",
    caption = "Data: Characters from the Star Wars universe"
    ) +
  theme(axis.title.x = element_blank())
```

![](08-regression_files/figure-html/fe_mods_compared-1.png)<!-- -->

FWIW, we'd normally expect our standard errors to blow up with clustering. Here that effect appears to be outweighed by the increased precision brought on by additional fixed effects. Still, I wouldn't put too much thought into it. Our clustering probably doesn't make much sense and was just used to demonstrate the package syntax.

#### Aside on standard errors

We've now seen some of the different options that **fixest** has for specifying different error structures. In short, run your model and then use either the `se` or `cluster` arguments in `summary.fixest()` (or `broom::tidy()`) if you aren't happy with the default clustering choice. There are two additional points that I want to draw your attention to.

First, if you're coming from another statistical language or package, adjusting the standard errors after the fact rather than in the original model call may seem slightly odd. But this behaviour is actually extremely powerful, because it allows us to analyse the effect of different error structures *on-the-fly* without having to rerun the entire model again. **fixest** is already the fastest game in town, but just think about the implied timesavings for really large models.^[To be clear, adjusting the standard errors via, say, `summary.fixest()` completes instantaneously. It's a testament to how well the package is put together and the [novel estimation method](https://wwwen.uni.lu/content/download/110162/1299525/file/2018_13) that Laurent (the package author) has derived.]

Second, reconciling standard errors across different software is a much more complicated process than you may realise. There are a number of unresolved theoretical issues to consider --- especially when it comes to multiway clustering --- and package maintainers have to make a number of arbitrary decisions about the best way to account for these. (See [here](https://github.com/sgaure/lfe/issues/1#issuecomment-530643808) for a detailed discussion.) Luckily, Laurent has taken the time to write out a [detailed vignette](https://cran.r-project.org/web/packages/fixest/vignettes/standard_errors.html) about how to replicate standard errors from other methods and software packages (including Stata's **reghdfe**).^[If you want a deep dive into the theory with even more simulations, then [this paper](https://cran.r-project.org/web/packages/sandwich/vignettes/sandwich-CL.pdf) by the authors of the **sandwich** paper is another excellent resource.]

### Random effects

Fixed effects models are more common than random effects models in economics (in my experience, anyway). I'd also advocate for [Bayesian hierachical models](http://www.stat.columbia.edu/~gelman/arm/) if we're going down the whole random effects path. However, it's still good to know that R has you covered for random effects models through the **plm** ([link](https://cran.r-project.org/web/packages/plm/)) and **nlme** ([link](https://cran.r-project.org/web/packages/nlme/index.html)) packages.^[As I mentioned above, **plm** also handles fixed effects (and pooling) models. However, I prefer **fixest** and **lfe** for the reasons already discussed.] I won't go into detail , but click on those links (especially the first one) if you would like to see some examples.

## Instrumental variables

As you would have guessed by now, there are a number of ways to run instrumental variable (IV) regressions in R. I'll walk through three options using the `AER::ivreg()`, `estimatr::iv_robust()`, and `lfe::felm()` functions, respectively. These are all going to follow a similar syntax, where the IV first-stage regression is specified after a **`|`** following the main regression. However, there are also some subtle and important differences, which is why I want to go through each of them. After that, I'll let you decide which of the three options is your favourite.

The dataset that we'll be using here is a panel of US cigarette consumption by state, which is taken from the **AER** package ([link](https://cran.r-project.org/web/packages/AER/vignettes/AER.pdf)). Let's load the data, add some modified variables, and then take a quick look at it. Note that I'm going to limit the dataset to 2005 only, given that I want to focus the IV syntax and don't want to deal with the panel structure of the data. (Though that's very easily done, as we've already seen.)


```r
## Get the data
data("CigarettesSW", package = "AER")
## Create a new data frame with some modified variables
cigs =
  CigarettesSW %>%
  mutate(
    rprice = price/cpi,
    rincome = income/population/cpi,
    rtax = tax/cpi,
    tdiff = (taxs - tax)/cpi
    ) %>%
  as_tibble()
## Create a subset of the data limited to 1995
cigs95 = cigs %>% filter(year==1995)
cigs95
```

```
## # A tibble: 48 x 13
##    state year    cpi population packs income   tax price  taxs rprice rincome
##    <fct> <fct> <dbl>      <dbl> <dbl>  <dbl> <dbl> <dbl> <dbl>  <dbl>   <dbl>
##  1 AL    1995   1.52    4262731 101.  8.39e7  40.5  158.  41.9   104.    12.9
##  2 AR    1995   1.52    2480121 111.  4.60e7  55.5  176.  63.9   115.    12.2
##  3 AZ    1995   1.52    4306908  72.0 8.89e7  65.3  199.  74.8   130.    13.5
##  4 CA    1995   1.52   31493524  56.9 7.71e8  61    211.  74.8   138.    16.1
##  5 CO    1995   1.52    3738061  82.6 9.29e7  44    167.  44     110.    16.3
##  6 CT    1995   1.52    3265293  79.5 1.04e8  74    218.  86.4   143.    21.0
##  7 DE    1995   1.52     718265 124.  1.82e7  48    166.  48     109.    16.7
##  8 FL    1995   1.52   14185403  93.1 3.34e8  57.9  188.  68.5   123.    15.4
##  9 GA    1995   1.52    7188538  97.5 1.60e8  36    157.  37.4   103.    14.6
## 10 IA    1995   1.52    2840860  92.4 6.02e7  60    191.  69.1   125.    13.9
## # … with 38 more rows, and 2 more variables: rtax <dbl>, tdiff <dbl>
```

Now, assume that we are interested in regressing the number of cigarettes packs consumed per capita on their average price and people's real incomes. The problem is that the price is endogenous (because it is simultaneously determined by demand and supply), so we need to instrument for it using different tax variables. That is, we want to run the following:

$$price_i = \pi_0 + \pi_1 tdiff_i + + \pi_2 rtax_i + v_i  \hspace{1cm} \text{(First stage)}$$
$$packs_i = \beta_0 + \beta_2\widehat{price_i} + \beta_1 rincome_i + u_i \hspace{1cm} \text{(Second stage)}$$

### Option 1: `AER::ivreg()`

Let's start with `AER::ivreg()` as our first IV regression option; if for no other reason than that's where our data are coming from. The key point from the below code chunk is that the first-stage regression is going to be specified after the **`|`** and will include *all* exogenous variables.


```r
# library(AER) ## Already loaded

## Run the IV regression 
iv_reg = 
  ivreg(
    log(packs) ~ log(rprice) + log(rincome) | ## The main regression. "rprice" is endogenous
      log(rincome) + tdiff + rtax, ## List all exogenous variables, including "rincome"
    data = cigs95
    )
summary(iv_reg, diagnostics = TRUE)
```

```
## 
## Call:
## ivreg(formula = log(packs) ~ log(rprice) + log(rincome) | log(rincome) + 
##     tdiff + rtax, data = cigs95)
## 
## Residuals:
##        Min         1Q     Median         3Q        Max 
## -0.6006931 -0.0862222 -0.0009999  0.1164699  0.3734227 
## 
## Coefficients:
##              Estimate Std. Error t value Pr(>|t|)    
## (Intercept)    9.8950     1.0586   9.348 4.12e-12 ***
## log(rprice)   -1.2774     0.2632  -4.853 1.50e-05 ***
## log(rincome)   0.2804     0.2386   1.175    0.246    
## 
## Diagnostic tests:
##                  df1 df2 statistic p-value    
## Weak instruments   2  44   244.734  <2e-16 ***
## Wu-Hausman         1  44     3.068  0.0868 .  
## Sargan             1  NA     0.333  0.5641    
## ---
## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
## 
## Residual standard error: 0.1879 on 45 degrees of freedom
## Multiple R-Squared: 0.4294,	Adjusted R-squared: 0.4041 
## Wald test: 13.28 on 2 and 45 DF,  p-value: 2.931e-05
```

I want to emphasise that you might find the above syntax a little counterintuitive --- or, at least, unusual --- if you're coming from a language like Stata.^[Assuming that you have already created the logged variables and subsetted the data, the Stata equivalent would be something like `ivreg log_packs = log_rincome (log_rprice = tdiff rtax)`.] Note that we didn't specify the endogenous variable (i.e. "rprice") directly. Rather, we told R which are the *exogenous* variables. It then figured out which were the endogenous variables that needed to be instrumented and ran the necessary first-stage regression(s) in the background. This approach actually makes quite a lot of sense if you think about the underlying theory of IV. But different strokes for different folks. 

The good news for those who prefer the Stata-style syntax is that `AER::ivreg()` also accepts an alternate way of specifying the first-stage. This time, we'll denote our endogenous "rprice" variable with `. -price` and include only the instrumental variables after the `|` break. Feel free to check yourself, but the outcome will be exactly the same.


```r
## Run the IV regression 
iv_reg2 = 
  ivreg(
    log(packs) ~ log(rprice) + log(rincome) | 
      . -log(rprice) + tdiff + rtax, ## Alternative way of specifying the first-stage.
    data = cigs95
  )
```


### Option 2: `estimatr::iv_robust()`

Our second IV option comes from the **estimatr** package that we saw earlier. This will default to using HC2 robust standard errors although, as before, we could specify other options if we so wished (including clustering). More importantly, note that the syntax is effectively identical to the previous example. All we need to do is change the function call from `AER::ivreg()` to `estimatr::iv_robust()`.


```r
# library(estimatr) ## Already loaded

## Run the IV regression with robust SEs
iv_reg_robust = 
  iv_robust( ## We only need to change the function call. Everything else stays the same.
    log(packs) ~ log(rprice) + log(rincome) | 
      log(rincome) + tdiff + rtax,
    data = cigs95
    )
summary(iv_reg_robust, diagnostics = TRUE)
```

```
## 
## Call:
## iv_robust(formula = log(packs) ~ log(rprice) + log(rincome) | 
##     log(rincome) + tdiff + rtax, data = cigs95)
## 
## Standard error type:  HC2 
## 
## Coefficients:
##              Estimate Std. Error t value  Pr(>|t|) CI Lower CI Upper DF
## (Intercept)    9.8950     0.9777  10.120 3.569e-13   7.9257  11.8642 45
## log(rprice)   -1.2774     0.2547  -5.015 8.739e-06  -1.7904  -0.7644 45
## log(rincome)   0.2804     0.2547   1.101 2.768e-01  -0.2326   0.7934 45
## 
## Multiple R-squared:  0.4294 ,	Adjusted R-squared:  0.4041 
## F-statistic:  15.5 on 2 and 45 DF,  p-value: 7.55e-06
```

### Option 3: `felm::lfe()`

Finally, we get to my personal favourite IV option using the `felm()` function from the **lfe** package. This is actually my favourite option; not only because I work mostly with panel data, but also because I find it has the most natural syntax.^[The **fixest** package that I concentrated on earlier, doesn't yet support IV regression. However, even if it is a little slower, `lfe::felm()` includes basically all of same benefits: support for high-level fixed effects, multiway clustering, etc.] In fact, it very closely resembles Stata's approach to writing out the first-stage, where you specify the endogenous variable(s) and the instruments only.


```r
# library(lfe) ## Already loaded

iv_felm = 
  felm(
    log(packs) ~ log(rincome) |
      0 | ## No FEs
      (log(rprice) ~ tdiff + rtax), ## First-stage. Note the surrounding parentheses
    data = cigs95
  )
summary(iv_felm)
```

```
## 
## Call:
##    felm(formula = log(packs) ~ log(rincome) | 0 | (log(rprice) ~      tdiff + rtax), data = cigs95) 
## 
## Residuals:
##      Min       1Q   Median       3Q      Max 
## -0.60069 -0.08622 -0.00100  0.11647  0.37342 
## 
## Coefficients:
##                    Estimate Std. Error t value Pr(>|t|)    
## (Intercept)          9.8950     1.0586   9.348 4.12e-12 ***
## log(rincome)         0.2804     0.2386   1.175    0.246    
## `log(rprice)(fit)`  -1.2774     0.2632  -4.853 1.50e-05 ***
## ---
## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
## 
## Residual standard error: 0.1879 on 45 degrees of freedom
## Multiple R-squared(full model): 0.4294   Adjusted R-squared: 0.4041 
## Multiple R-squared(proj model): 0.4294   Adjusted R-squared: 0.4041 
## F-statistic(full model):13.28 on 2 and 45 DF, p-value: 2.931e-05 
## F-statistic(proj model): 13.28 on 2 and 45 DF, p-value: 2.931e-05 
## F-statistic(endog. vars):23.56 on 1 and 45 DF, p-value: 1.496e-05
```

Note that in the above example, we inserted a "0" where the fixed effect slot goes, since we only used a subset of the data. Just for fun then, here's another IV regression with `felm()`. This time, I'll use the whole `cigs` data frame (i.e. not subsetting to 1995), and use both year and state fixed effects to control for the panel structure.


```r
iv_felm_all = 
  felm(
    log(packs) ~ log(rincome) |
      year + state | ## Now include FEs
      (log(rprice) ~ tdiff + rtax), 
    data = cigs ## Use whole panel data set
  )
summary(iv_felm_all)
```

```
## 
## Call:
##    felm(formula = log(packs) ~ log(rincome) | year + state | (log(rprice) ~      tdiff + rtax), data = cigs) 
## 
## Residuals:
##      Min       1Q   Median       3Q      Max 
## -0.08393 -0.03851  0.00000  0.03851  0.08393 
## 
## Coefficients:
##                    Estimate Std. Error t value Pr(>|t|)    
## log(rincome)         0.4620     0.3081   1.500    0.141    
## `log(rprice)(fit)`  -1.2024     0.1712  -7.024  9.4e-09 ***
## ---
## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
## 
## Residual standard error: 0.06453 on 45 degrees of freedom
## Multiple R-squared(full model): 0.9668   Adjusted R-squared: 0.9299 
## Multiple R-squared(proj model): 0.5466   Adjusted R-squared: 0.04281 
## F-statistic(full model):26.21 on 50 and 45 DF, p-value: < 2.2e-16 
## F-statistic(proj model): 27.71 on 2 and 45 DF, p-value: 1.436e-08 
## F-statistic(endog. vars):49.33 on 1 and 45 DF, p-value: 9.399e-09
```

## Other topics

### Marginal effects

Calculating marginal effect in a regression is utterly straightforward in cases where there are no non-linearities; just look at the coefficient values! However, that quickly goes out the window when you have interaction effects or non-linear models like probit, logit, etc. Luckily, the **margins** package ([link](https://cran.r-project.org/web/packages/margins)), which is modeled on its namesake in Stata, goes a long way towards automating the process. You can read more in the package [vignette](https://cran.r-project.org/web/packages/margins/vignettes/Introduction.html), but here's a very simple example to illustrate. 

Consider our earlier interaction effects regression, where we interested in how people's mass varied by height and gender. To get the average marginal effect (AME) of these dependent variables, we can just use the `margins::margins()` function.


```r
# library(margins) ## Already loaded

margins(ols_ie)
```

```
## Average marginal effects
```

```
## lm(formula = mass ~ gender * height, data = humans)
```

```
##  height gendermasculine
##   0.874           13.53
```

You can get standard errors by piping (or wrapping) the above object to the generic `summary()` function. 


```r
# summary(margins(ols_ie)) ## Also works
ols_ie %>% margins() %>% summary() 
```

```
##           factor     AME      SE      z      p    lower   upper
##  gendermasculine 13.5253 26.7585 0.5055 0.6132 -38.9203 65.9710
##           height  0.8740  0.4203 2.0797 0.0376   0.0503  1.6977
```

If we want to compare marginal effects at specific values --- e.g. how the AME of height on mass differs across genders --- then that's easily done too.


```r
ols_ie %>% 
  margins(
    variables = "height", ## The main variable we're interested in
    at = list(gender = c("masculine", "feminine")) ## How the main variable is modulated by at specific values of a second variable
    ) #%>% 
```

```
## Average marginal effects at specified values
```

```
## lm(formula = mass ~ gender * height, data = humans)
```

```
##  at(gender) height
##   masculine 0.8962
##    feminine 0.7333
```

```r
  # summary() ## If you want SEs etc.
```

If you're the type of person who prefers visualizations (like me), then you should consider `margins::cplot()`, which is the package's in-built method for constructing *conditional* effect plots.


```r
cplot(ols_ie, x = "gender", dx = "height", what = "effect")
```

![](08-regression_files/figure-html/margins3-1.png)<!-- -->

In this case,it doesn't make much sense to read a lot into the larger standard errors on the female group; that's being driven by a very small sub-sample size.

Finally, you can also use `cplot()` to plot the predicted values of your outcome variable (here: "mass"), conditional on one of your dependent variables. For example:


```r
par(mfrow=c(1, 2)) ## Just to plot these next two (base) figures side-by-side
cplot(ols_ie, x = "gender", what = "prediction")
```

```
##       xvals    yvals     upper    lower
## 1 masculine 84.19201  91.70295 76.68107
## 2  feminine 70.66667 122.57168 18.76166
```

```r
cplot(ols_ie, x = "height", what = "prediction")
```

```
##       xvals    yvals     upper    lower
## 1  150.0000 57.71242  86.90520 28.51964
## 2  152.1667 59.65426  87.02441 32.28411
## 3  154.3333 61.59610  87.15216 36.04003
## 4  156.5000 63.53793  87.29040 39.78546
## 5  158.6667 65.47977  87.44173 43.51781
## 6  160.8333 67.42161  87.60961 47.23361
## 7  163.0000 69.36344  87.79883 50.92806
## 8  165.1667 71.30528  88.01610 54.59446
## 9  167.3333 73.24711  88.27110 58.22313
## 10 169.5000 75.18895  88.57808 61.79983
## 11 171.6667 77.13079  88.95862 65.30296
## 12 173.8333 79.07262  89.44599 68.69926
## 13 176.0000 81.01446  90.09168 71.93724
## 14 178.1667 82.95630  90.97287 74.93972
## 15 180.3333 84.89813  92.19300 77.60326
## 16 182.5000 86.83997  93.85745 79.82249
## 17 184.6667 88.78181  96.01749 81.54612
## 18 186.8333 90.72364  98.63222 82.81507
## 19 189.0000 92.66548 101.59946 83.73149
## 20 191.1667 94.60732 104.81353 84.40110
```

![](08-regression_files/figure-html/margins4-1.png)<!-- -->

```r
par(mfrow=c(1, 1)) ## Reset plot defaults
```

Note that `cplot` automatically produces a data frame of the predicted effects too. This can be used to construct **ggplot2** versions of the figures instead of the (base) `cplot` defaults. See the package documentation for [more information](https://cran.r-project.org/web/packages/margins/vignettes/Introduction.html#ggplot2_examples).

*Aside:* One downside that I want to highlight briefly is that **margins** does [not yet work](https://github.com/leeper/margins/issues/128) with **fixest** (or **lfe**) objects, but there are [workarounds](https://github.com/leeper/margins/issues/128#issuecomment-636372023) in the meantime.

### Probit, logit and other generalized linear models

See `?stats::glm`.

### Synthetic control

See the **gsynth** package ([link](https://yiqingxu.org/software/gsynth/gsynth_examples.html)).

### Bayesian regression

We could spend a whole course on Bayesian models. The very, very short version is that R offers outstanding support for Bayesian models and data analysis. You will find convenient interfaces to all of the major MCMC and Bayesian software engines: [Stan](https://mc-stan.org/users/interfaces/rstan), [JAGS](http://mcmc-jags.sourceforge.net/), TensorFlow (via [Greta](https://greta-stats.org/)), etc. Here follows a *super* simple example using the **rstanarm** package ([link](http://mc-stan.org/rstanarm/)). Note that we did not install this package with the others above, as it can take fairly long and involve some minor troubleshooting.^[FWIW, on my machine (running Arch Linux) I had to install `stan` (and thus `rstanarm`) by running R through the shell. For some reason, RStudio kept closing midway through the installation process.]


```r
# install.packages("rstanarm") ## Run this first if you want to try yourself
library(rstanarm)
bayes_reg = 
  stan_glm(
    mass ~ gender * height,
    data = humans, 
    family = gaussian(), prior = cauchy(), prior_intercept = cauchy()
    )
```

```r
summary(bayes_reg)
```

```
## 
## Model Info:
##  function:     stan_glm
##  family:       gaussian [identity]
##  formula:      mass ~ gender * height
##  algorithm:    sampling
##  sample:       4000 (posterior sample size)
##  priors:       see help('prior_summary')
##  observations: 22
##  predictors:   4
## 
## Estimates:
##                          mean   sd     10%    50%    90% 
## (Intercept)             -65.8   74.5 -158.6  -65.6   28.3
## gendermasculine          -0.3   10.8   -7.6    0.1    7.8
## height                    0.8    0.5    0.2    0.8    1.4
## gendermasculine:height    0.1    0.1   -0.1    0.1    0.2
## sigma                    15.9    2.7   12.8   15.5   19.5
## 
## Fit Diagnostics:
##            mean   sd   10%   50%   90%
## mean_PPD 82.6    5.0 76.4  82.7  89.0 
## 
## The mean_ppd is the sample average posterior predictive distribution of the outcome variable (for details see help('summary.stanreg')).
## 
## MCMC diagnostics
##                        mcse Rhat n_eff
## (Intercept)            1.6  1.0  2239 
## gendermasculine        0.4  1.0   908 
## height                 0.0  1.0  2140 
## gendermasculine:height 0.0  1.0  1167 
## sigma                  0.1  1.0  2381 
## mean_PPD               0.1  1.0  3015 
## log-posterior          0.0  1.0  1373 
## 
## For each parameter, mcse is Monte Carlo standard error, n_eff is a crude measure of effective sample size, and Rhat is the potential scale reduction factor on split chains (at convergence Rhat=1).
```

```r
tidy(bayes_reg)
```

```
## Error: $ operator is invalid for atomic vectors
```


### Tables

#### Regression tables

There are loads of [different options](https://hughjonesd.github.io/huxtable/design-principles.html) here.^[FWIW, the **fixest** package also provides its own dedicated function for exporting regression models, namely `etable()`. This function is highly optimised and is capable of producing great looking tables with minimal effort, but is limited to `fixest` model objects only. More [here](https://cran.r-project.org/web/packages/fixest/vignettes/fixest_walkthrough.html#14_viewing_the_results_in_r) and [here](https://cran.r-project.org/web/packages/fixest/vignettes/exporting_tables.html).] These days, however, I find myself using the **modelsummary** package ([link](https://vincentarelbundock.github.io/modelsummary)) for creating and exporting regression tables. It is extremely flexible and handles all manner of models and output formats. **modelsummary** also supports automated coefficient plots and data summary tables, which I'll get back to in a moment. The [documentation](https://vincentarelbundock.github.io/modelsummary/articles/modelsummary.html) is outstanding and you should read it, but here is a bare-boned example just to demonstrate.


```r
# library(modelsummary) ## Already loaded

## Note: msummary() is an alias for modelsummary()
msummary(list(ols_dv2, ols_ie, ols_fe, ols_hdfe))
```

<!--html_preserve--><style>html {
  font-family: -apple-system, BlinkMacSystemFont, 'Segoe UI', Roboto, Oxygen, Ubuntu, Cantarell, 'Helvetica Neue', 'Fira Sans', 'Droid Sans', Arial, sans-serif;
}

#ykllpknaij .gt_table {
  display: table;
  border-collapse: collapse;
  margin-left: auto;
  margin-right: auto;
  color: #333333;
  font-size: 16px;
  background-color: #FFFFFF;
  width: auto;
  border-top-style: solid;
  border-top-width: 2px;
  border-top-color: #A8A8A8;
  border-right-style: none;
  border-right-width: 2px;
  border-right-color: #D3D3D3;
  border-bottom-style: solid;
  border-bottom-width: 2px;
  border-bottom-color: #A8A8A8;
  border-left-style: none;
  border-left-width: 2px;
  border-left-color: #D3D3D3;
}

#ykllpknaij .gt_heading {
  background-color: #FFFFFF;
  text-align: center;
  border-bottom-color: #FFFFFF;
  border-left-style: none;
  border-left-width: 1px;
  border-left-color: #D3D3D3;
  border-right-style: none;
  border-right-width: 1px;
  border-right-color: #D3D3D3;
}

#ykllpknaij .gt_title {
  color: #333333;
  font-size: 125%;
  font-weight: initial;
  padding-top: 4px;
  padding-bottom: 4px;
  border-bottom-color: #FFFFFF;
  border-bottom-width: 0;
}

#ykllpknaij .gt_subtitle {
  color: #333333;
  font-size: 85%;
  font-weight: initial;
  padding-top: 0;
  padding-bottom: 4px;
  border-top-color: #FFFFFF;
  border-top-width: 0;
}

#ykllpknaij .gt_bottom_border {
  border-bottom-style: solid;
  border-bottom-width: 2px;
  border-bottom-color: #D3D3D3;
}

#ykllpknaij .gt_col_headings {
  border-top-style: solid;
  border-top-width: 2px;
  border-top-color: #D3D3D3;
  border-bottom-style: solid;
  border-bottom-width: 2px;
  border-bottom-color: #D3D3D3;
  border-left-style: none;
  border-left-width: 1px;
  border-left-color: #D3D3D3;
  border-right-style: none;
  border-right-width: 1px;
  border-right-color: #D3D3D3;
}

#ykllpknaij .gt_col_heading {
  color: #333333;
  background-color: #FFFFFF;
  font-size: 100%;
  font-weight: normal;
  text-transform: inherit;
  border-left-style: none;
  border-left-width: 1px;
  border-left-color: #D3D3D3;
  border-right-style: none;
  border-right-width: 1px;
  border-right-color: #D3D3D3;
  vertical-align: bottom;
  padding-top: 5px;
  padding-bottom: 6px;
  padding-left: 5px;
  padding-right: 5px;
  overflow-x: hidden;
}

#ykllpknaij .gt_column_spanner_outer {
  color: #333333;
  background-color: #FFFFFF;
  font-size: 100%;
  font-weight: normal;
  text-transform: inherit;
  padding-top: 0;
  padding-bottom: 0;
  padding-left: 4px;
  padding-right: 4px;
}

#ykllpknaij .gt_column_spanner_outer:first-child {
  padding-left: 0;
}

#ykllpknaij .gt_column_spanner_outer:last-child {
  padding-right: 0;
}

#ykllpknaij .gt_column_spanner {
  border-bottom-style: solid;
  border-bottom-width: 2px;
  border-bottom-color: #D3D3D3;
  vertical-align: bottom;
  padding-top: 5px;
  padding-bottom: 6px;
  overflow-x: hidden;
  display: inline-block;
  width: 100%;
}

#ykllpknaij .gt_group_heading {
  padding: 8px;
  color: #333333;
  background-color: #FFFFFF;
  font-size: 100%;
  font-weight: initial;
  text-transform: inherit;
  border-top-style: solid;
  border-top-width: 2px;
  border-top-color: #D3D3D3;
  border-bottom-style: solid;
  border-bottom-width: 2px;
  border-bottom-color: #D3D3D3;
  border-left-style: none;
  border-left-width: 1px;
  border-left-color: #D3D3D3;
  border-right-style: none;
  border-right-width: 1px;
  border-right-color: #D3D3D3;
  vertical-align: middle;
}

#ykllpknaij .gt_empty_group_heading {
  padding: 0.5px;
  color: #333333;
  background-color: #FFFFFF;
  font-size: 100%;
  font-weight: initial;
  border-top-style: solid;
  border-top-width: 2px;
  border-top-color: #D3D3D3;
  border-bottom-style: solid;
  border-bottom-width: 2px;
  border-bottom-color: #D3D3D3;
  vertical-align: middle;
}

#ykllpknaij .gt_striped {
  background-color: rgba(128, 128, 128, 0.05);
}

#ykllpknaij .gt_from_md > :first-child {
  margin-top: 0;
}

#ykllpknaij .gt_from_md > :last-child {
  margin-bottom: 0;
}

#ykllpknaij .gt_row {
  padding-top: 8px;
  padding-bottom: 8px;
  padding-left: 5px;
  padding-right: 5px;
  margin: 10px;
  border-top-style: solid;
  border-top-width: 1px;
  border-top-color: #D3D3D3;
  border-left-style: none;
  border-left-width: 1px;
  border-left-color: #D3D3D3;
  border-right-style: none;
  border-right-width: 1px;
  border-right-color: #D3D3D3;
  vertical-align: middle;
  overflow-x: hidden;
}

#ykllpknaij .gt_stub {
  color: #333333;
  background-color: #FFFFFF;
  font-size: 100%;
  font-weight: initial;
  text-transform: inherit;
  border-right-style: solid;
  border-right-width: 2px;
  border-right-color: #D3D3D3;
  padding-left: 12px;
}

#ykllpknaij .gt_summary_row {
  color: #333333;
  background-color: #FFFFFF;
  text-transform: inherit;
  padding-top: 8px;
  padding-bottom: 8px;
  padding-left: 5px;
  padding-right: 5px;
}

#ykllpknaij .gt_first_summary_row {
  padding-top: 8px;
  padding-bottom: 8px;
  padding-left: 5px;
  padding-right: 5px;
  border-top-style: solid;
  border-top-width: 2px;
  border-top-color: #D3D3D3;
}

#ykllpknaij .gt_grand_summary_row {
  color: #333333;
  background-color: #FFFFFF;
  text-transform: inherit;
  padding-top: 8px;
  padding-bottom: 8px;
  padding-left: 5px;
  padding-right: 5px;
}

#ykllpknaij .gt_first_grand_summary_row {
  padding-top: 8px;
  padding-bottom: 8px;
  padding-left: 5px;
  padding-right: 5px;
  border-top-style: double;
  border-top-width: 6px;
  border-top-color: #D3D3D3;
}

#ykllpknaij .gt_table_body {
  border-top-style: solid;
  border-top-width: 2px;
  border-top-color: #D3D3D3;
  border-bottom-style: solid;
  border-bottom-width: 2px;
  border-bottom-color: #D3D3D3;
}

#ykllpknaij .gt_footnotes {
  color: #333333;
  background-color: #FFFFFF;
  border-bottom-style: none;
  border-bottom-width: 2px;
  border-bottom-color: #D3D3D3;
  border-left-style: none;
  border-left-width: 2px;
  border-left-color: #D3D3D3;
  border-right-style: none;
  border-right-width: 2px;
  border-right-color: #D3D3D3;
}

#ykllpknaij .gt_footnote {
  margin: 0px;
  font-size: 90%;
  padding: 4px;
}

#ykllpknaij .gt_sourcenotes {
  color: #333333;
  background-color: #FFFFFF;
  border-bottom-style: none;
  border-bottom-width: 2px;
  border-bottom-color: #D3D3D3;
  border-left-style: none;
  border-left-width: 2px;
  border-left-color: #D3D3D3;
  border-right-style: none;
  border-right-width: 2px;
  border-right-color: #D3D3D3;
}

#ykllpknaij .gt_sourcenote {
  font-size: 90%;
  padding: 4px;
}

#ykllpknaij .gt_left {
  text-align: left;
}

#ykllpknaij .gt_center {
  text-align: center;
}

#ykllpknaij .gt_right {
  text-align: right;
  font-variant-numeric: tabular-nums;
}

#ykllpknaij .gt_font_normal {
  font-weight: normal;
}

#ykllpknaij .gt_font_bold {
  font-weight: bold;
}

#ykllpknaij .gt_font_italic {
  font-style: italic;
}

#ykllpknaij .gt_super {
  font-size: 65%;
}

#ykllpknaij .gt_footnote_marks {
  font-style: italic;
  font-size: 65%;
}
</style>
<div id="ykllpknaij" style="overflow-x:auto;overflow-y:auto;width:auto;height:auto;"><table class="gt_table">
  
  <thead class="gt_col_headings">
    <tr>
      <th class="gt_col_heading gt_columns_bottom_border gt_left" rowspan="1" colspan="1"> </th>
      <th class="gt_col_heading gt_columns_bottom_border gt_left" rowspan="1" colspan="1">Model 1</th>
      <th class="gt_col_heading gt_columns_bottom_border gt_left" rowspan="1" colspan="1">Model 2</th>
      <th class="gt_col_heading gt_columns_bottom_border gt_left" rowspan="1" colspan="1">Model 3</th>
      <th class="gt_col_heading gt_columns_bottom_border gt_left" rowspan="1" colspan="1">Model 4</th>
    </tr>
  </thead>
  <tbody class="gt_table_body">
    <tr>
      <td class="gt_row gt_left">(Intercept)</td>
      <td class="gt_row gt_left">-84.252</td>
      <td class="gt_row gt_left">-61.000</td>
      <td class="gt_row gt_left"></td>
      <td class="gt_row gt_left"></td>
    </tr>
    <tr>
      <td class="gt_row gt_left"></td>
      <td class="gt_row gt_left">(65.786)</td>
      <td class="gt_row gt_left">(204.057)</td>
      <td class="gt_row gt_left"></td>
      <td class="gt_row gt_left"></td>
    </tr>
    <tr>
      <td class="gt_row gt_left">gendermasculine</td>
      <td class="gt_row gt_left">10.739</td>
      <td class="gt_row gt_left">-15.722</td>
      <td class="gt_row gt_left"></td>
      <td class="gt_row gt_left"></td>
    </tr>
    <tr>
      <td class="gt_row gt_left"></td>
      <td class="gt_row gt_left">(13.197)</td>
      <td class="gt_row gt_left">(219.544)</td>
      <td class="gt_row gt_left"></td>
      <td class="gt_row gt_left"></td>
    </tr>
    <tr>
      <td class="gt_row gt_left">height</td>
      <td class="gt_row gt_left">0.879</td>
      <td class="gt_row gt_left">0.733</td>
      <td class="gt_row gt_left">0.975</td>
      <td class="gt_row gt_left">0.756</td>
    </tr>
    <tr>
      <td class="gt_row gt_left"></td>
      <td class="gt_row gt_left">(0.407)</td>
      <td class="gt_row gt_left">(1.274)</td>
      <td class="gt_row gt_left">(0.044)</td>
      <td class="gt_row gt_left">(0.117)</td>
    </tr>
    <tr>
      <td class="gt_row gt_left">gendermasculine × height</td>
      <td class="gt_row gt_left"></td>
      <td class="gt_row gt_left">0.163</td>
      <td class="gt_row gt_left"></td>
      <td class="gt_row gt_left"></td>
    </tr>
    <tr>
      <td class="gt_row gt_left" style="border-bottom-width: 1px; border-bottom-style: solid; border-bottom-color: #000000;"></td>
      <td class="gt_row gt_left" style="border-bottom-width: 1px; border-bottom-style: solid; border-bottom-color: #000000;"></td>
      <td class="gt_row gt_left" style="border-bottom-width: 1px; border-bottom-style: solid; border-bottom-color: #000000;">(1.349)</td>
      <td class="gt_row gt_left" style="border-bottom-width: 1px; border-bottom-style: solid; border-bottom-color: #000000;"></td>
      <td class="gt_row gt_left" style="border-bottom-width: 1px; border-bottom-style: solid; border-bottom-color: #000000;"></td>
    </tr>
    <tr>
      <td class="gt_row gt_left">Num.Obs.</td>
      <td class="gt_row gt_left">22</td>
      <td class="gt_row gt_left">22</td>
      <td class="gt_row gt_left">58</td>
      <td class="gt_row gt_left">55</td>
    </tr>
    <tr>
      <td class="gt_row gt_left">R2</td>
      <td class="gt_row gt_left">0.444</td>
      <td class="gt_row gt_left">0.444</td>
      <td class="gt_row gt_left">0.997</td>
      <td class="gt_row gt_left">0.998</td>
    </tr>
    <tr>
      <td class="gt_row gt_left">R2 Adj.</td>
      <td class="gt_row gt_left">0.386</td>
      <td class="gt_row gt_left">0.352</td>
      <td class="gt_row gt_left">0.993</td>
      <td class="gt_row gt_left">1.008</td>
    </tr>
    <tr>
      <td class="gt_row gt_left">R2 Pseudo</td>
      <td class="gt_row gt_left"></td>
      <td class="gt_row gt_left"></td>
      <td class="gt_row gt_left"></td>
      <td class="gt_row gt_left"></td>
    </tr>
    <tr>
      <td class="gt_row gt_left">R2 Within</td>
      <td class="gt_row gt_left"></td>
      <td class="gt_row gt_left"></td>
      <td class="gt_row gt_left">0.662</td>
      <td class="gt_row gt_left">0.487</td>
    </tr>
    <tr>
      <td class="gt_row gt_left">AIC</td>
      <td class="gt_row gt_left">186.9</td>
      <td class="gt_row gt_left">188.9</td>
      <td class="gt_row gt_left">492.1</td>
      <td class="gt_row gt_left">513.1</td>
    </tr>
    <tr>
      <td class="gt_row gt_left">BIC</td>
      <td class="gt_row gt_left">191.3</td>
      <td class="gt_row gt_left">194.4</td>
      <td class="gt_row gt_left">558.0</td>
      <td class="gt_row gt_left">649.6</td>
    </tr>
    <tr>
      <td class="gt_row gt_left">Log.Lik.</td>
      <td class="gt_row gt_left">-89.465</td>
      <td class="gt_row gt_left">-89.456</td>
      <td class="gt_row gt_left">-214.026</td>
      <td class="gt_row gt_left">-188.552</td>
    </tr>
    <tr>
      <td class="gt_row gt_left">FE ×   homeworld</td>
      <td class="gt_row gt_left"></td>
      <td class="gt_row gt_left"></td>
      <td class="gt_row gt_left"></td>
      <td class="gt_row gt_left">X</td>
    </tr>
    <tr>
      <td class="gt_row gt_left">FE ×   species</td>
      <td class="gt_row gt_left"></td>
      <td class="gt_row gt_left"></td>
      <td class="gt_row gt_left">X</td>
      <td class="gt_row gt_left">X</td>
    </tr>
  </tbody>
  
  
</table></div><!--/html_preserve-->

</br>
One nice thing about **modelsummary** is that it plays very well with Rmarkdown and will automatically coerce your tables to the format that matches your document output: HTML, LaTeX/PDF, RTF, etc. Of course, you can also [specify the output type](https://vincentarelbundock.github.io/modelsummary/#saving-and-viewing-output-formats) if you aren't using Rmarkdown and want to export a table for later use. Finally, you can even specify special table formats like *threepartable* for LaTeX and, provided that you have called the necessary packages in your preamble, it will render correctly (see example [here](https://twitter.com/VincentAB/status/1265255622943150081)).

#### Summary tables

A variety of summary tables --- balance, correlation, etc. --- can be produced by the companion set of `modelsummary::datasummary*()` functions. Again, you should read the [documentation](https://vincentarelbundock.github.io/modelsummary/articles/datasummary.html) to see all of the options. But here's an example of a very simple balance table using a subset of our "humans" data frame.


```r
datasummary_balance(~ gender,
                    data = select(humans, gender, height, mass, birth_year, eye_color))
```

<!--html_preserve--><style>html {
  font-family: -apple-system, BlinkMacSystemFont, 'Segoe UI', Roboto, Oxygen, Ubuntu, Cantarell, 'Helvetica Neue', 'Fira Sans', 'Droid Sans', Arial, sans-serif;
}

#nqftesesrs .gt_table {
  display: table;
  border-collapse: collapse;
  margin-left: auto;
  margin-right: auto;
  color: #333333;
  font-size: 16px;
  background-color: #FFFFFF;
  width: auto;
  border-top-style: solid;
  border-top-width: 2px;
  border-top-color: #A8A8A8;
  border-right-style: none;
  border-right-width: 2px;
  border-right-color: #D3D3D3;
  border-bottom-style: solid;
  border-bottom-width: 2px;
  border-bottom-color: #A8A8A8;
  border-left-style: none;
  border-left-width: 2px;
  border-left-color: #D3D3D3;
}

#nqftesesrs .gt_heading {
  background-color: #FFFFFF;
  text-align: center;
  border-bottom-color: #FFFFFF;
  border-left-style: none;
  border-left-width: 1px;
  border-left-color: #D3D3D3;
  border-right-style: none;
  border-right-width: 1px;
  border-right-color: #D3D3D3;
}

#nqftesesrs .gt_title {
  color: #333333;
  font-size: 125%;
  font-weight: initial;
  padding-top: 4px;
  padding-bottom: 4px;
  border-bottom-color: #FFFFFF;
  border-bottom-width: 0;
}

#nqftesesrs .gt_subtitle {
  color: #333333;
  font-size: 85%;
  font-weight: initial;
  padding-top: 0;
  padding-bottom: 4px;
  border-top-color: #FFFFFF;
  border-top-width: 0;
}

#nqftesesrs .gt_bottom_border {
  border-bottom-style: solid;
  border-bottom-width: 2px;
  border-bottom-color: #D3D3D3;
}

#nqftesesrs .gt_col_headings {
  border-top-style: solid;
  border-top-width: 2px;
  border-top-color: #D3D3D3;
  border-bottom-style: solid;
  border-bottom-width: 2px;
  border-bottom-color: #D3D3D3;
  border-left-style: none;
  border-left-width: 1px;
  border-left-color: #D3D3D3;
  border-right-style: none;
  border-right-width: 1px;
  border-right-color: #D3D3D3;
}

#nqftesesrs .gt_col_heading {
  color: #333333;
  background-color: #FFFFFF;
  font-size: 100%;
  font-weight: normal;
  text-transform: inherit;
  border-left-style: none;
  border-left-width: 1px;
  border-left-color: #D3D3D3;
  border-right-style: none;
  border-right-width: 1px;
  border-right-color: #D3D3D3;
  vertical-align: bottom;
  padding-top: 5px;
  padding-bottom: 6px;
  padding-left: 5px;
  padding-right: 5px;
  overflow-x: hidden;
}

#nqftesesrs .gt_column_spanner_outer {
  color: #333333;
  background-color: #FFFFFF;
  font-size: 100%;
  font-weight: normal;
  text-transform: inherit;
  padding-top: 0;
  padding-bottom: 0;
  padding-left: 4px;
  padding-right: 4px;
}

#nqftesesrs .gt_column_spanner_outer:first-child {
  padding-left: 0;
}

#nqftesesrs .gt_column_spanner_outer:last-child {
  padding-right: 0;
}

#nqftesesrs .gt_column_spanner {
  border-bottom-style: solid;
  border-bottom-width: 2px;
  border-bottom-color: #D3D3D3;
  vertical-align: bottom;
  padding-top: 5px;
  padding-bottom: 6px;
  overflow-x: hidden;
  display: inline-block;
  width: 100%;
}

#nqftesesrs .gt_group_heading {
  padding: 8px;
  color: #333333;
  background-color: #FFFFFF;
  font-size: 100%;
  font-weight: initial;
  text-transform: inherit;
  border-top-style: solid;
  border-top-width: 2px;
  border-top-color: #D3D3D3;
  border-bottom-style: solid;
  border-bottom-width: 2px;
  border-bottom-color: #D3D3D3;
  border-left-style: none;
  border-left-width: 1px;
  border-left-color: #D3D3D3;
  border-right-style: none;
  border-right-width: 1px;
  border-right-color: #D3D3D3;
  vertical-align: middle;
}

#nqftesesrs .gt_empty_group_heading {
  padding: 0.5px;
  color: #333333;
  background-color: #FFFFFF;
  font-size: 100%;
  font-weight: initial;
  border-top-style: solid;
  border-top-width: 2px;
  border-top-color: #D3D3D3;
  border-bottom-style: solid;
  border-bottom-width: 2px;
  border-bottom-color: #D3D3D3;
  vertical-align: middle;
}

#nqftesesrs .gt_striped {
  background-color: rgba(128, 128, 128, 0.05);
}

#nqftesesrs .gt_from_md > :first-child {
  margin-top: 0;
}

#nqftesesrs .gt_from_md > :last-child {
  margin-bottom: 0;
}

#nqftesesrs .gt_row {
  padding-top: 8px;
  padding-bottom: 8px;
  padding-left: 5px;
  padding-right: 5px;
  margin: 10px;
  border-top-style: solid;
  border-top-width: 1px;
  border-top-color: #D3D3D3;
  border-left-style: none;
  border-left-width: 1px;
  border-left-color: #D3D3D3;
  border-right-style: none;
  border-right-width: 1px;
  border-right-color: #D3D3D3;
  vertical-align: middle;
  overflow-x: hidden;
}

#nqftesesrs .gt_stub {
  color: #333333;
  background-color: #FFFFFF;
  font-size: 100%;
  font-weight: initial;
  text-transform: inherit;
  border-right-style: solid;
  border-right-width: 2px;
  border-right-color: #D3D3D3;
  padding-left: 12px;
}

#nqftesesrs .gt_summary_row {
  color: #333333;
  background-color: #FFFFFF;
  text-transform: inherit;
  padding-top: 8px;
  padding-bottom: 8px;
  padding-left: 5px;
  padding-right: 5px;
}

#nqftesesrs .gt_first_summary_row {
  padding-top: 8px;
  padding-bottom: 8px;
  padding-left: 5px;
  padding-right: 5px;
  border-top-style: solid;
  border-top-width: 2px;
  border-top-color: #D3D3D3;
}

#nqftesesrs .gt_grand_summary_row {
  color: #333333;
  background-color: #FFFFFF;
  text-transform: inherit;
  padding-top: 8px;
  padding-bottom: 8px;
  padding-left: 5px;
  padding-right: 5px;
}

#nqftesesrs .gt_first_grand_summary_row {
  padding-top: 8px;
  padding-bottom: 8px;
  padding-left: 5px;
  padding-right: 5px;
  border-top-style: double;
  border-top-width: 6px;
  border-top-color: #D3D3D3;
}

#nqftesesrs .gt_table_body {
  border-top-style: solid;
  border-top-width: 2px;
  border-top-color: #D3D3D3;
  border-bottom-style: solid;
  border-bottom-width: 2px;
  border-bottom-color: #D3D3D3;
}

#nqftesesrs .gt_footnotes {
  color: #333333;
  background-color: #FFFFFF;
  border-bottom-style: none;
  border-bottom-width: 2px;
  border-bottom-color: #D3D3D3;
  border-left-style: none;
  border-left-width: 2px;
  border-left-color: #D3D3D3;
  border-right-style: none;
  border-right-width: 2px;
  border-right-color: #D3D3D3;
}

#nqftesesrs .gt_footnote {
  margin: 0px;
  font-size: 90%;
  padding: 4px;
}

#nqftesesrs .gt_sourcenotes {
  color: #333333;
  background-color: #FFFFFF;
  border-bottom-style: none;
  border-bottom-width: 2px;
  border-bottom-color: #D3D3D3;
  border-left-style: none;
  border-left-width: 2px;
  border-left-color: #D3D3D3;
  border-right-style: none;
  border-right-width: 2px;
  border-right-color: #D3D3D3;
}

#nqftesesrs .gt_sourcenote {
  font-size: 90%;
  padding: 4px;
}

#nqftesesrs .gt_left {
  text-align: left;
}

#nqftesesrs .gt_center {
  text-align: center;
}

#nqftesesrs .gt_right {
  text-align: right;
  font-variant-numeric: tabular-nums;
}

#nqftesesrs .gt_font_normal {
  font-weight: normal;
}

#nqftesesrs .gt_font_bold {
  font-weight: bold;
}

#nqftesesrs .gt_font_italic {
  font-style: italic;
}

#nqftesesrs .gt_super {
  font-size: 65%;
}

#nqftesesrs .gt_footnote_marks {
  font-style: italic;
  font-size: 65%;
}
</style>
<div id="nqftesesrs" style="overflow-x:auto;overflow-y:auto;width:auto;height:auto;"><table class="gt_table">
  
  <thead class="gt_col_headings">
    <tr>
      <th class="gt_col_heading gt_center gt_columns_bottom_border" rowspan="2" colspan="1"> </th>
      <th class="gt_center gt_columns_top_border gt_column_spanner_outer" rowspan="1" colspan="2">
        <span class="gt_column_spanner">feminine (N=9)</span>
      </th>
      <th class="gt_center gt_columns_top_border gt_column_spanner_outer" rowspan="1" colspan="2">
        <span class="gt_column_spanner">masculine (N=26)</span>
      </th>
      <th class="gt_col_heading gt_center gt_columns_bottom_border" rowspan="2" colspan="1">Diff. in Means</th>
      <th class="gt_col_heading gt_center gt_columns_bottom_border" rowspan="2" colspan="1">Std. Error</th>
    </tr>
    <tr>
      <th class="gt_col_heading gt_columns_bottom_border gt_center" rowspan="1" colspan="1">Mean</th>
      <th class="gt_col_heading gt_columns_bottom_border gt_center" rowspan="1" colspan="1">Std. Dev.</th>
      <th class="gt_col_heading gt_columns_bottom_border gt_center" rowspan="1" colspan="1">Mean </th>
      <th class="gt_col_heading gt_columns_bottom_border gt_center" rowspan="1" colspan="1">Std. Dev. </th>
    </tr>
  </thead>
  <tbody class="gt_table_body">
    <tr>
      <td class="gt_row gt_left">height</td>
      <td class="gt_row gt_right">160.2</td>
      <td class="gt_row gt_right">7.0</td>
      <td class="gt_row gt_right">182.3</td>
      <td class="gt_row gt_right">8.2</td>
      <td class="gt_row gt_right">22.1</td>
      <td class="gt_row gt_right">3.0</td>
    </tr>
    <tr>
      <td class="gt_row gt_left">mass</td>
      <td class="gt_row gt_right">56.3</td>
      <td class="gt_row gt_right">16.3</td>
      <td class="gt_row gt_right">87.0</td>
      <td class="gt_row gt_right">16.5</td>
      <td class="gt_row gt_right">30.6</td>
      <td class="gt_row gt_right">10.1</td>
    </tr>
    <tr>
      <td class="gt_row gt_left" style="border-bottom-width: 1px; border-bottom-style: solid; border-bottom-color: #000000;">birth_year</td>
      <td class="gt_row gt_right" style="border-bottom-width: 1px; border-bottom-style: solid; border-bottom-color: #000000;">46.4</td>
      <td class="gt_row gt_right" style="border-bottom-width: 1px; border-bottom-style: solid; border-bottom-color: #000000;">18.8</td>
      <td class="gt_row gt_right" style="border-bottom-width: 1px; border-bottom-style: solid; border-bottom-color: #000000;">55.2</td>
      <td class="gt_row gt_right" style="border-bottom-width: 1px; border-bottom-style: solid; border-bottom-color: #000000;">26.0</td>
      <td class="gt_row gt_right" style="border-bottom-width: 1px; border-bottom-style: solid; border-bottom-color: #000000;">8.8</td>
      <td class="gt_row gt_right" style="border-bottom-width: 1px; border-bottom-style: solid; border-bottom-color: #000000;">10.2</td>
    </tr>
    <tr>
      <td class="gt_row gt_left"></td>
      <td class="gt_row gt_right">N</td>
      <td class="gt_row gt_right">%</td>
      <td class="gt_row gt_right">N</td>
      <td class="gt_row gt_right">%</td>
      <td class="gt_row gt_right"></td>
      <td class="gt_row gt_right"></td>
    </tr>
    <tr>
      <td class="gt_row gt_left">blue</td>
      <td class="gt_row gt_right">3</td>
      <td class="gt_row gt_right">33</td>
      <td class="gt_row gt_right">9</td>
      <td class="gt_row gt_right">35</td>
      <td class="gt_row gt_right"></td>
      <td class="gt_row gt_right"></td>
    </tr>
    <tr>
      <td class="gt_row gt_left">blue-gray</td>
      <td class="gt_row gt_right">0</td>
      <td class="gt_row gt_right">0</td>
      <td class="gt_row gt_right">1</td>
      <td class="gt_row gt_right">4</td>
      <td class="gt_row gt_right"></td>
      <td class="gt_row gt_right"></td>
    </tr>
    <tr>
      <td class="gt_row gt_left">brown</td>
      <td class="gt_row gt_right">5</td>
      <td class="gt_row gt_right">56</td>
      <td class="gt_row gt_right">12</td>
      <td class="gt_row gt_right">46</td>
      <td class="gt_row gt_right"></td>
      <td class="gt_row gt_right"></td>
    </tr>
    <tr>
      <td class="gt_row gt_left">dark</td>
      <td class="gt_row gt_right">0</td>
      <td class="gt_row gt_right">0</td>
      <td class="gt_row gt_right">1</td>
      <td class="gt_row gt_right">4</td>
      <td class="gt_row gt_right"></td>
      <td class="gt_row gt_right"></td>
    </tr>
    <tr>
      <td class="gt_row gt_left">hazel</td>
      <td class="gt_row gt_right">1</td>
      <td class="gt_row gt_right">11</td>
      <td class="gt_row gt_right">1</td>
      <td class="gt_row gt_right">4</td>
      <td class="gt_row gt_right"></td>
      <td class="gt_row gt_right"></td>
    </tr>
    <tr>
      <td class="gt_row gt_left">yellow</td>
      <td class="gt_row gt_right">0</td>
      <td class="gt_row gt_right">0</td>
      <td class="gt_row gt_right">2</td>
      <td class="gt_row gt_right">8</td>
      <td class="gt_row gt_right"></td>
      <td class="gt_row gt_right"></td>
    </tr>
  </tbody>
  
  
</table></div><!--/html_preserve-->

</br>
Another package that I like a lot in this regard is **vtable** ([link](https://nickch-k.github.io/vtable)). Not only can it be used to construct descriptive labels like you'd find in Stata's "Variables" pane, but it is also very good at producing the type of "out of the box" summary tables that economists like. For example, here's the equivalent version of the above balance table.


```r
# library(vtable) ## Already loaded

## st() is an alias for sumtable()
st(select(humans, gender, height, mass, birth_year, eye_color), 
   group = 'gender')
```

<table>
<caption>Summary Statistics</caption>
 <thead>
<tr>
<th style="border-bottom:hidden; padding-bottom:0; padding-left:3px;padding-right:3px;text-align: center; " colspan="1"><div style="border-bottom: 1px solid #ddd; padding-bottom: 5px; ">gender</div></th>
<th style="border-bottom:hidden; padding-bottom:0; padding-left:3px;padding-right:3px;text-align: center; " colspan="3"><div style="border-bottom: 1px solid #ddd; padding-bottom: 5px; ">feminine</div></th>
<th style="border-bottom:hidden; padding-bottom:0; padding-left:3px;padding-right:3px;text-align: center; " colspan="3"><div style="border-bottom: 1px solid #ddd; padding-bottom: 5px; ">masculine</div></th>
</tr>
  <tr>
   <th style="text-align:left;"> Variable </th>
   <th style="text-align:left;"> N </th>
   <th style="text-align:left;"> Mean </th>
   <th style="text-align:left;"> SD </th>
   <th style="text-align:left;"> N </th>
   <th style="text-align:left;"> Mean </th>
   <th style="text-align:left;"> SD </th>
  </tr>
 </thead>
<tbody>
  <tr>
   <td style="text-align:left;"> height </td>
   <td style="text-align:left;"> 8 </td>
   <td style="text-align:left;"> 160.25 </td>
   <td style="text-align:left;"> 6.985 </td>
   <td style="text-align:left;"> 23 </td>
   <td style="text-align:left;"> 182.348 </td>
   <td style="text-align:left;"> 8.189 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> mass </td>
   <td style="text-align:left;"> 3 </td>
   <td style="text-align:left;"> 56.333 </td>
   <td style="text-align:left;"> 16.289 </td>
   <td style="text-align:left;"> 19 </td>
   <td style="text-align:left;"> 86.958 </td>
   <td style="text-align:left;"> 16.549 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> birth_year </td>
   <td style="text-align:left;"> 5 </td>
   <td style="text-align:left;"> 46.4 </td>
   <td style="text-align:left;"> 18.77 </td>
   <td style="text-align:left;"> 20 </td>
   <td style="text-align:left;"> 55.165 </td>
   <td style="text-align:left;"> 26.02 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> eye_color </td>
   <td style="text-align:left;"> 9 </td>
   <td style="text-align:left;">  </td>
   <td style="text-align:left;">  </td>
   <td style="text-align:left;"> 26 </td>
   <td style="text-align:left;">  </td>
   <td style="text-align:left;">  </td>
  </tr>
  <tr>
   <td style="text-align:left;"> ... blue </td>
   <td style="text-align:left;"> 3 </td>
   <td style="text-align:left;"> 33.3% </td>
   <td style="text-align:left;">  </td>
   <td style="text-align:left;"> 9 </td>
   <td style="text-align:left;"> 34.6% </td>
   <td style="text-align:left;">  </td>
  </tr>
  <tr>
   <td style="text-align:left;"> ... blue-gray </td>
   <td style="text-align:left;"> 0 </td>
   <td style="text-align:left;"> 0% </td>
   <td style="text-align:left;">  </td>
   <td style="text-align:left;"> 1 </td>
   <td style="text-align:left;"> 3.8% </td>
   <td style="text-align:left;">  </td>
  </tr>
  <tr>
   <td style="text-align:left;"> ... brown </td>
   <td style="text-align:left;"> 5 </td>
   <td style="text-align:left;"> 55.6% </td>
   <td style="text-align:left;">  </td>
   <td style="text-align:left;"> 12 </td>
   <td style="text-align:left;"> 46.2% </td>
   <td style="text-align:left;">  </td>
  </tr>
  <tr>
   <td style="text-align:left;"> ... dark </td>
   <td style="text-align:left;"> 0 </td>
   <td style="text-align:left;"> 0% </td>
   <td style="text-align:left;">  </td>
   <td style="text-align:left;"> 1 </td>
   <td style="text-align:left;"> 3.8% </td>
   <td style="text-align:left;">  </td>
  </tr>
  <tr>
   <td style="text-align:left;"> ... hazel </td>
   <td style="text-align:left;"> 1 </td>
   <td style="text-align:left;"> 11.1% </td>
   <td style="text-align:left;">  </td>
   <td style="text-align:left;"> 1 </td>
   <td style="text-align:left;"> 3.8% </td>
   <td style="text-align:left;">  </td>
  </tr>
  <tr>
   <td style="text-align:left;"> ... yellow </td>
   <td style="text-align:left;"> 0 </td>
   <td style="text-align:left;"> 0% </td>
   <td style="text-align:left;">  </td>
   <td style="text-align:left;"> 2 </td>
   <td style="text-align:left;"> 7.7% </td>
   <td style="text-align:left;">  </td>
  </tr>
</tbody>
</table>

</br>
In case you were wondering, `vtable::st()` does a clever job of automatically picking defaults and dropping "unreasonable" variables (e.g. list variables or factors with too many levels). Here's what we get if we just ask it to produce a summary table of the main "starwars" data frame.


```r
st(starwars)
```

<table>
<caption>Summary Statistics</caption>
 <thead>
  <tr>
   <th style="text-align:left;"> Variable </th>
   <th style="text-align:left;"> N </th>
   <th style="text-align:left;"> Mean </th>
   <th style="text-align:left;"> Std. Dev. </th>
   <th style="text-align:left;"> Min </th>
   <th style="text-align:left;"> Pctl. 25 </th>
   <th style="text-align:left;"> Pctl. 75 </th>
   <th style="text-align:left;"> Max </th>
  </tr>
 </thead>
<tbody>
  <tr>
   <td style="text-align:left;"> height </td>
   <td style="text-align:left;"> 81 </td>
   <td style="text-align:left;"> 174.358 </td>
   <td style="text-align:left;"> 34.77 </td>
   <td style="text-align:left;"> 66 </td>
   <td style="text-align:left;"> 167 </td>
   <td style="text-align:left;"> 191 </td>
   <td style="text-align:left;"> 264 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> mass </td>
   <td style="text-align:left;"> 59 </td>
   <td style="text-align:left;"> 97.312 </td>
   <td style="text-align:left;"> 169.457 </td>
   <td style="text-align:left;"> 15 </td>
   <td style="text-align:left;"> 55.6 </td>
   <td style="text-align:left;"> 84.5 </td>
   <td style="text-align:left;"> 1358 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> birth_year </td>
   <td style="text-align:left;"> 43 </td>
   <td style="text-align:left;"> 87.565 </td>
   <td style="text-align:left;"> 154.691 </td>
   <td style="text-align:left;"> 8 </td>
   <td style="text-align:left;"> 35 </td>
   <td style="text-align:left;"> 72 </td>
   <td style="text-align:left;"> 896 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> sex </td>
   <td style="text-align:left;"> 83 </td>
   <td style="text-align:left;">  </td>
   <td style="text-align:left;">  </td>
   <td style="text-align:left;">  </td>
   <td style="text-align:left;">  </td>
   <td style="text-align:left;">  </td>
   <td style="text-align:left;">  </td>
  </tr>
  <tr>
   <td style="text-align:left;"> ... female </td>
   <td style="text-align:left;"> 16 </td>
   <td style="text-align:left;"> 19.3% </td>
   <td style="text-align:left;">  </td>
   <td style="text-align:left;">  </td>
   <td style="text-align:left;">  </td>
   <td style="text-align:left;">  </td>
   <td style="text-align:left;">  </td>
  </tr>
  <tr>
   <td style="text-align:left;"> ... hermaphroditic </td>
   <td style="text-align:left;"> 1 </td>
   <td style="text-align:left;"> 1.2% </td>
   <td style="text-align:left;">  </td>
   <td style="text-align:left;">  </td>
   <td style="text-align:left;">  </td>
   <td style="text-align:left;">  </td>
   <td style="text-align:left;">  </td>
  </tr>
  <tr>
   <td style="text-align:left;"> ... male </td>
   <td style="text-align:left;"> 60 </td>
   <td style="text-align:left;"> 72.3% </td>
   <td style="text-align:left;">  </td>
   <td style="text-align:left;">  </td>
   <td style="text-align:left;">  </td>
   <td style="text-align:left;">  </td>
   <td style="text-align:left;">  </td>
  </tr>
  <tr>
   <td style="text-align:left;"> ... none </td>
   <td style="text-align:left;"> 6 </td>
   <td style="text-align:left;"> 7.2% </td>
   <td style="text-align:left;">  </td>
   <td style="text-align:left;">  </td>
   <td style="text-align:left;">  </td>
   <td style="text-align:left;">  </td>
   <td style="text-align:left;">  </td>
  </tr>
  <tr>
   <td style="text-align:left;"> gender </td>
   <td style="text-align:left;"> 83 </td>
   <td style="text-align:left;">  </td>
   <td style="text-align:left;">  </td>
   <td style="text-align:left;">  </td>
   <td style="text-align:left;">  </td>
   <td style="text-align:left;">  </td>
   <td style="text-align:left;">  </td>
  </tr>
  <tr>
   <td style="text-align:left;"> ... feminine </td>
   <td style="text-align:left;"> 17 </td>
   <td style="text-align:left;"> 20.5% </td>
   <td style="text-align:left;">  </td>
   <td style="text-align:left;">  </td>
   <td style="text-align:left;">  </td>
   <td style="text-align:left;">  </td>
   <td style="text-align:left;">  </td>
  </tr>
  <tr>
   <td style="text-align:left;"> ... masculine </td>
   <td style="text-align:left;"> 66 </td>
   <td style="text-align:left;"> 79.5% </td>
   <td style="text-align:left;">  </td>
   <td style="text-align:left;">  </td>
   <td style="text-align:left;">  </td>
   <td style="text-align:left;">  </td>
   <td style="text-align:left;">  </td>
  </tr>
</tbody>
</table>


### Figures

#### Coefficient plots

We've already worked through several examples of how to extract and compare model coefficients (e.g. [here](#Comparing_our_model_coefficients)). I use this "manual" approach to visualizing coefficient estimates all the time. However, our focus on **modelsummary** in the preceding section provides a nice segue to another one of the package's features: [`modelplot()`](https://vincentarelbundock.github.io/modelsummary/articles/modelplot.html). Consider the following, which shows both the degree to which `modelplot()` automates everything and the fact that it readily accepts regular **ggplot2** syntax.


```r
# library(modelsummary) ## Already loaded
mods = list('FE, no clustering' = summary(ols_fe, se = 'standard'),  # Don't cluster SEs 
            'HDFE, twoway clustering' = ols_hdfe)

modelplot(mods) +
  coord_flip() ## You can modify with normal ggplot2 commands
```

![](08-regression_files/figure-html/modplot-1.png)<!-- -->

#### Model validation and prediction

The easiest way to visually inspect model performance (i.e. validation and prediction) is with **ggplot2**. In particular, you should already be familiar with `geom_smooth()` from our earlier lectures. For instance,


```r
humans %>%
  ggplot(aes(x = mass, y = height, col = gender)) + 
  geom_point(alpha = 0.7) +
  geom_smooth(method = "lm", se = FALSE) + ## See ?geom_smooth for other methods
  scale_color_brewer(palette = "Set1")
```

```
## `geom_smooth()` using formula 'y ~ x'
```

![](08-regression_files/figure-html/smooth-1.png)<!-- -->

For further reference on data visualization with models, I highly encourage you to look over Chapter 6 of Kieran Healy's [*Data Visualization: A Practical Guide*](https://socviz.co/modeling.html). You will not only learn how to produce beautiful and effective model visualizations, but also pick up a variety of technical tips. I recommend that you pay particular attention attention to the section on [generating and plotting predictions](https://socviz.co/modeling.html#generate-predictions-to-graph), since that will form part of your next assignment.

## Further resources

- [Ed Rubin](https://twitter.com/edrubin) has outstanding [teaching notes](http://edrub.in/teaching.html) for econometrics with R on his website. This includes both [undergrad-](https://github.com/edrubin/EC421S19) and [graduate-](https://github.com/edrubin/EC525S19)level courses. 
- Speaking of books, several introductory texts are freely available, including [*Introduction to Econometrics with R*](https://www.econometrics-with-r.org/) (Christoph Hanck *et al.*) and [*Using R for Introductory Econometrics*](http://www.urfie.net/) (Florian Heiss).
- [Tyler Ransom](https://twitter.com/tyleransom) has a nice [cheat sheet](https://github.com/tyleransom/EconometricsLabs/blob/master/tidyRcheatsheet.pdf) for common regression tasks and specifications.
- [Itamar Caspi](https://twitter.com/itamarcaspi) has written a neat unofficial appendix to this lecture, [*recipes for Dummies*](https://itamarcaspi.rbind.io/post/recipes-for-dummies/). The title might be a little inscrutable if you haven't heard of the `recipes` package before, but basically it handles "tidy" data preprocessing, which is an especially important topic for machine learning methods. We'll get to that later in course, but check out Itamar's post for a good introduction.
- I promised to provide some links to time series analysis. The good news is that R's support for time series is very, very good. The [Time Series Analysis](https://cran.r-project.org/web/views/TimeSeries.html) task view on CRAN offers an excellent overview of available packages and their functionality.
