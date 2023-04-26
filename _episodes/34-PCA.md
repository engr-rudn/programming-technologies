---
title: "Principle Component Analysis (PCA)"
teaching: 0
exercises: 0
questions:
- "How can we perform unsupervised learning with Principle Component Analysis (PCA)?"
objectives:
- "Understand PCA using examples"
---
# Principle Component Analysis (PCA)
Principal component analysis (PCA) allows us to summarize and to visualize the information in a data set containing individuals/observations described by multiple inter-correlated quantitative variables. Each variable could be considered as a different dimension. If you have more than 3 variables in your data sets, it could be very difficult to visualize a multi-dimensional hyperspace.

Principal component analysis is used to extract the important information from a multivariate data table and to express this information as a set of few new variables called principal components. These new variables correspond to a linear combination of the originals. The number of principal components is less than or equal to the number of original variables.

The information in a given data set corresponds to the total variation it contains. The goal of PCA is to identify directions (or principal components) along which the variation in the data is maximal. In other words, PCA reduces the dimensionality of a multivariate data to two or three principal components, that can be visualized graphically, with minimal loss of information. It is particularly helpful in the case of "wide" datasets, where you have many variables for each sample.

As you already read in the introduction, PCA is particularly handy when you're working with "wide" data sets. But why is that? Well, in such cases, where many variables are present, you cannot easily plot the data in its raw format, making it difficult to get a sense of the trends present within. PCA allows you to see the overall "shape" of the data, identifying which samples are similar to one another and which are very different. This can enable us to identify groups of samples that are similar and work out which variables make one group different from another.

Note that, the PCA method is particularly useful when the variables within the data set are highly correlated. Correlation indicates that there is redundancy in the data. Due to this redundancy, PCA can be used to reduce the original variables into a smaller number of new variables ( = principal components) explaining most of the variance in the original variables.

**Caution:** PCAs primary purpose is NOT as a ways of feature removal! PCA can reduce dimensionality but it wont reduce the number of features / variables in your data. What this means is that you might discover that you can explain 99% of variance in your 1000 feature dataset by just using 3 principal components but you still need those 1000 features to construct those 3 principal components, this also means that in the case of predicting on future data you still need those same 1000 features on your new observations to construct the corresponding principal components.

#### Eigenvector & Eigenvalue
Just like many things in life, eigenvectors, and eigenvalues come in pairs: every eigenvector has a corresponding eigenvalue. Simply put, an eigenvector is a direction, such as "vertical" or "45 degrees", while an eigenvalue is a number telling you how much variance there is in the data in that direction. The eigenvector with the highest eigenvalue is, therefore, the first principal component.

So wait, there are possibly more eigenvalues and eigenvectors to be found in one data set?

That's correct! The number of eigenvalues and eigenvectors that exits is equal to the number of dimensions the data set has. In the example that you saw above, there were 2 variables, so the data set was two-dimensional. That means that there are two eigenvectors and eigenvalues. Similarly, you'd find three pairs in a three-dimensional data set.

There are two general methods to perform PCA in R :  
__- Spectral decomposition which examines the covariances / correlations between variables  
- Singular value decomposition which examines the covariances / correlations between individuals__

The function *princomp()* uses the **spectral decomposition** approach. The functions *prcomp()* and PCA()[FactoMineR] use the **singular value decomposition (SVD)**.

#### prcomp() and princomp() functions
The simplified format of these 2 functions are :

```{r}
# prcomp(x, scale = FALSE)
# princomp(x, cor = FALSE, scores = TRUE) 
```

1. Arguments for prcomp():
- x: a numeric matrix or data frame
- scale: a logical value indicating whether the variables should be scaled to have unit variance before the analysis takes place
2. Arguments for princomp():
- x: a numeric matrix or data frame
- cor: a logical value. If TRUE, the data will be centered and scaled before the analysis
- scores: a logical value. If TRUE, the coordinates on each principal component are calculated

The elements of the outputs returned by the functions prcomp() and princomp() includes :

| prcomp() 	| princomp() 	|                                   Description                                  	|
|:--------:	|:----------:	|:------------------------------------------------------------------------------:	|
|   sdev   	|    sdev    	|               the standard deviations of the principal components              	|
| rotation 	|  loadings  	|           the matrix of variable loadings (columns are eigenvectors)           	|
|  center  	|   center   	|                the variable means (means that were substracted)                	|
|   scale  	|    scale   	|    the variable standard deviations (the scaling applied to each variable )    	|
|     x    	|   scores   	| The coordinates of the individuals (observations) on the principal components. 	|

