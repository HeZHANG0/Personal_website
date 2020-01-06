---
title: "Predicting housing price using advanced regression"
date: 2019-05-05
published: true
tags: [regression, housing price, elastic net, random forest, boosting, predictive modeling]
excerpt: "Predicting housing price using advanced regression"
toc: true
toc_sticky: true
---

Authors: He Zhang, Bingying Feng, Dingzhe Leng

# Summary

Residential homes are hedonic pricing goods, whose prices are determined both by internal characteristics of the good being sold and external factors affecting it. Suppose we control the external factors, what internal features are strongly related to housing price? How good can we predict the housing price using the home features? These are the three questions that drive this project. The answers will be useful for 

1.Homebuyers: For them to choose their dream house given a budget.
2. Real estate developers  =>  choose the product to deliver 
3. Real estate agents  =>  provide valuable insights and identify the needs of customers 



## Data

The original data is retrieved from a Kaggle open competition. (https://www.kaggle.com/c/house-prices-advanced-regression-techniques/data) It covers data of Housing data that describes features of residential homes in Ames, Iowa sold between 2006 and 2010. There are 80 explanatory variables (23 nominal, 23 ordinal, 14 discrete, and 20 continuous)  describing the homes in different aspects such as space, amenities, conditions, zoning, etc (almost every aspect of residential homes).


## Method
In this project, we will build models to explain what features are related to home prices as well as models to predict home prices in Ames, Iowa.We will first conduct data cleaning, EDA and feature analysis. Then we will build Elastic net model to investigate the features that are related to home prices using methods. For the predictive models, we will incorporate more variable engineering to enhance model performance. Modern machine learning models such as Elastic net, Random Forest, Gradient Boosting and Neural Net Models will be employed in predictive modeling.


# Data Analysis
```{r, include=FALSE}
train<- read.csv("train.csv",sep=",")
test<- read.csv("test.csv",sep=",")
test<- test%>% mutate(SalePrice= "NA")
data<- rbind(train, test)
```

## EDA (Part I before data processing)
### Overlook
The response variable is `SalePrice`. There are 80 explanatory variables.
```{r, include=FALSE}
#names(data) The first column is id. Not used in analysis, but will be used for submitting results. The last column is sales price.
dim(data)
```

We created a histogram base of SalePrice (Appendix: Histogram of Property Sales Price) and find the property sales price is right-skewed. Most properties are below $200k.Given its distribution, we will later take the log of it to increase variability and approximate normality.
```{r, include=FALSE}
data$SalePrice<- as.numeric(data$SalePrice)
ggplot(data,aes(SalePrice))+
  geom_histogram(fill="#D48B6A", col= "black") + 
  labs(title="Histogram of property sales price")+
  theme_bw()
```


## Data processing
### Missing value
There are 34 variables with missing values. I dealt with these columns based on the data description.
```{r, include=FALSE}
NAcol<- which(colSums(is.na(data)) > 0)
sort(colSums(sapply(data[NAcol], is.na)),decreasing = T)
```


We find some missing values means no such feature present such as missing value is PoolQC means no pool, missing value in MiscFeature means no such feature. So we imputed missing value in PoolQC, MiscFeature, Alley, Fence, FireplaceQu, GarageType, GarageFinish, GarageQual, GarageCond, BsmtCond, BsmtExposure, BsmtQual, BsmtFinType2, BsmtFinType1, MasVnrType and MasVnrArea as "None" or 0 meaning no such feature.
```{r, include=FALSE}
data$PoolQC<- as.character(data$PoolQC) #NA means no pool
data$PoolQC[is.na(data$PoolQC)]<- "None"
data$PoolQC<- as.factor(data$PoolQC)

data$MiscFeature<- as.character(data$MiscFeature) #NA means no such feature
data$MiscFeature[is.na(data$MiscFeature)]<- "None"
data$MiscFeature<- as.factor(data$MiscFeature)

data$Alley<- as.character(data$Alley) #NA means no alley access
data$Alley[is.na(data$Alley)]<- "None"
data$Alley<- as.factor(data$Alley)

data$Fence<- as.character(data$Fence) #NA means no Fence
data$Fence[is.na(data$Fence)]<- "None"
data$Fence<- as.factor(data$Fence)

data$FireplaceQu<- as.character(data$FireplaceQu) #NA means no Fireplace
data$FireplaceQu[is.na(data$FireplaceQu)]<- "None"
data$FireplaceQu<- as.factor(data$FireplaceQu)

data$GarageType<- as.character(data$GarageType) #NA means no Garage
data$GarageType[is.na(data$GarageType)]<- "None"
data$GarageType<- as.factor(data$GarageType)

data$GarageFinish<- as.character(data$GarageFinish) #NA means no Garage
data$GarageFinish[is.na(data$GarageFinish)]<- "None"
data$GarageFinish<- as.factor(data$GarageFinish)

data$GarageQual<- as.character(data$GarageQual) #NA means no Garage
data$GarageQual[is.na(data$GarageQual)]<- "None"
data$GarageQual<- as.factor(data$GarageQual)

data$GarageCond<- as.character(data$GarageCond) #NA means no Garage
data$GarageCond[is.na(data$GarageCond)]<- "None"
data$GarageCond<- as.factor(data$GarageCond)

data$BsmtCond<- as.character(data$BsmtCond) #NA means no basement
data$BsmtCond[is.na(data$BsmtCond)]<- "None"
data$BsmtCond<- as.factor(data$BsmtCond)

data$BsmtExposure<- as.character(data$BsmtExposure) #NA means no basement
data$BsmtExposure[is.na(data$BsmtExposure)]<- "None"
data$BsmtExposure<- as.factor(data$BsmtExposure)

data$BsmtQual<- as.character(data$BsmtQual) #NA means no basement
data$BsmtQual[is.na(data$BsmtQual)]<- "None"
data$BsmtQual<- as.factor(data$BsmtQual)

data$BsmtFinType2<- as.character(data$BsmtFinType2) #NA means no basement
data$BsmtFinType2[is.na(data$BsmtFinType2)]<- "None"
data$BsmtFinType2<- as.factor(data$BsmtFinType2)

data$BsmtFinType1<- as.character(data$BsmtFinType1) #NA means no basement
data$BsmtFinType1[is.na(data$BsmtFinType1)]<- "None"
data$BsmtFinType1<- as.factor(data$BsmtFinType1)

data$MasVnrType<- as.character(data$MasVnrType) #NA means no masonry
data$MasVnrType[is.na(data$MasVnrType)]<- "None"
data$MasVnrType<- as.factor(data$MasVnrType)

data$MasVnrArea[is.na(data$MasVnrArea)]<- 0  #NA means no masonry
```

Some missing value could be imputed by geographical interpolation- taking the median value of the neighborhood such as The LotFrontage (Appendix: Box plot- lot frontage grouped by neighborhood).
```{r, include=FALSE}
ggplot(data, aes(Neighborhood,LotFrontage))+
geom_boxplot(varwidth=T, fill="plum") + 
    labs(title="Box plot- lot frontage grouped by neighborhood", 
         x="Neighborhood",
         y="Lot frontage")+
  theme_bw()
```

```{r,eval=T,echo=FALSE,results='hide'}
for (i in 1:nrow(data)){
        if(is.na(data$LotFrontage[i])){
               data$LotFrontage[i] <- as.integer(median(data$LotFrontage[data$Neighborhood==data$Neighborhood[i]], na.rm=TRUE)) 
        }
}
```

Some missing value can be replaced by other values, which is reasonable to an extent. `GarageYrBlt` is `NA` if there is no garage with this property. As `YearRemodAdd` equals to `YearBuilt` if there is no remodeling, we replace the `NA`s in `GarageYrBlt` with the `YearBuilt` of that property.
```{r,eval=T,echo=FALSE,results='hide'}
data$GarageYrBlt[is.na(data$GarageYrBlt)]= data$YearBuilt[is.na(data$GarageYrBlt)]
```

After taking care of missing values in the previous variables, we now have only 17 variables with missing values. And these variables have only 1 or 2 missing values.
```{r,eval=T,echo=FALSE,results='hide'}
NAcol<- which(colSums(is.na(data)) > 0)
sort(colSums(sapply(data[NAcol], is.na)),decreasing = T)
```

Now we impute the rest of the missing values.

`MSZoning` There are 4 missing values. As `RL` (Residential Low density) is the most common in this data, we assign `RL` to the missing values.
```{r,eval=T,echo=FALSE,results='hide'}
#table(data$MSZoning)
#data$Id[is.na(data$MSZoning)]
data$MSZoning[is.na(data$MSZoning)]<- "RL"
```

`Utilities` Only 1 property does not have all public utilities, so this is a variable of no use. We drop it.
```{r,eval=T,echo=FALSE,results='hide'}
#table(data$Utilities)
data<- data%>%select(-Utilities)
```

` BsmtHalfBath` 0 is the most common value, so we assign 0 to the two missing values.
```{r,eval=T,echo=FALSE,results='hide'}
#table(data$BsmtHalfBath)
data$BsmtHalfBath[is.na(data$BsmtHalfBath)]<- 0
```

`Exterior1st`, `Exterior2nd`. one observation has missing values in both cells. We assign the most common value `VinylSd`, `VinylSd` to them.
```{r,eval=T,echo=FALSE,results='hide'}
#data%>%filter(is.na(data$Exterior1st))
#table(data$Exterior1st)
#table(data$Exterior2nd)
data$Exterior1st[is.na(data$Exterior1st)]<- "VinylSd"
data$Exterior2nd[is.na(data$Exterior2nd)]<- "VinylSd"
```

`Electrical`,`KitchenQual`,`SaleType`. The most common values are `SBrkr`, `TA`, `WD`respectively. We assign these values to the missing value.
```{r,eval=T,echo=FALSE,results='hide'}
data$Electrical[is.na(data$Electrical)]<- "SBrkr"
data$KitchenQual[is.na(data$KitchenQual)]<- "TA"
data$SaleType[is.na(data$SaleType)]<- "WD"
```

`BsmtFullBath`We assign 0 to the missing value. We assign 0 to all other basement variables with NA as well.
```{r,eval=T,echo=FALSE,results='hide'}
#summary(data$BsmtFullBath)
#data[is.na(data$BsmtFullBath),]
data$BsmtFullBath[is.na(data$BsmtFullBath)]<- 0
data$BsmtFinSF1[is.na(data$BsmtFinSF1)]<- 0
data$BsmtFinSF2[is.na(data$BsmtFinSF2)]<- 0
data$BsmtUnfSF[is.na(data$BsmtUnfSF)]<- 0
data$TotalBsmtSF[is.na(data$TotalBsmtSF)]<- 0
```

`Functional` The predominant value is `Typ` typical functionality, so we assign it to the missing values.
```{r,eval=T,echo=FALSE,results='hide'}
data$Functional[is.na(data$Functional)]<- "Typ"
```

`GarageCars`,`GarageArea` We assign 0 to the missing values.
```{r,eval=T,echo=FALSE,results='hide'}
data$GarageArea[is.na(data$GarageArea)]<- 0
data$GarageCars[is.na(data$GarageCars)]<- 0
```

So far, all missing values have been imputed.
```{r,eval=T,echo=FALSE,results='hide'}
sum(is.na(data%>%select(-SalePrice))) 
```

### Variable structures
We now need to make sure that all categorical and numeric variables have the correct structures. We turned MSSubClass, MoSold, YrSold into factors. OverallQual and OverallCond can be wither factor or numeric, as they are ordinal. Here we keep them as numeric for now.
```{r,eval=T,echo=FALSE,results='hide'}
str(data)
data$MSSubClass<- as.factor(data$MSSubClass)
# OverallQual and OverallCond can be wither factor or numeric, as they are ordinal. Here I keep them as numeric for now.
data$MoSold<- as.factor(data$MoSold)# Change month and year to factors
data$YrSold<- as.factor(data$YrSold)
```
Months & years

The gragh in Appendix: "Box plot- sales price by month sold"" visualized how sales price varies across months. We see Janurary and April have relatively low median of sale price. July has lots of outliers. Janurary and July have the highest outlier values. Overall, the sale price does not vary much during month
```{r, include=FALSE}
ggplot(data, aes(MoSold, SalePrice))+
geom_boxplot(varwidth=T, fill="plum") + 
    labs(title="Box plot- sales price by month sold", 
         x="Month sold",
         y="Sales price")+
  theme_bw()
```

The graph in Appendix:"Box plot- sales price by Year sold" shows how sales price varies across years. From the diagram, we see the price does not vary much during years. 2007 has a couple of high outlier values. 

```{r, include=FALSE}
ggplot(data, aes(YrSold,SalePrice))+
geom_boxplot(varwidth=T, fill="plum") + 
    labs(title="Box plot- sales price by Year sold", 
         x="Sales price",
         y="Year sold")+
  theme_bw()
```

MSZoning Analysis

From the graph and table below in Appendix:"Distribution of MSZoning", it is obvious that most of houses in this dataset are built in the area of Residential Low Density, and follows by Residential Medium Density(460 houses). Few houes are built in Commercial, Floating Village and Residential High Density.Since a large amount of houses comes to the categoreis of Residential Low Density and Residential Medium Density, these two areas should be paid more attention for housing price analysis. 

```{r, include=FALSE}
ggplot(data, aes(x = MSZoning)) + 
geom_bar()+ 
ggtitle("Distribution of MSZoning")+
theme_bw()+
geom_text(stat='count',aes(label=..count..),vjust=-0.25)
```


From the boxplot in Appendix:"Box plot- sales price by MSZoning", we can tell that sale price in Residential Low Density zone has a lot of outliers, and the range is wide too. Commercial zone has relatively low sale price 

```{r, include=FALSE}
ggplot(data, aes(MSZoning,SalePrice))+
geom_boxplot(varwidth=T, fill="plum") + 
    labs(title="Box plot- sales price by MSZoning", 
         x="MSZoning",
         y="Sales price")+
  theme_bw()
```


SalePrice vs Numerical Values

We then visualized the relationship of saleprice between 4 numerical values: 

GrLivArea (Above grade (ground) living area square feet), 

TotalBsmtSF (Total square feet of basement area), 

TotRmsAbvGrd (Total rooms above grade (does not include bathrooms)), 

GarageArea (Size of garage in square feet).

```{r,eval=T,echo=FALSE,results='hide'}

# scatter plot of GrLiveArea
p1 <- ggplot(data, aes(x=GrLivArea, y=SalePrice)) + 
  geom_point(shape=1) +  
  geom_smooth(method=lm , color="red", se=FALSE)+
  ggtitle("Scatter plot of SalePrice and GrLivArea") +
  theme(plot.title = element_text(hjust = 0.4))

# scatter plot of TotalBsmtSF
p2 <- ggplot(data, aes(x=TotalBsmtSF, y=SalePrice)) + 
  geom_point(shape=1) +  
  geom_smooth(method=lm , color="red", se=FALSE)+
  ggtitle("Scatter plot of SalePrice and TotalBsmtSF") +
  theme(plot.title = element_text(hjust = 0.4))

#scatter plot of TotRmsAbvGrd
p3 <- ggplot(data, aes(x=TotRmsAbvGrd, y=SalePrice)) + 
  geom_point(shape=1) +  
  geom_smooth(method=lm , color="red", se=FALSE)+
  ggtitle("Scatter plot of SalePrice and TotRmsAbvGrd") +
  theme(plot.title = element_text(hjust = 0.4))

#scatter plot of GarageArea
p4 <- ggplot(data, aes(x=GarageArea, y=SalePrice)) + 
  geom_point(shape=1) +  
  geom_smooth(method=lm , color="red", se=FALSE)+
  ggtitle("Scatter plot of SalePrice and GarageArea") +
  theme(plot.title = element_text(hjust = 0.4))
```

All 4 variables have positive relationship with the sale price. GrLivArea and TotalBsmtSF have larger positive relationship then TotRmsAbvGrd and GarageArea
```{r,eval=T,echo=FALSE, warning=FALSE}
library(gridExtra)
grid.arrange(p1, p2,p3,p4)
```



### Feature Engineering
In order to get a normalized dataset, we log the saleprice and deleted the original sale price column. The LogSalePrice is our response variable. 
```{r,eval=T,echo=FALSE,results='hide'}
data<- data%>% mutate(LogSalePrice= log(SalePrice))%>%select(-SalePrice) #Take the log
```
ID is not useful, but we still restore it for reporting the results. Then we take ID out of our data set. 
```{r,eval=T,echo=FALSE,results='hide'}
test_id<- test$Id
data<- data%>% select(-Id)
```

We can also get the age of a house by taking `Yearsold`-`Yearbuilt`. We can also add 3 new features by the age of house. Isnew represents if a house is a new house. If the house age is 0, then mark 1 in Isnew to represent the house is a brand new one. If a house's age is not new, but the age of the house is less then 16 years, then we mark 1 in IsRecent to represent the house is recently built. If the house is more than 50 years old, then we mark 1 in IsOld to represent the house is old. 
```{r,eval=T,echo=FALSE,results='hide'}
data$YrSold<- as.numeric(as.character(data$YrSold))
data<- data%>% mutate(age= YrSold- YearBuilt)%>%
  mutate(Isnew= ifelse(age==0,1,0), IsRecent= ifelse(age>0& age<16,1,0), IsOld= ifelse(age>=50, 1,0))
data$YrSold<- as.factor(data$YrSold)
data$Isnew= as.factor(data$Isnew)
data$IsOld= as.factor(data$IsOld)
data$IsRecent= as.factor(data$IsRecent)
```

About the neighborhood, we not only want Iowa-specific results, but also generally interpretable results. So we are generating neighborhood feature data. A next step can be adding neighborhood data from census. (Appendix: Neighborhood)

```{r, include=FALSE}
nprice<- data%>% filter(!is.na(LogSalePrice))%>%group_by(Neighborhood)%>% summarise(median_price= median(LogSalePrice))
nprice<- nprice[order(nprice$median_price, decreasing = TRUE),]
ggplot(nprice)+
  geom_bar(aes(x=reorder(Neighborhood,median_price), y=median_price),stat = 'identity')
data<- data%>% mutate(IsPoor= ifelse(Neighborhood=="MeadowV"|Neighborhood=="IDOTRR"|Neighborhood=="BrDale",1,0))%>%
  mutate(IsRich= ifelse(Neighborhood=="NridgHt"|Neighborhood=="NoRidge"|Neighborhood=="StoneBr",1,0))
data$IsPoor= as.factor(data$IsPoor)
data$IsRich= as.factor(data$IsRich)
```

Total square footage is important by intuitive. So we add a column named Totalsqft by adding GrLivArea and TotalBsmtSF. 
```{r,eval=T,echo=FALSE,results='hide'}
data<- data%>% mutate(Totalsqft= GrLivArea+ TotalBsmtSF)
```

The porch variables are not providing much variability. So we consolidate it by adding OpenPorchSF, EnclosedPorch, X3SsnPorch, ScreenPorch together. We deleted all the porch variable and only consider the consolidated one "PorchArea"
```{r,eval=T,echo=FALSE,results='hide'}
data<- data%>% mutate(PorchArea= OpenPorchSF+ EnclosedPorch+ X3SsnPorch+ ScreenPorch) %>%
  select(-OpenPorchSF, -EnclosedPorch, -X3SsnPorch, -ScreenPorch)
```

The bathroom numbers. Now only the number of full bath ranks No.19 in the important features. A number of total bathrooms could be more helpful. Intuitively, we count full bath as 1 and half bath as 0.5. By using the following equation, we get TotalBath= BsmtFullBath+ 0.5*BsmtHalfBath+ FullBath+ 0.5*HalfBath. we deleted all other bathroom variables and will  only onsider the TotalBath. 
```{r,eval=T,echo=FALSE,results='hide'}
data<- data%>% 
  mutate(TotalBath= BsmtFullBath+ 0.5*BsmtHalfBath+ FullBath+ 0.5*HalfBath)%>%
  select(-BsmtFullBath,-BsmtHalfBath,-FullBath,-HalfBath)
```

### Final preparation
We make two boxplot for Totalsqft and LogSalePrice to have an overview of the data. In the Totalsqft boxplot, there seems to be two very large houses, and one very small house. LogSalePrice looks fine. (Appendix: Final Preparation)
```{r, include=FALSE}
par(mfrow=c(1,2))
boxplot(data$Totalsqft, main="Totalsqft") #There seems to be two very large homes. 
#data[order(data$Totalsqft,decreasing = T),] One in testing, one in training, so keep them here.
boxplot(data$LogSalePrice, main="LogSalePrice") #It's fine
```

Since there are many missing value in the response variable (LogSalePrice), we filtered out useful data and get our final dataset. We have 1460 observations and 79 variables and 1 response variable. 
```{r,eval=T,echo=FALSE,results='hide'}
data1 <- data[!is.na(data$LogSalePrice),]
dim(data1)
```
We reserve the testing data. 70% of the dataset is seperated to be training set, and the rest 30% is testing set. Now we have 1021 observations in the training set and 439 observations in the testing set. 
```{r,eval=T,echo=FALSE,results='hide'}
set.seed(101)
train <- sample(nrow(data1), 0.7*nrow(data1), replace=FALSE)
length(train)
data_train <- data1[train, ]
data_test <- data1[-train,]
dim(data_train)
dim(data_test)
```

# EDA (Part II after data processing)
## Correlations
Both the correlation matrixs and heat map shows similar relationships. 
The top continuous variables correlated with Sale Price:

OverallQual   0.817184418

Totalsqft     0.773276841

GrLivArea     0.700926653

GarageCars    0.680624807

TotalBath     0.673010594

GarageArea    0.650887556

TotalBsmtSF   0.612133975


```{r,eval=T,echo=FALSE, warning=FALSE}
cor<- correlation(data)
plot(cor)
```

```{r,eval=T,echo=FALSE, warning=FALSE}
plotData<- data%>%filter(!is.na(LogSalePrice))
plotData <-melt(cor(plotData[sapply(data, is.numeric)]))
ggplot(plotData ,
    aes(x = Var1, y = Var2, fill =value)) +
    geom_tile() +
    ylab("") +
    xlab("") +
scale_x_discrete(limits = rev(levels(plotData $Var2))) + #Flip the x- or y-axis
    scale_fill_gradient( low = "#56B1F7", high = "#132B43") +    
       guides(fill = guide_legend(title = "Correlation"))+
  theme(axis.text.x = element_text(angle=45))
```



# Models
## Random Forest Modeling

Before building the random forest model, we used the importance of random forest to see the important variables. This complements correlation analysis shows the top five important variables are `Totalsqft`, `OverallQual`, `GrLivArea`, `Neighborhood`,`age` which is similar to the elastic net model analysis. 
```{r,eval=T,echo=FALSE,results='hide'}
data1 <- data[!is.na(data$LogSalePrice),]
eda.rf<- randomForest(LogSalePrice~., data1, mtry=25, ntree=500,importance=TRUE)
imp_RF <- data.frame(eda.rf$importance)
imp_RF <- imp_RF[order(imp_RF$X.IncMSE, decreasing = TRUE),]
setDT(imp_RF, keep.rownames = TRUE)[]
```
```{r,eval=T,echo=FALSE, warning=FALSE}
ggplot(imp_RF[1:20,], aes(x=reorder(rn, X.IncMSE),y=X.IncMSE,fill="#D48B6A")) + 
  geom_bar(stat = 'identity') + labs(x = 'Variables', y= '% increase MSE if variable is randomly permuted') + coord_flip() + theme(legend.position="none")
```


```{r,eval=T,echo=FALSE,results='hide'}
dim(data1)
```

We used the training data set to build our random forest model using randomForest() functiom. We tune ntree and mtry, the two parameters of random forest.From the error ntree plot, we may need at least 100 trees to settle the OOB testing errors, so 500 trees are enough here. Now we fix ntree=500, We only want to compare the OOB mse[500] to see the mtry effects. Here we loopmtry from 1 to 30 and return the testing OOB errors. (Appendix: ntree plot)
```{r, include=FALSE}
fit.rf <-randomForest(LogSalePrice~., data_train, mtry=10, ntree=500)
plot(fit.rf, col="red", pch=16, type="p",main="ntree plot") 
```

The recommended mtry for reg trees are mtry=p/3=76/3 about 25 or 26. We run a loop around this recommended value and found smallest OOB mse at mtry= 25. We take mtry=25. (Appendix: mtry plot)
```{r, include=FALSE}
rf.error.p <- 1:10# set up a vector of length 30
for(p in 1:10){
  fit.rf <-randomForest(LogSalePrice~., data_train, mtry=20+p, ntree=500)
  rf.error.p[p] <- fit.rf$mse[500]
}
plot(21:30, rf.error.p, pch=16,xlab="mtry",ylab="OOB mse of mtry", main="mtry plot")
lines(21:30, rf.error.p)
```

The OOB error is 0.003064511 and the testing error is 0.0219367 Testing error is smaller than the training errir, but the testing error is also relatively small. So, this model predicts the testing dataset well. The following plot shows the error based on tree numbers. 
```{r,eval=T,echo=FALSE,results='hide'}
fit_rf <-randomForest(LogSalePrice~., data_train, mtry=25, ntree=500)
mse.oob <-mean((data_train$LogSalePrice-fit_rf$predicted)^2)
mse.oob
mse.train.rf<-mean((data_train$LogSalePrice-predict(fit_rf, data_train))^2)
mse.train.rf
mse.test.rf<-mean((data_test$LogSalePrice-predict(fit_rf, data_test))^2)
mse.test.rf
```

```{r,eval=T,echo=FALSE,warning=FALSE}
plot(fit_rf)
```


## Boosting tree

After trying different tuning parameter, we get n.trees = 20000, interaction.depth = 2, cv.folds = 5 for minimizing the training error. 
```{r,eval=T,echo=T,results='hide'}
fit_boosting <- gbm(
  formula = LogSalePrice~.,
  distribution = "gaussian",
  data = data_train,
  n.trees = 20000,
  interaction.depth = 2,
  shrinkage = 0.001,
  cv.folds = 5,
  n.cores = NULL, # will use all cores by default
  verbose = FALSE
  )  
pred.train <- predict(fit_boosting, n.trees = fit_boosting$n.trees, data_train)
caret::RMSE(pred.train, data_train$LogSalePrice)
```

The boosting tree model gives us the cross-validation error to be 0.02045268, the training error to be 0.250696 and the testing error tobe 0.3419559 The training and testing errors are very similar, which indicates the model estimates the testing dataset well. 

```{r,eval=T,echo=FALSE,results='hide'}
fit_boosting$cv.error[10000] #The cross-validation error of boosting. Is it useful or not?

pred.train <- predict(fit_boosting, n.trees = fit_boosting$n.trees, data_train)
mse.train.boosting <- caret::RMSE(pred.train, data_train$LogSalePrice)
mse.train.boosting

pred.test <- predict(fit_boosting, n.trees = fit_boosting$n.trees, data_test)
mse.test.boosting <- caret::RMSE(pred.test, data_test$LogSalePrice)
mse.test.boosting
```

## LASSO/Elastic Net

```{r,eval=T,echo=FALSE,results='hide'}
data1 <- data[!is.na(data$LogSalePrice),]
dim(data1)
```

After modifying the data set, a elastic net was used to reduce dimensionality. 
```{r,eval=T,echo=FALSE,results='hide'}
set.seed(101)
x <- model.matrix(LogSalePrice~., data=data1)[, -1]
y <- data.frame(LogSalePrice=as.numeric(data1$LogSalePrice))

set.seed(101)
train <- sample(nrow(data1), 0.7*nrow(data1), replace=FALSE)
length(train)
x_train <- x[train, ]
x_test <- x[-train,]
y_train <- y[train,]
y_test <- y[-train,]
data_train<-data.frame(x_train,LogSalePrice=y_train)
data_test<-data.frame(x_test,LogSalePrice=y_test)
summary(data_train)
dim(x_train)
```

Ridge regression doesn't cut any variable, but it gives us unique solutions. LASSO estimation can give us a smaller model for the ease of interpretaion. Elastic Net combines Ridge Regression and LASSO, by choosing alpha between 0 and 1. so that it will do feature selection, yet still benefit from Ridge Regression. First, we use the following plot to choose the alpha. The plot describes how mse changes with alpha. However, the plot is random here because mse depends on the split of nfolds. We run it several times and find that mse is always low at alpha=0.9. What's more, there is no big differences of mse for different alpha, so we stick with it in the following analysis.
```{r,eval=T,echo=FALSE}
#alpha
alpha<- seq(0,1,0.1)
MSE<- seq(0,1,0.1)
table<- data.frame(alpha,MSE)
for(i in 0:10){
  a= i/10
  fit.cv.1<- cv.glmnet(x_train, y_train,alpha=a,nfolds=10)
  table[i+1,2]=min(fit.cv.1$cvm)
}
# plot how mse changes with alpha
plot(table$alpha,table$MSE)
lines(table$alpha,table$MSE)
```

We use cross validation to select the lambda. From this plot we show that as ?? ??? ???, the impact of the shrinkage penalty grows, and the coefficient estimates will approach zero. To have a parsimonious model, we decide to use lambda.1se in our elastic net, which give us 54 variales in our final model. (We also try lambda.min and discuss the results in the appendix.)
```{r,eval=T,echo=FALSE}
fit.fl.cv <- cv.glmnet(x_train, y_train,alpha=0.9, nfolds=10 ) 
plot(fit.fl.cv)
```

Here are the non-zero coefficients and variables. After cleaning the results and sorting different levels of the same categorical variables, the elastic net returns 41 variables. We conclude that there are six types pf main factors with most effect on home price.   
- Area: lot size, shape, and configuration 
- Location: neighborhood locations, proximity to main road
- Garage: size, age, quality, and area
- Add-on features: street pave, material, basement, heating or AC, porch area, fireplace
- Age: original construction date, type of dwelling
- Zoning: the building class, the general zoning classification

As a house buyer, it's easy to notice the relationship between house price and house area, age and locatioin. Since cars are necessary to many families, the condition of garage is also taken into account when chooing houses. However, the majority people may not pay too much attention to add-on features and zoning. From our elastic net results, we can see that these add-on features are also determinants to the house price. It's not surprising that the quality of heating system, central air condition, and basement are key factors, but house buyer should know that they also pay for the porch area, fireplace, and street pave! Don't complain the narrow porch while enjoying the lower price of the house. Interestingly, the house in medium density has lower price. The possible reason is that the residental low density represents house and the residental high density represents luxury departments in the downtown, so the prices are both higher than the medium density.   

```{r,eval=T,echo=FALSE}
coef.1se <- coef(fit.fl.cv, s="lambda.1se") 
coef.1se <- coef.1se[which(coef.1se !=0),] 
coef.1se
lasso.coef<-data.frame(coef.1se )
```

The training error of this elastic net is 0.01275193, while the testing error is 0.03670769.
```{r,eval=T,echo=FALSE,results='hide'}
fit.lambda.1se<- glmnet(x_train, y_train, alpha=0.9,lambda = fit.fl.cv$lambda.1se,family = "gaussian")
mse.train.1se<- mean((y_train-predict(fit.lambda.1se,x_train))^2)
mse.test.1se<- mean((y_test-predict(fit.lambda.1se,x_test))^2)
```

The LASSO estimators are biased, so we use the same set of variables to refit our model using lm() function. The training error of this relaxed elastic net is 0.06878601, while the testing error is 0.01075683.
```{r,eval=T,echo=FALSE,results='hide'}
var.1se <- rownames(as.matrix(coef.1se)) 
lm.input <- as.formula(paste("LogSalePrice", "~", paste(var.1se[-1], collapse = "+"))) 
fit.1se.lm <- lm(lm.input, data=data_train)
mse.test.1se.lm <-mean((data_test$LogSalePrice-predict(fit.1se.lm,data_test, interval="prediction"))^2)
mse.train.1se.lm<- mean((y_train-predict(fit.1se.lm,data_train))^2)
```




## Neural Net
We also use neural net to address this problem. The neural net we learn in the class can deal with classification problems. We really want to if it can be used to solve problems with continues response variables. After searching for more information, we modify the original neural net. To be specific, we change the activation function in the output layer to the "linear" rather than "sigmoid", and also change the loss function to "mean_squared_error" instead of "categorical_crossentropy". Then, we train this neural net and choose epoch=22 according to the plot. 
```{r,eval=T,echo=FALSE,results='hide'}
model <- keras_model_sequential() %>%
 layer_dense(units = 16, activation = "relu", input_shape = c(286)) %>% # 1 layer with 16 neurons
 layer_dense(units = 16, activation = "relu") %>% 
 layer_dense(units = 1, activation = "linear")
model %>% compile(
 optimizer = "rmsprop",
 loss = "mean_squared_error",
 metrics = c("accuracy")
)
```

```{r,eval=T,echo=FALSE} 
val_indices <- 1:300
x_val <- x_train[val_indices,]
partial_x_train <- x_train[-val_indices,]
 
y_val <- y_train[val_indices]
partial_y_train <- y_train[-val_indices]

fit1 <- model %>% fit(
 partial_x_train,
 partial_y_train,
 epochs = 30,
 batch_size = 512,
 validation_data = list(x_val, y_val)
 )
plot(fit1)
```


```{r,eval=T,echo=FALSE}
model <- keras_model_sequential() %>%    #Retraining Model
 layer_dense(units = 16, activation = "relu", input_shape = c(286)) %>% 
 layer_dense(units = 16, activation = "relu") %>% 
 layer_dense(units = 1, activation = "linear")
 
model %>% compile(
 optimizer = "rmsprop",
 loss = "mean_squared_error",
 metrics = c("accuracy")
 )

model %>% fit(x_train, y_train, epochs = 22, batch_size = 512)
 
predict_x_train<-model %>% predict(x_train)
mse.train.nn<-mean((data_train$LogSalePrice-predict_x_train)^2)
paste("The training error of neural net is",mse.train.nn)

predict_x_test<-model %>% predict(x_test)
mse.test.nn<-mean((data_test$LogSalePrice-predict_x_test)^2)
paste("The testing error of neural net is",mse.test.nn,"Neural net gives us a reasonal result.")
```


## Model comparison
```{r,eval=T,echo=FALSE}
model.com<-data.frame(model=c("Elastic Net","Relaxed Elastic Net","Random Forest","Boosting","Neural Net" ),training_error=numeric(5),testing_error=numeric(5))

model.com[1,2]<-mse.train.1se
model.com[1,3]<-mse.test.1se

model.com[2,2]<-mse.train.1se.lm
model.com[2,3]<-mse.test.1se.lm

model.com[3,2]<-mse.train.rf
model.com[3,3]<-mse.test.rf
  
model.com[4,2]<-mse.train.boosting
model.com[4,3]<-mse.test.boosting

model.com[5,2]<-mse.train.nn
model.com[5,3]<-mse.test.nn

model.com

```
Overall, all the models give us the reasonal well results. From this table, we can see that elastic net and random forest give us relatively low training error and testing error. So both models can be our final models. 

## Conclusion
1. We use elastic net, random forest, boosting and neural net to predict the house price based on internal characteristics, like lot area, material, heating quality, and so on, and external factors, such as zoning, proximity to main road, and so on. 
2. We find that the house price mainly depends on six types of factors: Area, Location, Garage, Add-on features, Age and Zoning. It's easy to notice the relationship between house price and house area, age, locatioin and garage. It's also not surprising that the quality of heating system, central air condition, and basement are key factors, but house buyer should know that they also pay for the porch area, fireplace, and street pave! 
3. All of our models do pretty good job, which means we can predict the house price accurately based on physical characteristics. As we know, homes are hedonic pricing goods through intrinstic features which is all the information used to price the home, while controlling the exterior factors, like sales year fixed effect and local market. So we think the price well captures the information of the houses in this market. Few people in this area treat the house as the investment and hope to make profits by playing the market. 
4. Overall, we believe our models work well to predict house price, and they should work better for a healthy housing market.


# Appendix

```{r,eval=T,echo=FALSE,results='hide'}
data<- data%>% mutate(SalePrice= exp(LogSalePrice))%>%select(-LogSalePrice)
```
## Histogram of Property Sales Price
```{r,eval=T,echo=FALSE, warning=FALSE}
data$SalePrice<- as.numeric(data$SalePrice)
ggplot(data,aes(SalePrice))+
  geom_histogram(fill="#D48B6A", col= "black") + 
  labs(title="Histogram of property sales price")+
  theme_bw()
```

## Box plot- lot frontage grouped by neighborhood
```{r,eval=T,echo=FALSE, warning=FALSE}
ggplot(data, aes(Neighborhood,LotFrontage))+
geom_boxplot(varwidth=T, fill="plum") + 
    labs(title="Box plot- lot frontage grouped by neighborhood", 
         x="Neighborhood",
         y="Lot frontage")+
  theme_bw()+
  theme(axis.text.x = element_text(angle=45))
```


## Box plot- sales price by month sold
```{r,eval=T,echo=FALSE, warning=FALSE}
ggplot(data, aes(MoSold, SalePrice))+
geom_boxplot(varwidth=T, fill="plum") + 
    labs(title="Box plot- sales price by month sold", 
         x="Month sold",
         y="Sales price")+
  theme_bw()
```


## Box plot- sales price by Year sold
```{r,eval=T,echo=FALSE, warning=FALSE}
ggplot(data, aes(YrSold,SalePrice))+
geom_boxplot(varwidth=T, fill="plum") + 
    labs(title="Box plot- sales price by Year sold", 
         x="Sales price",
         y="Year sold")+
  theme_bw()
```


## Distribution of MSZoning
```{r,eval=T,echo=FALSE, warning=FALSE}
ggplot(data, aes(x = MSZoning)) + 
geom_bar()+ 
ggtitle("Distribution of MSZoning")+
theme_bw()+
geom_text(stat='count',aes(label=..count..),vjust=-0.25)
```


## Box plot- sales price by MSZoning
```{r,eval=T,echo=FALSE, warning=FALSE}
ggplot(data, aes(MSZoning,SalePrice))+
geom_boxplot(varwidth=T, fill="plum") + 
    labs(title="Box plot- sales price by MSZoning", 
         x="MSZoning",
         y="Sales price")+
  theme_bw()
```



```{r,eval=T,echo=FALSE,results='hide'}
data<- data%>% mutate(LogSalePrice= log(SalePrice))%>%select(-SalePrice) #Take the log
```
