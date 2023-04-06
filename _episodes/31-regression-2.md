---
title: "Regression Example"
teaching: 45
exercises: 30
questions:
- "How a Person's Alcohol Consumption Effects Their Blood Pressure?"
objectives:
- "Learn how to use linear regression to produce a model from data."
- "Learn how to model non-linear data using a logarithmic."
- "Learn how to measure the error between the original data and a linear model."
keypoints:
- "We can model linear data using a linear or least squares regression."
- "A linear regression model can be used to predict future values."
- "We should split up our training dataset and use part of it to test the model."
- "For non-linear data we can use logarithms to make the data linear."
---


<!-- 
<!-- ```{r setup, include=FALSE}
# load packages here
library(tidyverse)
library(ggfortify)
library(car)
``` -->
 -->
### Background and Introduction

Our data comes from the 2017-2018 National Health and Nutrition Examination 
Survey (NHANES), a recurring survey in which the CDC collects information from 
individuals on diet/nutrition and health status. NHANES data is collected using 
a stratified clustered, four-stage, random design to gain a nationally 
representative sample. According to the American Heart Association, alcohol 
consumption can affect your blood pressure, though this effect is quickly 
reversed if alcohol consumption is reduced. We are interested in how daily 
alcohol consumption affects systolic blood pressure reading from that same day. 

### Methods and Results

NHANES data is collected through an in-depth interview with individuals. 
Participants are selected using a stratified clustered, four-stage, random 
design to gain a nationally representative sample. Results are separated into 
smaller datasets by topic to make using the data more manageable. 
We merged two data sets from NHANES, one containing observations of a 
respondent’s alcohol consumption in grams on a specific day and the other data 
set containing observations from a systolic blood pressure reading in mmHg from 
those same respondents on that same day. After merging the data, we removed rows 
of data that had a 0 value for alcohol consumption, as there is no way to 
examine how blood pressure affects an individual’s alcohol consumption if that 
individual does not consume alcohol. 
From our exploratory data analyses, we noticed that we had a very 0-heavy 
predictor. Many respondents had not consumed any alcohol. We determined that 
these observations would not be appropriate to include in our model, as there is 
no way to examine how alcohol consumption affects an individual's systolic blood 
pressure if that individual does not consume alcohol. We removed the 
observations of individuals who did not consume any alcohol and proceeded with 
our analysis. 
We fitted a linear model to the raw data and found that several assumptions were 
violated. Our residuals were not normally distributed and centered at zero and 
the model likely didn’t describe all observations. We then performed a 1/Y 
transformation because the output from our BoxCox test was closest to -1. We 
checked the assumptions with our transformed data, found that all assumptions 
were met, and fit a new linear model to the transformed data.

The following table displays the variable names in this data set, along with 
their descriptions.

Variable   | Description
---------- | -------------
Alcohol    | Amount of alcohol consumed (grams)
Blood      | Systolic blood pressure (mmHg) 

We start by applying basic summary and exploratory statistics to this data to 
better understand the data and identify trends.

```{r, fig.align='center'}
nhanes <- read.csv("nhanes_sample.csv", header = TRUE) %>% 
  select(Blood, Alcohol) 
summary(nhanes)
nhanes_base_plot <- ggplot(data = nhanes, 
                           mapping = aes(x = Alcohol, y = Blood)) +
  geom_point() +
  theme_bw() +
  scale_x_continuous(limits = c(0, 450),
                     breaks = seq(0, 450, by = 50)) +
  scale_y_continuous(limits = c(80, 230)) +
  xlab("Alcohol Consumed (grams)") +
  ylab("Blood Pressure (mmHg)") +
  theme(aspect.ratio = 1)
nhanes_base_plot
# <...code to read in data and perform EDA...>
```

Here is the general linear model we want to fit:

$\widehat{Blood}_i = \beta_0 + \beta_1\times\text{Alcohol)}_i$

We now fit an initial model.

```{r, fig.align='center'}
# <...code to fit linear model and print out a summary to make sure the output
#     doesn't look strange...>
nhanes_lm <- lm(Blood ~ Alcohol, data = nhanes)
summary(nhanes_lm)
nhanes$residuals <- nhanes_lm$residuals
nhanes$fittedBlood <- nhanes_lm$fitted.values
```

