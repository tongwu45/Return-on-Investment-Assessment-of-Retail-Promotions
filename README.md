# Return-on-Investment-Assessment-of-Retail-Promotions


# Preparation and Set Up
Before we start, let us do some preparation work to get ready!

We need to:

  1. Create a folder and set working directory to that folder on your device,
  
  2. Download datasets needed for this tutorial onto your working directory, and
  
  3. Load datasets for analysis later as we proceed

The two datasets of this tutorial are made availabe to you. The files are Snickers_data, and softwaresalesdata, and are stored in an xlsx format.

To save these two datsets and for convenience of anlaysis for this tutorial, let us create a folder in your computer for and name the folder scanpro. And then let us download the two files to the folder (scanpro) that you just created.

In order to load the datasets to R, we need to run only part of the following chunk of codes, depending on the operating system of your device. To avoid running the codes that are not applicable and receiving error messages, please add “#” before each line of non-applicable codes in the chunk below. For example, if you are using a Mac, you need to put “#” before code setwd(“H:/downloads/scanpro”).

```r
# Please run the following codes depending on the operating system on your laptop/PC. 

# if you are using iOS, and your *scanpro* folder is created under "Downloads" on your Mac: you will need to first set your working directory to *scanpro* folder:

setwd("~/Downloads/scanpro")

# if you are using Windows, and your *scanpro* folder is created in your H drive: you will need to first set your working directory to *scanpro* folder:

#setwd("H:/downloads/scanpro")

# load the datasets, and name them "snicker" and "software", respectively.
snicker <- read_xlsx("Snickers_data.xlsx")
software <-read_xlsx("softwaresalesdata.xlsx")
```
NOW WE ARE READY TO GO!




# 1. The SCAN*PRO Model: A Basic Specification
Suppose you want to predict weekly sales of Snickers (the world’s best-selling candy bar) at your local supermarket. Factors that might influence sales of Snickers include:

  1. Price charged for Snickers bars
  
  2. Prices charged for competitors (e.g., Three Musketeers, Hershey’s Chocolate,and so on)
  
  3. Was the Snickers bar on display (i.e., on shelf)?
  
  4. Was there a national ad campaign for Snickers?
  
  5. Was there an ad for Snickers in the local Sunday paper?

Untangling how these factors affect unit sales of Snickers is difficult.

A model was developed by Dirk Wittink et al., called 𝑆𝐶𝐴𝑁∗𝑃𝑅𝑂 model to isolate the effect of these (and other factors) on sales of retail goods. The 𝑆𝐶𝐴𝑁∗𝑃𝑅𝑂 model and its variants have been widely used by A.C. Nielsen and other organizations to analyze retail sales.

## 1.1 Data
The 𝑆𝐶𝐴𝑁∗𝑃𝑅𝑂 model is often used to model the impact of various portions of the marketing mix. To predict sales you can model the effect of each part of the marketing mix as described above and create a final prediction for sales. Instead of simple additions, you do this by multiplying together the terms for each part of the marketing mix and throwing in a constant to correctly scale your final prediction.

To create this model you can use the snicker dataset that you loaded. The dataset shows:

  1. The weekly sales of Snickers bars at a local supermarket as well as the Snickers price (our price),
  
  2. The price of the major competitor (comp price), and
  
  3. Whether Snickers is on display (Display).



Let us take a brief look at the dataset:


```r
kable(head(snicker), caption = "Table 1 Snickers Raw Data")
```
Table 1 Snickers Raw Data
<img width="310" height="142" alt="截屏2026-02-03 下午3 46 12" src="https://github.com/user-attachments/assets/0b12a45c-43e0-4f45-affd-fe84b749b5b6" />


Reading the data, for example, in Week 1, 986 Snickers bars were sold; the average price of Snickers during Week 1 was $1.04; the average price of the main competitors was $0.81; and Snickers was on display (1 = Snickers on display, 0 = Snickers not on display). To simplify matters, let’s assume that Snickers sales do not exhibit seasonality.



## 1.2 Model Specification and Construction


To create a model to predict weekly sales, complete the following steps:

