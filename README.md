# Fitting regression models to costs in R

A typical issue with cost data is they are usually positively skewed (long positive tail), non-negative (if health care services are free!) and heteroscedastic. 
![cost data](https://github.com/1Genevieve/Cost_modelling/blob/master/cost.png)

As such, when modelling cost data (for instance, we want to predict the effect of treatment intervention vs. a comparator on cost), we want to fit a model that will give us the population mean (average effect of treatment on cost) while taking account of the skewness. When data are normally distributed (bell-shaped), we can use a linear regression model, which uses the ordinary least squares (OLS) method. In a linear model, the parameter beta tells us the average change in y for every unit change in x (the relationship is linear). Because data is not normally distributed, the beta estimator of the linear model misrepresents the relationship between x and y (beta says relationship is linear when it is actually not). It misrepresents because it uses the OLS method for fitting a line and OLS is assumed to fulfill the Gauss-Markov conditions. However, the skewed data means that the Gauss-Markov conditions are violated, so we cannot use the linear model for getting beta.

## What are the Gauss-Markov conditions?
When G-M conditions are upheld, OLS is the best linear unbiased (and efficient) estimator (BLUE).
1. The population process can be represented by a linear additive relationship between x and y.
2. The sample is a random sample of the population and all units of the sample come from the same population process.
3. Zero conditional mean of errors.
4. No perfect collinearity in the sample (and population) regressors.
5. Errors are homoskedastic.
6. No serial correlation (covariance between errors is 0).

## Fitting a standard linear regression
Let's try fitting a a linear regression model to the cost data shown above. We get a result like this in R:

![linear model](https://github.com/1Genevieve/Cost_modelling/blob/master/LM.JPG)

The intercept is the average change in cost that is an increase of £406.534 after controlling for age, sex and level of health as measured by SF6D. For every unit increase in age, the cost goes up by £5.035. For every unit increase in health (SF6DM0), the cost decreases by £551.301.

### Diagnostic plots
Doing the diagnostics helps us to determine whether we need to correct the model and how. It involves plotting the residuals and calculating the Akaike Information Criterion (AIC). Let's first look at the plot of residuals.

>plot( fitted(model.lr),resid(model.lr), ylab="residual", xlab="fitted value")
>abline(h = 0, lty = 2, col = "blue")
>qqnorm(residuals(model.lr))
>plot(model.lr)

![residual plots](https://github.com/1Genevieve/Cost_modelling/blob/master/Residual%20plots.jpg)

What we're looking for here is whether the residuals show a non-linear pattern. In the Residuals vs. Fitted plot, there is an upward deflection by about 550 (x-axis) due to the higher values of residuals. In the Q-Q plot, the observed values start to deviate from the fitted line at 1 (x-axis) and farther at 2. To me, this is an indication that the residuals are not normally distributed, violating G-M condition 5.

![scale](https://github.com/1Genevieve/Cost_modelling/blob/master/scale.jpg)

The Scale Location plot checks whether errors are homoskedastic, that is, they are spread evenly along the fitted line. In this case, they gradually spread farther away along the fitted line thus, there is a download deflection at about 450 (x-axis). The Residuals vs. Leverage plot does not show a case outside Cook's line (dashed line) meaning there are not influential extreme values.

The AIC is calculated using the formula: -2(log likelihood) + 2(number of fitted parameters) or using the R code AIC(model). In this case, it is 2386.301. The smaller the AIC, the better the model. If the difference between two models (i.e. LM and GLM) is 10 or more, then we have stronger evidence supporting our choice of model with the smaller AIC. We'll compare this with the AIC of the GLM.

### Plot the prediction, and compare to the observed costs
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

### Plot the prediction, and compare to the observed costs
Let's now compare the predicted and observed costs.

>par(mfrow=c(1,2))
>hist(pred.glm, col = "blue", breaks = 15,xlab=("predicted costs (GBP)"), main="GLM")
>hist(AccupunctureExample$Costs24, col = "blue", breaks = 15, xlab=("costs (GBP)"), main="observed costs")

![GLM compare](https://github.com/1Genevieve/Cost_modelling/blob/master/GLM_compare%20plots.jpg)


### References:
Young, T. (2019) Statistical Modelling of Cost Data. (Lecture) The University of Sheffield, 2019.

Barber, J. and Thompson, S. (2004) Multiple regression of cost data: use of genearlised linear models. Journal of Health Services Research & Policy, 9 (4), 197-204.

Understanding Diagnostic Plots for Linear Regression Analysis (Available at: https://data.library.virginia.edu/diagnostic-plots/)