Before we can interpret our model, we need to check the assumptions of the 
simple linear model to make sure the model fits the data well.

#### X vs Y is linear

```{r, fig.align='center'}
# <...code to check assumptions [you should use ALL diagnostics]...>
nhanes_base_plot
nhanes_resid_vs_fit <- autoplot(nhanes_lm, which = 1, ncol = 1, nrow = 1) +
  theme_bw() +
  theme(aspect.ratio = 1)
nhanes_resid_vs_fit
```

Linearity assumption is likely met. Scatter plot suggests that nothing would fit 
the data better than a line, and residuals vs fitted plot line appears roughly 
horizontal.

#### The residuals are independent

Independent assumption is met; data was collected using a random sample design 
and there is no reason to believe residuals are correlated about time or space.

#### The residuals are normally distributed and centered at zero

```{r, fig.align='center'}
# <...code to check assumptions [you should use ALL diagnostics]...>
# boxplot
nhanes_boxplot <- ggplot(data = nhanes, mapping = aes(y = residuals)) +
  geom_boxplot() +
  theme(aspect.ratio = 1)
nhanes_boxplot
# histogram
nhanes_hist <- ggplot(data = nhanes, mapping = aes(x = residuals)) + 
  geom_histogram(mapping = aes(y = ..density..), binwidth = 10) +
  theme(aspect.ratio = 1) +
  stat_function(fun = dnorm, 
                color = "red", 
                size = 2,
                args = list(mean = mean(nhanes$residuals), 
                            sd = sd(nhanes$residuals))) 
nhanes_hist
# qq plot
nhanes_qq <- autoplot(nhanes_lm, which = 2, ncol = 1, nrow = 1) +
  theme(aspect.ratio = 1)
nhanes_qq
#Shapiro wilk test
shapiro.test(nhanes_lm$residuals)
```

Residuals normally distributed and centered at zero assumption is likely 
violated. From our boxplot, histogram, and Q-Q plot, it appears that the data is 
right-skewed. The brown-forsythe test also returned a very significant p-value.

#### The residuals have equal/constant variance across all values of X

```{r, fig.align='center'}
# <...code to check assumptions [you should use ALL diagnostics]...>
# Residuals vs Fitted
nhanes_resid_vs_fit
# Brown Forsythe Test
grp <- as.factor(c(rep("lower", floor(dim(nhanes)[1] / 2)), 
                   rep("upper", ceiling(dim(nhanes)[1] / 2))))
leveneTest(nhanes[order(nhanes$Alcohol), "residuals"] ~ grp, center = median)
```

Constant Variance assumption is likely met. The levene test shows a p-value much 
larger than 0.05, which indicates the variance is constant. The residual vs. 
fitted plot has far more data points on the left side, but the residuals seem to 
be equally spread.

#### The model describes all observations 

```{r, fig.align='center'}
# Scatterplot
nhanes_base_plot
# Boxplot
nhanes_boxplot
# Q-Q
nhanes_qq
# Histogram
nhanes_hist
#get cooks distance value from all observations
nhanes$cooksd <- cooks.distance(nhanes_lm)
#plot cooks distance against the observation number
ggplot(data = nhanes) +
  geom_point(mapping = aes(x = as.numeric(rownames(nhanes)),
                           y = cooksd)) +
  theme_bw() +
  ylab("Cook's Distance") +
  xlab("Observation Number") +
  geom_hline(mapping = aes(yintercept = 4 / length(cooksd)),
             color = "red", linetype = "dashed") +
  theme(aspect.ratio = 1)
#DFFITS
nhanes$dffits <- dffits(nhanes_lm)
 
nhanes_dffits <- ggplot( data = nhanes) +
  geom_point(mapping = aes(x = as.numeric(rownames(nhanes)),
                       	y = abs(dffits))) +
  theme_bw() +
  ylab("Absolute Value of DFFITS for Acohol") +
  xlab("Observation Number") +
  geom_hline(mapping = aes(yintercept = 2 * sqrt(length(nhanes_lm$coefficients) /
                                  	             length(dffits))),
         	color = "red", linetype = "dashed") +
  theme(aspect.ratio = 1)
nhanes_dffits 
 
nhanes %>%
  mutate(rowNum = row.names(nhanes)) %>%
  filter(abs(dffits) > 2 * sqrt(length(nhanes_lm$coefficients) /
                              	length(dffits))) %>%
  arrange(desc(abs(dffits)))
#DFBETAS
nhanes$dfbetas_alcohol <- as.vector(dfbetas(nhanes_lm)[, 2])
nhanes_dfbetas_plot <- ggplot(data = nhanes) +
  geom_point(mapping = aes(x = as.numeric(rownames(nhanes)),
                           y = abs(dfbetas_alcohol))) +
  theme_bw() +
  geom_hline(mapping = aes(yintercept = 2 / sqrt(length(dfbetas_alcohol))),
             color = "red", linetype = "dashed") +
  xlab("Observation Number") +
  ylab("Absolute Value of DFBETAS for Alcohol") +
  theme(aspect.ratio = 1)
nhanes_dfbetas_plot
nhanes %>%
  mutate(rowNum = row.names(nhanes)) %>%
  filter(abs(dfbetas_alcohol) > 2 /
           sqrt(length(rownames(nhanes)))) %>%
  arrange(desc(abs(dfbetas_alcohol)))
```

