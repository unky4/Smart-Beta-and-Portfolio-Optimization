# Udacity's Artificial Intelligence for Trading Nanodegree

## Project: Smart Beta and Portfolio Optimization

## Table of content

1. [Project Overview](#overview)
2. [Project Description](#description)
3. [Project Overview](#overview) 
4. [Data Description](#data)
5. [Part 1: Smart Beta Portfolio](#first_part)
    1.  [Weights](#weights)
    2. [Returns](#returns)
    3. [Tracking Error](#tracking_error)
6. [Part 2: Portfolio Optimisation](#second_part)
    1. [Optimisation Description](#optimisation_description)
    2. [Optimised Portfolio](#optimised_portfolio)
    3. [Rebalance Portfolio Over Time](#rebalance_portfolio)
7. [Summary of What I Learned](#summary_learned)
8. [Packages and Files](#packages)

***

<a id='description'></a>
### Project Description

This project will build a smart beta portfolio, and in order to determine its performance, we compare it to a benchmark index by calculating the tracking error against this index. First, we build a portfolio using quadratic programming (to optimise the weights), then rebalance it and evaluate the (rebalanced) portfolio's performance by calculating its turnover. Ultimately, such a metric allows us to find the optimal rebalancing Frequency.

<a id='overview'></a>
### Project Overview

Smart beta has a broad meaning, but we can say in practice that when we use the universe of stocks from an index and then apply some weighting scheme (other than market cap weighting), it can be considered a type of smart beta fund. A Smart Beta portfolio generally gives investors exposure or "beta" to one or more types of market characteristics (or factors) that are believed to predict prices while giving investors diversified broad exposure to a particular market. Smart Beta portfolios generally target momentum, earnings quality, low volatility, and dividends or some combination. Smart Beta Portfolios are generally rebalanced infrequently and follow relatively simple rules or passively managed algorithms. Model changes to these types of funds are also rare, requiring prospectus filings with the US Security and Exchange Commission in the case of US-focused mutual funds or ETFs. Smart Beta portfolios are generally long-only; they do not short stocks.

In contrast, a purely alpha-focused quantitative fund may use multiple models or algorithms to create a portfolio. The portfolio manager retains discretion in upgrading or changing the types of models and how often to rebalance the portfolio in an attempt to maximise performance in comparison to a stock benchmark. In addition, managers may have discretion to short stocks in portfolios.

Imagine you are a portfolio manager who wishes to try different portfolio weighting methods. One way to design a portfolio is to look at specific accounting measures (fundamentals) that, based on past trends, indicate stocks that produce better results. For instance, we may start with a hypothesis that dividend-issuing stocks tend to perform better than stocks that do not. This may not always be true of all companies; for instance, Apple does not issue dividends but has had a good historical performance. The hypothesis about dividend-paying stocks may go something like this:
> Companies that regularly issue dividends may also be more prudent in allocating their available cash, indicating that they might be more conscious of prioritising shareholder interests. For instance, a CEO may decide to reinvest the cash into pet projects that produce low returns. Alternatively, the CEO may do some analysis, identify that reinvesting within the company produces lower returns than a diversified portfolio, and decide that shareholders would be better served if they were given the cash (in the form of dividends). So, according to this hypothesis, dividends may be both a proxy for how the company is doing (in terms of earnings and cash flow) and, at the same time, a signal that the company acts in the best interest of its shareholders. Of course, it is important to test whether this works in practice.

One may also have another hypothesis that wishes to design a portfolio that can be made into an ETF. For example, you may find that investors may wish to invest in passive beta funds but wish to have less risk exposure (less volatility) in their investments. The goal of having a low volatility fund that still produces returns similar to an index may be appealing to investors who have a shorter investment time horizon and so are more risk-averse. Hence, the objective of your proposed portfolio is to design a portfolio that closely tracks an index while also minimising the portfolio variance. Furthermore, if this portfolio can match the index's returns with less volatility, then it has a higher risk-adjusted return (same return, lower volatility).

Smart Beta ETFs can be designed with both of these two general methods (among others): alternative weighting and minimum volatility ETF.

<a id='data'></a>
### Data Description

We select large dollar volume stocks for this universe of stocks and use the end of day from [Quotemedia](https://www.quotemedia.com/). Note that we are using such a universe due to its high liquidity.

Unfortunately, Udacity does not have a [licence](https://github.com/udacity/artificial-intelligence-for-trading) to redistribute these data; however, they are working on alternatives to this problem.

<a id='first_part'></a>
### Part 1: Smart Beta Portfolio

In Part 1 of this project, we build a portfolio where the weights are chosen based on the dividend yields over time. Such a portfolio could be incorporated into a smart beta EFT which we will compare to a market cap weighted index to measure its performance.

> Note: in practice, one would probably get the index weights from a data vendor (such as companies that create indices, like MSCI, FTSE, Standard and Poor's), but we will simulate a market cap weighted index for this exercise.

<a id='weights'></a>
#### Weights

First, we need to calculate the index weights for each stock in the index and for each date which look like:

![Index Weights](/images/IndexWeights.png)

Then, we construct our portfolio's weights by calculating the weights for each stock based on its total dividend yield over time:

![EFT Weights](/images/ETFWeights.png)

<a id='returns'></a>
#### Returns

Now we can calculate the close returns for all stock and dates from the price data.
> Note: we are implementing returns and not log returns since we deal with volatility, so we do not have to use log returns.

![Close Returns](/images/close_returns.png)

With the returns of each stock computed, we can compute the returns for both the index and ETF.

![Index Returns](/images/IndexReturns.png)

![EFT Returns](/images/ETFReturns.png)

To compare the ETF and Index performance, we can calculate the tracking error. However, we first need to calculate both the index and ETF cumulative returns before that.

![Cumulative Returns](/images/cumulative_returns.png)

<a id='tracking_error'></a>
#### Tracking Error

Now that we have everything to check the performance of the smart beta portfolio, we can calculate the annualised tracking error against the index. For reference, we use the following annualised tracking error function:  
![Annualized Tracking Error Function](/images/atef.png | width=200)
Where _r<sub>p</sub>_ denotes the portfolio/ETF returns, and _r<sub>b</sub>_ is the benchmark returns.
> Note: When calculating the sample standard deviation, the delta degrees of freedom is 1 (which is also the default value).

***Hence, the annualised tracking error for our Smart Beta is 0.102.***

<a id='second_part'></a>
### Part 2: Portfolio Optimisation

Now, let us create a second portfolio. Although we reuse the market cap weighted index, this one will be independent of the dividend-weighted portfolio we created in [Part 1](#first_part).

On the one hand, we would like to minimise the portfolio's variance, and on the other hand, we also aim to closely track a market cap weighted index. In other words, we try to minimise the distance between the weights of our portfolio and the weights of the index.
![Portfolio Problem](/images/portfolio_problem.png | width=300)
where _m_ is the number of stocks in the portfolio, and 𝜆 is a scaling factor to be chosen.

Why are we doing this? One way that investors evaluate a fund is by how well it tracks its index. The fund is still expected to deviate from the index within a certain range in order to improve fund performance. A way for a fund to track the performance of its benchmark is by keeping its asset weights similar to the weights of the index. We would expect that if the fund has the same stocks as the benchmark and the same weights for each stock as the benchmark, the fund will yield about the same returns as the benchmark. By minimising a linear combination of both the portfolio risk and distance between portfolio and benchmark weights, we attempt to balance the desire to minimise portfolio variance to track the index.

<a id='optimisation_description'></a>
#### Optimisation Description

**Covariance:** To calculate the portfolio's variance, we calculate the covariance of the returns, denoted here by **P** which in an _mxm_ matrix (for _m_ stocks) containing the covariance between each pair of stocks.

![Covaraince](/images/CovarianceReturnsCorrelationMatrix.png)

**Portfolio variance:** Then using this covariance matrix **P**, we can calculate the portfolio's variance by _𝜎<sup>2</sup><sub>p</sub>=**x<sup>T</sup>Px**_.

**Distance from the weights:** Furthermore, we aim for portfolio weights that track the index closely, so we want to minimise the distance between them, which we calculate by simply taking the _L<sub>2</sub>_ norm between them.

**Objective function:** While we want to minimise the portfolio variance and the distance of the portfolio weights from the index weights, we also aim to choose a scaling constant _𝜆_. This allows us to choose how much priority we give to minimising the difference from the index relative to minimising the variance of the portfolio.

**Constraints:** We have two constraints in this model. First, we want the weights to sum up to 1, but we only aim for long positions (no shorting), so all weights must be positive.

<a id='optimised_portfolio'></a>
#### Optimised Portfolio

Now, we can generate the optimal ETF weights without rebalancing, which we can do by feeding in the covariance of the entire data history. However, we also need to feed in a set of index weights. We choose to go with the average weights of the index over time.

![Optimised Portfolio](/images/optimised_portfolio.png)

<a id='rebalance_portfolio'></a>
#### Rebalance Portfolio Over Time

Before, the single optimised ETF portfolio used the same weights for the entire history. However, this scenario might not yield the optimal weights for the entire period. So instead, let us rebalance the portfolio over the same period instead of using the same weights. Here, we rebalance our portfolio every _n_ number of days and use data from the past _k_ number of days, allowing us to compute the optimal weights for each of these periods.

***Finally, once we have finished rebalancing our portfolio, we can calculate the annual portfolio turnover, which, in our case, is 16.73%.***

<a id='summary_learned'></a>
### Summary of What I Learned

In this project, I learned how to construct a smart portfolio and use quadratic programming to calculate the weights for the stocks within this portfolio. Then, I compared such a portfolio to a benchmark index by calculating its tracking error against this index. This project also allowed me to gain knowledge about how to rebalance and evaluate such a portfolio. More specifically, I found the optimal rebalancing frequency by calculating the portfolio's annualised turnover.

<a id='packages'></a>
### Packages and Files

Files and Folders:
- `project_3_starter.ipynb` is the jupyter notebook of the project.
- `project_3_starter.html` is the HTML version of the notebook.
- graph folder includes all iterative plots that cannot be shown in the jupyter notebook.
- images folder contains the plots shown above in the documentation.

Custom packages:
- `helper` and `project_helper` packages contain utility and graph functions.
- `project_tests` include all unit tests for this project
- `tests` generates various test cases for the unit tests

The necessary libraries defined in `requirements.txt` are the followings:
- [colour==0.1.5](https://github.com/vaab/colour)
- [cvxpy==1.0.3](https://github.com/cvxgrp/cvxpy/)
- [cycler==0.10.0](https://matplotlib.org/cycler/)
- [numpy==1.13.3](http://www.numpy.org/)
- [pandas==0.21.1](https://github.com/pandas-dev/pandas)
- [plotly==2.2.3](https://plot.ly/python/)
- [pyparsing==2.2.0](https://github.com/pyparsing/pyparsing/)
- [python-dateutil==2.6.1](https://dateutil.readthedocs.io/en/stable/)
- [pytz==2017.3](https://pythonhosted.org/pytz/)
- [requests==2.18.4](http://docs.python-requests.org/en/master/)
- [scipy==1.0.0](https://www.scipy.org/)
- [scikit-learn==0.19.1](https://scikit-learn.org/stable/)
- [six==1.11.0](https://github.com/benjaminp/six)
- [tqdm==4.19.5](https://tqdm.github.io/)