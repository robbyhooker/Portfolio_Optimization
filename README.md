# Portfolio Optimization
 Exploring different portfolio optimization methods on a control set of securities
 
 ## Table of Contents

1. [Dataset](#dataset)
2. [Exploratory Data Analysis](#exploratory-data-analysis)
3. [Risk Parity](#risk-parity)
4. [Mean-Variance](#mean-variance)
5. [Genetic Algorithms](#genetic-algorithms)
6. [Future Work](#future-work)

## Dataset
In forming the list of control securties I wanted to ensure we diversify across asset classes such as stocks, bond, commodoties, real estate, etc. The simplest way to do so was to use ETFs such as SPY, which can represent asset classes.<br>

The is daily adjusted closing prices for the securities below. It is collected from yahoo finance using the Python package 'yfinance' and ranges from 2010-01-01 to 2023-01-01.
### Securities
Large-Cap US Equity: SPDR S&P 500 ETF (SPY)<br>
US Treasury Bonds: iShares 7-10 Year Treasury Bond ETF (IEF)<br>
Corporate Bonds: iShares iBoxx $ Investment Grade Corporate Bond ETF (LQD)<br>
Oil: United States Oil Fund (USO)<br>
Gold: SPDR Gold Shares (GLD)<br>
US Real Estate: Vanguard Real Estate ETF (VNQ)<br>
Global Real Estate: SPDR Dow Jones Global Real Estate ETF (RWO)<br>
Small-Cap Stocks: iShares Russell 2000 ETF (IWM)<br>
High Yield Bonds: iShares iBoxx $ High Yield Corporate Bond ETF (HYG)<br>
Broad Commodities: Invesco DB Commodity Index Tracking Fund (DBC)<br>
European Markets: Vanguard FTSE Europe ETF (VGK)<br>

## Exploratory Data Analysis
Before diving into the optimization methods I wanted to do some high level EDA on our Data<br>

In the correlation chart below we see how our securities relate to eachother<br>
The main observation here is that most of the securities have a positive correlation, except commodoties like USO, GLD, and DBC.<br>
Introducing such securities to the portfolio is theoretically good for the overall risk profile.<br>

![Securities Correlation Heatmap](assets/data_corr.png)

We can also plot the prices of each security over the time frame to give insights into the risk and return profiles of them individually. For example below we see that SPY offers high returns in the period but is more volatile (relativley):<br>

![Time Series of Security Prices](assets/time_series.png)

Normalzing the prices improves chart readability:

![Normalized Time Series of Security Prices](assets/normalized_tis.png)

## Risk Parity 
The theory behind Risk Parity Optimization is to purchase securites such that each one contributes an equal amount of risk to the portfolio. This can be achieved generally by purchasing less of the more risky securities and vice versa. Or mathematically:

![RC_i = RC_j \ \forall i,j](https://latex.codecogs.com/png.latex?\color{white}{RC_i%20%3D%20RC_j%20%5C%20%5Cforall%20i%2Cj})

Where<br>

![RC_i = \text{weight}_i \cdot \text{risk}_i](https://latex.codecogs.com/png.latex?\color{white}RC_i%20=%20\text{weight}_i%20\cdot%20\text{risk}_i) 

So with that, we impliment the simple algorithm in python, using scipy.minimize to minimize the difference in risk contribtution from each asset.<br>

We find the following results:<br>
Annualized Expected Portfolio Return: 4.57%<br>
Annualized Portfolio Risk (Volatility): 6.87%<br>\
This is an underwhelming result, but it is not suprising as we have some low volatility, low return securities in the portfolio which we would expect to get a lot of the capital distribution from this method.<br>

Indeed we see below that over 50% of capital is in Government and Corporate Bonds, as they have low risk profiles.
The problem with this is that we don't expose ourselves to enough potential returns (in my opinion)<br>

![Risk Parity Weights](assets/rp_pie.png)

Below are the cummulative returns of all the securities and the risk parity porfolio. I takeaway that this is a good strategy for someone who is quite risk averse, but still wants the benefits of diversification, rather than just buying, say, U.S. Treasuries.<br>

![Risk Parity Returns](assets/rp_returns.png)

I also think this would be an interesting method to apply to just the stock market, that way you could see some higher returns. For example instead of using the standard S&P 500 Index Fund, you could weight stocks according to risk parity.  

## Mean-Variance
My reference for this method: [View Mean-Variance CAPM PDF](https://www.columbia.edu/~mh2078/FoundationsFE/MeanVariance-CAPM.pdf)

In reading about Markowitz's mean-variance method and the CAPM model, I was able to gain the most intuition from the mathematical formulas and graphs, so I'll lean on those here.<br>

Also note that I don't consider our list of securities to contain a risk-free asset.<br>
We assume the we have n risky assets which make up the return vector 𝑅<br> 
We also assume that 𝑅 follows a Multivariate Normal distribution with mean vector 𝜇 covariance matrix 𝛴.

The mean-variance portfolio optimization problem is formulated as:

![Budget Contraint](assets/contraint.png)<br>
(Taken from above lecture notes)<br>

In english, this lays out the optmization problem as trying to find the weights (w) that produce the minimum risk (portfolio variance), given a target return (p)<br>
We also set constraints in implemtation: The sum of the weights is equal to 1 (all capital is invested) and each weight ∈ [0,1] (no options or leveraging)<br>

Given this, upon implimentation we can give the algorithm a target return rate and output the portfolio with the lowest risk.<br>
Using this, I derive both the portfolio with the highest Sharpe ratio, and the efficient portfolios frontier for our assets.<br>

In the below table are data for various porfolios across the efficient frontier. Note that securities not listed are weighted at 0.<br>
| Portfolio             | ETF  | Weight (%) | Return | Volatility |
|-----------------------|------|------------|--------|------------|
| Highest Sharpe Ratio  | SPY  | 32.15      | 0.06   | 0.06       |
|                       | IEF  | 66.26      |        |            |
|                       | GLD  | 1.59       |        |            |
| Portfolio 50          | IEF  | 22.69      | -0.03  | 0.28       |
|                       | USO  | 77.31      |        |            |
| Portfolio 150         | IEF  | 69.00      | 0.00   | 0.11       |
|                       | USO  | 31.00      |        |            |
| Portfolio 250         | SPY  | 7.95       | 0.04   | 0.05       |
|                       | IEF  | 66.52      |        |            |
|                       | HYG  | 21.45      |        |            |
|                       | DBC  | 4.08       |        |            |
| Portfolio 350         | SPY  | 46.54      | 0.07   | 0.08       |
|                       | IEF  | 50.94      |        |            |
|                       | GLD  | 2.52       |        |            |
| Portfolio 450         | SPY  | 81.41      | 0.11   | 0.15       |
|                       | IEF  | 13.58      |        |            |
|                       | GLD  | 5.01       |        |            |
| Portfolio 500         | SPY  | 100.00     | 0.13   | 0.18       |

My take away from this is that the mean variance algorithm is more flexible for investors than risk parity (above).<br>
We see that higher return portfolios are weighted in favor of riskier assets, with the most extreme example being all capital invested in SPY, which had the highest mean returns of all securities at hand.<br>
So, even though the max Sharpe ratio may suggest the investor to be less risky, there are alternative suggestions for someone looking for more return.  

The efficient portfolio frontier shows the entire spectrum of efficient portfolios, it follows that with higher returns come higher risk<br>
Finally, I draw a line at the minimum risk, as all portfolios below it would not be invested in by a rational person (they can gain higher returns for the same risk).<br>

![Efficient Portfolios Frontier](assets/mv_frontier.png)

## Genetic Algorithms
Reference for genetic algorithms: [View Mean-Variance CAPM PDF](chrome-extension://bdfcnmeidppjeaggnmidamkiddifkdib/viewer.html?file=https://www.graham-kendall.com/papers/sgk2005.pdf)

Genetic algorithms, generally, are meant to replicate evolutionary biology and natural seleciton

In our case, we'll impliment them to optimize a portfolio of securites

### Algorithm Description
First, we initialize a population of n portfolios with random weights<br>
Over m generations we create new populations based off prior portfolios with the highest level of fit<br>
In this case, fitness is determined by maximizing the Sharpe ratio, as used in the mean-variance optimization<br>
Portfolios in a given population with higher levels of fit will be assigned higher probabilities of 'reproduction':<br>

![Fitness Probabilty](assets/prob_fitness.png)

where:

𝑃𝑖 is the selection probability of individual 𝑖<br>
𝑓𝑖 is the fitness of individual 𝑖<br>
𝑁 is the total number of individuals in the population.<br>

Portfolios selected by their probability will then proceed into crossover<br>
In this we simulate reproduction by combining the weights of two parent portfolios around some random pivot index<br>
Each 'couple' will produce two 'children' consisting of their weights arrays before and after the pivot index.<br>

We also introduce mutation with a mutation rate of 0.01<br>
Individuals will be selected for mutation at random based on this rate, and have one of their weights altered randomly. 

So, with this we initialize `population_size`=200 and `num_generations`=10000, and run the algorithm<br>
We expect to see the fitness increase of the course of generations, thus generating a better portfolio.<br>

#### Optimal Portfolio Weights

| ETF  | Weight (%) |
|------|------------|
| SPY  | 54.28      |
| VGK  | 0.00       |
| IEF  | 30.37      |
| LQD  | 3.05       |
| USO  | 0.02       |
| GLD  | 2.80       |
| VNQ  | 0.53       |
| RWO  | 0.05       |
| IWM  | 0.35       |
| HYG  | 8.53       |
| DBC  | 0.00       |

#### Final Portfolio Performance

| Metric        | Value  |
|---------------|--------|
| Return        | 0.0854 |
| Volatility    | 0.1001 |

Below is the weights in a bar chart:<br>

![Genetic Weights](assets/genetic_weights.png)

We also see that that fitness trends upwards over generations, indicating our algorithm is working correctly. 

![Genetic Evolution](assets/genetic_evolution.png)

An interesting note on this algorithm is that each run outputs a different set of weights.<br>
Because of the random nature, and the fact that we aren't quite acheiving the max Sharpe ratio.<br>
This leads to much variability in returns and risk from run to run.<br>
Which makes me wonder, how would finance professionals would actually impliment the algorithm, and how they would finally select a porfolio?<br>

## Future Work

### Expanding the Dataset

The current dataset spans from 2010-01-01 to 2023-01-01 and includes a diverse set of asset classes. Future work could involve:

1. **Incorporating More Securities**: Including additional ETFs or individual stocks could provide a more granular view of the market.
2. **Extending the Time Frame**: Analyzing a longer historical period could offer more insights into the performance of various optimization methods over different market cycles.
3. **Alternative Data Sources**: Using alternative data, such as sentiment analysis from social media or macroeconomic indicators, could enhance the predictive power of the models.

### Real-World Constraints

To make the portfolio optimization more applicable to real-world scenarios, future work should consider:

1. **Transaction Costs**: Including transaction costs and other fees in the optimization process to better simulate real trading environments.
2. **Regulatory Requirements**: Adhering to regulatory constraints, such as diversification rules and leverage limits, which might affect institutional investors.

### Application to Different Markets

Applying the methods to different markets and asset classes can provide a broader perspective:

1. **Global Markets**: Expanding the analysis to include global markets and different asset classes such as emerging markets, commodities, and currencies.
2. **Sector-Specific Analysis**: Performing sector-specific optimizations to understand how these methods work in different industry sectors.
3. **Alternative Investments**: Including alternative investments such as private equity, hedge funds, and real estate in the portfolio optimization process.