All observations assumption is likely violated. DFBETAS, DFFITS, and Cook’s 
Distance plots show several potential influential points. Q-Q plot, Histogram, 
and Boxplot show potential outliers/influential points.

#### Additional predictor variables are not required

Additional Predictor variables would be helpful in this model. Consumption of 
fatty foods or foods that are high in cholesterol could predict blood pressure, 
as well as daily exercise habits or stress level. We will proceed with the 
analysis for this project, but we note the assumption is likely not met.

To summarize, after fitting the linear regression model and checking 
the assumptions, we notice multiple assumptions may not be met. Specifically, 
the residuals don't appear to be normally distributed, and the model likely does 
not describe all observations.

Given the above assumptions not being met, we will apply a Box-Cox 
transformation to find a good transformation for the response. If the model 
still doesn't describe all observations, we'll try the model with and without 
them.

```{r, fig.align='center'}
# <...code for Box-Cox here...>
bc <- boxCox(nhanes$Blood ~ nhanes$Alcohol)
bc$x[which.max(bc$y)]
nhanes$Blood_trans <- 1 / (nhanes$Blood)
nhanes_lm_trans <- lm(Blood_trans ~ Alcohol, data = nhanes)
summary(nhanes_lm_trans)
nhanes$residuals_trans <- nhanes_lm_trans$residuals
nhanes$fittedBlood_trans <- nhanes_lm_trans$fitted.values
nhanes_base_plot_trans <- 
  ggplot(data = nhanes, mapping = aes(x = Alcohol, y = Blood_trans)) +
  geom_point() +
  theme_bw() +
  ylab("Blood (mmHg)") +
  xlab("Alcohol (grams)") +
  theme(aspect.ratio = 1)
nhanes_base_plot_trans + geom_smooth(method = "lm", se = FALSE) 
```

The Box-Cox transform lead us to apply the reciprocal (1 / y) transformation to 
Blood pressure. This appeared to take care of our problem with normally 
distributed residuals. We now look at the model results as applied to the 
transformed data and assess the assumptions again that are relevant.

#### X vs Y is linear

```{r, fig.align='center'}
# <...code to check assumptions [you should use ALL diagnostics]...>
# Scatterplot
nhanes_base_plot_trans
# Resid vs fit
nhanes_resid_vs_fit_trans <- autoplot(nhanes_lm_trans, 
                                      which = 1, ncol = 1, nrow = 1) +
  theme_bw() +
  theme(aspect.ratio = 1)
nhanes_resid_vs_fit_trans
```

Linearity assumption is likely met. Scatter plot suggests that a line is the 
best descriptor of the relationship, and the residuals vs fitted plot line 
appears roughly horizontal. 

#### The residuals are normally distributed and centered at zero

