# Predicting-Crime-Rates

#### **Introduction**

In this analysis, we'll be performing linear regression using crime data from http://www.statsci.org/data/general/uscrime.txt. This is a commonly used dataset that in criminology. It contains information on various socio-economic and demographic factors and what the crime rates are for each state. The regression model will be used to predict the observed crime rate in a city with the following data:

M = 14.0
So = 0
Ed = 10.0
Po1 = 12.0
Po2 = 15.5
LF = 0.640
M.F = 94.0
Pop = 150
NW = 1.1
U1 = 0.120
U2 = 3.6
Wealth = 3200
Ineq = 20.1
Prob = 0.04
Time = 39.0

Finally, we will test the accuracy of the regression model by applying k-fold cross validation. 

**Columns**

|  |                                                                                       |
|----------|--------------------------------------------------------------------------------------------------|
| M        | Percentage of males aged 14–24 in total state population                                         |
| So       | Indicator variable for a southern state                                                         |
| Ed       | Mean years of schooling of the population aged 25 years or over                                 |
| Po1      | Per capita expenditure on police protection in 1960                                             |
| Po2      | Per capita expenditure on police protection in 1959                                             |
| LF       | Labour force participation rate of civilian urban males in the age-group 14–24                  |
| M.F      | Number of males per 100 females                                                                 |
| Pop      | State population in 1960 in hundred thousands                                                   |
| NW       | Percentage of nonwhites in the population                                                       |
| U1       | Unemployment rate of urban males 14–24                                                          |
| U2       | Unemployment rate of urban males 35–39                                                          |
| Wealth   | Median value of transferable assets or family income                                    |
| Ineq     | Income inequality: percentage of families earning below half the median income                  |
| Prob     | Probability of imprisonment: ratio of number of commitments to number of offenses               |
| Time     | Average time in months served by offenders in state prisons before their first release          |
| Crime    | Crime rate: number of offenses per 100,000 population in 1960                                   |







#### **Setting Up the Linear Regression Model**

In the following model, we use the `lm()` function to set up a linear regression model that is fitted to our `crime_data` data set. The first parameter `Crime~.` designates the "Crime" column as the response variable, and the other columns are its predictors. 

```{r}
crime_data <- read.table("C:/Users/leegu/OneDrive/Desktop/Projects/LinReg_CV/uscrime.txt", header = TRUE, stringsAsFactors = F)

# Linear regression model
lm_crime_data <- lm(Crime~., data = crime_data)

# Summarize Model
summary(lm_crime_data)
```
The summary of the `lm_crime_data` tells us some important information about our data and its regression model. Some of the key takeaways are the coefficients for each predictor and its p-values. A positive coefficient of a predictor means that as the predictor increases, the crime rate will also increase. The `Pr(>|t|)` column tells us the p-value for each predictor, and if the p-value is below 0.05 the model will mark the predictor's influence as statistically significant. The p-values that are significant are marked by one or more asterisks. 

According to our model, the `M` (% males aged 14–24), `Ed` (years of schooling for ages 25 years or over), `Ineq` (income inequality), `Prob` (probability of imprisonment) columns are significant. Po1 (per capita expenditure on police protection) and U2 (unemployment rate ages 35-39) are borderline significant, and they are marked by a single dot.

From all this, we can deduce several things about crime rates. The `M` column suggests that as the percentage of young males increases, the crime rates will also increase. The same could be said for the `Ineq` predictor; the more income inequality there is in a population, the higher crime rates tend to be. This makes sense because time and again, we see that crime is linked to poverty. 

The `Prob` predictor is also statistically significant, but it has a negative coefficient. This means that a higher probability of imprisonment will often lead to decrease in crime. This also seems reasonable because people will be much less willing to commit crimes if they knew they would be caught. 

The only statistically significant predictor that doesn't seem to make sense is the `Ed` column. This predictor has a significant positive coefficient, meaning that higher education rate will lead to more crimes. Higher education usually leads to lower crime rates by decreasing poverty. Our model suggests that the opposite is true, seemingly contradicting what we observed about the `Ineq` predictor. 

