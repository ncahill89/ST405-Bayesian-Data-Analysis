Bayesian Data Analysis
================
Dr Niamh Cahill (she/her)
Hierarchical Regression Models

## Hierarchical Linear Models

Hierarchical regression models are useful as soon as there are
predictors at different levels of variation. For example,

-   Studying scholastic achievement we may have information about
    individual students (for example, family background), class-level
    information (char- acteristics of the teacher), and also information
    about the school (educational policy, type of neighborhood).

-   Analysis of data obtained by stratified or cluster sampling using
    regression models for strata or clusters.

## Hierarchical Linear Models

Predictors can be introduced for each of the higher-level units in the
data e.g., for the classes in the educational example or for the strata
or clusters in the sampling example.

-   But this will in general dramatically increase the number of
    parameters in the model,

-   and sensible estimation of these is only possible through further
    modeling, in the form of a population distribution.

-   this can take a simple exchangeable or independent and identically
    distributed form, but it may also be reasonable to consider a
    further regression model at this second level, to allow for the
    predictors defined at this level.

In principle there is no limit to the number of levels of variation that
can be handled in this way and Bayesian methods provide ready guidance
on handling the estimation of unknown parameters.

## Math Scores Data

Suppose that we have some data on the math scores for students in 10
different schools.

-   Each school contributes a number of data points.

-   We also have predictors on number of completed homeworks and type of
    school (public, private)

-   School-specific regression lines can describe the relationship
    between math scores and homework completed at the school level.

-   If there is a lack of data for some schools then parameter estimates
    can be informed by all other schools.

-   We can consider the school type as a predictor for the school level
    parameters.

-   We can get an overall estimate of the relationship between math
    scores and no. of homeworks completed.

## Visualising the Data

![](8_BHM_Reg_files/figure-gfm/unnamed-chunk-2-1.png)<!-- -->

## Hierarcical Regression on Individuals within Groups

**Goal:** Model math scores with a linear regression and share
information across schools to provide information where data are sparse.
Include relevant predictors at the different levels in the hierarchy.

**Modelling Option**

$y_{i} \sim N(\mu_i,\sigma^2)$

$\mu_i = \alpha_{j[i]} + \beta x_i$ for $i = 1 \ldots n$

$\alpha_j \sim N(\mu_{\alpha},\sigma^2_{\alpha})$ for $j = 1 \ldots m$

where $x_i$ is the number of homeworks completed.

$\beta \sim Normal(0,10^2)$

$\sigma \sim Uniform(0,30)$

## Including Predictors at the Group Level

Additionally, type of school is potentially informative for
across-school variation in average math scores and in this case we can
further extend the model such that

$\alpha_j \sim N(\mu_{\alpha_j},\sigma^2_{\alpha})$ for $j = 1 \ldots m$

$\mu_{\alpha_j} = \gamma_0 + \gamma_1u_j$

where $u_j$ is the school type (public or private).

$\gamma_0 \sim Uniform(0,100)$

$\gamma_1 \sim Normal(0,10^2)$

$\sigma_{\alpha} \sim dt(0,2^2,1)$

## The model DAG

![](8_BHM_Reg_files/figure-gfm/unnamed-chunk-3-1.png)<!-- -->

## The JAGS model

``` r
bhregmodel = "
model{
  for(i in 1:N)
{
 y.i[i] ~ dnorm(mu.i[i],sigma^-2) # data model
 mu.i[i] <- alpha.j[school[i]] + beta*x.i[i]
} # end i loop

for(j in 1:n_school)
{
alpha.j[j] ~ dnorm(mu_alpha.j[j],sigma_alpha^-2)
mu_alpha.j[j] <- gamma0 + gamma1*u.j[j]
}

beta ~ dnorm(0,10^-2)
gamma0 ~ dunif(0,100)
gamma1 ~ dnorm(0,10^-2)

sigma ~ dunif(0,30)
sigma_alpha ~ dt(0,2^-2,1)T(0,)
 "
```