```{r, PCA tutorial by Hefin Ryes}
data(iris)
head(iris)
summary(iris)
mypca <- prcomp(iris[,-5], scale = T)
plot(iris$Sepal.Length, iris$Sepal.Width)
plot(scale(iris$Sepal.Length), scale(iris$Sepal.Width))
mypca
summary(mypca)
# Standard deviation: This is simply the eigenvalues in our case
# since the data has been centered and scaled (standardized)
# Proportion of Variance: This is the amount of variance the 
# component accounts for in the data, ie. PC1 accounts for >44%
# of total variance in the data alone!
# Cumulative Proportion: This is simply the accumulated amount
# of explained variance, ie. if we used the first 10 components
# we would be able to account for >95% of total variance in the data.
plot(mypca, type = "l")
biplot(mypca, scale = 0)
# Extract PCA score
str(mypca)
head(mypca$x)
iris2 <- cbind(iris, mypca$x[,1:2])
head(iris2)
# Plot with ggplot
library(ggplot2)
ggplot(iris2, aes(PC1, PC2, col = Species, fill = Species)) +
  stat_ellipse(geom = "polygon", col = "black", alpha = 0.5) +
  geom_point(shape = 21, col = "black")
ggplot(iris2, aes(PC1, PC2, col = Species, fill = Species)) +
  geom_point(shape = 21, col = "black")
# Correlation with variables and PCA
cor(iris[, -5], iris2[, 6:7])
```


## PCA by DataCamp
URL: https://www.datacamp.com/community/tutorials/pca-analysis-r

#### A Simple PCA
In this section, you will try a PCA using a simple and easy to understand dataset. You will use the mtcars dataset, which is built into R. This dataset consists of data on 32 models of car, taken from an American motoring magazine (1974 Motor Trend magazine). For each car, you have 11 features, expressed in varying units (US units).

Because PCA works best with numerical data, you'll exclude the two categorical variables (vs and am).

We are left with a matrix of 9 columns and 32 rows, which you pass to the prcomp() function, assigning your output to mtcars.pca. We will also set two arguments, center and scale, to be TRUE. Then you can have a peek at your PCA object with summary().

```{r, Compute the Principal Components 1}
data(mtcars)
head(mtcars)
#PCA is sensitive to variances of values. So, we have
# to convert standardize the values using scale.
# It expresses the values in terms of their sd.
pca_mtcars <- prcomp(mtcars[,c(1:7, 10, 11)], scale = T, center = T)
summary(pca_mtcars)
# Principle Components are ordered according to the significance.
# PC1 & PC2 combined can describe 86% (cumulative prop) of variance in the dataset. 
```
We get 9 principal components, which you call PC1-9. Each of these explains a percentage of the total variation in the dataset. That is to say: PC1 explains 63% of the total variance, which means that nearly two-thirds of the information in the dataset (9 variables) can be encapsulated by just that one Principal Component. PC2 explains 23% of the variance. So, by knowing the position of a sample in relation to just PC1 and PC2, we can get a very accurate view on where it stands in relation to other samples, as just PC1 and PC2 can explain 86% of the variance.

```{r, Compute the Principal Components 2}
pca_mtcars
str(pca_mtcars)
```
Our PCA object contains the following information:\

+ The center point (center), scaling (scale), standard deviation(sdev) of each principal component\ 
+ The relationship (correlation or anticorrelation, etc) between the initial variables and the principal components (rotation)\ 
+ The values of each sample in terms of the principal components (x)\

#### Plotting PCA

Now it's time to plot our PCA. You will make a biplot, which includes both the position of each sample in terms of PC1 and PC2 and also will show us how the initial variables map onto this. We will use the ggbiplot package, which offers a user-friendly and pretty function to plot biplots. A biplot is a type of plot that will allow you to visualize how the samples relate to one another in our PCA (which samples are similar and which are different) and will simultaneously reveal how each variable contributes to each principal component.

Before we  get started, let's first install ggbiplot!
```{r,Plotting PCA 1}
library(devtools)
# install_github("vqv/ggbiplot")
library(ggbiplot)
ggbiplot(pca_mtcars)
```
The axes are seen as arrows originating from the center point. Here, you see that the variables hp, cyl, and disp all contribute to PC1, with higher values in those variables moving the samples to the right on this plot. This lets you see how the data points relate to the axes, but it's not very informative without knowing which point corresponds to which sample (car).

