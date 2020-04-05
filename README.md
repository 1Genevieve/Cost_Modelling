# Fitting regression models to costs in R

A typical issue with cost data is they are usually positively skewed and non-negative (if health care services are free!). 
![cost data](https://github.com/1Genevieve/Cost_modelling/blob/master/cost.png)

As such, when modelling cost data (for instance, we want to predict the effect of treatment intervention vs. a comparator on cost), we want to fit a model that will give us the population mean (average effect of treatment on cost) while taking account of the skewness. When data are normally distributed (bell-shaped), we can use a linear regression model, which uses the ordinary least squares (OLS) method. In a linear model, the parameter beta tells us the average change in y for every unit change in x. Because data is not normally distributed as shown in the figure above, the beta estimator of the linear model misrepresents the relationship between x and y.

## Fitting a standard linear regression

When we fit a linear regression model to the cost data shown above, we get a result like this:

![linear model](https://github.com/1Genevieve/Cost_modelling/blob/master/LM.JPG)


# Plot the prediction, and compare to the observed costs
par(mfrow=c(1,2))
hist(pred.lr, col = "blue", breaks = 15,xlab=("predicted costs (GBP)"), main="linear regression")
hist(AccupunctureExample$Costs24, col = "blue", breaks = 15, xlab=("costs (GBP)"), main="observed costs")

# Standard model diagnostic plots
plot( fitted(model.lr),resid(model.lr), ylab="residual", xlab="fitted value")
abline(h = 0, lty = 2, col = "blue")
qqnorm(residuals(model.lr))
plot(model.lr)

#############
# Fit a GLM #
#############

#Why add 1 to each cost value?
AccupunctureExample$CostPlus1 <- AccupunctureExample$Costs24+1
View(AccupunctureExample$CostPlus1)

model.glm <- glm(CostPlus1~age+sex.new+SF6DM0,data=AccupunctureExample,family=Gamma(link="identity"))
summary(model.glm)
pred.glm <- predict(model.glm)
mean(pred.glm)
median(pred.glm)
sd(pred.glm)
range(pred.glm)
AIC(model.glm)

# Plot the prediction, and compare to the observed costs
par(mfrow=c(1,2))
hist(pred.glm, col = "blue", breaks = 15,xlab=("predicted costs (GBP)"), main="GLM")
hist(AccupunctureExample$Costs24, col = "blue", breaks = 15, xlab=("costs (GBP)"), main="observed costs")

# Standard model diagnostic plots
plot( fitted(model.glm),resid(model.glm), ylab="residual", xlab="fitted value")
abline(h = 0, lty = 2, col = "blue")
qqnorm(residuals(model.glm))


#############################
# Fitting a two part model  #
#############################

# Part1 Logistic regression
m.full.glm2p <- glm(zerocost~age+sex.new + SF6DM0,data=AccupunctureExample, family = binomial(link = "logit"))
summary(m.full.glm2p)

anova(m.full.glm2p, test="Chi")
glm.pred <- predict(m.full.glm2p, type="response") # Gives the probabilities
summary(glm.pred)

# Part2 Linear regression
m.costs.lrred <- lm(Costs24~age+sex.new + SF6DM0, data = AccupunctureExample, subset=Costs24 >= 1)
summary(m.costs.lrred)
AIC(m.costs.lrred)
AIC(m.full.glm2p)

lm.pred <-  predict(m.costs.lrred, AccupunctureExample) # Obtains predictions for the full acupuncture dataset
tpm.pred <- lm.pred*glm.pred
summary(lm.pred)
summary(tpm.pred)
sd(tpm.pred)


# Plot the prediction, and compare to the observed costs
par(mfrow=c(1,2))
hist(tpm.pred, col = "blue", breaks = 15,xlab=("predicted costs (GBP)"), main="two-part model")
hist(AccupunctureExample$Costs24, col = "blue", breaks = 15, xlab=("costs (GBP)"), main="observed costs")