1. Raise the price to an unknown power. (We call this power OWNELAS, meaning “own elasticity”.) This creates a term of the form (𝑂𝑢𝑟𝑃𝑟𝑖𝑐𝑒)𝑂𝑊𝑁𝐸𝐿𝐴𝑆
. The value of OWNELAS can estimate the price elasticity. You would expect OWNELAS to be negative. For example, if OWNELAS = -3, you can estimate that for any price you charge, a 1 percent price increase reduces demand 3 percent.

2. Raise the competitor’s price to an unknown power (COMPELAS, meaning “competitor elasticity”). This creates a term of the form (𝐶𝑜𝑚𝑝𝑃𝑟𝑖𝑐𝑒)𝐶𝑂𝑀𝑃𝐸𝐿𝐴𝑆
. The value of COMPELAS estimates a cross-elasticity of demand. You would expect COMPELAS to be positive and smaller in magnitude than OWNELAS. For example, if COMPELAS = 0.4, then a 1 percent increase in the competitor’s price increases demand for Snickers (for any set of prices) by 0.4 percent.

3. Model the effect of a display by a term that raises an unknown parameter (Call it Displayeffect) to the power DISPLAY (which is 1 if there is a display and 0 if there is no display). This term is of the form (𝐷𝑖𝑠𝑝𝑙𝑎𝑦𝑒𝑓𝑓𝑒𝑐𝑡)𝐷𝐼𝑆𝑃𝐿𝐴𝑌
. When there is a display, this term equals the value of Displayeffect, and when there is no display, this term equals 1 (i.e., 𝐷𝑖𝑠𝑝𝑙𝑎𝑦𝑒𝑓𝑓𝑒𝑐𝑡0
). Therefore, a value of, say, Displayeffect=1.2 indicates that after adjusting for prices, a display increases weekly sales by 20 percent.



Putting it all together, the final prediction for weekly sales is shown in Equation 1:

<img width="867" height="62" alt="截屏2026-02-03 下午3 47 00" src="https://github.com/user-attachments/assets/00df9cfb-6688-4ffa-b17e-b569726998ee" />


To estimate such a model, we can either apply optimization techniques to find the optimal combination of parameters that result in minimal estimation error (consider using optim function in R).

However, that will require multiple rounds of iterations and we will risk not reaching the global optimum. Alternatively we can transform equation 1.2 by taking the logarithm of it and estimate a linear model. The advantage of this transformation, which is shown below, is that the parameters of the original nonlinear model can be estimated using linear-regression techniques which are quite stable and robust and computatioinallly efficient.

Specifically, we have:

<img width="1142" height="70" alt="截屏2026-02-03 下午3 47 29" src="https://github.com/user-attachments/assets/6f2fc45e-6354-40b8-8628-4f87806a8056" />



After the transformation, in Equation 2, we are having a linear regression model with the log of OurPrice, the log of CompPrice, and DISPLAY as model variables that we directly extract from the data, and OWNELAS, COMPELAS, and Log(Displayeffect) as the model coefficients that need to be estimated.

  1. Equation 2 is one of the most commonly used forms of market mix modelling, i.e., the concave shape, which is characterized by diminishing returns to scale as the marketing activity increases (see the green curve in Figure 1 below). This aligns well with the expectation that as the intensity of discounts, displays and advertising increases, the returns diminish.
  
  2. Remember that another advantage of estimating a model specified like Equation 2 also lies in the fact that the model coefficients can be directly interpreted as elasticities.
  
  3. Note: Given that we are estimating Displayeffect as a model parameter instead of DISPLAY (i.e., the dummy variable indicating the existence of display ad), we treat 𝐿𝑜𝑔(𝐷𝑖𝑠𝑝𝑙𝑎𝑦𝑒𝑓𝑓𝑒𝑐𝑡) as the coefficient to be estimated in our linear model and then calculate the exponential of it to get the actual scale of the coefficient.


<img width="858" height="450" alt="截屏2026-02-03 下午3 48 30" src="https://github.com/user-attachments/assets/71373d64-6717-43b6-9e9e-7f94e6e26294" />


Figure 1: Marketing Mix Modelling

