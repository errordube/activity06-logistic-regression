Activity 6 - Multinomial Logistic Regression
================

## The Data

Today we will analyze data from an online Ipsos (a consulting firm)
survey that was conducted for a $\texttt{FiveThirthyEight}$ article [Why
Many Americans Don’t
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
- `educ`: Highest educational attainment category
- `race`: Race of respondent, census categories Note: all categories
  except Hispanic are non-Hispanic
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

These data can be read from the `data` folder in this Day 2 folder and
were originally downloaded from:
`https://github.com/fivethirtyeight/data/tree/master/non-voters`

**Notes**:

- Similarly to the data you used for the logistic regression portion of
  this activity, the researchers have the variable labeled `gender`, but
  it is unclear how this question was asked or what categorizations (if
  any) were provided to respondents to select from. We will use this as,
  “individuals that chose to provide their gender.”
- The authors use weighting to make the final sample more representative
  on the US population for their article. We will **not** use weighting
  in this activity, so we will treat the sample as a convenience sample
  rather than a random sample of the population.

Now…

- Below, create a new R code chunk and write the code to:
  - Load `{tidyverse}` and `{tidymodels}` and any other packages you
    want to use.
  - *Read* in the *CSV* file from the `data` folder and store it in an R
    dataframe called `nonvoters`.
  - `select` only the variables listed above to want to make
    viewing/managing the data (and the `augment` output later) easier.
- Give your R code chunk a meaningful name, then run your code chunk or
  knit your document.

``` r
library(tidyverse)
```

    ## ── Attaching core tidyverse packages ──────────────────────── tidyverse 2.0.0 ──
    ## ✔ dplyr     1.1.2     ✔ readr     2.1.4
    ## ✔ forcats   1.0.0     ✔ stringr   1.5.1
    ## ✔ ggplot2   3.4.3     ✔ tibble    3.2.1
    ## ✔ lubridate 1.9.3     ✔ tidyr     1.3.0
    ## ✔ purrr     1.0.1     
    ## ── Conflicts ────────────────────────────────────────── tidyverse_conflicts() ──
    ## ✖ dplyr::filter() masks stats::filter()
    ## ✖ dplyr::lag()    masks stats::lag()
    ## ℹ Use the conflicted package (<http://conflicted.r-lib.org/>) to force all conflicts to become errors

``` r
library(tidymodels)
```

    ## ── Attaching packages ────────────────────────────────────── tidymodels 1.1.0 ──
    ## ✔ broom        1.0.5     ✔ rsample      1.1.1
    ## ✔ dials        1.2.0     ✔ tune         1.1.1
    ## ✔ infer        1.0.4     ✔ workflows    1.1.3
    ## ✔ modeldata    1.1.0     ✔ workflowsets 1.0.1
    ## ✔ parsnip      1.1.0     ✔ yardstick    1.2.0
    ## ✔ recipes      1.0.6     
    ## ── Conflicts ───────────────────────────────────────── tidymodels_conflicts() ──
    ## ✖ scales::discard() masks purrr::discard()
    ## ✖ dplyr::filter()   masks stats::filter()
    ## ✖ recipes::fixed()  masks stringr::fixed()
    ## ✖ dplyr::lag()      masks stats::lag()
    ## ✖ yardstick::spec() masks readr::spec()
    ## ✖ recipes::step()   masks stats::step()
    ## • Use tidymodels_prefer() to resolve common conflicts.

``` r
nonvoters <- read_csv("~/STA 631/Activities/activity06-logistic-regression/day02-multinomial/data/nonvoters.csv")
```

    ## Rows: 5836 Columns: 119
    ## ── Column specification ────────────────────────────────────────────────────────
    ## Delimiter: ","
    ## chr   (5): educ, race, gender, income_cat, voter_category
    ## dbl (114): RespId, weight, Q1, Q2_1, Q2_2, Q2_3, Q2_4, Q2_5, Q2_6, Q2_7, Q2_...
    ## 
    ## ℹ Use `spec()` to retrieve the full column specification for this data.
    ## ℹ Specify the column types or set `show_col_types = FALSE` to quiet this message.

``` r
nonvoters <- nonvoters %>% select(ppage, educ, race, gender, income_cat, Q30, voter_category)
nonvoters
```

    ## # A tibble: 5,836 × 7
    ##    ppage educ                race        gender income_cat    Q30 voter_category
    ##    <dbl> <chr>               <chr>       <chr>  <chr>       <dbl> <chr>         
    ##  1    73 College             White       Female $75-125k        2 always        
    ##  2    90 College             White       Female $125k or m…     3 always        
    ##  3    53 College             White       Male   $125k or m…     2 sporadic      
    ##  4    58 Some college        Black       Female $40-75k         2 sporadic      
    ##  5    81 High school or less White       Male   $40-75k         1 always        
    ##  6    61 High school or less White       Female $40-75k         5 rarely/never  
    ##  7    80 High school or less White       Female $125k or m…     1 always        
    ##  8    68 Some college        Other/Mixed Female $75-125k        2 always        
    ##  9    70 College             White       Male   $125k or m…     1 always        
    ## 10    83 Some college        White       Male   $125k or m…     3 always        
    ## # ℹ 5,826 more rows

After doing this, answer the following questions:

1.  Why do you think the authors chose to only include data from people
    who were eligible to vote for at least four election cycles?

Answer: To ensure a comprehensive analysis of voting patterns over time,
allowing for a clearer distinction. This criterion facilitates a deeper
understanding of long-term voting behavior, minimizes the influence of
first-time voter biases, and captures changes in political engagement as
individuals experience different life stages and solidify their
political beliefs.

2.  In the FiveThirtyEight article, the authors include visualizations
    of the relationship between the [voter category and demographic
    variables](https://projects.fivethirtyeight.com/non-voters-poll-2020-election/images/NONVOTERS-1026-1.png?v=411f25ea).
    Select two of these demographic variables. Then, for each variable,
    create and interpret a plot to describe its relationship with
    `voter_category`.

``` r
ggplot(nonvoters, aes(x = educ, fill = voter_category)) +
  geom_bar(position = position_fill(), stat = "count") +
  scale_fill_brewer(palette = "Pastel1") +
  labs(title = "Education Level by Voter Category",
       x = "Education Level",
       y = "Proportion") +
  theme(axis.text.x = element_text(angle = 45, hjust = 1),
        legend.title = element_blank())
```

![](activity06-multinomial_files/figure-gfm/Plot%20for%20Education%20and%20Voter%20Category-1.png)<!-- -->

``` r
ggplot(nonvoters, aes(x = race, fill = voter_category)) +
  geom_bar(position = position_fill(), stat = "count") +
  scale_fill_brewer(palette = "Pastel2") +
  labs(title = "Race by Voter Category",
       x = "Race",
       y = "Proportion") +
  theme(axis.text.x = element_text(angle = 45, hjust = 1),
        legend.title = element_blank())
```

![](activity06-multinomial_files/figure-gfm/Plot%20for%20Race%20and%20Voter%20Category-1.png)<!-- -->

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

``` r
nonvoters <- nonvoters %>%
  mutate(party = case_when(
    Q30 == 1 ~ "Republican",
    Q30 == 2 ~ "Democrat",
    Q30 == 3 ~ "Independent",
    Q30 == 4 | Q30 == 5 | Q30 == -1 ~ "Other"  # Combining 'Other', 'No preference', and 'No response'
  ))

nonvoters <- nonvoters %>%
  mutate(voter_category = factor(voter_category, levels = c("rarely/never", "sporadic", "always")))

ggplot(nonvoters, aes(x = party, fill = voter_category)) +
  geom_bar(position = "fill") +
  scale_fill_manual(values = c("rarely/never" = "#FF5733", "sporadic" = "#CBDF05", "always" = "#23DF05")) + 
  labs(y = "Proportion", x = "Party", fill = "Voter Category") +
  theme_minimal() +
  theme(axis.text.x = element_text(angle = 45, hjust = 1))
```

![](activity06-multinomial_files/figure-gfm/data%20preparation-1.png)<!-- -->

## Fitting the model

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
equations for each category of the outcome variable **relative to the
baseline category**. If the outcome has $K$ possible categories, there
will be $K - 1$ equations as part of the multinomial logistic model.

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

Now we will fit a model using age, race, gender, income, and education
to predict voter category. This is using `{tidymodels}`.

- In the code chunk below, replace “verbatim” with “r”,
- Provide the code chunk a meaningful name/title, then run it.

``` r
# abbreviated recipe from previous activities
multi_mod <- multinom_reg() %>% 
  set_engine("nnet") %>% 
  fit(voter_category ~ ppage + educ + race + gender + income_cat, data = nonvoters)

multi_mod <- repair_call(multi_mod, data = nonvoters)

tidy(multi_mod) %>% 
  print(n = Inf) # This will display all rows of the tibble
```

    ## # A tibble: 22 × 6
    ##    y.level  term                      estimate std.error statistic   p.value
    ##    <chr>    <chr>                        <dbl>     <dbl>     <dbl>     <dbl>
    ##  1 sporadic (Intercept)              -0.887      0.167    -5.32    1.03e-  7
    ##  2 sporadic ppage                     0.0476     0.00229  20.8     5.05e- 96
    ##  3 sporadic educHigh school or less  -0.922      0.0957   -9.64    5.42e- 22
    ##  4 sporadic educSome college         -0.357      0.0938   -3.81    1.40e-  4
    ##  5 sporadic raceHispanic             -0.00655    0.126    -0.0521  9.58e-  1
    ##  6 sporadic raceOther/Mixed          -0.373      0.157    -2.38    1.74e-  2
    ##  7 sporadic raceWhite                -0.127      0.102    -1.25    2.10e-  1
    ##  8 sporadic genderMale               -0.0961     0.0707   -1.36    1.74e-  1
    ##  9 sporadic income_cat$40-75k        -0.127      0.110    -1.15    2.48e-  1
    ## 10 sporadic income_cat$75-125k       -0.000882   0.106    -0.00832 9.93e-  1
    ## 11 sporadic income_catLess than $40k -0.663      0.112    -5.91    3.43e-  9
    ## 12 always   (Intercept)              -1.85       0.185   -10.0     1.18e- 23
    ## 13 always   ppage                     0.0606     0.00252  24.0     2.29e-127
    ## 14 always   educHigh school or less  -1.35       0.107   -12.7     7.65e- 37
    ## 15 always   educSome college         -0.412      0.100    -4.10    4.13e-  5
    ## 16 always   raceHispanic             -0.417      0.147    -2.84    4.46e-  3
    ## 17 always   raceOther/Mixed          -0.683      0.182    -3.74    1.82e-  4
    ## 18 always   raceWhite                 0.0392     0.111     0.353   7.24e-  1
    ## 19 always   genderMale               -0.211      0.0779   -2.70    6.83e-  3
    ## 20 always   income_cat$40-75k        -0.0669     0.120    -0.559   5.76e-  1
    ## 21 always   income_cat$75-125k        0.147      0.113     1.29    1.95e-  1
    ## 22 always   income_catLess than $40k -0.756      0.125    -6.05    1.43e-  9

`{tidymodels}` is designed for cross-validation and so there needs to be
some “trickery” when we build models using the entire dataset. For
example, when you type `multi_mod$fit$call` in your **Console**, you
should see the following output:

    > multi_mod$fit$call
    nnet::multinom(formula = voter_category ~ ppage + educ + race + gender + income_cat, data = data, trace = FALSE)

The issue here is `data = data` and should be `data = nonvoters`. To
*repair* this, add the following to your previous R code chunk:

    multi_mod <- repair_call(multi_mod, data = nonvoters)

Re-run your code chunk, then type `multi_mod$fit$call` in your
**Console**, you should see the following output:

    > multi_mod$fit$call
    nnet::multinom(formula = voter_category ~ ppage + educ + race + gender + income_cat, data = nonvoters, trace = FALSE)

Yay!

Now, recall that the baseline category for the model is
`"rarely/never"`. Using your `tidy(multi_mod) %>% print(n = Inf)`
output, complete the following items:

3.  Write the model equation for the log-odds of a person that the
    “rarely/never” votes vs “always” votes. That is, finish this
    equation using your estimated parameters:

$$
\begin{equation*}
\log\left(\frac{\hat{p}_{\texttt{"always"}}}{\hat{p}_{\texttt{"rarely/never"}}}\right) = -1.8538234 + 0.0605977 \times \text{ppage} - 1.3530192 \times \text{I(educ = "High school or less")} - 0.4119994 \times \text{I(educ = "Some college")} - 0.4172005 \times \text{I(race = "Hispanic")} - 0.6827372 \times \text{I(race = "Other/Mixed")} + 0.0392386 \times \text{I(gender = "Male")} - 0.2105851 \times \text{I(income_cat = "$40-75k")} + 0.1466083 \times \text{I(income_cat = "$75-125k")} - 0.7563449 \times \text{I(income_cat = "Less than $40k")}
\end{equation*}
$$

4.  For your equation in (3), interpret the slope for `genderMale` in
    both log-odds and odds.

Answer: For males, the log-odds of always voting versus rarely/never
voting decreases by 0.2105850553 compared to females, holding all other
variables constant.

Odds_ratio= exp(−0.2105850553) = 0.810

**Note**: The interpretation for the slope for `ppage` is a little more
difficult to interpret. However, we could mean-center age (i.e.,
subtract the mean age from each age value) to have a more meaningful
interpretation.

## Predicting

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

- In the code chunk below, replace “verbatim” with “r”,
- Provide the code chunk a meaningful name/title, then run it.

``` r
voter_aug <- augment(multi_mod, new_data = nonvoters)

voter_aug
```

    ## # A tibble: 5,836 × 12
    ##    ppage educ     race  gender income_cat   Q30 voter_category party .pred_class
    ##    <dbl> <chr>    <chr> <chr>  <chr>      <dbl> <fct>          <chr> <fct>      
    ##  1    73 College  White Female $75-125k       2 always         Demo… always     
    ##  2    90 College  White Female $125k or …     3 always         Inde… always     
    ##  3    53 College  White Male   $125k or …     2 sporadic       Demo… sporadic   
    ##  4    58 Some co… Black Female $40-75k        2 sporadic       Demo… sporadic   
    ##  5    81 High sc… White Male   $40-75k        1 always         Repu… sporadic   
    ##  6    61 High sc… White Female $40-75k        5 rarely/never   Other sporadic   
    ##  7    80 High sc… White Female $125k or …     1 always         Repu… sporadic   
    ##  8    68 Some co… Othe… Female $75-125k       2 always         Demo… sporadic   
    ##  9    70 College  White Male   $125k or …     1 always         Repu… sporadic   
    ## 10    83 Some co… White Male   $125k or …     3 always         Inde… always     
    ## # ℹ 5,826 more rows
    ## # ℹ 3 more variables: `.pred_rarely/never` <dbl>, .pred_sporadic <dbl>,
    ## #   .pred_always <dbl>

``` r
voter_aug %>% 
  select(contains("pred"))
```

    ## # A tibble: 5,836 × 4
    ##    .pred_class `.pred_rarely/never` .pred_sporadic .pred_always
    ##    <fct>                      <dbl>          <dbl>        <dbl>
    ##  1 always                    0.0352          0.411        0.554
    ##  2 always                    0.0153          0.402        0.583
    ##  3 sporadic                  0.119           0.489        0.391
    ##  4 sporadic                  0.121           0.485        0.394
    ##  5 sporadic                  0.0930          0.505        0.402
    ##  6 sporadic                  0.204           0.472        0.324
    ##  7 sporadic                  0.0778          0.504        0.418
    ##  8 sporadic                  0.102           0.515        0.382
    ##  9 sporadic                  0.0516          0.475        0.474
    ## 10 always                    0.0380          0.454        0.508
    ## # ℹ 5,826 more rows

Here we can see all of the predicted probabilities. This is still rather
difficult to view so a [confusion
matrix](https://en.wikipedia.org/wiki/Confusion_matrix) can help us
summarize how well the predictions fit the actual values.

- In the code chunk below, replace “verbatim” with “r”,
- Provide the code chunk a meaningful name/title, then run it.

``` r
voter_conf_mat <- voter_aug %>% 
  count(voter_category, .pred_class, .drop = FALSE)

voter_conf_mat %>% 
  pivot_wider(
    names_from = .pred_class,
    values_from = n
  )
```

    ## # A tibble: 3 × 4
    ##   voter_category `rarely/never` sporadic always
    ##   <fct>                   <int>    <int>  <int>
    ## 1 rarely/never              586      815     50
    ## 2 sporadic                  271     1994    309
    ## 3 always                    243     1150    418

We can also visualize how well these predictions fit the original
values.

- In the code chunk below, replace “verbatim” with “r”,
- Provide the code chunk a meaningful name/title, then run it.

``` r
nonvoters %>% 
  ggplot(aes(x = voter_category)) +
  geom_bar() +
  labs(
    main = "Self-reported voter category"
    )
```

![](activity06-multinomial_files/figure-gfm/visualization%20of%20predicted%20vs%20self-reported-1.png)<!-- -->

``` r
voter_conf_mat %>% 
  ggplot(aes(x = voter_category, y = n, fill = .pred_class)) +
  geom_bar(stat = "identity") +
  labs(
    main = "Predicted vs self-reported voter category"
    )
```

![](activity06-multinomial_files/figure-gfm/visualization%20of%20predicted%20vs%20self-reported-2.png)<!-- -->

Answer the following question:

5.  What do you notice?

Answer:The first plot shows a bar chart with voter counts for
rarely/never, sporadic, and always voters, indicating the majority are
sporadic. The second plot is a stacked bar chart, breaking down the
accuracy of voter behavior predictions within each actual voting
category. It highlights that predictions for rarely/never and always
voters are relatively accurate, while predictions for sporadic voters
show more variation and error, suggesting the unpredictability of this
group’s voting behavior.

## Challenge: Explore with `party`

Fit the model that also includes `party` and discuss differences between
the above model and this model with the additional predictor variable.
Can you assess (think back to the MLR activity for how we tested two
models where one was a subset of another) the effect by including this
additional predictor variable?