You'll provide an argument to ggbiplot: let's give it the rownames of mtcars as labels. This will name each point with the name of the car in question:

```{r, Plotting PCA 2}
ggbiplot(pca_mtcars, labels=rownames(mtcars))
# we can also do this using base R
plot(pca_mtcars, type = "l")  # this is a scree plot
# biplot
biplot(pca_mtcars, scale = 0)
```
Now you can see which cars are similar to one another. For example, the Maserati Bora, Ferrari Dino and Ford Pantera L all cluster together at the top. This makes sense, as all of these are sports cars.

How else can you try to better understand your data?

#### Interpreting the results

Maybe if we look at the origin of each of the cars. We'll put them into one of three categories (categories?), one each for the US, Japanese and European cars. You make a list for this info, then pass it to the groups argument of ggbiplot. You'll also set the ellipse argument to be TRUE, which will draw an ellipse around each group.

```{r, Interpreting the results 1}
# Let's group the cars according to their country of origin
rownames(mtcars)
# First the cars are form Japan, and so on.
mtcars_country <- c(rep("Japan", 3), rep("US",4), 
                    rep("Europe", 7), rep("US",3), 
                    "Europe", rep("Japan", 3), 
                    rep("US",4), rep("Europe", 3), 
                    "US", rep("Europe", 3))
ggbiplot(pca_mtcars, ellipse=TRUE, labels=rownames(mtcars),
         groups= mtcars_country)
```
Now you see something interesting: the American cars form a distinct cluster to the right. Looking at the axes, you see that the American cars are characterized by high values for cyl, disp, and wt. Japanese cars, on the other hand, are characterized by high mpg. European cars are somewhat in the middle and less tightly clustered than either group.

Of course, you have many principal components available, each of which map differently to the original variables. You can also ask ggbiplot to plot these other components, by using the choices argument.

Let's have a look at PC3 and PC4:

```{r, Interpreting the results 2}
ggbiplot(pca_mtcars, ellipse = T, choices = c(3,4),
         labels = rownames(mtcars), groups= mtcars_country)
```

You don't see much here, but this isn't too surprising. PC3 and PC4 explain very small percentages of the total variation, so it would be surprising if you found that they were very informative and separated the groups or revealed apparent patterns.

Let's take a moment to recap: having performed a PCA using the mtcars dataset, we can see a clear separation between American and Japanese cars along a principal component that is closely correlated to cyl, disp, wt, and mpg. This provides us with some clues for future analyses; if we were to try to build a classification model to identify the origin of a car, these variables might be useful.

#### Graphical parameters with ggbiplot

There are also some other variables we can play with to alter your biplots. We can add a circle to the center of the dataset (circle argument):
```{r, Graphical parameters with ggbiplot}
ggbiplot(pca_mtcars, circle = T, labels = rownames(mtcars),
         groups = mtcars_country)
# We can also scale the samples (obs.scale) and the variables (var.scale):
ggbiplot(pca_mtcars, ellipse = T, obs.scale = 1, var.scale = 1,
         labels = rownames(mtcars), groups=mtcars_country)
# Let's see a plot without scaling
ggbiplot(pca_mtcars, ellipse = T, labels=rownames(mtcars),
         groups= mtcars_country)
# We can also remove the arrows altogether, using var.axes.
ggbiplot(pca_mtcars, ellipse = T, var.axes = F, 
         labels = rownames(mtcars), groups = mtcars_country)
```

#### Customize ggbiplot

As ggbiplot is based on the ggplot function, you can use the same set of graphical parameters to alter your biplots as you would for any ggplot. Here, you're going to:

- Specify the colours to use for the groups with scale_colour_manual()
- Add a title with ggtitle()
- Specify the minimal() theme
- Move the legend with theme()