## 1.3 Model Estimation

```r
# take logarithm of each variable, excpet "Display", which is a dummy variable.
# We do not have zeros or missing values in the three variables below, otherwise we need to add a small value, e.g., 1, to the variables when taking logs. 

snicker$log_sales <- log(snicker$Sales) 
snicker$log_our_price <- log(snicker$`Our price`)
snicker$log_comp_price <-log(snicker$`Comp price`)

# estimate linear model
scanpro <- lm(log_sales~log_our_price+log_comp_price+Display, data = snicker)
summary(scanpro)
```
```text
## 
## Call:
## lm(formula = log_sales ~ log_our_price + log_comp_price + Display, 
##     data = snicker)
## 
## Residuals:
##      Min       1Q   Median       3Q      Max 
## -0.18080 -0.05987 -0.01226  0.06823  0.22396 
## 
## Coefficients:
##                Estimate Std. Error t value Pr(>|t|)    
## (Intercept)     6.77144    0.02381 284.408  < 2e-16 ***
## log_our_price  -3.19451    0.13288 -24.041  < 2e-16 ***
## log_comp_price  0.29890    0.11813   2.530   0.0157 *  
## Display         0.20635    0.03107   6.641 7.56e-08 ***
## ---
## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
## 
## Residual standard error: 0.09866 on 38 degrees of freedom
## Multiple R-squared:  0.9466, Adjusted R-squared:  0.9424 
## F-statistic: 224.7 on 3 and 38 DF,  p-value: < 2.2e-16
```

## 1.4 Results Interpretation


Given theh model output, we should not forget the transform the coefficient for Display back by taking the exponental: $e^{0.20635} = 1.23 $

Now we can say that based on the model estimation:

  1. The display effect = 1.23. This implied that after adjusting for prices, a Snickers display increases weekly sales by 23 percent.
  
  2. The price elasticity for Snickers is -3.19, so a 1 percent increase in the price of Snickers can reduce Snicker’s sales by 3.19 percent.
  
  3. The cross-price elasticity for Snickers is 0.21, so a 1 percent increase in the competitor’s price can raise the demand for Snickers by 0.21 percent.
  


Substituting your parameter values into Equation 1, you can find your prediction for weekly sales is given by:

<img width="546" height="50" alt="截屏2026-02-03 下午3 49 44" src="https://github.com/user-attachments/assets/2d88a53f-a494-4ee2-8bc8-8d0609c645c8" />



Question: Why is the constant term 872.57 instead of 6.77 in Equation 3?



Now we have finished out task of estimating a 𝑆𝐶𝐴𝑁∗𝑃𝑅𝑂 model using Snicker data. Now we can explore another specification of 𝑆𝐶𝐴𝑁∗𝑃𝑅𝑂 model using another dataset.



# 2. Forecasting Software Sales: A Variant of SCAN*PRO Model (Optional)


## 2.1 Data


An intelligent modification of the 𝑆𝐶𝐴𝑁∗𝑃𝑅𝑂 approach introduced above can enable you to build more insightful and accurate sales forecasting models that include factors other than price and product displays.

To illustrate the idea, we use the dataset software that we have already loaded to our environment.

Suppose you are given the following information for your software sales:

  1. Quarterly software sales (in million) depend on quarterly PC shipments (in million).
  
  2. Sales are seasonal.
  
  3. Sales increase after a launch of software and drop off prior to the launch.

How can you build a model to forecast sales, given these information?

```r
kable(software[1:15,])
```
```text
Quarter	PC shipments	Sales	Quarter of year	Launch
1	4.4	0.3193439	1	NA
2	7.0	0.6016478	2	NA
3	5.5	0.6859068	3	NA
4	6.6	0.7791580	4	NA
5	4.2	0.3141329	1	NA
6	5.8	0.5167288	2	NA
7	6.0	0.6489136	3	NA
8	4.1	0.5082976	4	NA
9	4.8	0.3379526	1	NA
10	5.9	0.3734321	2	NA
11	4.8	0.7000000	3	yes
12	4.0	0.5828551	4	NA
13	7.5	0.5355897	1	NA
14	6.8	0.5606161	2	NA
15	6.2	0.6778529	3	NA
```

