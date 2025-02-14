Activity 6 - Logistic Regression
================

# Activity 6 - Day 1

## Task 2: Load the necessary packages

``` r
library(tidyverse)
library(tidymodels) 
library(lessR)
library(GGally)
```

## Task 3: Load the data

``` r
resume<-read_csv("https://www.openintro.org/data/csv/resume.csv",show_col_types = FALSE)
```

1.  I think this is an experimental study because they have monitored
    and kept track of the job applications call back rates

2.  I think the variable “received\_callback” was a numeric variable
    (More of a binary variable). The value “o” represents that the
    applicant did not get a call back and the value “1” represents that
    the applicant got a call back for the job application.

``` r
# Adding a new varibale called condition to show whether the applicant received a call back

resume<-resume %>% mutate(condition=ifelse(resume$received_callback=="1","Yes","No"))
```

``` r
# creating barplot visualization for the received_callback variable

plot1<-resume%>%ggplot(aes(x=received_callback))+
  geom_bar(fill = "lightblue", colour = "black")+
  geom_text(aes(label=..count..),stat = "count",vjust=1.6)+
  scale_x_continuous(breaks = c(0,1))+
  labs(title = "Barplot for received callback",
       x="Received a callback",
       y="Count")+
  theme(plot.title = element_text(hjust = 0.5))+
  theme_test()

plot1
```

![](activity06_files/figure-gfm/unnamed-chunk-4-1.png)<!-- -->

3.  I have choosen a bar plot to visulaize the numerical summary of
    applicants callback status which you can see above.

4.  Below is the numerical summary table that I have recreated and also
    a pie chart to visualize the numerical summary table

``` r
# Creating a summary table to display the numerical summary table of the applicants callback status

summary_table <- resume %>% 
  group_by(condition) %>% # Variable to be transformed
  count() %>% 
  ungroup() %>% 
  mutate(percent = (`n` / sum(`n`))*100)%>%
  mutate(across(where(is.numeric), round, 2))

print.data.frame(summary_table)
```

    ##   condition    n percent
    ## 1        No 4478   91.95
    ## 2       Yes  392    8.05

``` r
# Creating a piechart to visualize applicants condition of received callback

plot2<-ggplot(summary_table, aes(x = "", y = percent, fill = condition)) +
  geom_col(color = "black") +
  geom_label(aes(label = percent), color = "black",
            position = position_stack(vjust = 0.5),
            show.legend = FALSE) +
  labs(title = "Percentage of applicants callback status",
       caption = "Reference: https://r-charts.com/part-whole/pie-chart-percentages-ggplot2/")+
  guides(fill = guide_legend(title = "Received a call back")) +
  scale_fill_brewer(palette = "Pastel1")+
  coord_polar(theta = "y") + 
  theme_void()

plot2
```

![](activity06_files/figure-gfm/unnamed-chunk-5-1.png)<!-- -->

5.  According to the results of 3 and 4, the majority of applicants
    (91.95%) did not receive a callback for their application, while
    only a few applicants (8.05%) did.

## Task 4: Probability and odds

6.  The probability that a randomly selected résumé/person will be
    called back is **0.08**

7.  The odds that a randomly selected résumé/person will be called back
    is **0.087**

## Task 5: Logistic regression

8.  The probability that a randomly selected résumé/person perceived as
    Black will be called back is **0.064**

9.  The odds that a randomly selected résumé/person perceived as Black
    will be called back is **0.069**

``` r
# The {tidymodels} method for logistic regression requires that the response be a factor variable
resume <- resume %>% 
  mutate(received_callback = as.factor(received_callback))

resume_mod <- logistic_reg() %>%
  set_engine("glm") %>%
  fit(received_callback ~ race, data = resume, family = "binomial")

tidy(resume_mod) %>% 
  knitr::kable(digits = 3)
```

| term        | estimate | std.error | statistic | p.value |
|:------------|---------:|----------:|----------:|--------:|
| (Intercept) |   -2.675 |     0.083 |   -32.417 |       0 |
| racewhite   |    0.438 |     0.107 |     4.083 |       0 |

10. The estimated regression equation is

$$
 \begin{aligned}
\widehat{\texttt{received\\_callback}}&= \hat{\beta}_0 + \hat{\beta}_1 \times \texttt{race}\\
&= -2.67 + 0.44 \times \text{race}
\end{aligned}
$$

**Note:** Since the Race variable is a categorical variable with two
levels, R has created a dummy variable for race in the model. So if the
applicant is perceived as “Black” then race is “0” and if the applicant
is perceived it as “white” then race is “1”.

11. The simplified estimated regression equation corresponding to
    résumés/persons perceived as Black.

$$
 \begin{aligned}
\widehat{\texttt{received\\_callback}}&= -2.67 + 0.44   \times \text{race} \\
&= 2.675 + 0.438(0) \\
\widehat{\texttt{received\\_callback}}&= -2.67
\end{aligned}
$$