```{r, fig.align='center'}
# <...code to check assumptions [you should use ALL diagnostics]...>
# Boxplot
nhanes_boxplot_trans <- 
  ggplot(data = nhanes, mapping = aes(y = residuals_trans)) +
  geom_boxplot() +
  ylab("residuals") +
  theme(aspect.ratio = 1)
nhanes_boxplot_trans
# Histogram
nhanes_hist_trans <- ggplot(data = nhanes, mapping = aes(x = residuals_trans)) + 
  # when using this code for future data sets, make sure to change the binwidth: 
  geom_histogram(mapping = aes(y = ..density..), binwidth = 0.001) +
  theme(aspect.ratio = 1) +
  xlab("residuals") +
  stat_function(fun = dnorm, 
                color = "red", 
                size = 2,
                args = list(mean = mean(nhanes$residuals_trans), 
                            sd = sd(nhanes$residuals_trans))) 
nhanes_hist_trans
# QQ Plot
nhanes_qq_trans <- autoplot(nhanes_lm_trans, which = 2, ncol = 1, nrow = 1) +
   theme_bw() +
   theme(aspect.ratio = 1)
nhanes_qq_trans
# Shapiro wilk test
shapiro.test(nhanes$residuals_trans)
```

Residuals are normally distributed and centered at zero assumption is met. The 
boxplot and histogram appear normal and centered at zero, and the Q-Q plot 
follows the horizontal line well. The Shapiro-Wilk test returned an 
insignificant p-value.

#### The residuals have equal/constant variance across all values of X

```{r, fig.align='center'}
# <...code to check assumptions [you should use ALL diagnostics]...>
# Residuals vs fitted
nhanes_resid_vs_fit_trans
# Brown-Forsythe
grp <- as.factor(c(rep("lower", floor(dim(nhanes)[1] / 2)), 
                   rep("upper", ceiling(dim(nhanes)[1] / 2))))
leveneTest(nhanes[order(nhanes$Alcohol), "residuals_trans"] ~ grp, center = median)
```

Constant Variance assumption is likely met. The Brown-Forsythe test shows a 
p-value much larger than 0.05, which indicates the variance is constant. The 
residual vs. fitted plot looks a little more spread on the right side, but the 
difference is likely negligible.

#### The model describes all observations 

```{r, fig.align='center'}
# Scatterplot
nhanes_base_plot_trans
# Boxplot
nhanes_boxplot_trans
# Q-Q
nhanes_qq_trans
# Histogram
nhanes_hist_trans
# get cooks distance value from all observations
nhanes$cooksd_trans <- cooks.distance(nhanes_lm_trans)
# plot cooks distance against the observation number
ggplot(data = nhanes) +
  geom_point(mapping = aes(x = as.numeric(rownames(nhanes)),
                           y = cooksd_trans)) +
  theme_bw() +
  ylab("Cook's Distance") +
  xlab("Observation Number") +
  geom_hline(mapping = aes(yintercept = 4 / length(cooksd_trans)),
             color = "red", linetype = "dashed") +
  theme(aspect.ratio = 1)
# DFFITS
nhanes$dffits_trans <- dffits(nhanes_lm_trans)
 
nhanes_dffits_trans <- ggplot( data = nhanes) +
  geom_point(mapping = aes(x = as.numeric(rownames(nhanes)),
                       	y = abs(dffits_trans))) +
  theme_bw() +
  ylab("Absolute Value of DFFITS for Alcohol") +
  xlab("Observation Number") +
  geom_hline(mapping = aes(yintercept = 2 * 
                             sqrt(length(nhanes_lm_trans$coefficients) /
                                  	             length(dffits_trans))),
         	color = "red", linetype = "dashed") +
  theme(aspect.ratio = 1)
nhanes_dffits_trans
 
nhanes %>%
  mutate(rowNum = row.names(nhanes)) %>%
  filter(abs(dffits_trans) > 2 * sqrt(length(nhanes_lm_trans$coefficients) /
                              	length(dffits_trans))) %>%
  arrange(desc(abs(dffits_trans)))
# DFBETAS
nhanes$dfbetas_alcohol_trans <- as.vector(dfbetas(nhanes_lm_trans)[, 2])
nhanes_dfbetas_plot_trans <- ggplot(data = nhanes) +
  geom_point(mapping = aes(x = as.numeric(rownames(nhanes)),
                           y = abs(dfbetas_alcohol_trans))) +
  theme_bw() +
  geom_hline(mapping = aes(yintercept = 2 / sqrt(length(dfbetas_alcohol_trans))),
             color = "red", linetype = "dashed") +
  xlab("Observation Number") +
  ylab("Absolute Value of DFBETAS for Alcohol") +
  theme(aspect.ratio = 1)
nhanes_dfbetas_plot_trans
nhanes %>%
  mutate(rowNum = row.names(nhanes)) %>%
  filter(abs(dfbetas_alcohol_trans) > 2 /
           sqrt(length(rownames(nhanes)))) %>%
  arrange(desc(abs(dfbetas_alcohol_trans)))
```


