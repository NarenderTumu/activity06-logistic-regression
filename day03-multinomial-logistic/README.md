Day 3 - Multinomial Logistic Regression
================

In this repository/directory you should see one items:

- `README.md` - this document.
- `activity06-multinomial.Rmd` - the file you will complete in RStudio
  for this week.

## Task 1: Pull your changes from GitHub into RStudio

You successfully updated your GitHub repo from the main
[`README`](../README), but we still need to update your RStudio files.
Read these directions first, then work through them.

- In the **Git** pane of RStudio (upper right-hand area), locate and
  click on the
  <img src="../README-img/pull-icon.png" alt="knit" width = "20"/>
  **Pull** icon.
- After a few moments (you might need to provide your GitHub username
  and PAT), your **Files** pane will be updated with the new items.

The important item that was brought into your RStudio session is the
`day03-multinomial-logistic` subfolder that contains the
`activity06-multinomial.Rmd` file. Remember that it is always a good
habit to **pull** from GitHub before starting to work after you take a
break. For example, if you were collaborating on a project with others,
they might do work at different times than you (or even at the same
time). In fact, you will be required to solve any problems on your end
before **push**ing to GitHub - a “feature” of Git.

## Today

Today we will analyze data from an online Ipsos survey that was
conducted for a $\texttt{FiveThirthyEight}$ article [Why Many Americans
Don’t
Vote](https://projects.fivethirtyeight.com/non-voters-poll-2020-election/).
You can read more about the survey design and respondents in the
`README` of their [GitHub
repo](https://github.com/fivethirtyeight/data/tree/master/non-voters)
for the data.

Briefly, respondents were asked a variety of questions about their
political beliefs, thoughts on multiple issues, and voting behavior. We
will focus on the demographic variables and the respondent’s party
identification to understand whether a person is a probable voter (with
levels always, sporadic, rarely/never).

The specific variables we will use are (definitions are from the
[`nonvoters_codebook.pdf`](https://github.com/fivethirtyeight/data/blob/master/non-voters/nonvoters_codebook.pdf)):

- `ppage`: Age of respondent
- `educ`: Highest educational attainment category.
- `race`: Race of respondent, census categories. Note: all categories
  except Hispanic are non-Hispanic.
- `gender`: Gender of respondent
- `income_cat`: Household income category of respondent
- `Q30`: Response to the question “Generally speaking, do you think of
  yourself as a…”
  - 1: Republican
  - 2: Democrat
  - 3: Independent
  - 4: Another party, please specify
  - 5: No preference
  - -1: No response
- `voter_category`: past voting behavior:
  - **always**: respondent voted in all or all-but-one of the elections
    they were eligible in
  - **sporadic**: respondent voted in at least two, but fewer than
    all-but-one of the elections they were eligible in
  - **rarely/never**: respondent voted in 0 or 1 of the elections they
    were eligible in

These data can be read from the following GitHub URL:
`https://raw.githubusercontent.com/fivethirtyeight/data/master/non-voters/nonvoters_data.csv`

**Notes**:

- Similarly to the data you used for Days 1 & 2, the researchers have
  the variable labeled `gender`, but this is likely meant to be `sex`.
- The authors use weighting to make the final sample more representative
  on the US population for their article. We will **not** use weighting
  in this activity, so we will treat the sample as a convenience sample
  rather than a random sample of the population.

Now…

- Create a new R code chunk and write the code to:
  - Load `{tidyverse}` and `{tidymodels}` and any other packages you
    want to use.
  - *Read* in the *CSV* file from the provided URL and store it in an R
    dataframe called \`voters\`\`.
  - `rename` the variable `gender` to `sex`.
  - `select` only the variables listed above to want to make viewing the
    data (and `augment` output later) easier to view.
- Then, run your code chunk or knit your document.

After doing this, answer the following questions:

1.  Why do you think the authors chose to only include data from people
    who were eligible to vote for at least four election cycles?
2.  In the FiveThirtyEight article, the authors include visualizations
    of the relationship between the [voter category and demographic
    variables](https://projects.fivethirtyeight.com/non-voters-poll-2020-election/images/NONVOTERS-1026-1.png?v=411f25ea).
    Select two demographic variables. Then, for each variable, interpret
    the plot to describe its relationship with voter category.

We need to do some data preparation before we fit our multinomial
logistic regression model.

- Create a new R code chunk and address these items:
  - The variable `Q30` contains the respondent’s political party
    identification. *Create a new variable* called `party` in the
    dataset that simplifies `Q30` into four categories: “Democrat”,
    “Republican”, “Independent”, “Other” (“Other” should also include
    respondents who did not answer the question).
  - The variable `voter_category` identifies the respondent’s past voter
    behavior. *Convert* this to a factor variable and ensure (*hint*:
    explore `relevel`) that the “rarely/never” level is the baseline
    level, followed by “sporadic”, then “always”.
- Then, run your code chunk or knit your document. Check that your
  changes are correct by creating a stacked bar graph using your new
  `Q30` variable as the $y$-axis and the `voter_category` represented
  with different colors. **Challenge**: Can you use the same color
  palette (*hint*: this is a handy tool, <https://pickcoloronline.com/>)
  that $\texttt{FiveThirthyEight}$ used in their article?

### Aside

I noticed during your presentations of the first mini-competition that
some of you are already aware of cross-validation techniques. Also, many
of you have asked about how to compare two or more models to determine
which is the “best” from a statistical perspective. I am intentionally
focusing on the various kinds of models first and using less intensive
methods for assessing models (i.e., how good is the one model we are
fitting).

I think that it is awesome that some of you are already familiar with
cross-validation methods. For those of you less familiar with these
techniques, we will cover this in **Module 4: Model Assessment &
Selection** (this is Chapter 5 in *ISL*). As we have primarily been
focusing on assessing how well one model fits the data using graphical
methods (e.g., residual analysis and potential outlier detection) and
numerical summaries (e.g., $R^2$, *VIF* and correlation of dependent
variables, high leverage points). Remember that we have also looked at
how to address concerns of linearity or non-constant variability of the
residuals with variable transformations. In **Module 4: Model Assessment
& Selection** (Chapters 6 & 13 in *ISL*) we will explore ways to compare
between multiple potential models.

## Task 2: Fitting the model

Previously, we have explored logistic regression where the
outcome/response/independent variable has two levels (e.g., “has
feature” and “does not have feature”). We then used the logistic
regression model

$$
\begin{equation*}
\log\left(\frac{\hat{p}}{1-\hat{p}}\right) = \hat\beta_0 + \hat\beta_1x_1 + \hat\beta_2x_2 + \cdots + \hat\beta_px_p
\end{equation*}
$$

Another way to think about this model is if we are interested in
comparing our “has feature” category to the *baseline* “does not have
feature” category. If we let $y = 0$ represent the *baseline category*,
such that $P(y_i = 1 | X's) = \hat{p}_i1$ and
$P(y_i = 0 | X's) = 1 - \hat{p}_{i1} = \hat{p}_{i0}$, then the above
equation can be rewritten as:

$$
\begin{equation*}
\log\left(\frac{\hat{p}_{i1}}{\hat{p}_{i0}}\right) = \hat\beta_0 + \hat\beta_1x_{i1} + \hat\beta_2x_{i2} + \cdots + \hat\beta_px_{ip}
\end{equation*}
$$

Recall that:

- The slopes ($\hat\beta_p$) represent when $x_p$ increases by one
  ($x_p$) unit, the odds of $y = 1$ compared to the baseline $y = 0$ are
  expected to multiply by a factor of $e^{\hat\beta_p}$. -The intercept
  ($\hat\beta0$) respresents when all $x_j = 0$ (for
  $j = 1, \ldots, p$), the predicted odds of $y = 1$ versus the baseline
  $y = 0$ are $e^{\hat\beta_0}$.

For a multinomial (i.e., more than two categories, say, labeled
$k = 1, 2, \ldots, K$) outcome variable,
$P(y = 1) = p_1, P(y = 2) = p_2, \ldots, P(y = K) = p_k$, such that

$$
\begin{equation*}
\sum_{k = 1}^K p_k = 1
\end{equation*}
$$

This is called the **multinomial distribution**.

For a multinomial logistic regression model it is helpful to identify a
baseline category (say, $y = 1$). We then fit a model such that
$P(y = k) = p_k$ is a model of the $x$’s.

$$
\begin{equation*}
\log\left(\frac{\hat{p}_{ik}}{\hat{p}_{i1}}\right) = \hat\beta_{0k} + \hat\beta_{1k}x_{i1} + \hat\beta_{2k}x_{i2} + \cdots + \hat\beta_{pk}x_{ip}
\end{equation*}
$$

Notice that for a multinomial logistic model, we will have separate
equations for each category of the outcome variable relative to the
baseline category. If the outcome has $K$ possible categories, there
will be $K - 1$ equestions as part of the multinomial logistic model.

Suppose we have an outcome variable $y$ with three possible levels coded
as “A”, “B”, “C”. If “A” is the baseline category, then

$$
\begin{equation*}
\begin{aligned}
\log\left(\frac{\hat{p}_{iB}}{\hat{p}_{iA}}\right) &= \hat\beta_{0B} + \hat\beta_{1B}x_{i1} + \hat\beta_{2B}x_{i2} + \cdots + \hat\beta_{pB}x_{ip} \\
\log\left(\frac{\hat{p}_{iC}}{\hat{p}_{iA}}\right) &= \hat\beta_{0C} + \hat\beta_{1C}x_{i1} + \hat\beta_{2C}x_{i2} + \cdots + \hat\beta_{pC}x_{ip} \\
\end{aligned}
\end{equation*}
$$

Now we will fit a model using age, race, sex, income, and education to
predict voter category. This is using `{tidymodels}`! I will provide you
with updated information for Day 2’s activity on Thu, Mar 2.

- Create a new R code chunk and type the following, then run your code
  chunk or knit your document.

  ``` r
  multi_mod <- multinom_reg() %>% 
    set_engine("nnet") %>% 
    fit(voter_category ~ ppage + educ + race + sex + income_cat, data = voters)

  tidy(multi_mod) %>% 
    print(n = Inf) # This will display all rows of the tibble
  ```

`{tidymodels}` is designed for cross-validation and so there needs to be
some “trickery” when we build models using the entire dataset. For
example, when you type `multi_mod$fit$call` in your **Console**, you
should see the following output:

    > multi_mod$fit$call
    nnet::multinom(formula = voter_category ~ ppage + educ + race + sex + income_cat, data = data, trace = FALSE)

The issue here is `data = data` and should be `data = voters`. To
*repair* this, add the following to your previous R code chunk:

``` r
multi_mod <- repair_call(multi_mod, data = voters)
```

Re-run your code chunk, then type `multi_mod$fit$call` in your
**Console**, you should see the following output:

    > multi_mod$fit$call
    nnet::multinom(formula = voter_category ~ ppage + educ + race + sex + income_cat, data = voters, trace = FALSE)

Yay!

Now, recall that the baseline category for the model is
`"rarely/never"`. Using your `tidy(multi_mod) %>% print(n = Inf)`
output, complete the following items:

3.  Write the model equation for the log-odds of a person that the
    “rarely/never” votes vs “always” votes. That is, finish this
    equation using your estimated parameters:

$$
\begin{equation*}
\log\left(\frac{\hat{p}_{\texttt{"always"}}}{\hat{p}_{\texttt{"rarely/never"}}}\right) = \cdots
\end{equation*}
$$

4.  For your equation in (3), interpret the slope for `sexMale` in both
    log-odds and odds.

**Note**: The interpretation for the slope for `ppage` is a little more
difficult to interpret. However, we could mean-center age (i.e.,
subtract the mean age from each age value) to have a more meaningful
interpretation. I will provide you with an example of this when I
“correct” the Day 2 activity with `{tidymodels}` code - for Thu, Mar 2.

## Task 4: Predicting

We could use this model to calculate probabilities. Generally, for
categories $2, \ldots, K$, the probability that the $i^{th}$ observation
is in the $k^{th}$ category is,

$$
\begin{equation*}
\hat{p}_{ik} = \frac{e^{\hat\beta_{0j} + \hat\beta_{1j}x_{i1} + \hat\beta_{2j}x_{i2} + \cdots + \hat\beta_{pj}x_{ip}}}{1 + \sum_{k = 2}^Ke^{\hat\beta_{0k} + \hat\beta_{1k}x_{1i} + \hat\beta_{2k}x_{2i} + \cdots + \hat\beta_{pk}x_{pi}}}
\end{equation*}
$$

And the baseline category, $k = 1$,

$$
\begin{equation*}
\hat{p}_{i1} = 1 - \sum_{k = 2}^K \hat{p}_{ik}
\end{equation*}
$$

However, we will let R do these calculations.

- Create a new R code chunk and type the following, then run your code
  chunk or knit your document.

  ``` r
  voter_aug <- augment(multi_mod, new_data = voters)

  voter_aug

  voter_aug %>% 
    select(contains("pred"))
  ```

Here we can see all of the predicted probabilities. This is still rather
difficult to view so a [confusion
matrix](https://en.wikipedia.org/wiki/Confusion_matrix) can help us
summarize how well the predictions fit the actual values.

- Create a new R code chunk and type the following, then run your code
  chunk or knit your document.

  ``` r
  voter_conf_mat <- voter_aug %>% 
    count(voter_category, .pred_class, .drop = FALSE)

  voter_conf_mat %>% 
    pivot_wider(
      names_from = .pred_class,
      values_from = n
    )
  ```

We can also visualize how well these predictions fit the original
values.

- Create a new R code chunk and type the following, then run your code
  chunk or knit your document.

  ``` r
  voters %>% 
    ggplot(aes(x = voter_category)) +
    geom_bar() +
    labs(
      main = "Self-reported voter category"
      )

  voter_conf_mat %>% 
    ggplot(aes(x = voter_category, y = n, fill = .pred_class)) +
    geom_bar(stat = "identity") +
    labs(
      main = "Predicted vs self-reported voter category"
      )
  ```

Answer the following question:

5.  What do you notice?

## Challenge: Explore with `party`

Fit the model that also includes `party` and discuss differences between
the above model and this model with the additional predictor variable.

## What is next?

Thu, Mar 2 will be an in-class project work day (a good day to start
your next assignment). I will provide you with a document in a
`day04-additional-checks` folder that uses `{tidymodels}` on the
`resume` data. This document will also walk you through checking the
condition of linearity (really if log-odds are linear with the
explanatory variables).