The data you’ve got is sales from a software firm for 48 quarters. For example, 700,000 units were sold in Quarter 11 (Sales= 0.7 million). During this quarter there was a software launch (launch = yes), the quarter was the third quarter of the year (Quarter of year= 3), and 4.8 million PCs were shipped (PC shipmemnts = 4.7).

Now let us take a look at the dataset:

```r
# The "Launch" variable is a string variable; we need to convert it to numeric first and then change the "NA"'s to "0", "yes" to 1, so that Launch is an dummy variable (also called indicator variable).

software$Launch<-as.factor(software$Launch)
software$Launch<-as.numeric(software$Launch)
software[is.na(software)] <- 0

# We drop the 49th row for analysis because it has a missing value of Sales. There are several ways to fill in the missing value (e.g., using the average sales in the data), yet here we decide to simplify our world by dropping it.
software <- software[1:48,] 

# Take a look at the first 10 rows of data
kable(head(software,11), caption = "Table 2 Software Sales")
```
Table 2 Software Sales
<img width="384" height="251" alt="截屏2026-02-03 下午3 51 25" src="https://github.com/user-attachments/assets/d33dc44b-6ec9-410b-b870-792ff85aa298" />




## 2.2 Model Specification
The equation to forecast sales involves the following parameters:

  1. A seasonal index for each quarter. These seasonal indexes should average to 1. A seasonal index of, say, 1.1 for Quarter 2 means that after adjusting for other factors, sales in the second quarter is 10 percent more than an average quarter
  
  2. Factors (similar to the effect of the display in the Snickers example) that measure the bump up in sales the quarter of a launch (LAUNCH1) and the quarter after the launch (LAUNCH2)
  
  3. A factor (LAUNCH-1) that measures the decline in sales the quarter before the launch
  
  4. Because it seems reasonable to assume that an increase in PC shipments will increase sales of your product, you can include a term of the form 𝐵𝑎𝑠𝑒∗𝑃𝐶𝑆𝑎𝑙𝑒𝑠
   in your forecast. Here Base is a changing cell that scales your forecast to minimize MAPE.It is analogous to the constant term in Equation 1.1.
  


Putting it all together, your model for predicting quarterly sales (in millions) is given by Equation 4.

<img width="681" height="47" alt="截屏2026-02-03 下午3 52 08" src="https://github.com/user-attachments/assets/95176edb-d574-48c3-b804-90fce6460609" />


When building your sales forecasting model, you can assume that in lieu of minimizing the sum of squared errors (SSE) you can minimize the mean absolute percentage error (MAPE).

Minimizing SSE emphasizes on avoiding large outliers, and minimizing MAPE emphasizes on minimizing the magnitude of the typical prediction error.

Also, many practitioners prefer MAPE over SSE as a measure of forecast accuracy. This is probably because MAPE is measured as a percentage of the dependent variable, whereas it is difficult to interpret the units of SSE (in this case 𝑢𝑛𝑖𝑡𝑠2
). It is clear that we cannot use linear regression to estimate the model specified in Equation 2 anymore. We will need to use the optim function in R to solve this optimization problem to find the best parameters that gives the minimal MAPE.

As a start we give the parameters an initial values to complete the function.

  1. Base: 1, which in effect scales PC Sales into sales of our software
```r
base <- 1
```

  2. Seasonal index: let’s for now set 0.5, 1, 1, and 1.5 for season 1, 2, 3, and 4, respectively. (The average of these index is also 1). A simple rationale could be that we assume the sales should be the lowest during the first quarter of each year and should peak at the last quarter. Of course, you can set other starting values as you want.
```r
seasonal.factors<- data.frame(cbind(c(1,2,3,4), c(0.5,1,1,1.5)))
colnames(seasonal.factors) <- c("Quarter of year", "initial factor")
kable(seasonal.factors, caption = "Table 3 seasonal factor")
```
<img width="251" height="138" alt="截屏2026-02-03 下午3 53 04" src="https://github.com/user-attachments/assets/47bb35a9-3160-4b33-b42f-ecb92b371243" />