The model describes all observations assumption is possibly met. The Histogram, 
Boxplot, and Q-Q plot no longer show significant outliers or potential 
influential points. The DFBETAS calculated numbers seem to cancel each other out 
to where the slope would not make a large difference. DFFITS values also seem to 
cancel each other out to where including all observations would not make a 
significant difference compared to excluding them. Cooks distance shows the same 
potential problem points as DFFITS and DFBETAS. To know for sure, we'll need to
make models with and without the top 5 influential points.


To summarize, all assumptions appear to be met except for "the model describes 
all observations." There are 5 main potential influential points. To assess the 
impact of these points, we will create a new data set with these points removed. 
We then apply the transformations to Alcohol and Blood and fit another linear 
model.

```{r, fig.align='center'}
nhanes_copy <- read.csv("nhanes_sample copy.csv", header = TRUE)
summary(nhanes_copy)
nhanes_copy$Blood_trans <- 1 / (nhanes_copy$Blood)
nhanes_copy_lm_trans <- lm(Blood_trans ~ Alcohol, data = nhanes_copy)
summary(nhanes_copy_lm_trans)
nhanes_copy$residuals_trans <- nhanes_copy_lm_trans$residuals
nhanes_copy$fittedBlood_trans <- nhanes_copy_lm_trans$fitted.values
nhanes_copy_base_plot_trans <- ggplot(data = nhanes_copy, 
                                      mapping = aes(x = Alcohol, y = Blood_trans)) +
  geom_point() +
  theme_bw() +
  ylab("Blood (mmHg)") +
  xlab("Alcohol (grams)") +
  theme(aspect.ratio = 1)
nhanes_base_plot_trans + geom_smooth(method = "lm", se = FALSE)
nhanes_copy_base_plot_trans + geom_smooth(method = "lm", se = FALSE) 
```

We noticed that the influential points almost completely cancel each other out 
as we predicted, so we decided to use the first model going forward.

Now that we have a model with all assumptions met, we would like to use the 
model to make inferences and predictions. Here is our fitted linear model:

$$\widehat{\frac{1}{Blood_i}} = 0.008195 - 0.000001568\times\text{Alcohol}_i$$

To start, we will assess the model slopes, confidence intervals, and hypothesis 
tests.

```{r, fig.align='center'}
# <...code to print out a summary of the model and create confidence
#     intervals...>
summary(nhanes_lm_trans)
confint(nhanes_lm_trans, level = 0.95, parm = "Alcohol")
```
The range of -3.432521e-06 to 2.975724e-07 indicates that there is not a 
significant relationship between alcohol consumption and blood pressure due to 0 
being within the range. Like-wise, a non-significant p-value of 0.0994 indicates 
that alcohol is not a significant source of variability in predicting systolic 
blood pressure. 

We now want to get predictions for new subjects. We are particularly interested 
in the predicted average blood pressure for a subject that consumes 30 grams of 
alcohol. We will use this information to create confidence and prediction 
intervals for average blood pressure.  

```{r, fig.align='center'}
# <...code for confidence and prediction intervals...>
predict(nhanes_lm_trans, newdata = data.frame(Alcohol = 30), 
        interval = "confidence", level = 0.95)
predict(nhanes_lm_trans, newdata = data.frame(Alcohol = 30), 
        interval = "prediction", level = 0.95)
```

We are 95% confident that the average systolic blood pressure is between 0.00806 
and 0.00823 mmHg when one's alcohol consumption is 30 grams. 

We are 95% confident that a new observation's systolic blood pressure will fall 
between 0.006009727 and 0.010286 mmHg when one's alcohol consumption is 30 grams

