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
Now, we need to load in the dataset we are going to use, coming from the .csv file we named `collegedata.csv`
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

We can go deeper and make a barplot showing the average parental mean income for each college
tier. First, we need to make a table to simplify the data so it is easier to code.

```r
table = college %>% group_by(collegetier) %>%
  summarize(avg_pmean = mean(pmean))
view(table)
```

`%>%` this function is a pipe operator, and is similar in use to an f(x) function. It is in the `dplyr`
package, so make sure that is loaded before using this function. Here, it allows us to name this function `table` while also allowing us to tell the computer to use the `college` dataset and use the `group_by` function simultaneously.

`group_by` this function allows us to choose the variable we want to categorize
our barplot with.

`avg_pmean = mean(pmean)` this sets the other half of the table, the response, as the
average of the mean parental income. This function tells the computer to split up the parental mean income by college tier, and to then find the mean parental income of each college tier. So, our graph will show the parental mean income of each college tier.

```r
table %>% ggplot(aes(x = collegetier, y = avg_pmean)) +
  geom_col(aes(fill = collegetier)) + 
  labs(title = "Parental Income and College Tier", x = "College Tier", 
       y = "Parental Mean Income") 

```
Now, we have a graph where the x axis is each college tier, and the height, or y axis, is determined by the mean parental income of that college tier.

We set the x variable to college tier, and the height to the parental mean income, the variable we created earlier. 

`geom_col` makes the heights of graphs representative of values in the data, while
`geom_bar` makes the heights of the graphs representative of the counts in the data. For example, if we used `geom_bar`, the height of each column would be representative of how many cases are in each tier. For example, if there were 60 people in the first tier, the height of tier 1 would be 60, not the parental mean income.

`labs` assigns labels to the axes

If we wanted to flip the axes, we can use the `coord_flip()` function

```r
table %>% ggplot(aes(x = collegetier, y = avg_pmean)) +
  geom_col(aes(fill = collegetier)) +
  labs(title = "Parental Income and College Tier", x = "College Tier",
       y = "Parental Mean Income") +
  coord_flip()
```

## Creating a Barplot in Base R

In base R, we can create items without using outside packages, such as GGPlot2. Here, we are going to create a barplot with the cohort dataset.

```r
# insert dataset and assign it a name
co = read.csv("~/downloads/cohort.csv")

# create barplot
barplot(cohort)
```
We can add different labels for the x and y axes or the main title.

```r
barplot(cohort, xlab = "Parent Income",
        ylab = "Count", main = "Parental Income")
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