Table 3 seasonal factor



  3. Launch factors:
    LAUNCH-1 which represents the drop in sales in the quarter before a launch
    
    LAUNCH1 which represents the increase in sales during a launch quarter
    
    LAUNCH2 which represents the increases in sales the quarter after a launch

We set the initial values of these launch factors as 2, 1, 0.8, and 0.2 respectively. A rationale that justifies our choices would be that sales should be the highest during a launch, then one quarter after the launch, then one quarter prior to the launch, and finally the lowest during other quarters without a recent launch.


```r
launch.factors <- data.frame(cbind(c(1, 2, -1, 0), c(2,1,0.8,0.2)))
colnames(launch.factors) <- c("launchcode", "initial factor")
kable(launch.factors, caption = "Table 4 Launch Factors")
```
Table 4 Launch Factors
<img width="222" height="136" alt="截屏2026-02-03 下午3 54 10" src="https://github.com/user-attachments/assets/c9333b29-48c0-4e3b-9af9-4245d3569c23" />


To reflect such variation, we will need to create a new variable named “launch code” based on the values in the Launch column of the original dataset.


```r
# We are adding a new column of code named "launchcode" in the data file, based on the values in the column "Launch".

software <- mutate(software, launchcode = 
                     ifelse(Launch == "1", 1, 
                            ifelse(lag(Launch) == "1", 2, 
                                   ifelse(lead(Launch) == "1", -1, 0))))
software[is.na(software)] <- 0
launchcode <- software[6]

# Take a look at the data file now with a new variable "launchcode" added

kable(head(software, 12), caption = "Table 5 Software Sales with Launch Code")
```
Table 5 Software Sales with Launch Code
<img width="574" height="336" alt="截屏2026-02-03 下午3 54 39" src="https://github.com/user-attachments/assets/1a900ff4-e953-49ca-af96-1943b7add51a" />


From the above table we can see that because there was a launch in quarter 11,based on the rule that we described above, the launchcode in quarter 10 (i.e., one quarter before the launch) is -1, the launchcode of quarter 11 (i.e., the quarter of launch) is 1, and the launchcode of quarter 12 (i.e., one quarter after the launch) is 2.

Other information from dataset that’s needed for computation includes: quarter of the year, product shipments used for prediction and sales for computing the error.



## 2.3 Model Construction


Now you have all the parameters and variables ready for computing 𝑆𝐶𝐴𝑁∗𝑃𝑅𝑂
 forecast, you can write a function based on formula (2).

We are writing a function in the following chunk of code. The function is called “scanpro.forecast”. The purpose of this function is to obtain the sales predictions based on given values of the parameters, and calculate the model performance metric (in this case, the MAPE).

It is just like any other built-in functions in some packages that we already know of, e.g., the “lm” function in the “lme4” package that we used for forecasting in the previous session. The only difference is that, instead of installing packages and using the function, we are writing the function ourselves!



The code of the function is displayed below. The function has 5 inputs: x, quarterofyear, launchcode, pcshipment, and sales

  1. x is the vector of parameters that you are trying to optimize. x should contain 9 elements each being: seasonal_factor (s_1, s_2, s_3, s_4, each standing for one quarter), launch_factor (l_1, l_2, l_-1, l_0, as defined earlier), and base. The seasonal factors, launch factors and base are extracted from the parameter vector based on their position. Seasonal effect and launch effect are then derived based on Quarter of the year and Launch code using if condition.
  
  2. quarterofyear, launchcode, pcshipment, sales are data input directly from the dataset–recall that we have these columns in our data file “software”. Here, “sales” is the actual sales, or sales in the dataset. We will be able to calculate predicted sales based on the estimated parameters in x and Equation 2. Then we can compare the actual sales with the predicted sales, and finally calculate the model performance metric, MAPE.

The output of the function is the MAPE.


