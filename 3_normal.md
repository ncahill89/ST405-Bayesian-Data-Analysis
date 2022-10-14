Bayesian Analysis - Single Parameter Normal Model
================
Dr Niamh Cahill (she/her)


## Example: Cognitive Test Scores

Data (y) are available on the cognitive test scores of three- and
four-year-old children in the USA. The sample contains 434 observations.

![](3_normal_files/figure-gfm/unnamed-chunk-1-1.png)<!-- -->

## Normal distribution with known variance

We will assume a normal model for the data where
$y_i|\mu,\sigma^2 \sim N(\mu, \sigma^2)$. Assume $\sigma^2$ is known
where $\sigma = 20.4$

-   Specify the likelihood for $\mu$

$p(y|\mu, \sigma^2) = \prod_{i=1}^{n} \frac{1}{\sqrt {2\pi\sigma^2}}exp \bigg(-\frac{1}{2\sigma^2}(y_i - \mu)^2\bigg)$

-   Specify a prior for $\mu$

$\mu \sim N(\mu_0, \sigma^2_{_0})$

-   Use Bayes’ rule to obtain the posterior distribution

$p(\mu|y) \propto p(y|\mu)p(\mu)$

-   As it turns out, the posterior is also a normal distribution

$$\mu|y \sim N \bigg(\frac{n\bar{y}/\sigma^2 + \mu_0/\sigma^2_{0}}{n/\sigma^2 + 1/\sigma^2_{0}}, {\frac{1}{n/\sigma^2 + 1/\sigma^2_{0}}}\bigg)$$

## Prior vs Posterior for $\mu$

-   Choose prior mean and variance, e.g., $\mu_0 = 80$,
    $\sigma_{0} = 10$

![](3_normal_files/figure-gfm/unnamed-chunk-2-1.png)<!-- -->

-   Result: $\hat{\mu} =$ 86.73, 95% CI: (84.82, 88.64)

## Normal distribution with known mean

-   Assume $\mu$ is known where $\mu = 86.79$

-   Usually work with precision i.e., $\tau = 1/\sigma^2$

-   Specify a prior for $\tau$

    -   Popular prior for the precision of a normal distribution is a
        gamma prior e.g., $\tau \sim Gamma(a, b)$ where
        $E[\tau] = \frac{a}{b}$ and $Var[\tau] = \frac{a}{b^2}$

    -   $p(\tau|a,b) = \frac{b^a}{\Gamma(a)}\tau^{a-1}e^{-b\tau}$ for
        $\tau >0$ and $a,b > 0$

-   Use Bayes’ rule to obtain the posterior distribution

$p(\tau|y) \propto p(y|\tau)p(\tau)$

-   The posterior will also be a gamma distribution

$\tau|y \sim Gamma \bigg(a + n/2, b + 1/2 \sum_{i=1}^n (y_i - \mu)^2\bigg)$

## Prior vs Posterior for $\tau$

-   Choose parameter values for the prior distribution for $\tau$, e.g.,
    $a = 0.1$, $b = 0.1$

![](3_normal_files/figure-gfm/unnamed-chunk-3-1.png)<!-- -->

-   Result: Through simulation from the posterior for $\sigma$ we find
    $\hat{\sigma} = 20.23$, 95% CI: (18.96, 21.63)
    
## Appendix: R code

```{r}
library(rstanarm)
library(tidyverse)

y <- kidiq$kid_score
n <- length(y)

#-------------------
# exact solutions
# prior and posterior for mu, conditional on sigma (assuming sigma = sd.y)
sd.y<- 20.4

# prior parameters
mu0 <- 80
sigma0 <- 10

# Posterior of mu, assuming sigma = sd.y (sample SD)
mupost.mean = (n*mean(y)/(sd.y^2) + mu0/(sigma0^2))/(n/(sd.y^2) + 1/(sigma0^2))
mupost.sd = sqrt(1/(n/(sd.y^2) + 1/(sigma0^2)))

# Credible interval
lwr <- qnorm(0.05, mean = mupost.mean, sd = mupost.sd)
upr <- qnorm(0.95, mean = mupost.mean, sd = mupost.sd)

# set up for plotting
mu_grid <- seq(50,120)
mu_prior <- dnorm(mu_grid,mu0,sigma0)
mu_posterior <- dnorm(mu_grid,mupost.mean,mupost.sd)

mu_res <- tibble::tibble(mu_grid, mu_prior,mu_posterior) %>% 
  tidyr::pivot_longer(-mu_grid,
                      names_to = "distribution",
                      values_to = "value") %>% 
  dplyr::mutate(distribution = factor(distribution, levels = c("mu_prior", "mu_posterior")))

# plot prior and posterior
ggplot(mu_res, aes(x = mu_grid, y = value, colour = distribution)) +
  geom_line() +
  labs(colour = "") +
  facet_wrap(~distribution, scales = "free_y") +
  ylab("Density") +
  xlab(expression(mu))


#-------------------
# exact solutions
# prior and posterior for tau, conditional on mu (assuming mu = ybar)
mu.y <- 87

# prior parameters
a <- 2
b <- 4

# Posterior of tau
post.a = a + n/2
post.b = b + 1/2*(sum((y-mu.y)^2))

sqrt(1/(post.a/post.b))

# Credible intervals
lwr <- qgamma(0.025, post.a, post.b)
upr <- qgamma(0.975, post.a, post.b)

# Set up for plotting
tau_grid <- seq(0,3,by = 0.01)
tau_prior <- dgamma(tau_grid,a,b)
tau_posterior <- dgamma(tau_grid,post.a,post.b)

tau_res <- tibble::tibble(tau_grid, tau_prior,tau_posterior) %>% 
  tidyr::pivot_longer(-tau_grid,
                      names_to = "distribution",
                      values_to = "value") %>% 
  dplyr::mutate(distribution = factor(distribution, levels = c("tau_prior", "tau_posterior")))

# plot prior and posterior
ggplot(tau_res, aes(x = tau_grid, y = value, colour = distribution)) +
  geom_line() +
  labs(colour = "") +
  ylab("Density")+
  xlab(expression(tau))+
  facet_wrap(~distribution, scales = "free_y")

# sample from the posterior for tau
tau_samps <- rgamma(10000,post.a,post.b)
plot(density(tau_samps))

# get sigma samples and summaries
sigma_samps <- sqrt(1/tau_samps)
plot(density(sigma_samps))
quantile(sigma_samps, probs = c(0.025,0.5,0.975))
```
