# Rbarplottutorial
everything about barplots in R!

# Barplot

Barplots are useful for showing the relationship between numeric and categorical variables.
We can make a simple barplot using the college tier as the category and the count as the
y variable:

``` r
# load libraries and datasets
library(tidyverse)
library(ggplot2)
```
Now, we need to load in the dataset we are going to use, coming from the .csv file we named `collegedata.csv` This can be downloaded at the Dropbox link here: https://www.dropbox.com/s/mxpl8uvcis148ip/collegedata.csv?dl=0

We need to assign that dataset to a variable so it is easier to use within R, so we set it equal to the name
`college`

```r
college = read.csv("~/downloads/collegedata.csv")
```

In this barplot, we want to use the college tier, which in the dataset are numbers from 1 - 14, as the factor
at the bottom of our barplot, the x variable. To do this, we use the `as.factor` function which converts a column into a factor. Here, it is already a numeric factor, but if it were for example, the names of 14 colleges, using the `as.factor` function would convert those into factors that could be used as the x axis of the barplot.
``` r
# establish variable
college$collegetier = as.factor(college$collegetier)
```

Now, we use the `barplot()` function to actually make the barplot.

```r
# make barplot
barplot(table(college$collegetier))
```

This makes a very simple bar graph showing the counts for each variable. 

## Using averages for barplot height
We can go deeper and make a barplot showing the average parental mean income for each college
tier. First, we need to create a separate dataframe of averages of the college tiers. To do this, we are going to use the `aggregate()` function:

### Create new dataframe
```r
# load libraries
library(ggplot2)

# import college dataset
college = read.csv("~/downloads/collegedata.csv")

# create new dataframe
college.means = aggregate(x = college,
                          by = list(college$collegetier),
                          FUN = mean)
View(college.means)
```

`college.means` is the name of our new dataframe

`aggregate()` this function is used to group multiple data points together for a singular value rather than multiple datapoints

`x = college` this is referring to the dataset we are pulling from, which is the `college` dataset

`by = list(college$collegetier` This tells the computer which variable we want to sort the data by, which is college tier

`FUN = mean` FUN is shorthand for function, so we are telling the computer the function that we want it to use is the mean.

### Plot data
Now, we are going to use `geom_col()` to actually graph the function. 

```r
ggplot() +
  geom_col(data = college.means,
           aes(x = college.means$collegetier,
               y = college.means$pmean))

```




## Creating a Barplot in Base R

In base R, we can create items without using outside packages, such as GGPlot2. Here, we are going to create a basic barplot of the counts of collegetier

```r
# insert dataset and assign it a name
college = read.csv("~/downloads/collegedata.csv")

# create barplot
barplot(college$collegetier)
```
We can add different labels for the x and y axes or the main title.

```r
barplot(college$collegetier, xlab = "Counts",
        ylab = "College Tier",
        main = "Counts of Students in Each College Tier")

```


## Creating a Dataframe
Sometimes, it is required that we make our own dataset within R, which is called a dataframe. Here,
I have pulled data from an Opportunity Insights dataset. 

``` r
# load libraries
library(ggplot2)
library(tidyverse)
library(dplyr)
library(breakDown)
library(reshape2)

# create a dataframe and stacked barplots
gf = data.frame(
  parentpercentile=c("1", "2", "3", "4", "5"),
  p1=c(.144, .125, .114, .103, .109), 
  p2=c(.216, .195, .178, .163, .162),
  p3=c(.230, .228, .218, .205, .186),
  p4=c(.219, .238, .250, .256, .238),
  p5=c(.190, .214, .240, .272, .305))

view(gf)
```
I calculated these values myself, and you can do this with any set of numbers. `p1`, `p2`, and so 
on represent parental income quintiles, and the percentages given within the dataset represent
the income percentile the children of those parents ended up in. For example, out of the
children born in the first income quintile, 1-20%, 14.4% will end up in the first income quintile
themselves, or 11.4% of those children will end up in the third income quintile.


`gf` is the name of the dataframe we have created, and is how we will refer to our
new dataframe.

`c` indicates that we are using names, and not numerical values.

```r
# compress gf into a ggplot
pf = ggplot(data = gf, aes(x = parentpercentile,
                           y = value,
                           fill = p1))

```

Now, we have compressed `gf` into a `ggplot` named `pf`. 

```r
# melt data
dat.m = melt(gf, id.vars = "parentpercentile")
head(dat.m)
```
`head(dat.m)` this allows us to see the first six lines of `dat.m`


Next, we have to "melt" the data. Melting a dataframe makes it easier for the
computer to read through the dataset, therefore easier for us to work with it.
It formats the data into a longer, skinnier dataset. This makes it much easier to subset,
graph, or do anything else with the data. `head(dat.m)` allows you to view the melted data,
helping us to visualize what "melting" does

```r
dat.m = dat.m %>%
  group_by(parentpercentile) %>%
  mutate(label_y = cumsum(value))
```
Next, we format our melted data so we can graph it next. `group_by(parentpercentile)` 
establishes that we want to group all data by the parental income percentile. The `mutate`
function establishes a new variable, but preserves existing variables.

```r
# graph
ggplot(dat.m, aes(x = parentpercentile, y = value,
                  fill = variable)) +
  geom_bar(stat = "identity")
```
Now we have finally graphed the data. All graphs are the same height because
we are using percentages of a whole. 

`geom_bar(stat = "identity")` this is set equal to "identity" because the data contains
y values in the column, and identity is the default setting to achieve that.

`fill = variable` The fill of the barplot comes from the variables in the dataframe, so 
we set the fill equal to whatever our variables are.

## Grouping Within Barplots

For this example, I am going to use a Titanic dataset from Kaggle, which can be found with this link: https://www.dropbox.com/s/00j17o6xzxsdjo4/titanictest.csv?dl=0

Let's say we want to create a barplot showing the fare of men and women and if they were in first, second, or third class on the ship. 

```r
# name data
train = read.csv("~/downloads/titanictest.csv", stringsAsFactors = FALSE, header = TRUE)
View(train)

# install packages and load them
install.packages("ggplot2")
library(ggplot2)
```

Now, we can begin to create the barplot.

```r
summary_data <- tapply(train$Fare, list(sex = train$Sex,
                                        pclass = train$Pclass),
                       FUN = mean, na.rm = TRUE)
summary_data

> summary_data
        pclass
sex              1        2        3
  female 115.59117 26.43875 13.73513
  male    75.58655 20.18465 11.82635
  ```
  
From this, we can see that women in first class on average paid more for their ticket than did anyone else.

Now, let's graph the data:
```r
barplot.one = barplot(summary_data, xlab = "Class", ylab = "Fare",
        beside = TRUE,
        legend.text = rownames(summary_data),
        args.legend = list(title = "Gender",
                           x = "topright",
                           inset = c(.2,0)))

```

`summary_data` establishes that we want to pull data from the summary data we pulled earlier.

`xlab` and `ylab` are labels for the x and y axes.

`beside = TRUE` tells the computer to put the female and male columns next to each other in the graph.

`args.legend` creates a legend in the graph.