```{r, Customize ggbiplot}
ggbiplot(pca_mtcars, ellipse = T, labels=rownames(mtcars),
         groups= mtcars_country) +
  scale_colour_manual(name="Origin", 
                      values= c("forest green", "red3", "dark blue"))+
  ggtitle("PCA of mtcars dataset") +
  theme_linedraw() +
  theme(legend.position = "right")
# install.packages("ggthemes") 
library(ggthemes)
# theme_economist
ggbiplot(pca_mtcars, ellipse = T, labels=rownames(mtcars),
         groups= mtcars_country) +
  scale_colour_manual(name="Origin", 
                      values= c("forest green", "red3", "dark blue"))+
  ggtitle("PCA of mtcars dataset") +
  theme_economist() + 
  scale_fill_economist() +
  theme(legend.position = "right")
# theme_stata
ggbiplot(pca_mtcars, ellipse = T, labels=rownames(mtcars),
         groups= mtcars_country) +
  scale_colour_manual(name="Origin", 
                      values= c("forest green", "red3", "dark blue"))+
  ggtitle("PCA of mtcars dataset") +
  theme_stata() + 
  scale_fill_stata() +
  theme(legend.position = "right")
# theme_hc
ggbiplot(pca_mtcars, ellipse = T, labels=rownames(mtcars),
         groups= mtcars_country) +
  scale_colour_manual(name="Origin", 
                      values= c("forest green", "red3", "dark blue"))+
  ggtitle("PCA of mtcars dataset") +
  theme_hc() + 
  theme(legend.position = "right")
```

# Adding a new sample
Okay, so let's say you want to add a new sample to your dataset. This is a very special car, with stats unlike any other. It's super-powerful, has a 60-cylinder engine, amazing fuel economy, no gears and is very light. It's a "spacecar", from Jupiter.

Can you add it to your existing dataset and see where it places in relation to the other cars?

You will add it to mtcars, creating mtcarsplus, then repeat your analysis. You might expect to be able to see which region's cars it is most like.

```{r, adding a new sample}
spacecar <- c(1000,60,50,500,0,0.5,2.5,0,1,0,0)
mtcarsplus <- rbind(mtcars, spacecar)
tail(mtcarsplus)
mtcars.countryplus <- c(mtcars_country, "Jupiter")
mtcarsplus.pca <- prcomp(mtcarsplus[,c(1:7,10,11)], center = T, scale = T)
ggbiplot(mtcarsplus.pca, ellipse = T, var.axes = F,
         labels=c(rownames(mtcars), "spacecar"), groups=mtcars.countryplus) +
  scale_colour_manual(name="Origin", 
                      values= c("forest green", "red3", "violet", "dark blue"))+
  ggtitle("PCA of mtcars dataset, with extra sample added")+
  theme_minimal()+
  theme(legend.position = "bottom")
```

But that would be a naive assumption! The shape of the PCA has changed drastically, with the addition of this sample. When you consider this result in a bit more detail, it actually makes perfect sense. In the original dataset, you had strong correlations between certain variables (for example, cyl and mpg), which contributed to PC1, separating your groups from one another along this axis. However, when you perform the PCA with the extra sample, the same correlations are not present, which warps the whole dataset. In this case, the effect is particularly strong because your extra sample is an extreme outlier in multiple respects.

If you want to see how the new sample compares to the groups produced by the initial PCA, you need to project it onto that PCA.

# PCA with factoextra

URL: http://www.sthda.com/english/articles/31-principal-component-methods-in-r-practical-guide/118-principal-component-analysis-in-r-prcomp-vs-princomp/

```{r, factoextra}
# install.packages("factoextra")
library(factoextra)
```
```{r, data decathlon}
data(decathlon2)
str(decathlon2)
# View(decathlon2)
decathlon2.active <- decathlon2[1:23, 1:10]
head(decathlon2.active[, 1:5])
```
```{r, calculation}
res.pca <- prcomp(decathlon2.active, scale = TRUE)
summary(res.pca)
fviz_eig(res.pca, xlab = "Principle Component", barfill = "royalblue", 
         barcolor = "royalblue", linecolor = "black", addlabels = T)
```

Right, so how many principle components do we want? We obviously want to be able to explain as much variance as possible but to do that we would need all 30 components, at the same time we want to reduce the number of dimensions so we definitely want less than 30!

Since we standardized our data and we now have the corresponding eigenvalues of each PC we can actually use these to draw a boundary for us. Since an eigenvalues <1 would mean that the component actually explains less than a single explanatory variable we would like to discard those. If our data is well suited for PCA we should be able to discard these components while retaining at least 70–80% of cumulative variance. Lets plot and see:

```{r}
# Scree Plot
screeplot(res.pca, type = "l", npcs = 15, main = "Screeplot of the first 10 PCs")
abline(h = 1, col="red", lty=5)
legend("topright", legend=c("Eigenvalue = 1"),
       col=c("red"), lty=5, cex=0.6)
# Cumulative Plot
cumpro <- cumsum(res.pca$sdev^2 / sum(res.pca$sdev^2))
plot(cumpro[0:15], xlab = "PC #", ylab = "Amount of explained variance", main = "Cumulative variance plot")
abline(v = 6, col="blue", lty=5)
abline(h = 0.88759, col="blue", lty=5)
legend("topleft", legend=c("Cut-off @ PC6"),
       col=c("blue"), lty=5, cex=0.6)
```

We notice  that the first 3 components has an Eigenvalue >1 and explains almost 70% of variance, this is great! We can effectively reduce dimensionality from 10 to 3 while only “loosing” about 30% of variance!

__The selection of PC based on Eigenvalue >1 is  known as Kaiser-Guttman Rule.__

We also notice that we can actually explain more than 60% of variance with just the first two components. 

```{r, customization}
fviz_pca_ind(res.pca, 
             col.ind = "cos2", # Color by the quality of representation
             repel = T, # Avoid text overlapping
             gradient.cols = c("#1CC6D4", "#1D9DDE","#2465C7"))
fviz_pca_var(res.pca, 
             col.var = "contrib", 
             repel = T,
             gradient.cols = c("#00AFBB", "#E7B800", "#FC4E07"))
fviz_pca_biplot(res.pca,
                repel = T, 
                col.ind = "cyan3",
                col.var = "red")
```

```{r, fviz_pca_}
# Eigenvalues
eig.val <- get_eigenvalue(res.pca)
eig.val
  
# Results for Variables
res.var <- get_pca_var(res.pca)
res.var$coord          # Coordinates
res.var$contrib        # Contributions to the PCs
res.var$cos2           # Quality of representation 
# Results for individuals
res.ind <- get_pca_ind(res.pca)
res.ind$coord          # Coordinates
res.ind$contrib        # Contributions to the PCs
res.ind$cos2           # Quality of representation 
```

# Testing on the remaining data

```{r, test}
# Data for the supplementary individuals
ind.sup <- decathlon2[24:27, 1:10]
ind.sup[, 1:5]
ind.sup.coord <- predict(res.pca, newdata = ind.sup)
ind.sup.coord[, 1:4]
# Plot of active individuals
p <- fviz_pca_ind(res.pca, repel = TRUE)
p
# Add supplementary individuals
fviz_add(p, ind.sup.coord, color ="blue", repel = T)
```

```{r, grouping}
groups <- as.factor(decathlon2$Competition[1:23])
fviz_pca_ind(res.pca,
             col.ind = groups, # color by groups
             palette = c("#00AFBB",  "#FC4E07"),
             addEllipses = T, # Concentration ellipses
             ellipse.type = "confidence",
             legend.title = "Groups",
             repel = T)
```

# PCA from Towardsdatascience
URL: https://towardsdatascience.com/principal-component-analysis-pca-101-using-r-361f4c53a9ff

#### Tumor classification

For this article we’ll be using the Breast Cancer Wisconsin data set from the UCI Machine learning repo as our data. 


```{r, data}
wdbc <- read.csv(url('http://archive.ics.uci.edu/ml/machine-learning-databases/breast-cancer-wisconsin/wdbc.data'), header = F)
features <- c("radius", "texture", "perimeter", "area", 
              "smoothness", "compactness", "concavity", 
              "concave_points", "symmetry", "fractal_dimension")
names(wdbc) <- c("id", "diagnosis", 
                 paste0(features,"_mean"), 
                 paste0(features,"_se"), 
                 paste0(features,"_worst"))
```

#### Why PCA?

Right, so now we’ve loaded our data and find ourselves with 30 variables (thus excluding our response “diagnosis” and the irrelevant ID-variable). Now some of you might be saying “30 variable is a lot” and some might say “Pfft.. Only 30? I’ve worked with THOUSANDS!!” but rest assured that this is equally applicable in either scenario..!

There’s a few pretty good reasons to use PCA. The plot at the very beginning af the article is a great example of how one would plot multi-dimensional data by using PCA, we actually capture 63.3% (Dim1 44.3% + Dim2 19%) of variance in the entire dataset by just using those two principal components, pretty good when taking into consideration that the original data consisted of 30 features which would be impossible to plot in any meaningful way.