``` r
# To calculate the odds of a logistic regression model, use the exp() function to back transform the obtained coefficient.

odds <- exp(-2.67)

# Probability can be caluculated with the help of odds value as shown below

probability<- odds/(1+odds)

log_odds<-log(odds)

probability<-round(probability,digits=3)
odds<- round(odds,digits = 3)
```

12. If a randomly selected résumé/person perceived as Black, the log
    odds that they will be called back is **-2.67**

13. If a randomly selected résumé/person perceived as Black, the odds
    that they will be called back is **0.069**, this was the same value
    that I obtained in question 9. We can back transform a log odds
    value by using **exp()** function to obtain odds value.

14. If a randomly selected résumé/person perceived as Black, the
    probability that they will be called back is **0.065**, this was the
    same value that I obtained in question 8.

# Activity 6 -Day 2

``` r
resume_select <- resume %>% dplyr::rename(sex=gender) %>% 
  filter(job_city == "Chicago") %>% 
  mutate(race = case_when(race == "white" ~ "White",TRUE ~ "Black"),
         sex = case_when(sex == "f" ~ "female",TRUE ~ "male")) %>% 
  select(received_callback, years_experience, race, sex)
```

1.  -   In the above code, since we are trying to build a multiple
        logistics regression model. We are creating another data set
        that only has the variables we are using for the model.
    -   The variable “gender” was renamed to “sex”.
    -   we filtered the applicants based on their job\_city which is
        Chicago since we are trying to find difference in callback rates
        for jobs in Chicago.
    -   By using the mutate function the values of the variables “race”
        and “sex” were changed. Like the first letter for values of race
        were capitalized. The values of variable “sex” were coded as “m”
        for male and “f” for female. So, we changed those into “male”
        and “female”.
    -   After all these, we have used the select function to keep only
        the required variables for the model in the data set.

## Task 2: Relationship Exploration

``` r
ggbivariate(resume_select,"received_callback")+scale_fill_manual(values = c("#E69F00","#999999"))
```

![](activity06_files/figure-gfm/unnamed-chunk-9-1.png)<!-- -->

2.  From the above we can see the whether the explanatory variables has
    any impact on the applicants getting a callback for jobs in chicago.
    I can see that the male applicants getting more callbacks than the
    female applicants, which is not too significant but there is a
    slight difference. It was the same with the race variable, where
    whit appliants got a little more callbacks than those of Black
    applicants.

## Task 3: Fitting the model

``` r
mult_log_mod <- glm(received_callback ~ years_experience + race + sex, data = resume_select, family = "binomial")

tidy(mult_log_mod)
```

    ## # A tibble: 4 × 5
    ##   term             estimate std.error statistic  p.value
    ##   <chr>               <dbl>     <dbl>     <dbl>    <dbl>
    ## 1 (Intercept)       -3.28      0.181     -18.1  4.41e-73
    ## 2 years_experience   0.0449    0.0163      2.75 5.94e- 3
    ## 3 raceWhite          0.426     0.157       2.72 6.53e- 3
    ## 4 sexmale            0.580     0.203       2.86 4.28e- 3

``` r
tidy(mult_log_mod, exponentiate = TRUE) %>% 
  knitr::kable(digits = 3)
```

| term              | estimate | std.error | statistic | p.value |
|:------------------|---------:|----------:|----------:|--------:|
| (Intercept)       |    0.038 |     0.181 |   -18.082 |   0.000 |
| years\_experience |    1.046 |     0.016 |     2.751 |   0.006 |
| raceWhite         |    1.532 |     0.157 |     2.720 |   0.007 |
| sexmale           |    1.786 |     0.203 |     2.857 |   0.004 |

2.  For each additional year of experience for an applicant in Chicago,
    we expect the odds of an applicant receiving a call back to increase
    by 1.046 units. Assuming that the applicants have similar time in
    spent in college, similar inferred races, and similar inferred sex.

## Task 4: Assessing model fit

``` r
# To store residuals and create row number variable
mult_log_aug <- augment(mult_log_mod, type.predict = "response", 
                      type.residuals = "deviance") %>% 
                      mutate(id = row_number())

# Plot residuals vs fitted values
ggplot(data = mult_log_aug, aes(x = .fitted, y = .resid)) + 
geom_point() + 
geom_hline(yintercept = 0, color = "red") + 
labs(x = "Fitted values", 
     y = "Deviance residuals", 
     title = "Deviance residuals vs. fitted")
```

![](activity06_files/figure-gfm/unnamed-chunk-12-1.png)<!-- -->

``` r
# Plot residuals vs row number
ggplot(data = mult_log_aug, aes(x = id, y = .resid)) + 
geom_point() + 
geom_hline(yintercept = 0, color = "red") + 
labs(x = "id", 
     y = "Deviance residuals", 
     title = "Deviance residuals vs. id")
```

![](activity06_files/figure-gfm/unnamed-chunk-12-2.png)<!-- -->