The `Ed` predictor could be a result of overfitting of the data. Overfitting occurs when there are too many predictors/data points and the model is responding to random fluctuations and noise, resulting in recognizing inaccurate relationships in the data. Inaccuracies due to overfitting is definitely something to keep in mind as we make our prediction on crime rate.  

#### **Predicting the Observed Crime Rate**

Now that the linear regression has been set, we can use our model to make a prediction about the crime rate for this particular city. We'll designate `test_point` as the data frame that contains the set of predictors for our city. We use the `predict()` function to make our prediction. 

```{r}
test_point <- data.frame(M = 14.0, So = 0, Ed = 10.0, Po1 = 12.0, Po2 = 15.5, LF = 0.640, M.F = 94.0, Pop = 150, NW = 1.1, U1 = 0.120, U2 = 3.6, Wealth = 3200, Ineq = 20.1, Prob = 0.04, Time = 39.0)

pred_model <- predict(lm_crime_data, test_point)
pred_model 

```
The value that was predicted by our model is **155.4348**. This means that the estimated crime rate for 100,000 individuals in the city comes out to **155.4348**. 

Upon reviewing the dataset, it was found that this is a very unlikely estimate. The lowest crime rate in the "Crime" column of our dataset is 373, and the highest crime rate gets as high as 1993. Our predicted crime rate does not reasonably fit into the expected range of the dataset. As we suspected earlier, the model's estimate of 155.4348 is a result of overfitting. Since the problem with our dataset is that it has too many predictors, we'll run the same model again, but only include predictors that we deemed to be statistically significant. The predictors we'll include are as follows: `M`, `Ed`, `Ineq`, `Prob`, `Po1`, and `U2`. 

#### **Prediction With Corrected Data**

```{r}
lm_crime_data2 <- lm(Crime~ M + Ed + Po1 + U2 + Ineq + Prob, data = crime_data)
summary(lm_crime_data2)

test_point2 <- data.frame(M = 14.0, Ed = 10.0, Po1 = 12.0, U2 = 3.6, Ineq = 20.1, Prob = 0.04)

pred_model2 <- predict(lm_crime_data2, test_point2)
pred_model2 
```
We can see that a summary of our new linear regression model that specifically targets only the statistically significant predictors have p-values that are all lower than 0.05. This will significantly reduce overfitting that results from noise and insignificant data. 

Our new and improved prediction of crime rates for our city comes out to be **1304.245**. For every 100,000 individuals in this city, the estimated crime rate according to our linear regression model is **1304.245**. Seeing how this reasonably fits within the dataset's 373 to 1993 range, we can conclude that this is a much more reliable estimate of crime rate. 

#### **Cross Validation**

We can use the k-fold cross validation method to further assess the accuracy of our linear regression model. We'll be using `cv.lm()` function, where `crime_data` is the dataset and `lm_crime_data` is our original linear regression model, while `lm_crime_data2` is the improved linear regression model. Cross validation will be performed twice for each linear regression model to see if we can observe improvements between the two. We designate 4 folds for this cross validation. 

**Model 1 Cross Validation (Before Removing Overfitted Data)**

```{r}
library(DAAG)
lm_crime_data_cv <- cv.lm(crime_data, lm_crime_data, m=4)
```

**Model 2 Cross Validation (After Removing Overfitted Data)**
```{r}
lm_crime_data_cv2 <- cv.lm(crime_data, lm_crime_data2, m=4)
```

When we compare the two linear regression models, the main point to highlight is the difference in the mean square values. The first model has an overall mean square value of 94179.93, indicating that this model has high prediction errors. The second model has an overall mean square value of 48203.02, which is a much lower error rate. The error rate was basically cut in half, so we can be assured that our estimated crime rate of 1304.245 predicted by the second model is a reliable and reasonable prediction. 

## Works Cited 

D’Allegro, J. (2021b, October 19). Just what factors into the value of your used car?. Investopedia. https://www.investopedia.com/articles/investing/090314/just-what-factors-value-your-used-car.asp 