## Results

    ## # A tibble: 23 × 8
    ##    `1:10` .variable   .value .lower .upper .width .point .interval
    ##     <int> <chr>        <dbl>  <dbl>  <dbl>  <dbl> <chr>  <chr>    
    ##  1      1 alpha.j      44.1  40.9    46.9    0.95 median qi       
    ##  2      2 alpha.j      40.8  36.9    44.7    0.95 median qi       
    ##  3      3 alpha.j      45.6  40.9    52.0    0.95 median qi       
    ##  4      4 alpha.j      42.2  38.7    45.5    0.95 median qi       
    ##  5      5 alpha.j      47.5  44.3    50.9    0.95 median qi       
    ##  6      6 alpha.j      44.7  41.6    47.7    0.95 median qi       
    ##  7      7 alpha.j      58.2  54.5    61.9    0.95 median qi       
    ##  8      8 alpha.j      44.4  38.6    50.3    0.95 median qi       
    ##  9      9 alpha.j      44.6  41.4    47.8    0.95 median qi       
    ## 10     10 alpha.j      45.2  42.2    48.4    0.95 median qi       
    ## 11     NA beta          1.35  0.478   2.18   0.95 median qi       
    ## 12      1 mu_alpha.j   44.5  41.8    47.1    0.95 median qi       
    ## 13      2 mu_alpha.j   44.5  41.8    47.1    0.95 median qi       
    ## 14      3 mu_alpha.j   44.5  41.8    47.1    0.95 median qi       
    ## 15      4 mu_alpha.j   44.5  41.8    47.1    0.95 median qi       
    ## 16      5 mu_alpha.j   44.5  41.8    47.1    0.95 median qi       
    ## 17      6 mu_alpha.j   44.5  41.8    47.1    0.95 median qi       
    ## 18      7 mu_alpha.j   57.7  49.6    64.0    0.95 median qi       
    ## 19      8 mu_alpha.j   44.5  41.8    47.1    0.95 median qi       
    ## 20      9 mu_alpha.j   44.5  41.8    47.1    0.95 median qi       
    ## 21     10 mu_alpha.j   44.5  41.8    47.1    0.95 median qi       
    ## 22     NA sigma         7.86  7.14    8.71   0.95 median qi       
    ## 23     NA sigma_alpha   2.57  0.914   5.19   0.95 median qi

## Results: School specific intercepts

Recall: $\alpha_j \sim N(\gamma_0 + \gamma_1u_j, \sigma_{\alpha})$

![](8_BHM_Reg_files/figure-gfm/unnamed-chunk-7-1.png)<!-- -->

## Results: Posterior samples of $\mu_{ij}$

![](8_BHM_Reg_files/figure-gfm/unnamed-chunk-9-1.png)<!-- -->

## Results: Regression lines + 95% Credible Intervals

![](8_BHM_Reg_files/figure-gfm/unnamed-chunk-10-1.png)<!-- -->

## Results: Overall regression lines for each school type

![](8_BHM_Reg_files/figure-gfm/unnamed-chunk-11-1.png)<!-- -->

## Posterior predicitive checks (density overlay)

![](8_BHM_Reg_files/figure-gfm/unnamed-chunk-12-1.png)<!-- -->

## Posterior predicitive checks (test statistic)

Test statistic = max(y) - min(y)

![](8_BHM_Reg_files/figure-gfm/unnamed-chunk-13-1.png)<!-- -->

## Further extend the model to include varying slopes

$y_{i} \sim N(\mu_i,\sigma^2)$

$\mu_i = \alpha_{j[i]} + \beta_{j[i]} x_i$ for $i = 1 \ldots n$

$\beta \sim Normal(\mu_{\beta}, \sigma_{\beta})$

![](8_BHM_Reg_files/figure-gfm/unnamed-chunk-14-1.png)<!-- -->

## Results: Compare Regression lines

![](8_BHM_Reg_files/figure-gfm/unnamed-chunk-18-1.png)<!-- -->

## Posterior predicitive checks (density overlay)

![](8_BHM_Reg_files/figure-gfm/unnamed-chunk-19-1.png)<!-- -->

## Posterior predicitive checks (test statistic)

Test statistic = max(y) - min(y)

![](8_BHM_Reg_files/figure-gfm/unnamed-chunk-20-1.png)<!-- -->