```r
# start defining the function
scanpro.forecast <- function(x, quarterofyear, launchcode, pcshipment, sales){
 
  # seasonaleffect is determined by "quarterofyear": e.g., if quarterofyear is 1, then it means we are dealing with the 1st quarter, hence seasonaleffect should be the 1st element of the parameter vector x (i.e., x[1]) in our prediction calculation.
  seasonaleffect <- ifelse(quarterofyear ==1, x[1], ifelse(
    quarterofyear ==2, x[2], ifelse(quarterofyear ==3, x[3], x[4])))
  
  # launcheffect is also determined by the specific value of the launchcode in a quarteR, e.g., if launchcode is 1, then we use the 5th element from the parameter vector x (i.e., x[5]) in our prediction calculation.
  launcheffect <- ifelse(launchcode == 1, x[5], ifelse(
    launchcode == 2, x[6], ifelse(launchcode == -1, x[7], x[8])))
  
  base <- x[9] # the 9th element of parameter vector x is the Base

  # Sales prediction calculation based on Equation 2:
  predict <- base * pcshipment * seasonaleffect * launcheffect
  
  # calculating the MAPE and save the value in "fit"
  fit<- mean(abs((sales - predict)/sales))
  
  # display the value of fit, i.e., the MAPE of this round of estimation.
  return (fit)
}
```

## 2.4 Model Estimation


Now that the function is ready, yet we cannot only run it once and stop. Instead, we need to repeat our estimations several times with different compositions of the parameter vector x, until we get a minimum MAPE.

That is why we need to fit this function into the optim function to solve for the optimal parameters that gives the lower errors in forecast. We save the optimization results in an item named “result”.

We are mostly interested in two components in “result”:

  1. “par”, which refers to the estimated parameters, and
  2. “value”, which is the MAPE of the ouput model.

```r
set.seed(666)
result <- optim(c(0.5,1,1,1.5,2,1,0.8,0.2,1), fn = scanpro.forecast, 
      quarterofyear = software$`Quarter of year`, 
      launchcode = software$launchcode, pcshipment = software$`PC shipments`, 
      sales = software$Sales, 
      method = "L-BFGS-B")

result[1:2]
```
```text
## $par
## [1] 0.6970292 0.8289199 1.1275037 1.2320062 1.3554287 1.1929444 0.8283103
## [8] 1.0628158 0.0950542
## 
## $value
## [1] 0.0476572
```


A solution is reached which gives a MAPE of 0.047. The seasonal factors are (0.697, 0.829, 1.128, 1.232) and the launch effect factors are (1.355, 1.193, 0.928, 1.063) with base equals to 0.095.

Put these parameters back to Equation 2, we have:

<img width="988" height="56" alt="截屏2026-02-03 下午3 56 04" src="https://github.com/user-attachments/assets/2a0beef7-c216-453d-b5c4-9348456aaa9b" />


where 𝑠1,𝑠2, 𝑠3, 𝑠4,𝑙1, 𝑙2, 𝑙−1, and 𝑙0 are all dummy variables, so that if we are at Quarter 3 (i.e., quarterofyear = 3), and right in the quarter of product launch (i.e., launchcode = 1), then we should have 𝑠3=1 and 𝑙1=1, while all others being zero. Our equation above actually becomes:

<img width="926" height="53" alt="截屏2026-02-03 下午3 56 58" src="https://github.com/user-attachments/assets/b3ab4298-1fc6-4186-9c9c-e6f5f4366f6b" />

which, if we clean things up:

<img width="453" height="50" alt="截屏2026-02-03 下午3 57 14" src="https://github.com/user-attachments/assets/37c7f2cf-7500-49b1-9f28-9d30fa72a8a8" />


Question: How do you interpret the results in Equation 5?

We know that optimizations are highly dependent on the initial values we pick for the parameters. To further improve the model performance, we can consider trying different sets of starting values and re-estimate the model. For example, a common practice is to use the parameters from the previous round of estimation as the initial values of the current round of estimation. Let’s do this manually.

For example, instead of having the initial values as x=(0.5,1,1,1.5,2,1,0.8,0.2,1), we try x=(0.3,1,1,1.7,2,1,0.8,0.2,3) instead:

