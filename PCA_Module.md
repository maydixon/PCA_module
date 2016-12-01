# Module: Principal Component Analysis
Claire Hemingway, Sebastian Stockmaier, and May Dixon  
December 1, 2016  


    


## Preliminaries
- Install this packages in ***R***: {HSAUR} (the .rmd won't knit without this)

## Objectives
> In this module we will examine Principal Component Analysis, a method for analysing continuous multivariate datasets. We will learn what PCAs can be used for, will walk though a basic version by hand in R, and will finally run an analysis together using real world data. 
Key concepts: 
**Principle components**, 
**Eigenvectors**, 
**Eigenvalues**, 
**Loading**. 

## The basics  
*This module can be found at: https://github.com/maydixon/PCA_module*
   
**Principal Component Analysis** or **PCA** is a common  statistical technique that helps to describe relationships and structure in data without losing variation. PCA's are non-parametric, and they assume that high variances describe something interesting in the data. PCA's are good tools for : 

* Visualizing multivariate data

* Finding the correlations between variables in multivariate data

* Demonstrating the most important sources of variance in your data

* Reducing highly dimensional data into fewer, more managable or visualizable dimensions

* De-correlating variables

* Often an intermediate step used to reveal associations between variables
   
##### Conceptual underpinnings

PCA can be thought of as creating natural axes that fit the data and describe it. 

**[Drawing a toy version on the board]**

How we normally plot our data, with our variables as axes (Fig 1A), and the same data rotated so that PC1 and PC2 are the axes (Fig 1B):

<img src="img/reg.axis.png" width="420px"/>
<img src="img/PCA.axis.png" width="420px"/>

* the direction of the principal components (PCs) represent the eigenvectors
* the length of the PCs represent the eigenvalues
* the correlation between the original variables and the PCs represent the *loadings*: how much each PC is correlated with the original variable

Here is a neat animation showing how PC1 is fit, with the data projected onto possible eigenvectors. You can see that the best PC1 minimizes the distance between the points and the vector: 

 <img src="img/Q7HIP.gif" width="420px"/>
 

  
This can extend to many dimensions. 


##Real world uses:
You could predict the route that a species will evolve towards an adaptive peak, based on the existing variation in the population:   


<img src="img/adaptive_peak.png" width="420px"/>
<img src="img/Gmax_dolf.png" width="420px"/>     

> In: Schluter, D. Adaptive radiation along genetic lines of least resistance. *Evolution*. 1996, 50(5). 


Or, you can measure a bunch of different ecological traits in closely related species, and see whether any can be reliably used to distinguish between the species:  

<img src="img/PCA_overlapping_ecospace.png" width="420px"/>
<img src="img/PCA_seperate_eco_space.png" width="420px"/>   
   
> In: Courvreur T. P., Porter-Morgan H., Wieringa J. J., and Chatrou, L. W. Little ecological divergence associated with speciation in two African rain forest tree genera. *BMC Evolutionary Biology*. 2011, 11(296). 

  

   
So, now that we've worked thorugh a toy model, let's get into the math:

### Some Background Mathematics...


To understand what Principal Component Analysis (PCA) actually does, we need to remember some basics background mathematics. This might be quite simple, and a lot of what we have done in class will show up again, but it will help to understand the process of PCA.

Things you need to remember throughout this module are:

- Variance
- Covariance and Covariance Matrix
- Correlation Matrix
- Eigenvalues/ Eigenvectors



#### 1) Variance
Besides Standard deviation, variance is another measure of how spread out out a dataset is. In fact, it is almost identical to the standard deviation. It simply is the standard deviation squared. R has a build-in functon called var(), and we have derived Variance in earlier sections of this course. We will need this information to cover the next section, covariance.

<img src="img/Variance.jpg" width=260px/>


```r
> x1 <- c(0, 8, 12, 20)
> x2 <- c(8, 9, 11, 12)
> Results <- cbind(mean(x1), mean(x2), var(x1), var(x2))
> colnames(Results) <- c("Mean X1", "Mean X2", "Var X1", "Var X2")
> Results
```

```
##      Mean X1 Mean X2   Var X1   Var X2
## [1,]      10      10 69.33333 3.333333
```



#### 2) Covariance
SD and Variance are purely one-dimensional. Examples are for instance heights of all monkeys in a cage, or armlength of all students in this room. However throughout this course we have repeatedly seen datasets with more than one dimension. Usually the aim of our analysis is to describe relationships between those dimensions. For instance our dataset includes the height of all monkeys but also the arm lenght, and we want to find out how related those dimensions are. Covariance is a simple way to find out how dimensions vary from the mean with respect to each other. Again, R has a build in function cov().

<img src="img/covariance.jpg" width=260px/>


```r
> cov(x1, x2)
```

```
## [1] 14.66667
```

Recall that covariance is measured between two dimensions. If we have a dataset with more than two dimensions there is more than one covariance measurment that can be calculated. An example is a three dimensional dataset with dimensions x, y, z. In this scenario we can calculate cov(x,y), cov(x,z), cov(z,y). If we calculate all the possible combinations of covariance between the dimensions, and put them in a matrix, we obtain a covariance matrix (again we can simply use the cov() function for this).

<img src="img/covariance matrix.jpg" width=260px/>


```r
> x <- c(1, 3, 6, 4)
> y <- c(30, 20, 10, 18)
> z <- c(0.3, 0.7, 0.5, 0.4)
> M <- cbind(x, y, z)
> cov(M)
```

```
##             x           y           z
## x   4.3333333 -17.0000000  0.11666667
## y -17.0000000  67.6666667 -0.61666667
## z   0.1166667  -0.6166667  0.02916667
```
However, there are some problems with covariance: 

- Covariances are hard to compare! For instance if you calculate covariance of a set of heights and weights (in meters and kilograms, respectively), you will get a different covariance when you use other units (e.g you use the imperial system instead of the metric system). 

- Differences in scale will flaw the covariance matrix. For instance we have a variable in our dataset describing the body length of a whale, and one describing the diameter of its smallest tooth. Covariance between those two variables will be hard to interpret.

This is why we have a tool to normalize covariance matrices... 



#### 3) Correlation matrices

The correlation matrix divides the covariance matrix by something that represents the diversity and scale in the covariates ,and provides a new value, a correlation. This correlation lays between -1 and 1. E.g for the correlation between variable x and y, we divide its covariance by the square root of the variance of x times the variance of y. 

<img src="img/correlation.jpg" width=260px/>

Again, we luckily don't have to do this by hand. R has a build-in function cor(). 

```r
> cor(x, y)
```

```
## [1] -0.9927742
```

And again, if we perform this operation on more than two variables, cor() will also create a correlation matrix for you.


```r
> cor(M)
```

```
##            x          y          z
## x  1.0000000 -0.9927742  0.3281651
## y -0.9927742  1.0000000 -0.4389547
## z  0.3281651 -0.4389547  1.0000000
```




#### 4) Eigenvectors and Eigenvalues. 

Imagine we are doing Matrix multiplication. Specifically we are multiplying a matrix with a vector (or in math jargon, we "transform" a vector using a matrix). If the resulting vector is an integer of the original vector, we call the original vector an EIGENVECTOR. The number we have to multiply the original vector with to obtain the resulting vector, is called its EIGENVALUE. The matrix functions as a transforming element, "stretching" the vector without changing its direction. Sounds confusing, let's dive into vector space and see if we can make it easier to understand.

Non-eigenvector

<img src="img/Noneigenvector.jpg" width=400px/>

This shows that we if we take a random vector, transformation with the covariance/correlation matrix of the dataset will move this vector in both, direction and length. Interestingly, if we would now use the correlation matrix to transform the resulting vector again and again, the slope and direction would start to stabilize. 

And than there are those magical eigenvectors:

<img src="img/Eigenvector.jpg" width=400px/>

Transformation of this vector by the covariance matrix does not result in directional changes. Instead it only influences the length of the vector. You see that the resulting vector is an integer of the Eigenvector multiplied by a certain number (4 in our case). This is the Eigenvalue of the Eigenvector, and is pretty much a measure of data spread! Note that together, Eigenvectors and their corresponding Eigenvalue represent direction and spread that explain variance in our dataset!


Important things to remember about Eigenvectors

- Eigenvectors can only be found for square matrices.
- Given a n x n matrix there will usually be n eigenvectors. E.g. a 3 x 3 matrix has three Eigenvectors.
- Even if an Eigenvector is scaled before throwing it in in the transformation, it will still yield the same Eigenvalue! 
- All Eigenvectors are perpendicular. This means they are at right angles to each other, and we can express data in terms of them!

Well, how would one find those magical Eigenvectors? It is only easy(ish) if you have small matrix. With larger matrices, complicated iterative methods come into play, which are beyond of what we want to cover today --> Today we will use R, and its eigen() function (:




### Alright, with this knowledge we should be ready to go and learn a little bit more about PCA. PCA follows a fairly simple and straightforward recipe:





#### 1) Get some data (or in our case make some up).

We will only use two dimensions in this example, so we can plot the data at each step to visualize what the PCA does. Imagine we went to a bat cave and we measured eyesize and nostrile size of individuals from different species. If you examine the dataset, you will see that both columns (eyesize and nostril size) are in the same scale/unit. This is important because It will determine if we can use the covariance matrix, or if we have to calculate the correlation matrix in upcoming transformation steps (more later!). In this case we can use the covariance matrix for transformation.



```r
> # create a dataset
> Bat.eyesize <- c(2.5, 0.5, 2.2, 1.9, 3.1, 2.3, 2, 1, 1.5, 1.1)
> Bat.nostrils <- c(2.4, 0.7, 2.9, 2.2, 3, 2.7, 1.6, 1.1, 1.6, 0.9)
> Bat.species <- c("Desmodus", "Pipistrellus", "Desmodus", "Noctula", "Vampyrum", 
+     "Desmodus", "Noctula", "Pipistrellus", "Noctula", "Pipistrellus")
> Dataset <- cbind(Bat.species, Bat.eyesize, Bat.nostrils)
> Dataset <- as.data.frame(Dataset)
> Dataset$Bat.eyesize <- as.numeric(Bat.eyesize)
> Dataset$Bat.nostrils <- as.numeric(Bat.nostrils)
> head(Dataset)
```

```
##    Bat.species Bat.eyesize Bat.nostrils
## 1     Desmodus         2.5          2.4
## 2 Pipistrellus         0.5          0.7
## 3     Desmodus         2.2          2.9
## 4      Noctula         1.9          2.2
## 5     Vampyrum         3.1          3.0
## 6     Desmodus         2.3          2.7
```

```r
> # let's just plot it for fun
> library(ggplot2)
> p <- ggplot(Dataset, aes(x = Bat.eyesize, y = Bat.nostrils, color = as.factor(Bat.species)))
> p <- p + geom_point(size = 4)
> p <- p + labs(y = "Bat nostril size (cm)", x = "Bat eyesize (cm)")
> p <- p + theme_bw()
> p <- p + theme(legend.title = element_blank())
> p
```

![](img/unnamed-chunk-6-1.png)<!-- -->



#### 2) Centering of the data by substracting the mean

For a PCA to work properly, we have to substract the mean from each of the data dimensions. This will put the origin 0 in the center of our "data cloud", it will center it. We will add the adjusted values to the original dataset in the first part of the code, and then plot the data to see how things have changed.  


```r
> Dataset$Bat.eyesize.adjust <- (Dataset$Bat.eyesize - mean(Dataset$Bat.eyesize))
> Dataset$Bat.nostrils.adjust <- as.numeric((Dataset$Bat.nostrils - mean(Dataset$Bat.nostrils)))
> head(Dataset)
```

```
##    Bat.species Bat.eyesize Bat.nostrils Bat.eyesize.adjust
## 1     Desmodus         2.5          2.4               0.69
## 2 Pipistrellus         0.5          0.7              -1.31
## 3     Desmodus         2.2          2.9               0.39
## 4      Noctula         1.9          2.2               0.09
## 5     Vampyrum         3.1          3.0               1.29
## 6     Desmodus         2.3          2.7               0.49
##   Bat.nostrils.adjust
## 1                0.49
## 2               -1.21
## 3                0.99
## 4                0.29
## 5                1.09
## 6                0.79
```

```r
> p <- ggplot(Dataset, aes(x = Bat.eyesize.adjust, y = Bat.nostrils.adjust, color = Bat.species))
> p <- p + geom_point(size = 4)
> p <- p + coord_fixed()
> p <- p + labs(y = "Bat nostril size", x = "Bat eyesize")
> p <- p + theme_bw()
> p <- p + theme(legend.title = element_blank())
> p <- p + geom_vline(xintercept = 0)
> p <- p + geom_hline(yintercept = 0)
> p
```

![](img/unnamed-chunk-7-1.png)<!-- -->




#### 3) Calculate the covariance matrix. 

Again this is pretty simple using the cov() function in R. Since the data is two dimensional, the covariance matrix will be 2 x 2. Remember here that if our dataset includes different scales/units, we have to calculate the correlation matrix instead, which is defined by the cor() function. 


```r
> covM <- cov(Dataset[, c(2, 3)])  # Only picking the original values from the Dataset
> covM
```

```
##              Bat.eyesize Bat.nostrils
## Bat.eyesize    0.6165556    0.6154444
## Bat.nostrils   0.6154444    0.7165556
```

Note that the non-diagonal elements in the covariance matrix are positive, we should expect that both x and y increase together (which they actually do)!




#### 4) Calculation of Eigenvectors and Eigenvalues

Next, the PCA will calculate the Eigenvectors and Eigenvalues of our Covariance matrix. We can use the build-in function eigen() for this process. As said before, the calculations get very complicated. Especially once the dimensionality of the matrix increases. For visualization we will extract the Eigenvectors, and plot them as directional lines with our data (this will prove that the eigenvectors describe the direction of data spread).

(Since we have two points for each Eigenvector (origin (0,0) and its actual value), we can calculate the slope! It is simply the ratio of the Eigenvector. 



```r
> Eigen <- eigen(covM)
> Eigen$slopes[1] <- Eigen$vectors[1, 1]/Eigen$vectors[2, 1]
> Eigen$slopes[2] <- Eigen$vectors[1, 2]/Eigen$vectors[2, 2]
> head(Eigen)
```

```
## $values
## [1] 1.2840277 0.0490834
## 
## $vectors
##           [,1]       [,2]
## [1,] 0.6778734 -0.7351787
## [2,] 0.7351787  0.6778734
## 
## $slopes
## [1]  0.9220526 -1.0845368
```

```r
> p <- ggplot(Dataset, aes(x = Bat.eyesize.adjust, y = Bat.nostrils.adjust, color = Bat.species))
> p <- p + geom_point(size = 4)
> p <- p + coord_fixed()
> p <- p + labs(y = "Bat nostril size", x = "Bat eyesize")
> p <- p + theme_bw()
> p <- p + theme(legend.title = element_blank())
> p <- p + geom_vline(xintercept = 0)
> p <- p + geom_hline(yintercept = 0)
> p <- p + geom_abline(intercept = 0, slope = Eigen$slopes[1], colour = "green")
> p <- p + geom_abline(intercept = 0, slope = Eigen$slopes[2], colour = "red")
> p
```

![](img/unnamed-chunk-9-1.png)<!-- -->

If you look at the plot, you can see that the data has quite a strong pattern. As expected from the covariance matrix, the two variables do indeed increase together. The plot also shows the two Eigenvectors which are perpendicular to each other. One goes through the middle of the points, like drawing a line of best fit (green)! This Eigenvector is showing us the direction and spread that explains the biggest variance in our dataset. The second Eigenvector (red) gives us the other, less important direction/spread that explains the second biggest variance in our dataset. Since we have two dimensions, we have two eigenvectors. 




#### 5) Choosing principal components

The Eigenvector with the highest eigenvalue is also called the first principle component of the data set (PC 1). In our example it is represented by the green line. It is the most significant relationship between the data dimensions. Therefore in the next step the PCA orders the Eigenvectors by Eigenvalue from highest to lowest (again, remember the Eigenvalue is pretty much a measure of data spread), and we can decide to ignore components of lesser significance. This process is called creating a feature vector. If you leave components out, the final dataset will have less dimensions than the original one! There are several diagnostic tools for visual PC selection, and we will talk about one of them later. 

Below you will see how we picked both eigenvectors (PC 1 and 2) from our result derived with the eigen() function. 


```r
> feat.vec <- Eigen$vectors
> feat.vec
```

```
##           [,1]       [,2]
## [1,] 0.6778734 -0.7351787
## [2,] 0.7351787  0.6778734
```




#### 6) Deriving the new data set (PCA scores), projecting the data to the new dimensions 
This is the final step in the PCA, the calculation of our PCA scores. We have choosen the components that we wish to keep in our data, and formed a feature vectors. We now take the transpose of the feature vector and multiply it on the left of the original data set (also transposed).


<img src="img/PCA scores.jpg" width=400px/>



```r
> feat.vec.tr <- t(feat.vec)
> feat.vec.tr
```

```
##            [,1]      [,2]
## [1,]  0.6778734 0.7351787
## [2,] -0.7351787 0.6778734
```

```r
> row_data_adj <- t(Dataset[, 4:5])
> PCA_scores <- data.frame(t(feat.vec.tr %*% row_data_adj))
> names(PCA_scores) <- c("x", "y")
> PCA_scores
```

```
##              x           y
## 1   0.82797019 -0.17511531
## 2  -1.77758033  0.14285723
## 3   0.99219749  0.38437499
## 4   0.27421042  0.13041721
## 5   1.67580142 -0.20949846
## 6   0.91294910  0.17528244
## 7  -0.09910944 -0.34982470
## 8  -1.14457216  0.04641726
## 9  -0.43804614  0.01776463
## 10 -1.22382056 -0.16267529
```

```r
> Dataset <- cbind(Dataset, PCA_scores)
```

First, lets see what happens if we plot both eigenvectors (both principal components)
In this case we keep both dimensions.

```r
> p <- ggplot(Dataset, aes(x = x, y = y, color = as.factor(Bat.species)))
> p <- p + geom_point(size = 4)
> p <- p + labs(y = "Principal Component 2", x = "Principal Component 1")
> p <- p + theme_bw()
> p <- p + theme(legend.title = element_blank())
> p <- p + geom_vline(xintercept = 0, color = "red")
> p <- p + geom_hline(yintercept = 0, color = "green")
> p
```

![](img/unnamed-chunk-12-1.png)<!-- -->

Now, we can see what happens if we remove the second, less significant principal component. 

```r
> Dataset$m <- rep(0, 10)  # adding a column of zeros merely for plotting purposes. 
> head(Dataset)
```

```
##    Bat.species Bat.eyesize Bat.nostrils Bat.eyesize.adjust
## 1     Desmodus         2.5          2.4               0.69
## 2 Pipistrellus         0.5          0.7              -1.31
## 3     Desmodus         2.2          2.9               0.39
## 4      Noctula         1.9          2.2               0.09
## 5     Vampyrum         3.1          3.0               1.29
## 6     Desmodus         2.3          2.7               0.49
##   Bat.nostrils.adjust          x          y m
## 1                0.49  0.8279702 -0.1751153 0
## 2               -1.21 -1.7775803  0.1428572 0
## 3                0.99  0.9921975  0.3843750 0
## 4                0.29  0.2742104  0.1304172 0
## 5                1.09  1.6758014 -0.2094985 0
## 6                0.79  0.9129491  0.1752824 0
```

```r
> p <- ggplot(Dataset, aes(x = x, y = m, color = as.factor(Bat.species)))
> p <- p + geom_point(size = 4)
> p <- p + labs(x = "Principal Component 1")
> p <- p + theme_bw()
> p <- p + theme(legend.title = element_blank(), axis.title.y = element_blank(), 
+     axis.text.y = element_blank())
> p <- p + geom_hline(yintercept = 0, color = "green")
> p
```

![](img/unnamed-chunk-13-1.png)<!-- -->

The second plot is mainly eyecandy, it is just a number line because we want to demonstrate that we compressed the dataset from two to one dimension without loosing too much information. We got rid of one uneccessary dimension because the variance it contributed to the data was insignificant! Hooray, the PCA did its job! 

Now this was quite a hustle, and included a lot of steps. Luckily R has build-in functions for these kind of analysis:

Here is an example of a dataset in the HSAUR package in R (Handbook of Statistical Analysese Using R) of athletes from different countries that compete in a heptathlon. In this dataset we can see here that there are seven different events that each athlete is ranked on. 



```
## Loading required package: tools
```

```
##                     hurdles highjump  shot run200m longjump javelin
## Joyner-Kersee (USA)   12.69     1.86 15.80   22.56     7.27   45.66
## John (GDR)            12.85     1.80 16.23   23.65     6.71   42.56
## Behmer (GDR)          13.20     1.83 14.20   23.10     6.68   44.54
## Sablovskaite (URS)    13.61     1.80 15.23   23.92     6.25   42.78
## Choubenkova (URS)     13.51     1.74 14.76   23.93     6.32   47.46
## Schulz (GDR)          13.75     1.83 13.50   24.65     6.33   42.82
## Fleming (AUS)         13.38     1.80 12.88   23.59     6.37   40.28
## Greiner (USA)         13.55     1.80 14.13   24.48     6.47   38.00
## Lajbnerova (CZE)      13.63     1.83 14.28   24.86     6.11   42.20
## Bouraga (URS)         13.25     1.77 12.62   23.59     6.28   39.06
##                     run800m score
## Joyner-Kersee (USA)  128.51  7291
## John (GDR)           126.12  6897
## Behmer (GDR)         124.20  6858
## Sablovskaite (URS)   132.24  6540
## Choubenkova (URS)    127.90  6540
## Schulz (GDR)         125.79  6411
## Fleming (AUS)        132.54  6351
## Greiner (USA)        133.65  6297
## Lajbnerova (CZE)     136.05  6252
## Bouraga (URS)        134.74  6252
```

When doing multivariate analyses, we want to know whether any of our variables behave in similar ways. From this we can figure out what processes govern the distribution of these samples. 

For this example, we want to know whether being good in any one event is a predictor for how you will do in the other events. With this data, we can first look at pairwaise scatter plot to get an overall idea of how the variables are correlated with one another. 


```r
> hep <- heptathlon[, -8]  #remove the scores from the dataset in column 8
> pairs(hep)
```

![](img/unnamed-chunk-15-1.png)<!-- -->

We can see that most of these variable are highly correlated to one another and thats when we want to do PCA to try and reduce the dimensionality. Before we can run the PCA, we must choose whether we want to use the correlation matrix or the covariance matrix. The covariance matrix centers each variable on the mean but the scale of the variable still matters. Variables that originally had really high varaiance will control the resulting PCA plot much more strongly. Variables that had tiny variance will be unimportant. *Should only use the covariance matrix when all of the variables are in comparable units and the differences between their original variances are meaningful for interpretation. The correlation matrix uses the correlation coefficents. This adjusts each variable so it is centered around the mean and standardized to a variance of 1. 

In this example, we can see that some of the events, such as high jump, have average value of 1.8, whereas the 800m run has an average of 120. Because of this, it makes sense to use the correlation matrix. This is the more common assocation matrix for PCA and differentiating between the two can be important for interpretation (we can run both in this example to show the difference).



SO, let's run the PCA:

In the base package in R
There are two base functions in R for running a PCA: 
there is prcomp() and princomp() that more or less give you the same thing

Here is an example with prcomp:

```r
> PC.hep.cor <- prcomp(hep, scale = TRUE)  #using the correlation matrix
> print(PC.hep.cor)
```

```
## Standard deviations:
## [1] 2.1119364 1.0928497 0.7218131 0.6761411 0.4952441 0.2701029 0.2213617
## 
## Rotation:
##                 PC1         PC2         PC3         PC4         PC5
## hurdles   0.4528710 -0.15792058 -0.04514996  0.02653873 -0.09494792
## highjump -0.3771992  0.24807386  0.36777902 -0.67999172 -0.01879888
## shot     -0.3630725 -0.28940743 -0.67618919 -0.12431725 -0.51165201
## run200m   0.4078950  0.26038545  0.08359211 -0.36106580 -0.64983404
## longjump -0.4562318  0.05587394 -0.13931653 -0.11129249  0.18429810
## javelin  -0.0754090 -0.84169212  0.47156016 -0.12079924 -0.13510669
## run800m   0.3749594 -0.22448984 -0.39585671 -0.60341130  0.50432116
##                  PC6         PC7
## hurdles  -0.78334101 -0.38024707
## highjump -0.09939981 -0.43393114
## shot      0.05085983 -0.21762491
## run200m   0.02495639  0.45338483
## longjump -0.59020972  0.61206388
## javelin   0.02724076  0.17294667
## run800m   0.15555520  0.09830963
```

```r
> # scale=TRUE bases the PCA on the corrlation matrix
> 
> PC.hep.cov <- prcomp(hep, scale = FALSE)  #using the covariance matrix
> print(PC.hep.cov)
```

```
## Standard deviations:
## [1] 8.3646430 3.5909752 1.3856976 0.5857131 0.3238168 0.1471221 0.0332496
## 
## Rotation:
##                   PC1           PC2         PC3         PC4         PC5
## hurdles   0.069508692 -0.0094891417  0.22180829 -0.32737674 -0.80702932
## highjump -0.005569781  0.0005647147 -0.01451405  0.02123856  0.14013823
## shot     -0.077906090  0.1359282330 -0.88374045 -0.42500654 -0.10442207
## run200m   0.072967545 -0.1012004268  0.31005700 -0.81585220  0.46178680
## longjump -0.040369299  0.0148845034 -0.18494319  0.20419828  0.31899315
## javelin   0.006685584  0.9852954510  0.16021268 -0.03216907  0.04880388
## run800m   0.990994208  0.0127652701 -0.11655815  0.05827720  0.02784756
##                   PC6          PC7
## hurdles   0.424850883 -0.083123145
## highjump  0.098373568 -0.984881131
## shot     -0.051744802 -0.015649644
## run200m   0.082486244  0.051312974
## longjump  0.894592570  0.142110352
## javelin   0.006170438  0.005033005
## run800m  -0.002987043  0.001041451
```

```r
> # scale=FALSE bases the PCA on the covariance matrix
```

There can be as many principle components as there are variables, but the goal of this technique is to simplify! To figure out how many principal components to include in your analysis, a common technique is to use a scree plot. This show the proportion of variance explained by each component. There are several rules-of-thumb for figuring out how many components to include. Can either have some threshold of components required to explain a majority of the variance (typically ~80%) OR ignore any component that explains less than 1% of the variance. Most people just use the first two or three.


```r
> plot(PC.hep.cor)
```

![](img/unnamed-chunk-17-1.png)<!-- -->

```r
> summary(PC.hep.cor)  #proportion of variance is the eigenvalue for each PC. We can see that 95% of our data is explained with the first four, but 81% is explained.
```

```
## Importance of components:
##                           PC1    PC2     PC3     PC4     PC5     PC6
## Standard deviation     2.1119 1.0928 0.72181 0.67614 0.49524 0.27010
## Proportion of Variance 0.6372 0.1706 0.07443 0.06531 0.03504 0.01042
## Cumulative Proportion  0.6372 0.8078 0.88223 0.94754 0.98258 0.99300
##                           PC7
## Standard deviation     0.2214
## Proportion of Variance 0.0070
## Cumulative Proportion  1.0000
```


So, if we continue with the first two principal components, we can look how the data fall out. The black lines are each individual athlete and the red lines tell us which variable, or sporting event, is aligning on each component.

```r
> biplot(PC.hep.cor)
```

![](img/unnamed-chunk-18-1.png)<!-- -->

We can also look at the biplot for the covariance matrix to compare.

```r
> biplot(PC.hep.cov)
```

![](img/unnamed-chunk-19-1.png)<!-- -->


Loadings are the correlations between the original varibales and the axis. They indicate the direction and magnitude of the icnrease in that variable.The higher the component loading, the more important that variable is to the component. 

```r
> # here is the same example wtih princomp
> pc.hep.cor <- princomp(hep, scores = TRUE, cor = TRUE)
> # first argument is the dataset, second argument scores=TRUE which means
> # that it will take the data and transform it into the reduced space; the
> # third variable specifies to use the correlation matrix and not the
> # covariance matrix to do this (same as scale=TRUE in the other function.)
> 
> summary(pc.hep.cor)
```

```
## Importance of components:
##                           Comp.1    Comp.2     Comp.3     Comp.4
## Standard deviation     2.1119364 1.0928497 0.72181309 0.67614113
## Proportion of Variance 0.6371822 0.1706172 0.07443059 0.06530955
## Cumulative Proportion  0.6371822 0.8077994 0.88222998 0.94753952
##                            Comp.5     Comp.6      Comp.7
## Standard deviation     0.49524412 0.27010291 0.221361710
## Proportion of Variance 0.03503811 0.01042223 0.007000144
## Cumulative Proportion  0.98257763 0.99299986 1.000000000
```

```r
> summary(PC.hep.cor)
```

```
## Importance of components:
##                           PC1    PC2     PC3     PC4     PC5     PC6
## Standard deviation     2.1119 1.0928 0.72181 0.67614 0.49524 0.27010
## Proportion of Variance 0.6372 0.1706 0.07443 0.06531 0.03504 0.01042
## Cumulative Proportion  0.6372 0.8078 0.88223 0.94754 0.98258 0.99300
##                           PC7
## Standard deviation     0.2214
## Proportion of Variance 0.0070
## Cumulative Proportion  1.0000
```

```r
> # you can see that the outputs for these are almost idential. i.e. it doesnt
> # matter which function in the base R package you use.
> 
> pc.hep.cor$loadings
```

```
## 
## Loadings:
##          Comp.1 Comp.2 Comp.3 Comp.4 Comp.5 Comp.6 Comp.7
## hurdles   0.453 -0.158                       0.783 -0.380
## highjump -0.377  0.248 -0.368  0.680               -0.434
## shot     -0.363 -0.289  0.676  0.124 -0.512        -0.218
## run200m   0.408  0.260         0.361 -0.650         0.453
## longjump -0.456         0.139  0.111  0.184  0.590  0.612
## javelin         -0.842 -0.472  0.121 -0.135         0.173
## run800m   0.375 -0.224  0.396  0.603  0.504 -0.156       
## 
##                Comp.1 Comp.2 Comp.3 Comp.4 Comp.5 Comp.6 Comp.7
## SS loadings     1.000  1.000  1.000  1.000  1.000  1.000  1.000
## Proportion Var  0.143  0.143  0.143  0.143  0.143  0.143  0.143
## Cumulative Var  0.143  0.286  0.429  0.571  0.714  0.857  1.000
```

Here is another example using the base package in R:   

```r
> pottery
```

```
##    Al2O3 Fe2O3  MgO  CaO Na2O  K2O TiO2   MnO   BaO
## 1   18.8  9.52 2.00 0.79 0.40 3.20 1.01 0.077 0.015
## 2   16.9  7.33 1.65 0.84 0.40 3.05 0.99 0.067 0.018
## 3   18.2  7.64 1.82 0.77 0.40 3.07 0.98 0.087 0.014
## 4   16.9  7.29 1.56 0.76 0.40 3.05 1.00 0.063 0.019
## 5   17.8  7.24 1.83 0.92 0.43 3.12 0.93 0.061 0.019
## 6   18.8  7.45 2.06 0.87 0.25 3.26 0.98 0.072 0.017
## 7   16.5  7.05 1.81 1.73 0.33 3.20 0.95 0.066 0.019
## 8   18.0  7.42 2.06 1.00 0.28 3.37 0.96 0.072 0.017
## 9   15.8  7.15 1.62 0.71 0.38 3.25 0.93 0.062 0.017
## 10  14.6  6.87 1.67 0.76 0.33 3.06 0.91 0.055 0.012
## 11  13.7  5.83 1.50 0.66 0.13 2.25 0.75 0.034 0.012
## 12  14.6  6.76 1.63 1.48 0.20 3.02 0.87 0.055 0.016
## 13  14.8  7.07 1.62 1.44 0.24 3.03 0.86 0.080 0.016
## 14  17.1  7.79 1.99 0.83 0.46 3.13 0.93 0.090 0.020
## 15  16.8  7.86 1.86 0.84 0.46 2.93 0.94 0.094 0.020
## 16  15.8  7.65 1.94 0.81 0.83 3.33 0.96 0.112 0.019
## 17  18.6  7.85 2.33 0.87 0.38 3.17 0.98 0.081 0.018
## 18  16.9  7.87 1.83 1.31 0.53 3.09 0.95 0.092 0.023
## 19  18.9  7.58 2.05 0.83 0.13 3.29 0.98 0.072 0.015
## 20  18.0  7.50 1.94 0.69 0.12 3.14 0.93 0.035 0.017
## 21  17.8  7.28 1.92 0.81 0.18 3.15 0.90 0.067 0.017
## 22  14.4  7.00 4.30 0.15 0.51 4.25 0.79 0.160 0.019
## 23  13.8  7.08 3.43 0.12 0.17 4.14 0.77 0.144 0.020
## 24  14.6  7.09 3.88 0.13 0.20 4.36 0.81 0.124 0.019
## 25  11.5  6.37 5.64 0.16 0.14 3.89 0.69 0.087 0.009
## 26  13.8  7.06 5.34 0.20 0.20 4.31 0.71 0.101 0.021
## 27  10.9  6.26 3.47 0.17 0.22 3.40 0.66 0.109 0.010
## 28  10.1  4.26 4.26 0.20 0.18 3.32 0.59 0.149 0.017
## 29  11.6  5.78 5.91 0.18 0.16 3.70 0.65 0.082 0.015
## 30  11.1  5.49 4.52 0.29 0.30 4.03 0.63 0.080 0.016
## 31  13.4  6.92 7.23 0.28 0.20 4.54 0.69 0.163 0.017
## 32  12.4  6.13 5.69 0.22 0.54 4.65 0.70 0.159 0.015
## 33  13.1  6.64 5.51 0.31 0.24 4.89 0.72 0.094 0.017
## 34  11.6  5.39 3.77 0.29 0.06 4.51 0.56 0.110 0.015
## 35  11.8  5.44 3.94 0.30 0.04 4.64 0.59 0.085 0.013
## 36  18.3  1.28 0.67 0.03 0.03 1.96 0.65 0.001 0.014
## 37  15.8  2.39 0.63 0.01 0.04 1.94 1.29 0.001 0.014
## 38  18.0  1.50 0.67 0.01 0.06 2.11 0.92 0.001 0.016
## 39  18.0  1.88 0.68 0.01 0.04 2.00 1.11 0.006 0.022
## 40  20.8  1.51 0.72 0.07 0.10 2.37 1.26 0.002 0.016
## 41  17.7  1.12 0.56 0.06 0.06 2.06 0.79 0.001 0.013
## 42  18.3  1.14 0.67 0.06 0.05 2.11 0.89 0.006 0.019
## 43  16.7  0.92 0.53 0.01 0.05 1.76 0.91 0.004 0.013
## 44  14.8  2.74 0.67 0.03 0.05 2.15 1.34 0.003 0.015
## 45  19.1  1.64 0.60 0.10 0.03 1.75 1.04 0.007 0.018
```

```r
> pairs(pottery)
```

![](img/unnamed-chunk-21-1.png)<!-- -->

```r
> pot.pc <- princomp(pottery, scores = TRUE, cor = TRUE)
> 
> pot.pc$loadings
```

```
## 
## Loadings:
##       Comp.1 Comp.2 Comp.3 Comp.4 Comp.5 Comp.6 Comp.7 Comp.8 Comp.9
## Al2O3  0.348  0.328 -0.119        -0.322  0.776        -0.219       
## Fe2O3 -0.327  0.395  0.264        -0.343         0.244  0.504  0.482
## MgO   -0.435 -0.190 -0.151        -0.281        -0.442 -0.492  0.483
## CaO           0.501  0.478  0.497        -0.227 -0.171 -0.393 -0.170
## Na2O  -0.217  0.456        -0.578  0.530  0.158 -0.321              
## K2O   -0.456        -0.102        -0.389        -0.310  0.286 -0.667
## TiO2   0.340  0.301        -0.492 -0.493 -0.521        -0.147       
## MnO   -0.455        -0.140 -0.153                0.718 -0.427 -0.202
## BaO           0.378 -0.792  0.385  0.136 -0.199         0.114  0.103
## 
##                Comp.1 Comp.2 Comp.3 Comp.4 Comp.5 Comp.6 Comp.7 Comp.8
## SS loadings     1.000  1.000  1.000  1.000  1.000  1.000  1.000  1.000
## Proportion Var  0.111  0.111  0.111  0.111  0.111  0.111  0.111  0.111
## Cumulative Var  0.111  0.222  0.333  0.444  0.556  0.667  0.778  0.889
##                Comp.9
## SS loadings     1.000
## Proportion Var  0.111
## Cumulative Var  1.000
```

```r
> pot.pc$scores
```

```
##          Comp.1      Comp.2      Comp.3      Comp.4      Comp.5
## 1  -0.023849217  1.81662699  0.94273718 -0.70579260 -0.75732367
## 2   0.233231791  1.64956650  0.12094476 -0.21184570  0.11041602
## 3   0.125604739  1.27084610  1.02397180 -0.83369012 -0.28531707
## 4   0.325076863  1.70125087 -0.22210249 -0.18191939  0.13500145
## 5   0.169110844  1.90592478 -0.08711105  0.10355407  0.26519962
## 6   0.365747598  1.36283311  0.27261725  0.21721046 -0.76237729
## 7  -0.016035943  2.40398083  0.77431105  1.24102746  0.18539056
## 8   0.112255644  1.44608218  0.44117091  0.31206974 -0.54677478
## 9  -0.002112492  1.04766880  0.30304586 -0.24455074  0.21964842
## 10  0.086712589  0.09044388  1.76871681 -0.62815145  0.16371509
## 11  0.754864487 -1.09521799  1.83602879  0.37511269  0.64030246
## 12  0.089033002  0.99046307  1.47205721  1.22564436  0.21807731
## 13 -0.239033580  1.15650079  1.38561142  1.00149168  0.30009721
## 14 -0.359144925  2.05392583 -0.46089843 -0.06888727  0.37571206
## 15 -0.288474022  2.08307379 -0.41067019 -0.10726942  0.49094655
## 16 -1.210690329  2.77033022 -0.25826701 -1.59776479  1.46014441
## 17  0.010278219  1.85833215  0.02269332 -0.11978381 -0.35717665
## 18 -0.489413297  3.20025482 -0.73223968  0.56359517  0.80013668
## 19  0.513354259  0.78572014  0.76941984  0.30947264 -1.26451232
## 20  0.804075246  0.59746886  0.27943726  0.69246298 -0.89581839
## 21  0.344614840  0.83710547  0.31266185  0.60221427 -0.54404403
## 22 -2.705686670  0.45056482 -1.36105499 -1.02401726  0.34357317
## 23 -1.979799318 -0.35949920 -1.49126871  0.31024152 -0.33670167
## 24 -1.868347887 -0.32386198 -1.27111984  0.05509184 -0.68410558
## 25 -2.096067202 -2.68510221  1.57617743 -0.68998757 -0.59745592
## 26 -2.297408912 -0.46451435 -1.70304511  0.68732017 -0.42971065
## 27 -1.726625223 -2.19028336  1.53394922 -0.76460906  0.45292023
## 28 -2.238305683 -1.92026850 -0.64700970  0.31736103  1.16502636
## 29 -2.059785179 -2.02958384 -0.04782880  0.15546784 -0.03326139
## 30 -2.126136759 -1.41798805 -0.10568540  0.05979381  0.68213974
## 31 -3.565986728 -1.09053990 -0.91306796  0.04953716 -0.86266656
## 32 -3.598011474 -0.61285141 -0.34107340 -1.37406032  0.46499775
## 33 -2.639540462 -0.93504360 -0.59509035  0.15983432 -0.65771359
## 34 -2.244827567 -2.11143857  0.07179194  0.86138761 -0.02241796
## 35 -1.999772172 -2.39388275  0.64517591  0.68064098 -0.38709859
## 36  2.680313560 -2.12563210  0.17243596  0.68617183  0.96836140
## 37  3.437918034 -1.15676918  0.06973265 -1.16186102 -0.62228963
## 38  2.999718331 -1.36050554 -0.50093236  0.08489148  0.33406899
## 39  3.305745592 -0.24689465 -2.16861926  0.38930979  0.01944097
## 40  3.792720785 -0.27922334 -0.76926592 -0.87489640 -0.94572423
## 41  2.830656350 -1.99636216  0.40992112  0.10200277  0.69632904
## 42  2.968027398 -1.00988186 -1.30689077  0.63163669  0.55085532
## 43  3.117340811 -2.01535101  0.34802488 -0.29040299  0.61999169
## 44  3.190263869 -0.96538315 -0.15378810 -1.18888871 -0.71425217
## 45  3.518390190 -0.69288529 -1.00560490  0.19383423  0.04424965
##         Comp.6        Comp.7      Comp.8      Comp.9
## 1   0.68626602  0.2081277433  0.25305088  0.31551244
## 2  -0.10253317 -0.0573068042  0.17450339  0.05572727
## 3   0.63871698  0.2854493623 -0.17745272 -0.06163149
## 4  -0.16426980 -0.0785010172  0.32898738  0.10007837
## 5   0.25315922 -0.3290127220  0.13490882  0.07926497
## 6   0.42592957  0.1081576336 -0.14279591 -0.04006770
## 7  -0.67194619 -0.4136939511 -0.56345565 -0.33849067
## 8   0.22317273 -0.0336298469 -0.13594123 -0.16576388
## 9  -0.12138300 -0.1157318375  0.46900072 -0.06482262
## 10 -0.17482738 -0.0740758269  0.26158459 -0.08986550
## 11 -0.21472106  0.2481252003  0.25222250  0.53783646
## 12 -0.81250254 -0.1297963673 -0.24037951 -0.18823209
## 13 -0.63595617  0.2301986642 -0.36218992 -0.23223574
## 14  0.09775898  0.1086008602  0.12392293  0.17530674
## 15 -0.03752362  0.2825385576  0.07882795  0.28862256
## 16  0.07972234 -0.2755488317  0.12798275 -0.19168747
## 17  0.42700900  0.0087616865 -0.16070474  0.17063446
## 18 -0.40135147 -0.1310219898 -0.13266740  0.07714383
## 19  0.50771080  0.3628086294 -0.18290771 -0.08218185
## 20  0.26462629 -0.0737612006  0.43474876  0.31167392
## 21  0.31795697  0.2489829151  0.05309277  0.07911381
## 22  0.35475690  0.3080950020 -0.06293320 -0.20788783
## 23 -0.14750496  0.9588516527  0.35721667 -0.17427023
## 24  0.05880935  0.3964764005  0.35157146 -0.20546829
## 25  0.02720737 -0.3256099849 -0.19421576  0.41968177
## 26 -0.07225150 -0.3547274236  0.28784631  0.44171371
## 27 -0.09630368  0.5885557976  0.15718314  0.12534109
## 28 -0.64511853  0.8419007129 -0.53204468  0.11403119
## 29 -0.25565868 -0.5563410691 -0.17100752  0.76711689
## 30 -0.32413594 -0.6829137716  0.32993852  0.05991136
## 31  0.19356146  0.0294363354 -0.95087238  0.33507768
## 32  0.32556493 -0.3324648787 -0.46878771 -0.40635837
## 33 -0.01551464 -0.8404558130  0.22019802 -0.20672423
## 34 -0.05062438  0.2330139855  0.32583172 -0.65816688
## 35  0.01892630 -0.1942835730  0.43061914 -0.68328069
## 36  1.35399012 -0.0320479989  0.03949824  0.07024779
## 37 -1.20837070  0.0849537367 -0.02438921  0.01214938
## 38  0.39652544 -0.1347706144  0.04050084 -0.06313377
## 39 -0.57952149  0.0003147358  0.10377051  0.19788890
## 40  0.24503306 -0.3514549948 -0.44628395 -0.49199532
## 41  0.85440287 -0.1152162096 -0.05284106 -0.18568308
## 42  0.33374498 -0.1207903123 -0.01257694 -0.05904830
## 43  0.19992468  0.0688253478 -0.19183494 -0.03895490
## 44 -1.68500666  0.0353057714  0.15535658 -0.06717632
## 45  0.13254915  0.1156763083 -0.28608244  0.16905264
```

```r
> plot(pot.pc)
```

![](img/unnamed-chunk-21-2.png)<!-- -->

```r
> biplot(pot.pc)
```

![](img/unnamed-chunk-21-3.png)<!-- -->



      Some further helpful resources:
      3D visualization tool:
      http://setosa.io/ev/principal-component-analysis/
      http://stats.stackexchange.com/questions/2691/making-sense-of-principal-component-analysis-eigenvectors-eigenvalues
      
      Image credits: 
      http://www.sthda.com/english/wiki/principal-component-analysis-the-basics-you-should-read-r-software-and-data-mining
      Kirkpatrick, Mike. Population Genetics Lecture QT#2
      stackoverflow user "amoeba": http://stats.stackexchange.com/questions/2691/making-sense-of-principal-component-analysis-eigenvectors-eigenvalues
    
    