A very powerful consideration is to acknowledge that we never specified a response variable or anything else in our PCA-plot indicating whether a tumor was “benign” or “malignant”. It simply turns out that when we try to describe variance in the data using the linear combinations of the PCA we find some pretty obvious clustering and separation between the “benign” and “malignant” tumors! This makes a great case for developing a classification model based on our features!

Another major “feature” (no pun intended) of PCA is that it can actually directly improve performance of your models.

#### PCA on our tumor data
So now we understand a bit about how PCA works and that should be enough for now. Lets actually try it out:

```{r, pca on tumor data}
wdbc.pr <- prcomp(wdbc[c(3:32)], center = TRUE, scale = TRUE)
summary(wdbc.pr)
```

So, how many components do we want? We obviously want to be able to explain as much variance as possible but to do that we would need all 30 components, at the same time we want to reduce the number of dimensions so we definitely want less than 30!

Since we standardized our data and we now have the corresponding eigenvalues of each PC we can actually use these to draw a boundary for us. Since an eigenvalues <1 would mean that the component actually explains less than a single explanatory variable we would like to discard those. If our data is well suited for PCA we should be able to discard these components while retaining at least 70–80% of cumulative variance. Lets plot and see:

```{r, component number}
screeplot(wdbc.pr,
          type = "l",
          npcs = 15,
          main = "Screeplot of the first 15 PCs")
abline(h = 1, col = "red", lty = 5)
legend("topright", legend = c("Eigenvalue = 1"), col = "red",
       lty = 5, cex = 0.8)
cumpro <- cumsum(wdbc.pr$sdev ^ 2 / sum(wdbc.pr$sdev ^ 2))
plot(cumpro[0:15],
     xlab = "PC #",
     ylab = "Amount of explained variance",
     main = "Cumulative variance plot")
abline(v = 6, col = "blue", lty = 5)
abline(h = 0.88759, col = "blue", lty = 5)
legend( "topleft", legend = c("Cut-off @ PC6"),
        col = c("blue"),lty = 5, cex = 0.6)
```

We notice is that the first 6 components has an Eigenvalue >1 and explains almost 90% of variance, this is great! We can effectively reduce dimensionality from 30 to 6 while only “loosing” about 10% of variance!

We also notice that we can actually explain more than 60% of variance with just the first two components. Let’s try plotting these:

```{r, plot}
plot(wdbc.pr$x[,1],wdbc.pr$x[,2], xlab="PC1 (44.3%)", ylab = "PC2 (19%)", main = "PC1 / PC2 - plot")
```

Alright, this isn’t really too telling but consider for a moment that this is representing 60%+ of variance in a 30 dimensional dataset. But what do we see from this? There’s some clustering going on in the upper/middle-right. Lets also consider for a moment what the goal of this analysis actually is. We want to explain difference between malignant and benign tumors. Let’s actually add the response variable (diagnosis) to the plot and see if we can make better sense of it:

```{r, factoextra tumor }
library("factoextra")
fviz_pca_ind(wdbc.pr, geom.ind = "point", pointshape = 21, 
             pointsize = 2, fill.ind = wdbc$diagnosis,
             col.ind = "black", palette = "jco", addEllipses = TRUE,
             label = "var", col.var = "black", repel = TRUE,
             legend.title = "Diagnosis") +
  ggtitle("2D PCA-plot from 30 feature dataset") +
  theme(plot.title = element_text(hjust = 0.5))
```

# Dimensionality-Reduction-in-R by DataCamp

```{r, intro}
# install.packages("ggcorrplot")
library(ggcorrplot)
data(mtcars)
mtcars$cyl <- as.numeric(as.character(mtcars$cyl))
cor_mtcars <- cor(mtcars, use = "complete.obs")
head(round(cor_mtcars, 2))
ggcorrplot(cor_mtcars)
```

#### Exploring multivariate data
We've loaded a data frame called cars into your workspace. Go ahead and explore it! It includes features of a big range of brands of cars from 2004. In this exercise, you will explore the dataset and attempt to draw useful conclusions from the correlation matrix. Recall that correlation reveals feature resemblance and it will help us infer how cars are related to each other based on their features' values. To this end, you will discover how difficult it is to trace patterns based solely on the correlation structure.