We also plotted the confidence and prediction intervals across all values of 
alcohol. 

```{r, fig.align='center'}
# <...code for plot with confidence and prediction intervals...>
Alcohol_values <- seq(min(nhanes$Alcohol), max(nhanes$Alcohol), length = 1000)
conf_int_mean <- predict(nhanes_lm_trans, 
                         newdata = data.frame(Alcohol = Alcohol_values), 
                 interval = "confidence", level = 0.95)
preds <- data.frame("Alcohol_values" = Alcohol_values, conf_int_mean)
# linear model with Blood log transformed
# Sequence of Alcohol values that we are interested in using to predict Blood  
Alcohol_values <- seq(min(nhanes$Alcohol), max(nhanes$Alcohol), length = 1000)
# 95% confidence intervals of **log(Blood)** across those values of Alcohol
conf_int_mean_trans <- predict(nhanes_lm_trans, 
                               newdata = data.frame(Alcohol = Alcohol_values), 
                               interval = "confidence",
                               level = 0.95)
# Predictions of **Blood** (back-transformed) across those values of Alcohol
conf_int_mean_preds <- 1 / (conf_int_mean_trans)  # use exp to "undo" log trans
# Store results in a data frame for plotting
preds <- data.frame("Alcohol_values" = Alcohol_values, conf_int_mean_preds)
# Plot the predictions
nhanes_base_plot + 
  geom_line(data = preds, mapping = aes(x = Alcohol_values, y = fit), 
            color = "blue", size = 1.5) + 
  geom_line(data = preds, mapping = aes(x = Alcohol_values, y = lwr), 
            color = "#d95f02", size = 1.5) +
  geom_line(data = preds, mapping = aes(x = Alcohol_values, y = upr), 
            color = "#d95f02", size = 1.5)
# Prediction Interval 
pred_int_mean_trans <- predict(nhanes_lm_trans, 
                               newdata = data.frame(Alcohol = Alcohol_values), 
                               interval = "prediction",
                               level = 0.95)
# Predictions of **Blood** (back-transformed) across those values of Alcohol
pred_int_mean_preds <- 1 / (pred_int_mean_trans)  # use exp to "undo" log trans
# Store results in a data frame for plotting
prediction_preds <- data.frame("Alcohol_values" = Alcohol_values, 
                               pred_int_mean_preds)
# Plot the predictions
nhanes_base_plot + 
  geom_line(data = prediction_preds, mapping = aes(x = Alcohol_values, y = fit), 
            color = "blue", size = 1.5) + 
  geom_line(data = prediction_preds, mapping = aes(x = Alcohol_values, y = lwr), 
            color = "#d95f02", size = 1.5) +
  geom_line(data = prediction_preds, mapping = aes(x = Alcohol_values, y = upr), 
            color = "#d95f02", size = 1.5)
```

Next, we are interested in how well the model fits the data. To do this, we look 
at metrics such as $R^2$, the RMSE, MSE, and MAE. 

```{r, fig.align='center'}
# MSE
anova <- aov(nhanes_lm_trans)  # get ANOVA components
nhanes_anova <- summary(anova)[[1]]  # save data in a usable form
nhanes_anova
mse <- nhanes_anova["Residuals", "Sum Sq"] / nhanes_anova["Residuals", "Df"]
mse
# RMSE
rsme <- sqrt(mse)
rsme
# R-Squared
summary(nhanes_lm_trans)$r.squared
# MAE
mae <- sum(abs(nhanes$Blood - nhanes$fittedBlood)) / nhanes_lm$df.residual
mae
```

It is estimated that 1.183722e-06 is the average squared distance between the 
estimated values and what is estimated. A MSE of this size is optimal as it has 
a very small distance between the estimated values and what is estimated. 
0.00108799 is the average error performed by the model in predicting the 
outcome. In other words, 0.00108799 is the amount of spread in the residuals, 
which is very small. We want the RMSE to be close to 0 because less error is 
better. 
The proportion of total variation in runoff explained by precipitation is 
0.004134457. This is a less optimal proportion because an R^2 value closer to 
one is better. This means that most of the variation is not explained by alcohol 
consumption.  

### Summary and Conclusions
