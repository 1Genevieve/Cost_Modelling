# Cost_modelling
# Fitting regression models to costs in R

## Cost data are typically skewed.
![skewed cost data](/Desktop/cost.png)

#Inspect data
table(AccupunctureExample$treatment)
table(AccupunctureExample$sex)
summary(AccupunctureExample$age)
hist(AccupunctureExample$age)
summary(AccupunctureExample$Costs24)
hist(AccupunctureExample$Costs24)
summary(AccupunctureExample$SF6DM0)
hist(AccupunctureExample$SF6DM0)
 
#Missing values
sum(is.na(AccupunctureExample$id))
sum(is.na(AccupunctureExample$sex))
sum(is.na(AccupunctureExample$treatment))
sum(is.na(AccupunctureExample$Costs24))
sum(is.na(AccupunctureExample$SF6DM0))


# Create factors for categorical variables
as.factor(AccupunctureExample$sex)
AccupunctureExample$sex.new<-as.factor(AccupunctureExample$sex)
AccupunctureExample$treatment.new<-as.factor(AccupunctureExample$treatment)

hist(AccupunctureExample$Costs24, col = "blue", breaks = 15, xlab=("costs (GBP)"), main="Accupuncture Example")
table(AccupunctureExample$treatment)
mean(AccupunctureExample$Costs24)
median(AccupunctureExample$Costs24)
sd(AccupunctureExample$Costs24)
range(AccupunctureExample$Costs24)

AccupunctureExample$zerocost <- ifelse(AccupunctureExample$Costs24 == 0, 0, 1)
table(AccupunctureExample$zerocost)

####################################
# Fit a standard linear regression #
####################################


model.lr <- lm(AccupunctureExample$Costs24~AccupunctureExample$age+AccupunctureExample$treatment.new+AccupunctureExample$sex.new + AccupunctureExample$SF6DM0)
model.lr <- lm(AccupunctureExample$Costs24~AccupunctureExample$age+AccupunctureExample$sex.new + AccupunctureExample$SF6DM0)
summary(model.lr)
pred.lr <- predict(model.lr)
mean(pred.lr)
median(pred.lr)
sd(pred.lr)
range(pred.lr)
AIC(model.lr)

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