```r
set.seed(666)
result <- optim(c(0.3,1,1,1.7,2,1,0.8,0.2,3), fn = scanpro.forecast, 
      quarterofyear = software$`Quarter of year`, 
      launchcode = software$launchcode, pcshipment = software$`PC shipments`, 
      sales = software$Sales, 
      method = "L-BFGS-B")

result[1:2]
```
```text
## $par
## [1] 0.50512168 0.05161839 0.07034928 0.89801671 1.19044990 0.09278086 0.06715953
## [8] 0.08064583 1.73847487
## 
## $value
## [1] 0.4079018
```
As you can see above, we have obtained a different set of estimated parameters, with the MAPE being 0.408, significantly higher than 0.048 above. Hence the estimation above should be better and we should use the above one to interpret our model results. Of course, you can experiment different combinations of the starting values yourself.



## 2.5 Result Interpretation


Recall that we have Equation 5 as our final outcome:

<img width="945" height="46" alt="截屏2026-02-03 下午3 58 00" src="https://github.com/user-attachments/assets/e23eea58-93e4-48ce-938e-f6d05f03e8cf" />

from which we can yield the following insights.

  1. Quarter 4 has the highest sales with sales 23.2 percent better than average, and Quarter 1 has the lowest sales (28.3 percent below average).
  
  2. All other things being equal (ceteris paribus) the quarter of a launch, sales increase by 35.5 percent.
  
  3. The quarter after a launch, sales increase (ceteris paribus) by 19.3 percent.
  
  4. The quarter before a launch, sales decrease (ceteris paribus) by 17.2 percent.
  
  5. During an average quarter, an increase in PC shipments of 100 PCs leads to 9.5 more units sold.



## 2.6 Predicting Future Sales
Assuming that now you are requested to give a forecast of sales in Quarter 50 and you are given the following information about Quarter 50:

  1. 6 million PC shipments will be made in Quarter 50
  
  2. There will be no launch planned in quarters 48–51.



To predict, you need to first adjust the scanpro formula to return the forecast sales:

```r
# Now we call this function scanpro.forecast.forecast. Instead of returning the model MAPE, we want the function to return predicted sales this time:

scanpro.forecast.forecast <- function(x, quarterofyear, launchcode, pcshipment){
  base <- x[9]
  seasonaleffect <- ifelse(quarterofyear ==1, x[1], ifelse(
    quarterofyear ==2, x[2], ifelse(quarterofyear ==3, x[3], x[4])))
  
  launcheffect <- ifelse(launchcode == 1, x[5], ifelse(
    launchcode == 2, x[6], ifelse(launchcode == -1, x[7], x[8])))
  predict <- base * pcshipment * seasonaleffect * launcheffect
  
  # Everything the same up till here; the only difference lies in the return() below. Here we did not bother to calculate the "fit", or the MAPE like we used to, but rather we stop at "predict".
  return (predict)
}
```

Next you can call scanpro.forecast.forecast function to find out the sales on quarter 50. You will need to increase all input data by one value accordingly except for parameters and sales.

  1. quarterofyear: add quarter “2” （why? because quarter 50 is the second quarter of a year, hence 2)
  
  2. launchcode: add “0” (why? because there’s no launch planned)
  
  3. pcshipment: add “6” (why? because there’s 6 million PC shipments in quarter 50)



Now we run this “scanpro.forecast.forecast” function, by using the best parameter combination that we obtained above.

```r
scanpro.forecast.forecast(c(0.6970292, 0.8289199, 1.1275037, 1.2320062, 1.3554287, 1.1929444, 0.8283103, 1.0628158, 0.0950542), 
                          quarterofyear = c(software$`Quarter of year`, 2), 
                          launchcode = c(software$launchcode, 0), 
                          pcshipment =c(software$`PC shipments`, 6))[49]
```
```text
## [1] 0.5024503
```
Your forecast for Quarter 50 sales is around 0.502 million or 502,000 units.



To summarize, in this project I have learned to assess effectiveness of retail promotions in driving sales through estimating a 𝑆𝐶𝐴𝑁∗𝑃𝑅𝑂 model. Specifically, we have estiamted a basic 𝑆𝐶𝐴𝑁∗𝑃𝑅𝑂 model and its variation to estimate and forecast sales while incorporating different types of promotions. 



