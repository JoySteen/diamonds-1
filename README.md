<!-- README.md is generated from README.Rmd. Please edit that file -->



Diamonds
========

This page is intended to accompany the article *Seeing Diamonds: Statistical Graphics in the Introductory Course* by David Kahle.

Setting up the basic elements
-----------------------------

Load in the required libraries.

``` {.R}
library(ggplot2); theme_set(theme_bw(14))
library(plyr)
library(reshape2)
```

Export the diamonds dataset as a .csv file
------------------------------------------

``` {.R}
# write.csv(diamonds, "diamonds.csv", row.names = FALSE)
file.show("diamonds.csv")

length(unique(diamonds$price))

n <- nrow(diamonds)
p <- ncol(diamonds)
```

One dimensional discrete graphics
---------------------------------

Compute the frequency and relative frequency distributions.

``` {.R}
table(diamonds$cut)
round(table(diamonds$cut) / n, 3)
sum(round(table(diamonds$cut) / n, 3)) # take the .001 off of ideal
```

The bar chart:

``` {.R}
qplot(cut, data = diamonds) +
   xlab("Cut") + ylab("Frequency")
```

The pie chart:

``` {.R}
ggplot(
    aes(x = 1, y = V1, fill = cut), 
   data = ddply(diamonds, "cut", nrow)
  ) +
  geom_bar(stat = "identity") +
  coord_polar(theta = "y") +
  scale_x_continuous("", breaks = NULL) +
  scale_y_continuous("", breaks = NULL)
```

One dimensional continuous graphics
-----------------------------------

Extract the 50 diamonds to make the graphics:

``` {.R}
set.seed(1)
fiftyRows <- sample(n, 50)
fiftyDiamonds <- diamonds[fiftyRows,]
```

### Scatterplots

Make the 1d scatterplot:

``` {.R}
qplot(carat, 0L, data = fiftyDiamonds, size = I(8)) +
  geom_hline(yintercept = 0) +
  scale_x_continuous("Carat", lim = c(0,2.5)) +
  scale_y_continuous("", lim = c(-1,1), breaks = NULL)
```

Sample the points to jitter on the *y* axis and then make the jittered scatterplot:

``` {.R}
fiftyDiamonds$yValues <- runif(50, -1, 1)
qplot(carat, yValues, data = fiftyDiamonds, size = I(8)) +
  geom_hline(yintercept = 0) +
  scale_x_continuous("Carat", lim = c(0,2.5)) +
  scale_y_continuous("", lim = c(-1,1), breaks = NULL)
```

Use alpha blending:

``` {.R}
qplot(carat, yValues, data = fiftyDiamonds, size = I(8), alpha = I(.33)) +
  geom_hline(yintercept = 0) +
  scale_x_continuous("Carat", lim = c(0,2.5)) +
  scale_y_continuous("", lim = c(-1,1), breaks = NULL)
```

And resize the points:

``` {.R}
qplot(carat, yValues, data = fiftyDiamonds, size = I(5), alpha = I(.33)) +
  geom_hline(yintercept = 0) +
  scale_x_continuous("Carat", lim = c(0,2.5)) +
  scale_y_continuous("", lim = c(-1,1), breaks = NULL)
```

Do all three at once:

``` {.R}
qplot(carat, runif(n, -1, 1), data = diamonds, size = I(5), alpha = I(.01)) +
  geom_hline(yintercept = 0) +
  scale_x_continuous("Carat", lim = c(0,2.5)) +
  scale_y_continuous("", lim = c(-1,1), breaks = NULL)
```

### Histograms

The basic histogram (binwidth = .17):

``` {.R}
qplot(carat, data = diamonds) +
  scale_x_continuous("Carat", lim = c(0,5)) +
  ylab("Frequency")
```

The histograms with binwidth 2 and .01:

``` {.R}
qplot(carat, data = diamonds, binwidth = 2) +
  scale_x_continuous("Carat", lim = c(0,5)) +
  ylab("Frequency")


qplot(carat, data = diamonds, binwidth = .01) +
  scale_x_continuous("Carat", lim = c(0,5)) +
  ylab("Frequency") + geom_vline(xintercept = .9)
```

Compute the price differences between just under 1 and just over 1 carat diamonds:

``` {.R}
slightlySmallerDiamonds <- subset(diamonds, .99 <= carat & carat < 1)
nrow(slightlySmallerDiamonds)
mean(slightlySmallerDiamonds$price)
median(slightlySmallerDiamonds$price)

slightlyBiggerDiamonds <- subset(diamonds, 1 <= carat & carat < 1.01)
nrow(slightlyBiggerDiamonds)
mean(slightlyBiggerDiamonds$price)
median(slightlyBiggerDiamonds$price)

qplot(price, data = slightlySmallerDiamonds)

qplot(price, data = slightlyBiggerDiamonds)

t.test(
  slightlyBiggerDiamonds$price,
  slightlySmallerDiamonds$price, 
  alternative = "greater"
)
```

### Kernel density estimators

