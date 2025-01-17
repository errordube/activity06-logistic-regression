Activity 6 - Logistic Regression
================

## Load the necessary packages

Again, we will use two packages from Posit: `{tidyverse}` and
`{tidymodels}`.

- Below a new R code chunk and write the code to:
  - Load `{tidyverse}` and `{tidymodels}` and any other packages you
    want to use.
- Give your R code chunk a meaningful name, then run your code chunk or
  knit your document.

``` r
library(tidyverse)
library(tidymodels)
```

## Load the data

### The data

The data we are working with is again from the OpenIntro site. From
OpenIntro’s [description of the
data](https://www.openintro.org/data/index.php?data=resume):

> This experiment data comes from a study that sought to understand the
> influence of race and gender on job application callback rates. The
> study monitored job postings in Boston and Chicago for several months
> during 2001 and 2002 and used this to build up a set of test cases.
> Over this time period, the researchers randomly generating résumés to
> go out to a job posting, such as years of experience and education
> details, to create a realistic-looking résumé. They then randomly
> assigned a name to the résumé that would communicate the applicant’s
> gender and race. The first names chosen for the study were selected so
> that the names would predominantly be recognized as belonging to black
> or white individuals. For example, Lakisha was a name that their
> survey indicated would be interpreted as a black woman, while Greg was
> a name that would generally be interpreted to be associated with a
> white male.

Review the description page. If you still have questions, review the
**Source** (also linked to shortly) at the bottom of the description
page. On an initial reading, there are some concerns with how this study
is designed. In [the
article](https://www.nber.org/system/files/working_papers/w9873/w9873.pdf),
the authors do point out some of these concerns (in Sections 3.5 and
5.1).

Read in the `resume` **CSV** file that is in `day01-logistic/data`.

- Below, create a new R code chunk and write the code to:
  - *Read* in the *CSV* file from the `data` folder and store it in an R
    dataframe called `resume`.
- Give your R code chunk a meaningful name, then run your code chunk or
  knit your document.

``` r
resume <- read_csv("~/STA 631/Activities/activity06-logistic-regression/day01-logistic/data/resume.csv")
```

    ## Rows: 4870 Columns: 30
    ## ── Column specification ────────────────────────────────────────────────────────
    ## Delimiter: ","
    ## chr (10): job_city, job_industry, job_type, job_ownership, job_req_min_exper...
    ## dbl (20): job_ad_id, job_fed_contractor, job_equal_opp_employer, job_req_any...
    ## 
    ## ℹ Use `spec()` to retrieve the full column specification for this data.
    ## ℹ Specify the column types or set `show_col_types = FALSE` to quiet this message.

Recall that you can use `dplyr::glimpse` to see some meta information
about the R data frame. After doing this and your review of the data’s
description page, answer the following questions:

``` r
glimpse(resume)
```

    ## Rows: 4,870
    ## Columns: 30
    ## $ job_ad_id              <dbl> 384, 384, 384, 384, 385, 386, 386, 385, 386, 38…
    ## $ job_city               <chr> "Chicago", "Chicago", "Chicago", "Chicago", "Ch…
    ## $ job_industry           <chr> "manufacturing", "manufacturing", "manufacturin…
    ## $ job_type               <chr> "supervisor", "supervisor", "supervisor", "supe…
    ## $ job_fed_contractor     <dbl> NA, NA, NA, NA, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, N…
    ## $ job_equal_opp_employer <dbl> 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1,…
    ## $ job_ownership          <chr> "unknown", "unknown", "unknown", "unknown", "no…
    ## $ job_req_any            <dbl> 1, 1, 1, 1, 1, 0, 0, 1, 0, 0, 1, 1, 1, 1, 0, 0,…
    ## $ job_req_communication  <dbl> 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 1, 1, 0, 0,…
    ## $ job_req_education      <dbl> 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0,…
    ## $ job_req_min_experience <chr> "5", "5", "5", "5", "some", NA, NA, "some", NA,…
    ## $ job_req_computer       <dbl> 1, 1, 1, 1, 1, 0, 0, 1, 0, 0, 1, 1, 1, 1, 0, 0,…
    ## $ job_req_organization   <dbl> 0, 0, 0, 0, 1, 0, 0, 1, 0, 0, 0, 0, 1, 1, 0, 0,…
    ## $ job_req_school         <chr> "none_listed", "none_listed", "none_listed", "n…
    ## $ received_callback      <dbl> 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0,…
    ## $ firstname              <chr> "Allison", "Kristen", "Lakisha", "Latonya", "Ca…
    ## $ race                   <chr> "white", "white", "black", "black", "white", "w…
    ## $ gender                 <chr> "f", "f", "f", "f", "f", "m", "f", "f", "f", "m…
    ## $ years_college          <dbl> 4, 3, 4, 3, 3, 4, 4, 3, 4, 4, 4, 4, 4, 4, 4, 1,…
    ## $ college_degree         <dbl> 1, 0, 1, 0, 0, 1, 1, 0, 1, 1, 1, 1, 1, 1, 1, 0,…
    ## $ honors                 <dbl> 0, 0, 0, 0, 0, 1, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0,…
    ## $ worked_during_school   <dbl> 0, 1, 1, 0, 1, 0, 1, 0, 0, 1, 0, 1, 0, 1, 0, 0,…
    ## $ years_experience       <dbl> 6, 6, 6, 6, 22, 6, 5, 21, 3, 6, 8, 8, 4, 4, 5, …
    ## $ computer_skills        <dbl> 1, 1, 1, 1, 1, 0, 1, 1, 1, 0, 1, 1, 1, 1, 0, 1,…
    ## $ special_skills         <dbl> 0, 0, 0, 1, 0, 1, 1, 1, 1, 1, 1, 0, 1, 0, 1, 1,…
    ## $ volunteer              <dbl> 0, 1, 0, 1, 0, 0, 1, 1, 0, 1, 1, 0, 0, 0, 1, 0,…
    ## $ military               <dbl> 0, 1, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0,…
    ## $ employment_holes       <dbl> 1, 0, 0, 1, 0, 0, 0, 1, 0, 0, 1, 0, 1, 0, 0, 0,…
    ## $ has_email_address      <dbl> 0, 1, 0, 1, 1, 0, 1, 1, 0, 1, 1, 1, 0, 0, 1, 0,…
    ## $ resume_quality         <chr> "low", "high", "low", "high", "high", "low", "h…

1.  Is this an observational study or an experiment? Explain.

# This study is an experiment based the researchers created resumes and then randomly assigned names to these resumes that were meant to signal the gender and race of the applicants. By doing this, i feel they controlled the experimental conditions.

2.  The variable of interest is `received_callback`. What type of
    variable is this? What do the values represent?

# The variable `received_callback` is a binary variable. A type of categorical variable that can take on only two possible values.

`1`: The application received a callback. `0`: The application did not
receive a callback.

3.  For `received_callback`, create an appropriate data visualization
    using `{ggplot2}`. Be sure to provide more descriptive labels (both
    axes labels and value labels - there are many ways to do this) as
    well as an appropriate title.

``` r
ggplot(resume, aes(x = factor(received_callback, labels = c("No Callback", "Received Callback")))) +
  geom_bar(fill = c("#FF9999", "#66CC99")) +
  labs(title = "Distribution of Job Application Callbacks",
       x = "Callback Status",
       y = "Count") +
  theme_minimal() +
  theme(axis.text.x = element_text(angle = 45, hjust = 1))
```

![](activity06-logistic_files/figure-gfm/bar-plot-1.png)<!-- -->

4.  Below, I provide you with a numerical summary table that should
    reiterate (i.e., provides numerical values) your plot in (3). First,
    look at each line of code and describe what I am doing. Then,
    replace “verbatim” with “r” before the code chunk title to produce
    this table.

``` r
resume %>% 
  mutate(received_callback = case_when(
    received_callback == 0 ~ "No",
    received_callback == 1 ~ "Yes"
  )) %>% 
  count(received_callback) %>% 
  mutate(percent = round(n / sum(n) * 100, 2)) %>% 
  knitr::kable()
```

| received_callback |    n | percent |
|:------------------|-----:|--------:|
| No                | 4478 |   91.95 |
| Yes               |  392 |    8.05 |

5.  Using the output from (3) and (4), what do you notice?

# There is significant difference between the number of callbacks received. A vast majority of application did not received the callback. In (3) it is represented visually and in (4) it is represented in a table format.

## Probability and odds

Using your output from (3) and (4), answer the following questions:

6.  What is the probability that a randomly selected résumé/person will
    be called back?

``` r
number_of_callbacks = 392
total_number_of_resumes = 4870

probability_of_callback = number_of_callbacks / total_number_of_resumes
probability_of_callback
```

    ## [1] 0.08049281

7.  What are the [**odds**](https://en.wikipedia.org/wiki/Odds) that a
    randomly selected résumé/person will be called back?

$$
\ odds = p(event) / 1-p(event)
$$

``` r
odds_of_callback = probability_of_callback / (1 - probability_of_callback)
odds_of_callback
```

    ## [1] 0.08753908

## Logistic regression

Logistic regression is one form of a *generalized linear model*. For
this type of model, the outcome/response variable takes one one of two
levels (sometimes called a binary variable or a two-level categorical
variable).

In our activity, $Y_i$ takes the value 1 if a résumé receives a callback
and 0 if it did not. Generally, we will let the probability of a
“success” (a 1) be $p_i$ and the probability of a “failure” (a 0) be
$1 - p_i$. Therefore, the odds of a “success” are:

$$
\frac{Pr(Y_i = 1)}{Pr(Y_i = 0)} = \frac{p_i}{1-p_i}
$$

From your reading, you saw that we use the *logit function* (or *log
odds*) to model binary outcome variables:

$$
\begin{equation*}
\log\left(\frac{p_i}{1-p_i}\right) = \beta_0 + \beta_1 X
\end{equation*}
$$

To keep things simpler, we will first explore a logistic regression
model with a two-level categorical explanatory variable: `race` - the
inferred race associated to the first name on the résumé. Below is a
two-way table (also known as a contingency table or crosstable), where
the rows are the response variable levels, the columns are the
explanatory variable levels, and the cells are the percent (and number
of in parentheses). Note that the values in each column add to 100%.
Replace “verbatim” with “r” before the code chunk title to produce this
table.

``` r
resume %>% 
  mutate(received_callback = case_when(
    received_callback == 0 ~ "No",
    received_callback == 1 ~ "Yes"
  ),
  race = case_when(
    race == "black" ~ "Black",
    race == "white" ~ "White"
  )) %>% 
  group_by(race, received_callback) %>% 
  summarise(n = n()) %>% 
  mutate(percent = round(n / sum(n) * 100, 2),
         percent_n = glue::glue("{percent} ({n})")) %>% 
  select(received_callback, race, percent_n) %>% 
  pivot_wider(
    names_from = race,
    values_from = percent_n
  ) %>% 
  knitr::kable()
```

    ## `summarise()` has grouped output by 'race'. You can override using the
    ## `.groups` argument.

| received_callback | Black        | White        |
|:------------------|:-------------|:-------------|
| No                | 93.55 (2278) | 90.35 (2200) |
| Yes               | 6.45 (157)   | 9.65 (235)   |

Using the above table, answer the following question:

6.  What is the probability that a randomly selected résumé/person
    perceived as Black will be called back?

# 6.45% or 0.0645

7.  What are the **odds** that a randomly selected résumé/person
    perceived as Black will be called back?

``` r
probability_callback_black = 6.45
odds_callback_black = probability_callback_black / (100 - probability_callback_black)
odds_callback_black
```

    ## [1] 0.06894709

This process of calculating conditional (e.g., if a résumé/person
perceived as Black is called back) odds will be helpful as we fit our
logistic model. We will now begin to use the `{tidymodel}` method for
fitting models. A similar approach could be used for linear regression
models and you are encouraged to find out how to do this in your past
activities.

- Replace “verbatim” with “r” before the code chunk title to produce the
  logistic model output.

``` r
# The {tidymodels} method for logistic regression requires that the response be a factor variable
resume <- resume %>% 
  mutate(received_callback = as.factor(received_callback))

logistic_spec <- logistic_reg() %>%
  set_engine("glm")

logistic_spec
```

    ## Logistic Regression Model Specification (classification)
    ## 
    ## Computational engine: glm

``` r
resume_mod <- logistic_spec %>%
  fit(received_callback ~ race, data = resume, family = "binomial")

tidy(resume_mod) %>% 
  knitr::kable(digits = 3)
```

| term        | estimate | std.error | statistic | p.value |
|:------------|---------:|----------:|----------:|--------:|
| (Intercept) |   -2.675 |     0.083 |   -32.417 |       0 |
| racewhite   |    0.438 |     0.107 |     4.083 |       0 |

After doing this, respond to the following questions:

8.  Write the estimated regression equation. Round to 3 digits.

# The variable `RaceWhite` is a dummy variable where:

`RaceWhite` = 0 for resumes perceived as Black `RaceWhite` = 1 for
resumes perceived as White

$$
log(p/1-p) = -2.675 + 0.438 *   racewhite
$$

9.  Using your equation in (8), write the *simplified* estimated
    regression equation corresponding to résumés/persons perceived as
    Black. Round to 3 digits.

$$
log(p/1-p) = -2.675 
$$

Based on your model, if a randomly selected résumé/person perceived as
Black,

10. What are the log-odds that they will be called back?

$$
log(p/1-p) = -2.675 
$$

11. What are the odds that they will be called back? How does this
    relate back to your answer from (7)? *Hint*: In (9) you obtained the
    log-odds (i.e., the natural log-odds). How can you back-transform
    this value to obtain the odds?

# This relates back to the answer from (7), where we calculated the odds based on data. The model provides a similar value.

$$
odds = exp(log-odds)
odds = exp(-2.675) = 0.068906
$$

12. What is the probability that will be called back? How does this
    related back to your answer from (6)? *Hint* Use the odds in (11) to
    calculate this value.

# It provides the same value, demonstrating that the logistic regression model’s estimates align with the observed data.

``` r
odds_black = exp(-2.675)
probability_black = odds_black / (1 + odds_black)
odds_black 
```

    ## [1] 0.06890683

``` r
probability_black
```

    ## [1] 0.06446477

13. How does the output from following code relate to what you obtained
    before (8)? How can you use it help you answer (12)? Replace
    “verbatim” with “r” before the code chunk title to produce the
    logistic model output.

# The odds ratio for racewhite, which is 1.550. This confirms the earlier calculation and interpretation that the odds of receiving a callback are 1.550 times higher for individuals with White-sounding names compared to those with Black-sounding names.

# The odds ratio for racewhite (1.550) directly tells us how much more likely White individuals are to receive a callback compared to Black individuals, reaffirming the calculation we did earlier.

# The `conf.low` and `conf.high` columns give us the lower and upper bounds of the 95% confidence interval for the odds ratios. For racewhite, the interval ranges from 1.257 to 1.915, indicating that we are 95% confident that the true odds ratio falls within this range.

``` r
tidy(resume_mod, exponentiate = TRUE, conf.int = TRUE) %>% 
  knitr::kable(digits = 3)
```

| term        | estimate | std.error | statistic | p.value | conf.low | conf.high |
|:------------|---------:|----------:|----------:|--------:|---------:|----------:|
| (Intercept) |    0.069 |     0.083 |   -32.417 |       0 |    0.058 |     0.081 |
| racewhite   |    1.550 |     0.107 |     4.083 |       0 |    1.257 |     1.915 |

## Challenge: Extending to Mulitple Logistic Regression

We will explore the following question: Is there a difference in call
back rates in Chicago jobs, after adjusting for the an applicant’s years
of experience, years of college, race, and gender? Specifically, we will
fit the following model, where $\hat{p}$ is the estimated probability of
receiving a callback for a job in Chicago.

$$
\begin{equation*}
\log\left(\frac{\hat{p}}{1-\hat{p}}\right) = \hat\beta_0 + \hat\beta_1 \times (\texttt{years\\_experience}) + \hat\beta_2 \times (\texttt{race:White}) + \hat\beta_3 \times (\texttt{gender:male})
\end{equation*}
$$

Note that the researchers have the variable labeled `gender`. Like with
`race`, they limited their resume/name generation to only two
categorizations: “male” and “female”. The authors do not address this
decision in their article or provide any context as to what they mean by
“gender”.

- Replace “verbatim” with “r” before the code chunk title to produce
  this table.

``` r
resume_subet <- resume %>% 
  filter(job_city == "Chicago") %>% 
  mutate(race = case_when(
         race == "white" ~ "White",
         TRUE ~ "Black"
       ),
       gender = case_when(
         gender == "f" ~ "female",
         TRUE ~ "male"
       )) %>% 
  select(received_callback, years_experience, race, gender)
```

Describe what the above code does in the context of this problem.

# The code filters the `resume` dataset for entries from Chicago, recodes `race` and `gender` into “White”/“Black” and “female”/“male”, and selects only the relevant columns for analysis.

## Relationship Exploration

There are many variables in this model. Let’s explore each explanatory
variable’s relationship with the response variable. Note that I tried to
explore this using `GGally::ggbivariate`, but kept running into an error
that I did not have time to explore.

- Create a new R code chunk and create an appropriate data visualization
  to explore the relationship between `resume_subet` and each of the
  explanatory variables, then run your code chunk or knit your document.

``` r
# Plot for 'race' and callback rates
ggplot(resume_subet, aes(x = race, fill = received_callback)) +
  geom_bar(position = "fill") +
  labs(title = "Callback Rates by Race", x = "Race", y = "Proportion") +
  scale_fill_manual(values = c("0" = "red", "1" = "green"), labels = c("No Callback", "Received Callback")) +
  theme_minimal()
```

![](activity06-logistic_files/figure-gfm/Plot%20for%20race%20and%20callback%20rates-1.png)<!-- -->

``` r
# Plot for 'gender' and callback rates
ggplot(resume_subet, aes(x = gender, fill = received_callback)) +
  geom_bar(position = "fill") +
  labs(title = "Callback Rates by Gender", x = "Gender", y = "Proportion") +
  scale_fill_manual(values = c("0" = "red", "1" = "green"), labels = c("No Callback", "Received Callback")) +
  theme_minimal()
```

![](activity06-logistic_files/figure-gfm/Plot%20for%20gender%20and%20callback%20rates-1.png)<!-- -->

``` r
# Plot for 'years_experience' and callback rates
ggplot(resume_subet, aes(x = received_callback, y = years_experience, color = received_callback)) +
  geom_boxplot() +
  labs(title = "Years of Experience and Callback Rates", x = "Callback Received", y = "Years of Experience") +
  scale_color_manual(values = c("0" = "red", "1" = "green"), labels = c("No Callback", "Received Callback")) +
  theme_minimal()
```

![](activity06-logistic_files/figure-gfm/Plot%20for%20years_experience%20and%20callback%20rates-1.png)<!-- -->

After doing this, answer the following question:

14. Describe any patterns. What do you notice?

# These visualizations indicate patterns regarding callbacks:

1.  **Race**: There’s a noticeable disparity in callback rates by race,
    with individuals perceived as White receiving a higher proportion of
    callbacks than those perceived as Black.

2.  **Gender**: Callback rates appear similar across genders, with a
    slightly higher proportion of callbacks for resumes labeled as male,
    but the difference is not as much as it is by race.

3.  **Years of Experience**: The box plots suggest that there isn’t a
    significant difference in the median years of experience between
    those who did and did not receive callbacks. However, there is a
    wider range of experience among those who did not receive callbacks,
    indicated by the longer box and more outliers.

## Fitting the model

Aside: I kept running into an issue using `{tidymodels}` to fit this
model so I defaulted back to a method that I know works using `glm`. I
will keep exploring why I was experiencing issues and update you all
with a more modern method later this semester. Using the logistic model
code above, create a new code chunk below to fit the model to address
our research question.

Focusing on the estimated coefficient for `years_experience`, we would
say:

> For each additional year of experience for an applicant in Chicago, we
> expect the *log odds* of an applicant receiving a call back to
> increase by 0.045 units. Assuming applicants have similar time in
> spent in college, similar inferred races, and similar inferred gender.

This interpretation is somewhat confusing because we are describing this
in *log odds*. Fortunately, we can convert these back to odds using the
following transformation:

$$
\text{odds} = e^{\log(\text{odds})}
$$

You saw how to do this in (13)

``` r
chicago_log_model <- glm(received_callback ~ years_experience + race + gender, 
                         family = "binomial", data = resume_subet)

summary(chicago_log_model)
```

    ## 
    ## Call:
    ## glm(formula = received_callback ~ years_experience + race + gender, 
    ##     family = "binomial", data = resume_subet)
    ## 
    ## Coefficients:
    ##                  Estimate Std. Error z value Pr(>|z|)    
    ## (Intercept)      -3.27861    0.18132 -18.082  < 2e-16 ***
    ## years_experience  0.04487    0.01631   2.751  0.00594 ** 
    ## raceWhite         0.42645    0.15679   2.720  0.00653 ** 
    ## gendermale        0.58011    0.20306   2.857  0.00428 ** 
    ## ---
    ## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
    ## 
    ## (Dispersion parameter for binomial family taken to be 1)
    ## 
    ##     Null deviance: 1333.7  on 2703  degrees of freedom
    ## Residual deviance: 1313.5  on 2700  degrees of freedom
    ## AIC: 1321.5
    ## 
    ## Number of Fisher Scoring iterations: 5

After doing this, answer the following question:

15. Interpret the estimated coefficient for `years_experience`.

``` r
coef_years_experience <- coef(chicago_log_model)["years_experience"]
# Convert log odds to odds
odds_years_experience <- exp(coef_years_experience)
odds_years_experience
```

    ## years_experience 
    ##          1.04589

## Assessing model fit

Now we want to check the residuals of this model to check the model’s
fit. As we saw for multiple linear regression, there are various kinds
of residuals that try to adjust for various features of the data. Two
new residuals to explore are *Pearson residuals* and *Deviance
residuals*.

**Pearson residuals**

The Pearson residual corrects for the unequal variance in the raw
residuals by dividing by the standard deviation.

$$
\text{Pearson}_i = \frac{y_i - \hat{p}_i}{\sqrt{\hat{p}_i(1 - \hat{p}_i)}}
$$

**Deviance residuals**

Deviance residuals are popular because the sum of squares of these
residuals is the deviance statistic. We will talk more about this later
in the semester.

$$
d_i = \text{sign}(y_i - \hat{p}_i)\sqrt{2\Big[y_i\log\Big(\frac{y_i}{\hat{p}_i}\Big) + (1 - y_i)\log\Big(\frac{1 - y_i}{1 - \hat{p}_i}\Big)\Big]}
$$

Since Pearson residuals are similar to residuals that we have already
explored, we will instead focus on the deviance residuals.

- Replace “verbatim” with “r” before the code chunk title to produce
  this table. You might need to update other R objects in this code - my
  model was called `mult_log_mod`

``` r
# To store residuals and create row number variable
mult_log_aug <- augment(chicago_log_model, type.predict = "response", 
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

![](activity06-logistic_files/figure-gfm/residual-plots-1.png)<!-- -->

``` r
# Plot residuals vs row number
ggplot(data = mult_log_aug, aes(x = id, y = .resid)) + 
geom_point() + 
geom_hline(yintercept = 0, color = "red") + 
labs(x = "id", 
     y = "Deviance residuals", 
     title = "Deviance residuals vs. id")
```

![](activity06-logistic_files/figure-gfm/residual-plots-2.png)<!-- -->

Here we produced two residual plots: the deviance residuals against the
fitted values and the deviance variables against the index id (an index
plot). The index plot allows us to easily see some of the more extreme
observations - there are a lot ($|d_i| > 2$ is quiet alarming). The
residual plot may look odd (why are there two distinct lines?!?), but
this is a pretty typical shape when working with a binary response
variable (the original data is really either a 0 or a 1). In general
because there are so many extreme values in the index plot, this model
leaves room for improvement.
