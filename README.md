# Fitting regression models to costs in R

This is my interpretation of the R exercises on cost modelling as part of a lecture on Statistical Modelling of Cost Data in The University of Sheffield.

A typical issue with cost data is they are usually positively skewed (long positive tail) and non-negative and multimodal. 
![cost data](https://github.com/1Genevieve/Cost_modelling/blob/master/cost.png)

As such, when modelling cost data (for instance, we want to predict the effect of treatment intervention vs. a comparator on cost adjusting for characteristics as age, sex and health status), we want to fit a model that will give us the population mean (average effect of treatment on cost) while taking account of the skewness. 

When data are normally distributed (bell-shaped), we can use a linear regression model, which uses the ordinary least squares (OLS) method. In a linear model, the parameter beta tells us the average change in y for every unit change in x (the relationship is linear). Given a large number of samples, that average change will fall in the center of the bell-shaped curve, which represents the population mean. If data is not normally distributed, the beta estimator of the linear model will misrepresent the relationship between x and y. 

Also, as a rule, when using the OLS method for fitting a line, data is assumed to fulfill the Gauss-Markov conditions. However, the skewed data means that G-M conditions are violated. Thus, we cannot use the linear model for getting beta.

### What are the Gauss-Markov conditions?
When G-M conditions are upheld, OLS is the best linear unbiased (and efficient) estimator (remember BLUE?!).
1. The population process can be represented by a linear additive relationship between x and y.
2. The sample is a random sample of the population and all units of the sample come from the same population process.
3. Zero conditional mean of errors (the expectation of the error term given the predictor variables must be 0/covariance is 0).
4. No perfect collinearity in the sample (and population) regressors.
5. Errors are homoskedastic.
6. No serial correlation (covariance between errors is 0).

## Fitting a standard linear regression
Let's try fitting a a linear regression model to the cost data shown above. We get a result like this in R:
>model.lr <- lm(AccupunctureExample$Costs24~AccupunctureExample$age+AccupunctureExample$sex.new + AccupunctureExample$SF6DM0)

>summary(model.lr)

![linear model](https://github.com/1Genevieve/Cost_modelling/blob/master/LM.JPG)

The second line of the output is the formula of the linear regression model: the dependent variable is cost and the regressors are age, sex and health. We are concerned with the average effect of acupuncture treatment on cost adjusting for age, sex and health (SF6D) in the population. Since our data comes from a sample of the population, we are fitting a normal distribution to our data in order to derive the average effect on the population and that average will fall in the middle of the bell-shaped curve (according to Central Limit Theorem!).

The third line, residuals assess G-M condition 5. A residual is the difference between the observed value of the response variable (cost) and the value predicted by the model. For all positive and negative observed values, we want the difference to be symmetrical for all values of predictor X. As you would expect from our skewed data (compare min vs max, 1st vs 3rd quartile), the residuals do not appear symmetrical. The predicted value is nearer some observed values than others.

The fourth line is the coefficients -- intercept and slope (beta). The intercept is the average cost of treatment for all individuals in our sample, that is, £406.534 after adjusting for age, sex and level of health. The beta says that for every unit increase in age, the cost goes up by £5.035. For every unit increase in health (SF6DM0), the cost decreases by £551.301.

The Std. Error tells how much response variable vary from the coefficients. The lower the Std. Errors, the better the fit of our model. For instance, for every unit increase in health, cost decreases by £551 but can vary by £496.

The t-values measure the number of standard deviations the coefficients away are from 0. The larger the t-value, the more the regressor  variable (i.e. age, sex, health) indicates a potential influence on the outcome variable. As you can see, they are also very small compared to the Std. Errors.

Pr(>t) means the probability of observing the coefficient (intercept and slope) at or above t. For instance, at t=0.928, the probability of observing the relationship between age and cost by chance is 0.355. If we set alpha at 0.05, 0.355 is a high probability thus, such relationship is not significant and is due to chance. The same can be said for the other coefficients. Thus, the effect of acupuncture treatment on cost after adjusting for age, sex and health is not significant compared to a comparator treatment.

Let's go to the last 3 lines of the model output: residual standard error, multiple R-squared and F-statistic. The RSE tells how good the fit of our linear model is for all actual observed values of regressor variables and response variable. However, this fit is not perfect; there is a residual error. The RSE tells how much the response variable goes off. As mentioned earlier, the intercept is the average cost of treatment for all individuals in our sample, that is, £406.534 after adjusting for age, sex and level of health. The RSE is 675.6 meaning the cost could go off by 166%! The Degress of Freedom tells how many data points we used -- 146 out of 150. The 4 data points substracted are the intercept, age, sex and treatment. Logically, these 4 data points would not be accounted for in the estimation of RSE.  

Multiple R-squared is the proportion of variance and measures how much of our response variable (cost) is explained by the predictor variable. The average cost in the population can vary due to many influences and in this case, 1.7% is explained by the predictor variables. This means a lot of variable influence cost that we have not accounted for.


### Diagnostic plots
Diagnostics helps us make a decision whether we need to correct the model and how. I mean, we've seen the output and formed our judgement whether our model is good, but is it bad enough to consider an alternative model? Diagnostics involves plotting the residuals and calculating the Akaike Information Criterion (AIC). Let's first look at the plot of residuals.

>plot( fitted(model.lr),resid(model.lr), ylab="residual", xlab="fitted value")
>abline(h = 0, lty = 2, col = "blue")
>qqnorm(residuals(model.lr))
>plot(model.lr)

![residual plots](https://github.com/1Genevieve/Cost_modelling/blob/master/Residual%20plots.jpg)

What we're looking for here is whether the residuals show a non-linear pattern. In the Residuals vs. Fitted plot, there is an upward deflection by about 550 (x-axis) due to the higher values of residuals. In the Q-Q plot, the observed values start to deviate from the fitted line at 1 (x-axis) and farther at 2. To me, this is an indication that the residuals are not normally distributed, violating G-M condition 5.

![scale](https://github.com/1Genevieve/Cost_modelling/blob/master/scale.jpg)

The Scale Location plot checks whether errors are homoskedastic, that is, they are spread evenly along the fitted line. In this case, they gradually spread farther away along the fitted line thus, there is a download deflection at about 450 (x-axis). The Residuals vs. Leverage plot does not show a case outside Cook's line (dashed line) meaning there are not influential extreme values.

The AIC is calculated using the formula: -2(log likelihood) + 2(number of fitted parameters) or using the R code AIC(model). In this case, it is 2386.301. The smaller the AIC, the better the model. If the difference between two models (i.e. LM and GLM) is 10 or more, then we have stronger evidence supporting our choice of model with the smaller AIC. We'll compare this with the AIC of the GLM.

### Predicted vs. observed costs
Now, let's plot the predicted cost when using the linear model and compare that to the observed cost.

>par(mfrow=c(1,2))
>hist(pred.lr, col = "blue", breaks = 15,xlab=("predicted costs (GBP)"), main="linear regression")
>hist(AccupunctureExample$Costs24, col = "blue", breaks = 15, xlab=("costs (GBP)"), main="observed costs")

![compare costs](https://github.com/1Genevieve/Cost_modelling/blob/master/compare.JPG)


## Fitting a Generalised Linear Model (GLM)
GLM's allow for estimating average effect of skewed cost data because it is flexible -- they allow for different non-normal distributions (e.g gaussian, poisson, gamma), and additive or multiplicative effect of covariates through an identity link function or log link function, respectively. Because cost data are skewed, we would fit a skewed (e.g. gamma, poisson) rather than a normal (gaussian) distribution function. We will also fit a log link function, which seems more realistic for the data we have.

>model.glm <- glm(CostPlus1~age+sex.new+SF6DM0,data=AccupunctureExample,family=Gamma(link="identity"))

![GLM](https://github.com/1Genevieve/Cost_modelling/blob/master/GLM.JPG)

Here, the average change in cost is an increase of £479. For every unit increase in age, cost increases by £1.74, and for every increase in health, cost decresease by £439.

### Diagnostic plots
Let's now do the diagnostics for the GLM and compare with those of the linear regression model.

>plot( fitted(model.glm),resid(model.glm), ylab="residual", xlab="fitted value")
>abline(h = 0, lty = 2, col = "blue")
>qqnorm(residuals(model.glm))

![GLM diagnostics](https://github.com/1Genevieve/Cost_modelling/blob/master/GLM_diagnostics.jpg)

The AIC is 1955.901. Compare this with the AIC of the linear regression model that is 2386.301. The GLM AIC is smaller and the difference is more than 10, indicating that GLM is the better fitting model.

### Predicted vs. observed costs
Let's now compare the predicted and observed costs.

>par(mfrow=c(1,2))
>hist(pred.glm, col = "blue", breaks = 15,xlab=("predicted costs (GBP)"), main="GLM")
>hist(AccupunctureExample$Costs24, col = "blue", breaks = 15, xlab=("costs (GBP)"), main="observed costs")

![GLM compare](https://github.com/1Genevieve/Cost_modelling/blob/master/GLM_compare%20plots.jpg)


### Bibliography
1. Young, T. (2019) Statistical Modelling of Cost Data. (Lecture) The University of Sheffield, 2019.

2. Barber, J. and Thompson, S. (2004) Multiple regression of cost data: use of genearlised linear models. Journal of Health Services Research & Policy, 9 (4), 197-204.

3. Bommae Kim (2015) Understanding Diagnostic Plots for Linear Regression Analysis (Available at: https://data.library.virginia.edu/diagnostic-plots/)

4. Felipe Rego (2015). Quick Guide: Interpreting Simple Linear Model Output in R: (Available at: https://feliperego.github.io/blog/2015/10/23/Interpreting-Model-Output-In-R)