```{r}
# Explore cars with summary()
str(mtcars)
# Get the correlation matrix with cor()
correl <- cor(mtcars[,c(1:7,10,11)], use = "complete.obs")
# Use ggcorrplot() to explore the correlation matrix
ggcorrplot(correl)
# Conduct hierarchical clustering on the correlation matrix
ggcorrplot_clustered <- ggcorrplot(correl, hc.order = TRUE, type = "lower")
ggcorrplot_clustered
```

```{r, FactoMineR}
# install.packages("FactoMineR")
library(FactoMineR)
str(mtcars)
mtcars_pca <- PCA(mtcars[,c(1:7, 10,11)], ncp = 6)
# Names of variables within mtcars_pca
names(mtcars_pca)
mtcars_pca$eig
# PC by quality of representation
mtcars_pca$var$cos2
#PC by contribution
mtcars_pca$var$contrib
# Most correlated values to PC1
dimdesc(mtcars_pca)
```
####PCA with FactoMineR
As you saw in the video, FactoMineR is a very useful package, rich in functionality, that implements a number of dimensionality reduction methods. Its function for doing PCA is PCA() - easy to remember! Recall that PCA(), by default, generates 2 graphs and extracts the first 5 PCs. __You can use the ncp argument to manually set the number of dimensions to keep.__

You can also use the summary() function to get a quick overview of the indices of the first three principal components. Moreover, for extracting summaries of some of the rows in a dataset, you can specify the nbelements argument. You'll have a chance to practice all of these and more in this exercise!

As in the previous lesson, the cars dataset is available in your workspace.

```{r, factomineR work}
# Run a PCA for the 10 non-binary numeric variables of cars
# The graph argument is set to F to suppress the default graphical output PCA().
pca_output_ten_v <- PCA(mtcars[,c(1:7, 10,11)], ncp = 6, graph = FALSE)
pca_output_ten_v
# Get the summary of the first 100 cars.
summary(pca_output_ten_v, nbelements = 20)
# Get the variance of the first 3 new dimensions.
pca_output_ten_v$eig[,2][1:3]
# Get the cumulative variance.
pca_output_ten_v$eig[,3][1:3]
```
#### Exploring PCA()

PCA() provides great flexibility in its usage. You can choose to ignore some of the original variables or individuals in building a PCA model by supplying PCA() with the ind.sup argument for supplementary individuals and quanti.sup or quali.sup for quantitative and qualitative variables respectively. Supplementary individuals and variables are rows and variables of the original data ignored while building the model.

Your learning objectives in this exercise are:  
- To conduct PCA considering parts of a dataset
- To inspect the most correlated variables with a specified principal component
- To find the contribution of variables in the designation of the first 5 principal components

Go for it! The cars dataset is available in your workspace.

```{r}
# Run a PCA on cars by setting the first 8 variables as 
# quantitative supplementary variables and the last 2 
# variables as qualitative supplementary ones. The graph 
# argument is set to F for not displaying the default graphical output.
# Run a PCA with active and supplementary variables
# pca_output_all <- PCA(cars, quanti.sup = 1:8, quali.sup = 20:21, graph = FALSE)
# Get the most correlated variables
# inspect the most correlated variables with 
# the first two principal components by specifying the axes argument.
# dimdesc(pca_output_all, axes = 1:2)
# Run a PCA on the first 100 car categories
# pca_output_hundred <- PCA(cars, quanti.sup = 1:8, quali.sup = 20:21, ind.sup = 101:nrow(cars), graph = FALSE)
# Trace variable contributions in pca_output_hundred
# pca_output_hundred$var$contrib
```

#### PCA with ade4
Alright! Now that you've got some real hands-on experience with FactoMineR, let's have a look at ade4, a well-known and well-maintained R package with a large number of numerical methods for building and handling PCA models.

dudi.pca() is the main function that implements PCA for ade4 and by default, it is interactive: It lets the user insert the number of retained dimensions. For suppressing the interactive mode and inserting the number of axes within the dudi.pca() function, you need to set the scannf argument to FALSE and then use the nf argument for setting the number of axes to retain. So, let's put ade4 into practice and compare it with FactoMineR.

```{r}
# install.packages("ade4")
library(ade4)
# Run a PCA using the 10 non-binary numeric variables.
cars_pca <- dudi.pca(mtcars[,c(1:7, 10,11)], scannf = F, nf = 4)
# Explore the summary of cars_pca.
summary(cars_pca)
# Explore the summary of pca_output_ten_v.
pca_output_ten_v
```
#### Plotting cos2

