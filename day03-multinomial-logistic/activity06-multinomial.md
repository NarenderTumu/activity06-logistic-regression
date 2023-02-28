Activity 6 - Multinomial Logistic Regression
================

``` r
library(tidyverse)
library(tidymodels)
```

``` r
voters<-read_csv("https://raw.githubusercontent.com/fivethirtyeight/data/master/non-voters/nonvoters_data.csv",show_col_types = FALSE)

# using rename() function to rename the column gender to sex
voters<-rename(voters,sex=gender)

# Using the select() function to only select and keep the variables that we are using for this activity as there are 119 variables in the dataset
voters<-select(voters,c("ppage","educ","race","sex","income_cat","Q30","voter_category"))
```

1.  Because including people who were recently eligible will result in a
    large number of empty or missing values in the data set because the
    people were recently eligible to vote.

2.  I have choose the demographic variables “Income”and “Age” from the
    plot to interpret their relation with their voter category.

-   First, we can see from the plot between “Income” and voter category
    that close to 50% of people earning less than \$40k are not
    interested in voting, whereas it is less than 25% of people earning
    more than 125k. The percentage of people voting was almost the same
    in all earning groups.
-   When it comes to age, we can see that there is a decreasing trend of
    voting with increasing age, with less than 10% of people in the age
    group of 35-49 voting. The age group of 65 and up has the highest
    turnout of any age group, with nearly 50% voting.

``` r
# creating a new variable "party" that has simplified categories of the variable Q30

voters<-voters%>% mutate(voters, party=ifelse(Q30==1,"Republican",ifelse(Q30==2,"Democrat",ifelse(Q30==3,"Independent","Other"))))
```