``` {.R}
# the x-axis sequence on which to eval the kernels
s <- seq(0, 2.5, length.out = 501)

# evaluate the kernels at each
little_densities <- lapply(as.list(fiftyDiamonds$carat), function(mu){
  dnorm(s, mu, sd = .05) / 50
})

# aggregate and sum
df <- as.data.frame(t(plyr:::list_to_dataframe(little_densities)))
names(df) <- paste0("mu=", fiftyDiamonds$carat)
df$x <- s
mdf <- melt(df, id = "x")
kde <- ddply(mdf, "x", function(df) sum(df$value) )

# plot
ggplot() +
  geom_line(
    aes(x = x, y = value, group = variable),
    size = .2,
    data = mdf
  ) +
  geom_line(
    aes(x = x, y = V1),
    size = 1, color = "red",
    data = kde
  ) +
  scale_x_continuous("Carat", lim = c(0,2.5)) +
  ylab("Density") 
```

Make the associated histogram:

``` {.R}
qplot(carat, ..density.., data = fiftyDiamonds, geom = "histogram") +
  scale_x_continuous("Carat", lim = c(0,2.5)) +
  ylab("Density")
```

Continuous-continuous graphics
------------------------------

### The two-dimensional scatterplot

The basic 2d scatterplot:

``` {.R}
qplot(carat, price, data = diamonds) +
  scale_x_continuous("Carat") +
  scale_y_continuous("Price")
```

Use alpha blending and resize:

``` {.R}
qplot(carat, price, data = diamonds, alpha = I(.05), size = I(1)) +
  scale_x_continuous("Carat") +
  scale_y_continuous("Price")
```

### The two-dimensional histogram

Make the basic 2d histogram:

``` {.R}
qplot(carat, price, data = diamonds, geom = "bin2d") +
  scale_x_continuous("Carat") +
  scale_y_continuous("Price") +
  scale_fill_continuous("Frequency")
```

Make an elaborate 2d histogram:

``` {.R}
qplot(carat, price, data = diamonds, geom = "hex", bins = 100) +
  scale_x_continuous("Carat") +
  scale_y_continuous("Price") +
  scale_fill_gradientn("Frequency",
    colours = c("#132B43", "#56B1F7", "yellow", "red"),
    values = c(0.00, 0.025, 0.15, 1.00),
    breaks = c(100,250,500,1000,2000)
  )
```

Add a smoother:

``` {.R}
qplot(carat, price, data = diamonds, geom = "hex", bins = 100) +
  stat_smooth(color = "red", size = 2) +
  scale_x_continuous("Carat") +
  scale_y_continuous("Price") +
  scale_fill_gradientn("Frequency",
    colours = c("#132B43", "#56B1F7", "yellow", "red"),
    values = c(0.00, 0.025, 0.15, 1.00),
    breaks = c(100,250,500,1000,2000)
  )
```

### The contour plot

(The one in the paper is made with Mathematica.)

``` {.R}
ggplot(aes(x = carat, y = price), data = diamonds) +
  stat_density2d() +
  scale_x_continuous("Carat", lim = c(0,4)) +
  scale_y_continuous("Price", lim = c(0,18000))
```

Discrete-continuous graphics
----------------------------

The naive scatterplot:

``` {.R}
qplot(clarity, price, data = diamonds) +
  scale_x_discrete("Clarity") +
  scale_y_continuous("Price")
```

Adding jittering and alpha blending:

``` {.R}
qplot(clarity, price, data = diamonds, geom = "jitter", alpha = I(.05)) +
  scale_x_discrete("Clarity") +
  scale_y_continuous("Price")
```

The boxplot:

``` {.R}
qplot(clarity, price, data = diamonds, geom = "boxplot") +
  scale_x_discrete("Clarity") +
  scale_y_continuous("Price")
```

And the violin plot:

``` {.R}
qplot(clarity, price, data = diamonds, geom = "violin") +
  scale_x_discrete("Clarity") +
  scale_y_continuous("Price")
```

The more complex graphic:

``` {.R}
diamonds$size <- cut(diamonds$carat, c(0,.5,1,1.5,2,2.5,5))

qplot(clarity, price, 
    data = subset(diamonds, .5 < carat & carat <=2.5), 
    geom = "boxplot", fill = cut, outlier.size = .25
  ) + facet_grid(size ~ ., scales = "free") +
  scale_x_discrete("Clarity") +
  scale_y_continuous("Price") +
  scale_fill_discrete("Cut")
```

Discrete-discrete graphics
--------------------------

Scatter plot (with jittering, alpha-blending, and resizing):

``` {.R}
qplot(clarity, cut, data = diamonds, geom = "jitter", 
    alpha = I(.05), size = I(1.5)
  ) +
  scale_x_discrete("Clarity") +
  scale_y_discrete("Cut")
```

### Bar charts

The dodged bar chart:

``` {.R}
qplot(clarity, data = diamonds, geom = "bar", 
  fill = cut, position = "dodge")
```

The stacked bar chart:

``` {.R}
qplot(clarity, data = diamonds, geom = "bar", 
  fill = cut, position = "stack")
```

The pre-mosaic plot :

``` {.R}
props <- ddply(diamonds, "clarity", function(df){
  table(df$cut) / nrow(df)
})

mprops <- melt(props, id = "clarity")
ggplot(data = mprops) +
  geom_bar(
    aes(x = clarity, y = value, fill = variable), 
    stat = "identity"
  ) +
  scale_x_discrete("Clarity") +
  scale_y_continuous("Relative Frequency") +
  scale_fill_discrete("Cut")
```

### The mosaic plot

``` {.R}
source("ggmosaic.r")
ggmosaic(clarity, cut, data = diamonds)
```