You're getting the hang of PCA now! As Alex demonstrated in the video, an important index included in your PCA models is the squared cosine, abbreviated in FactoMineR and factoextra as cos2. This shows how accurate the representation of your variables or individuals on the PC plane is.

The factoextra package is excellent at handling PCA models built using FactoMineR. Here, you're going to explore the functionality of factoextra. You'll be using the pca_output_all object that you computed earlier and create plots based on its cos2. Visual aids are key to understanding cos2.

```{r}
# Create a factor map for the variables.
# fviz_pca_var(pca_output_all, select.var = list(cos2 = 0.7), repel = TRUE)
# Create a barplot for the variables with the highest cos2 in the 2nd PC.
# fviz_cos2(pca_output_all, choice = "var", axes = 2, top = 10)
```

#### Plotting contributions

In this exercise, you will be asked to prepare a number of plots to help you get a better feeling of the variables' contributions on the extracted principal components. It is important to keep in mind that the contributions of the variables essentially signify their importance for the construction of a given principal component.

```{r}
# Create a factor map for the top 5 variables with the highest contributions.
# fviz_pca_var(pca_output_all, select.var = list(contrib = 5), repel = TRUE)
# Create a factor map for the top 5 individuals with the highest contributions.
# fviz_pca_ind(pca_output_all, select.ind = list(contrib = 5), repel = TRUE)
# Create a barplot for the variables with the highest contributions to the 1st PC.
# fviz_contrib(pca_output_all, choice = "var", axes = 1, top = 5)
# Create a barplot for the variables with the highest contributions to the 2nd PC.
# fviz_contrib(pca_output_all, choice = "var", axes = 2, top = 5)
```

#### Biplots and their ellipsoids

As mentioned in the video, biplots are graphs that provide a compact way of summarizing the relationships between individuals, variables, and also between variables and individuals within the same plot. Moreover, ellipsoids can be added on top of a biplot and offer a much better overview of the biplot based on the groupings of variables and individuals.

In this exercise, your job is to create biplots and ellipsoids using factoextra's graphical utilities.

```{r}
# Create a biplot with no labels for all individuals with the geom argument.
# fviz_pca_biplot(pca_output_all)
# Create ellipsoids for wheeltype columns respectively.
# fviz_pca_ind(pca_output_all, habillage = cars$wheeltype, addEllipses = TRUE)
# Create the biplot with ellipsoids
# fviz_pca_biplot(pca_output_all, habillage = cars$wheeltype, addEllipses = TRUE, alpha.var = "cos2")
```

#### The Kaiser-Guttman rule and the Scree test

In the video, you saw the three most common methods that people utilize to decide the number of principal components to retain:

- Scree test (constructing the screeplot)
- The Kaiser-Guttman rule
- Parallel Analysis

Your task now is to apply all of them on the R's built-in airquality dataset!

```{r}
# Conduct a PCA on the airquality dataset
pca_air <- PCA(airquality)
# Apply the Kaiser-Guttman rule
summary(pca_air, ncp = 4)
# Perform the screeplot test
fviz_screeplot(pca_air, ncp = 5)
```

#### Parallel Analysis with paran()
In this exercise, you will use two R functions for conducting parallel analysis for PCA:

- paran() of the paran package and
- fa.parallel() of the psych package.

fa.parallel() has one advantage over the paran() function; it allows you to use more of your data while building the correlation matrix. On the other hand, paran() does not handle missing data and you should first exclude missing values before passing the data to the function. For checking out the suggested number of PCs to retain, fa.parallel()'s output object includes the attribute ncomp.

The built-in R dataset airquality, on which you will be doing your parallel analyses, describes daily air quality measurements in New York from May to September 1973 and includes missing values.

```{r}
# install.packages("paran")
# install.packages("psych")
library(paran)
library(psych)
# Subset the complete rows of airquality.
airquality_complete <- airquality[complete.cases(airquality), ]
# Conduct a parallel analysis with paran().
air_paran <- paran(airquality_complete, seed = 1)
# Check out air_paran's suggested number of PCs to retain.
air_paran$Retained
# Conduct a parallel analysis with fa.parallel().
air_fa_parallel <- fa.parallel(airquality)
# Check out air_fa_parallel's suggested number of PCs to retain.
air_fa_parallel$ncomp
```