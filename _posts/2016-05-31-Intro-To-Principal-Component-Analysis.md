---
layout: post
title:  "Introduction to Principal Component Analysis (PCA) in Unsupervised Learning Settings"
date: 2016-05-31
excerpt: "Principal Component Analysis (PCA) is a popular method used in statistical learning approaches."
image: "/images/pca1.jpg"
permalink: /blog/2016/05/31/Intro-To-Principal-Component-Analysis
---





Principal Component Analysis (PCA) is a popular method used in statistical learning approaches. PCA can be used to achieve dimensionality reduction in regression settings allowing us to explain a high-dimensional dataset with a smaller number of representative variables which, in combination, describe most of the variability found in the original high-dimensional data.

PCA can also be used in unsupervised learning problems to discover, visualise an explore patterns in high-dimensional datasets when there is not specific response variable. Additionally, PCA can aid in clustering exercises and segmentation models.





In this article, we're going to look at PCA as a tool for unsupervised learning, data exploration and visualisation (remembering that visualisation is a key step in the exploratory data analysis process). 

In high-dimensional datasets, the ability to visualise patterns among all variables is challenging. One way we can achieve this is by generating two-dimensional scatterplots containing all data points on two of all possible features. However, this technique does not help much when the number of features / variables is large. Consider, for example, the Motor Trend Car Road Test dataset from the 1974 Motor Trend US magazine (this dataset can be found in the `library(datasets)` package):


```r
# Load necessary libraries
library(datasets)
library(ggplot2)
library(FactoMineR)
library(scales)
library(rgl)
library(knitr)
library(scatterplot3d)

# only display the top 5 records of the mtcars dataset
dta = mtcars
head(dta)
```

```
##                    mpg cyl disp  hp drat    wt  qsec vs am gear carb
## Mazda RX4         21.0   6  160 110 3.90 2.620 16.46  0  1    4    4
## Mazda RX4 Wag     21.0   6  160 110 3.90 2.875 17.02  0  1    4    4
## Datsun 710        22.8   4  108  93 3.85 2.320 18.61  1  1    4    1
## Hornet 4 Drive    21.4   6  258 110 3.08 3.215 19.44  1  0    3    1
## Hornet Sportabout 18.7   8  360 175 3.15 3.440 17.02  0  0    3    2
## Valiant           18.1   6  225 105 2.76 3.460 20.22  1  0    3    1
```

Note that this specific dataset example is only illustrative and is not really a highly-dimensional dataset (cases when the number of features is larger than the number of data points) but it's an easier example to assimilate the concept.

The **mtcars** dataset contains 32 observations each representing a specific car model and 11 variables each with a different measure derived from the road test or from the properties of the car.





Even in this basic example, visualising patterns among all variables in the data by running pairwise scatterplots is a challenging endeavour:



```r
# Plot dataset
plot(dta, main="Pairwise Scatterplot of all variables in the mtcars dataset",
     col="blue", cex=0.3, pch=16)
```

<span class="image fit"><img src="{{ "/images/PCA_files/figure-html/unnamed-chunk-2-1.png" | absolute_url }}" alt="" /></span>


By trying to visualise our dataset of cars (with 11 variables) through the pairwise scatterplot method, we end up having to look through more than 50 individual scatterplots! Now imagine having a dataset with a very large number of variables (hundreds) and trying to visualise and make sense of all of this!

That's where PCA comes in quite handy.

PCA is a method that allows you to find low-dimensional representations of your data that can explain most of the variation and captures as much information about the dataset as possible. These low-dimensional representations are, essentially, called **Principal Components** and end up, simplistically speaking, taking the form of new variables. PCA can also be interpreted as low-dimensional linear surfaces that are as closest as possible to each individual observation in a dataset.

PCA computes the first principal component of a set of features / variables by a normalized (**centered to mean zero**) linear combination of the features that have the largest variance, which after a few mathematical computations, arrives on scores (Z1) for the first principal component. The second principal component (Z2), again, is the linear combination of all features / variables across all data points that have a maximum variance but that are not related with Z1. By making the second principal component uncorrelated to Z1, we are forcing it to be orthogonal (or perpendicular to the direction in which Z1 is projected). The idea here being that if we were to project the PCAs onto a two-dimensional plane, the second PCA would be perpendicular to the first PCA (hence, capturing the variance across both dimensions). Obviously, in a dataset with more than two variables, we can find multiple principal components.





Let's get back to our example dataset of cars:

```r
# Look at the differences in the mean for each variable
summary(dta)
```

```
##       mpg             cyl             disp             hp       
##  Min.   :10.40   Min.   :4.000   Min.   : 71.1   Min.   : 52.0  
##  1st Qu.:15.43   1st Qu.:4.000   1st Qu.:120.8   1st Qu.: 96.5  
##  Median :19.20   Median :6.000   Median :196.3   Median :123.0  
##  Mean   :20.09   Mean   :6.188   Mean   :230.7   Mean   :146.7  
##  3rd Qu.:22.80   3rd Qu.:8.000   3rd Qu.:326.0   3rd Qu.:180.0  
##  Max.   :33.90   Max.   :8.000   Max.   :472.0   Max.   :335.0  
##       drat             wt             qsec             vs        
##  Min.   :2.760   Min.   :1.513   Min.   :14.50   Min.   :0.0000  
##  1st Qu.:3.080   1st Qu.:2.581   1st Qu.:16.89   1st Qu.:0.0000  
##  Median :3.695   Median :3.325   Median :17.71   Median :0.0000  
##  Mean   :3.597   Mean   :3.217   Mean   :17.85   Mean   :0.4375  
##  3rd Qu.:3.920   3rd Qu.:3.610   3rd Qu.:18.90   3rd Qu.:1.0000  
##  Max.   :4.930   Max.   :5.424   Max.   :22.90   Max.   :1.0000  
##        am              gear            carb      
##  Min.   :0.0000   Min.   :3.000   Min.   :1.000  
##  1st Qu.:0.0000   1st Qu.:3.000   1st Qu.:2.000  
##  Median :0.0000   Median :4.000   Median :2.000  
##  Mean   :0.4062   Mean   :3.688   Mean   :2.812  
##  3rd Qu.:1.0000   3rd Qu.:4.000   3rd Qu.:4.000  
##  Max.   :1.0000   Max.   :5.000   Max.   :8.000
```

Remember that one of the criteria for one to use PCA is to center variables to have mean zero. This is so Principal Components are not affected by the differences in scales found across all different variables in the data. For example, note how the mean for **disp** (displacement in cubic inches) is very different from the mean of say **carb** (number of carburettors) for example. If we do not scale our variables, the Principal Components in this case would be significantly influenced by the **disp** variable.

Let's run a Principal Component Analysis on the cars dataset:

```r
# Run Principal Component on the data
PC_res = PCA(dta, scale.unit=TRUE, ncp = dim(dta)[2], graph=FALSE)
summary(PC_res)
```

```
## 
## Call:
## PCA(X = dta, scale.unit = TRUE, ncp = dim(dta)[2], graph = FALSE) 
## 
## 
## Eigenvalues
##                        Dim.1   Dim.2   Dim.3   Dim.4   Dim.5   Dim.6
## Variance               6.608   2.650   0.627   0.270   0.223   0.212
## % of var.             60.076  24.095   5.702   2.451   2.031   1.924
## Cumulative % of var.  60.076  84.172  89.873  92.324  94.356  96.279
##                        Dim.7   Dim.8   Dim.9  Dim.10  Dim.11
## Variance               0.135   0.123   0.077   0.052   0.022
## % of var.              1.230   1.117   0.700   0.473   0.200
## Cumulative % of var.  97.509  98.626  99.327  99.800 100.000
## 
## Individuals (the 10 first)
##                       Dist    Dim.1    ctr   cos2    Dim.2    ctr   cos2  
## Mazda RX4         |  2.234 | -0.657  0.204  0.087 |  1.735  3.551  0.604 |
## Mazda RX4 Wag     |  2.081 | -0.629  0.187  0.091 |  1.550  2.833  0.555 |
## Datsun 710        |  2.987 | -2.779  3.653  0.866 | -0.146  0.025  0.002 |
## Hornet 4 Drive    |  2.521 | -0.312  0.046  0.015 | -2.363  6.584  0.879 |
## Hornet Sportabout |  2.456 |  1.974  1.844  0.646 | -0.754  0.671  0.094 |
## Valiant           |  3.014 | -0.056  0.001  0.000 | -2.786  9.151  0.855 |
## Duster 360        |  3.187 |  3.003  4.264  0.888 |  0.335  0.132  0.011 |
## Merc 240D         |  2.841 | -2.055  1.998  0.523 | -1.465  2.531  0.266 |
## Merc 230          |  3.733 | -2.287  2.474  0.375 | -1.984  4.639  0.282 |
## Merc 280          |  1.907 | -0.526  0.131  0.076 | -0.162  0.031  0.007 |
##                    Dim.3    ctr   cos2  
## Mazda RX4         -0.601  1.801  0.072 |
## Mazda RX4 Wag     -0.382  0.728  0.034 |
## Datsun 710        -0.241  0.290  0.007 |
## Hornet 4 Drive    -0.136  0.092  0.003 |
## Hornet Sportabout -1.134  6.412  0.213 |
## Valiant            0.164  0.134  0.003 |
## Duster 360        -0.363  0.656  0.013 |
## Merc 240D          0.944  4.439  0.110 |
## Merc 230           1.797 16.094  0.232 |
## Merc 280           1.493 11.103  0.613 |
## 
## Variables (the 10 first)
##                      Dim.1    ctr   cos2    Dim.2    ctr   cos2    Dim.3
## mpg               | -0.932 13.143  0.869 |  0.026  0.026  0.001 | -0.179
## cyl               |  0.961 13.981  0.924 |  0.071  0.191  0.005 | -0.139
## disp              |  0.946 13.556  0.896 | -0.080  0.243  0.006 | -0.049
## hp                |  0.848 10.894  0.720 |  0.405  6.189  0.164 |  0.111
## drat              | -0.756  8.653  0.572 |  0.447  7.546  0.200 |  0.128
## wt                |  0.890 11.979  0.792 | -0.233  2.046  0.054 |  0.271
## qsec              | -0.515  4.018  0.266 | -0.754 21.472  0.569 |  0.319
## vs                | -0.788  9.395  0.621 | -0.377  5.366  0.142 |  0.340
## am                | -0.604  5.520  0.365 |  0.699 18.440  0.489 | -0.163
## gear              | -0.532  4.281  0.283 |  0.753 21.377  0.567 |  0.229
##                      ctr   cos2  
## mpg                5.096  0.032 |
## cyl                3.073  0.019 |
## disp               0.378  0.002 |
## hp                 1.960  0.012 |
## drat               2.598  0.016 |
## wt                11.684  0.073 |
## qsec              16.255  0.102 |
## vs                18.388  0.115 |
## am                 4.234  0.027 |
## gear               8.397  0.053 |
```

The summary output of PCA shows a bunch of results. It displays all 11 Principal Components created which contain the scores and their associated PVEs.

Notice that the first two Principal Components explain 84.17% of the variance in the data. Let's plot the results:

```r
# Plot the results of the two first Principal Components
biplot(PC_res$ind$coord, PC_res$var$coord, scale=0, cex=0.7, main="Biplot for the first two Principal Components", xlab = "First Principal Component (explains ~60%)", ylab="Second Principal Component (explains ~24%)")
```

<span class="image fit"><img src="{{ "/images/PCA_files/figure-html/unnamed-chunk-5-1.png" | absolute_url }}" alt="" /></span>

The graph above displays the first two principal components which explain more than 84% of the variance in the cars dataset. The black car names represent the scores for the first two principal components. The red arrows represent the first two principal components loading vectors.





For example, we can see that the first PC (horizontal axis) places more positive weight on variables such as cyl (number of cylinders in a car), disp(displacement in cubic inches of a car) and places negative weights on variables such as mpg (miles per gallon). So we could say that the first PC places cars that are 'powerful and heavy' on the right side and places cars that are 'economical' and 'versatile' on the left side.

The second PC (vertical axis) places positive weight on variables such as gear (number of forward gears a car has) and am (transmission type a=auto and m=manual) and negative weight on variables such as qsec (time in seconds it takes for a car to travel 1/4 mile). So we could argue that the second PC places cars that are 'manual with lots of gears' on the top and cars that are 'more classic-looking and slower' on the bottom.

Pick the Maseratti Bora which is located on the upper right hand corner of our biplot. According to our interpretation, cars placed on the top and to the left of the graph, should be 'powerful and heavy' (right side) and 'manual with lots of gears' (top side). As car lovers would know, a Maseratti is a sports car, and it's a powerful car. Conversely, a Toyota Corona (positioned on the left lower corner of our biplot) is a more versatile and slower car. 

We have identified that the first two PC explain more than 84% of the variance in the data.





However, how many Principal Components can be or are created? In general, the number of Principal Components created  will be the minimum value found of either (n-1) where n is the number of data points in a dataset or (p) where p is the number of variables in your data. So for example, in our cars dataset, the minimum value between 31 (number of observations - 1) and 11 (number of variables in the cars dataset) is the latter (11). So the expected number of Principal Components will be 11.


```r
# Plot the rotation matrix with the coordinates
PC_res$var$coord
```

```
##           Dim.1       Dim.2       Dim.3        Dim.4       Dim.5
## mpg  -0.9319502  0.02625094 -0.17877989 -0.011703525  0.04861531
## cyl   0.9612188  0.07121589 -0.13883907 -0.001345754  0.02764565
## disp  0.9464866 -0.08030095 -0.04869285  0.133237930  0.18624400
## hp    0.8484710  0.40502680  0.11088579 -0.035139337  0.25528375
## drat -0.7561693  0.44720905  0.12765473  0.443850788  0.03655308
## wt    0.8897212 -0.23286996  0.27070586  0.127677743 -0.03546673
## qsec -0.5153093 -0.75438614  0.31929289  0.035347223 -0.07783859
## vs   -0.7879428 -0.37712727  0.33960355 -0.111555360  0.28340603
## am   -0.6039632  0.69910300 -0.16295845 -0.015817187  0.04244016
## gear -0.5319156  0.75271549  0.22949350 -0.137434658  0.02284570
## carb  0.5501711  0.67330434  0.41858505 -0.065832458 -0.17079759
##            Dim.6         Dim.7         Dim.8         Dim.9      Dim.10
## mpg   0.05004636  0.1352413926  0.2643640975  0.0654243559 -0.03177274
## cyl  -0.07753399  0.0210655949  0.0809209881  0.0149987209  0.19307910
## disp  0.15463426  0.0788163447 -0.0004004013  0.0550781718 -0.01126416
## hp   -0.03286009 -0.0005501946  0.0779528673 -0.1598347609 -0.05653171
## drat -0.11246761  0.0077674570 -0.0112861725 -0.0130185049  0.02315201
## wt    0.21387027 -0.0076013843  0.0030050869  0.0997869332 -0.02153254
## qsec  0.15201955  0.0183928603  0.0812768533 -0.1466631306  0.06174396
## vs   -0.08924701 -0.0977488250 -0.0090921558  0.0995327801  0.03627885
## am    0.26257362 -0.2159989575  0.0209456686 -0.0131580558  0.04055512
## gear  0.11203788  0.2225426856 -0.1178452002 -0.0004815997  0.04877625
## carb -0.08441920 -0.0642155284  0.1386968855  0.0473652092 -0.01648332
##            Dim.11
## mpg  -0.018543702
## cyl  -0.020889557
## disp  0.098082614
## hp   -0.038082296
## drat -0.005869197
## wt   -0.084251143
## qsec  0.026927434
## vs    0.001249351
## am    0.004428008
## gear -0.007944389
## carb  0.047451368
```

Above you can see that there are 11 distinct principal components as expected with their associated score vectors.

Also, from performing PCA, one may wonder how much information is lost by projecting a 'wide' dataset onto a few principal components? The answer here is in the Proportion of Variance Explained (PVE) by each Principal Component, which is always a positive quantity that explains the variance of each Principal Component. The PVE is a positive quantity and the cumulative sum of PVEs will be 100%. So, once we plot the Principal Components, it's always relevant to display the PVE of each Principal Component.





The next natural question is how many Principal Components should one consider? In general, the number of Principal Components in a dataset will be the minimum of either (n-1) where n is the number of data points or (p) where p is the number of variables in your data.

There is one technique to decide on the number of principal components one should use (for explaining most of the variance found in the data) and this is called the **scree plot**. In essence, a scree plot is a graph that displays the PVE on the vertical axis and the number of principal components found. And the way one chooses the number of principal components is by eyeballing the scree plot and identifying a point at which the proportion of variance explained by each subsequent principal component drops off (similar to the **elbow** method in K-means when you are trying to find the number of clusters to use).

Let's plot the PVE explained by each component from our PCA:

```r
# Plot scree plot of the PCs after calculating PVEs
pve = PC_res$eig[1]/sum(PC_res$eig[1])
pc = seq(1,11)
plot_dta = cbind(pc, pve)

ggplot(data=plot_dta, aes(x=pc, y=unlist(pve), group=1)) +
    geom_line(size=1.5) +
    geom_point(size=3, fill="white") +
    xlab("Principal Components") +
    ylab("Proportion of Variance Explained") +
    scale_x_continuous(breaks = c(1:11), minor_breaks = NULL) +
    scale_y_continuous(labels=percent) +
    ggtitle("Scree Plot for Principal Components and Variance")
```

<span class="image fit"><img src="{{ "/images/PCA_files/figure-html/unnamed-chunk-7-1.png" | absolute_url }}" alt="" /></span>

We can see from the scree plot above that the point at which the proportion of variance explained by each subsequent principal component drops off is from the 3rd principal component. 



Let's now plot the first three principal components in an interactive 3D scatterplot (you can click on the graph and drag it around to observe each individual point in different angles. You can also zoom in and out using your mouse scroll wheel):


```r
# Plot 3D PCA after defining the 3 dimensions from PCA

dims = as.data.frame(PC_res$ind$coord)
dims_3 = dims[,c(1:3)]
plot3d(dims_3$Dim.1, dims_3$Dim.3, dims_3$Dim.2, type="s", aspect=FALSE, col="blue", size=1, main="3D plot of the first three Principal Components of the mtcars dataset", sub="Click on and drag it around or zoom in or out to see it in different angles", xlab="First PC (~60%)", ylab="Third PC (~6%)", zlab="Second PC (~24%)")
text3d(dims_3$Dim.1, dims_3$Dim.3, dims_3$Dim.2,row.names(dims_3), cex=0.7, adj=c(1,1))
```

<script>/*
* Copyright (C) 2009 Apple Inc. All Rights Reserved.
*
* Redistribution and use in source and binary forms, with or without
* modification, are permitted provided that the following conditions
* are met:
* 1. Redistributions of source code must retain the above copyright
*    notice, this list of conditions and the following disclaimer.
* 2. Redistributions in binary form must reproduce the above copyright
*    notice, this list of conditions and the following disclaimer in the
*    documentation and/or other materials provided with the distribution.
*
* THIS SOFTWARE IS PROVIDED BY APPLE INC. ``AS IS'' AND ANY
* EXPRESS OR IMPLIED WARRANTIES, INCLUDING, BUT NOT LIMITED TO, THE
* IMPLIED WARRANTIES OF MERCHANTABILITY AND FITNESS FOR A PARTICULAR
* PURPOSE ARE DISCLAIMED.  IN NO EVENT SHALL APPLE INC. OR
* CONTRIBUTORS BE LIABLE FOR ANY DIRECT, INDIRECT, INCIDENTAL, SPECIAL,
* EXEMPLARY, OR CONSEQUENTIAL DAMAGES (INCLUDING, BUT NOT LIMITED TO,
* PROCUREMENT OF SUBSTITUTE GOODS OR SERVICES; LOSS OF USE, DATA, OR
* PROFITS; OR BUSINESS INTERRUPTION) HOWEVER CAUSED AND ON ANY THEORY
* OF LIABILITY, WHETHER IN CONTRACT, STRICT LIABILITY, OR TORT
* (INCLUDING NEGLIGENCE OR OTHERWISE) ARISING IN ANY WAY OUT OF THE USE
* OF THIS SOFTWARE, EVEN IF ADVISED OF THE POSSIBILITY OF SUCH DAMAGE.
* Copyright (2016) Duncan Murdoch - fixed CanvasMatrix4.ortho,
* cleaned up.
*/
/*
CanvasMatrix4 class
This class implements a 4x4 matrix. It has functions which
duplicate the functionality of the OpenGL matrix stack and
glut functions.
IDL:
[
Constructor(in CanvasMatrix4 matrix),           // copy passed matrix into new CanvasMatrix4
Constructor(in sequence<float> array)           // create new CanvasMatrix4 with 16 floats (row major)
Constructor()                                   // create new CanvasMatrix4 with identity matrix
]
interface CanvasMatrix4 {
attribute float m11;
attribute float m12;
attribute float m13;
attribute float m14;
attribute float m21;
attribute float m22;
attribute float m23;
attribute float m24;
attribute float m31;
attribute float m32;
attribute float m33;
attribute float m34;
attribute float m41;
attribute float m42;
attribute float m43;
attribute float m44;
void load(in CanvasMatrix4 matrix);                 // copy the values from the passed matrix
void load(in sequence<float> array);                // copy 16 floats into the matrix
sequence<float> getAsArray();                       // return the matrix as an array of 16 floats
WebGLFloatArray getAsCanvasFloatArray();           // return the matrix as a WebGLFloatArray with 16 values
void makeIdentity();                                // replace the matrix with identity
void transpose();                                   // replace the matrix with its transpose
void invert();                                      // replace the matrix with its inverse
void translate(in float x, in float y, in float z); // multiply the matrix by passed translation values on the right
void scale(in float x, in float y, in float z);     // multiply the matrix by passed scale values on the right
void rotate(in float angle,                         // multiply the matrix by passed rotation values on the right
in float x, in float y, in float z);    // (angle is in degrees)
void multRight(in CanvasMatrix matrix);             // multiply the matrix by the passed matrix on the right
void multLeft(in CanvasMatrix matrix);              // multiply the matrix by the passed matrix on the left
void ortho(in float left, in float right,           // multiply the matrix by the passed ortho values on the right
in float bottom, in float top,
in float near, in float far);
void frustum(in float left, in float right,         // multiply the matrix by the passed frustum values on the right
in float bottom, in float top,
in float near, in float far);
void perspective(in float fovy, in float aspect,    // multiply the matrix by the passed perspective values on the right
in float zNear, in float zFar);
void lookat(in float eyex, in float eyey, in float eyez,    // multiply the matrix by the passed lookat
in float ctrx, in float ctry, in float ctrz,    // values on the right
in float upx, in float upy, in float upz);
}
*/
CanvasMatrix4 = function(m)
{
if (typeof m == 'object') {
if ("length" in m && m.length >= 16) {
this.load(m[0], m[1], m[2], m[3], m[4], m[5], m[6], m[7], m[8], m[9], m[10], m[11], m[12], m[13], m[14], m[15]);
return;
}
else if (m instanceof CanvasMatrix4) {
this.load(m);
return;
}
}
this.makeIdentity();
};
CanvasMatrix4.prototype.load = function()
{
if (arguments.length == 1 && typeof arguments[0] == 'object') {
var matrix = arguments[0];
if ("length" in matrix && matrix.length == 16) {
this.m11 = matrix[0];
this.m12 = matrix[1];
this.m13 = matrix[2];
this.m14 = matrix[3];
this.m21 = matrix[4];
this.m22 = matrix[5];
this.m23 = matrix[6];
this.m24 = matrix[7];
this.m31 = matrix[8];
this.m32 = matrix[9];
this.m33 = matrix[10];
this.m34 = matrix[11];
this.m41 = matrix[12];
this.m42 = matrix[13];
this.m43 = matrix[14];
this.m44 = matrix[15];
return;
}
if (arguments[0] instanceof CanvasMatrix4) {
this.m11 = matrix.m11;
this.m12 = matrix.m12;
this.m13 = matrix.m13;
this.m14 = matrix.m14;
this.m21 = matrix.m21;
this.m22 = matrix.m22;
this.m23 = matrix.m23;
this.m24 = matrix.m24;
this.m31 = matrix.m31;
this.m32 = matrix.m32;
this.m33 = matrix.m33;
this.m34 = matrix.m34;
this.m41 = matrix.m41;
this.m42 = matrix.m42;
this.m43 = matrix.m43;
this.m44 = matrix.m44;
return;
}
}
this.makeIdentity();
};
CanvasMatrix4.prototype.getAsArray = function()
{
return [
this.m11, this.m12, this.m13, this.m14,
this.m21, this.m22, this.m23, this.m24,
this.m31, this.m32, this.m33, this.m34,
this.m41, this.m42, this.m43, this.m44
];
};
CanvasMatrix4.prototype.getAsWebGLFloatArray = function()
{
return new WebGLFloatArray(this.getAsArray());
};
CanvasMatrix4.prototype.makeIdentity = function()
{
this.m11 = 1;
this.m12 = 0;
this.m13 = 0;
this.m14 = 0;
this.m21 = 0;
this.m22 = 1;
this.m23 = 0;
this.m24 = 0;
this.m31 = 0;
this.m32 = 0;
this.m33 = 1;
this.m34 = 0;
this.m41 = 0;
this.m42 = 0;
this.m43 = 0;
this.m44 = 1;
};
CanvasMatrix4.prototype.transpose = function()
{
var tmp = this.m12;
this.m12 = this.m21;
this.m21 = tmp;
tmp = this.m13;
this.m13 = this.m31;
this.m31 = tmp;
tmp = this.m14;
this.m14 = this.m41;
this.m41 = tmp;
tmp = this.m23;
this.m23 = this.m32;
this.m32 = tmp;
tmp = this.m24;
this.m24 = this.m42;
this.m42 = tmp;
tmp = this.m34;
this.m34 = this.m43;
this.m43 = tmp;
};
CanvasMatrix4.prototype.invert = function()
{
// Calculate the 4x4 determinant
// If the determinant is zero,
// then the inverse matrix is not unique.
var det = this._determinant4x4();
if (Math.abs(det) < 1e-8)
return null;
this._makeAdjoint();
// Scale the adjoint matrix to get the inverse
this.m11 /= det;
this.m12 /= det;
this.m13 /= det;
this.m14 /= det;
this.m21 /= det;
this.m22 /= det;
this.m23 /= det;
this.m24 /= det;
this.m31 /= det;
this.m32 /= det;
this.m33 /= det;
this.m34 /= det;
this.m41 /= det;
this.m42 /= det;
this.m43 /= det;
this.m44 /= det;
};
CanvasMatrix4.prototype.translate = function(x,y,z)
{
if (x === undefined)
x = 0;
if (y === undefined)
y = 0;
if (z === undefined)
z = 0;
var matrix = new CanvasMatrix4();
matrix.m41 = x;
matrix.m42 = y;
matrix.m43 = z;
this.multRight(matrix);
};
CanvasMatrix4.prototype.scale = function(x,y,z)
{
if (x === undefined)
x = 1;
if (z === undefined) {
if (y === undefined) {
y = x;
z = x;
}
else
z = 1;
}
else if (y === undefined)
y = x;
var matrix = new CanvasMatrix4();
matrix.m11 = x;
matrix.m22 = y;
matrix.m33 = z;
this.multRight(matrix);
};
CanvasMatrix4.prototype.rotate = function(angle,x,y,z)
{
// angles are in degrees. Switch to radians
angle = angle / 180 * Math.PI;
angle /= 2;
var sinA = Math.sin(angle);
var cosA = Math.cos(angle);
var sinA2 = sinA * sinA;
// normalize
var length = Math.sqrt(x * x + y * y + z * z);
if (length === 0) {
// bad vector, just use something reasonable
x = 0;
y = 0;
z = 1;
} else if (length != 1) {
x /= length;
y /= length;
z /= length;
}
var mat = new CanvasMatrix4();
// optimize case where axis is along major axis
if (x == 1 && y === 0 && z === 0) {
mat.m11 = 1;
mat.m12 = 0;
mat.m13 = 0;
mat.m21 = 0;
mat.m22 = 1 - 2 * sinA2;
mat.m23 = 2 * sinA * cosA;
mat.m31 = 0;
mat.m32 = -2 * sinA * cosA;
mat.m33 = 1 - 2 * sinA2;
mat.m14 = mat.m24 = mat.m34 = 0;
mat.m41 = mat.m42 = mat.m43 = 0;
mat.m44 = 1;
} else if (x === 0 && y == 1 && z === 0) {
mat.m11 = 1 - 2 * sinA2;
mat.m12 = 0;
mat.m13 = -2 * sinA * cosA;
mat.m21 = 0;
mat.m22 = 1;
mat.m23 = 0;
mat.m31 = 2 * sinA * cosA;
mat.m32 = 0;
mat.m33 = 1 - 2 * sinA2;
mat.m14 = mat.m24 = mat.m34 = 0;
mat.m41 = mat.m42 = mat.m43 = 0;
mat.m44 = 1;
} else if (x === 0 && y === 0 && z == 1) {
mat.m11 = 1 - 2 * sinA2;
mat.m12 = 2 * sinA * cosA;
mat.m13 = 0;
mat.m21 = -2 * sinA * cosA;
mat.m22 = 1 - 2 * sinA2;
mat.m23 = 0;
mat.m31 = 0;
mat.m32 = 0;
mat.m33 = 1;
mat.m14 = mat.m24 = mat.m34 = 0;
mat.m41 = mat.m42 = mat.m43 = 0;
mat.m44 = 1;
} else {
var x2 = x*x;
var y2 = y*y;
var z2 = z*z;
mat.m11 = 1 - 2 * (y2 + z2) * sinA2;
mat.m12 = 2 * (x * y * sinA2 + z * sinA * cosA);
mat.m13 = 2 * (x * z * sinA2 - y * sinA * cosA);
mat.m21 = 2 * (y * x * sinA2 - z * sinA * cosA);
mat.m22 = 1 - 2 * (z2 + x2) * sinA2;
mat.m23 = 2 * (y * z * sinA2 + x * sinA * cosA);
mat.m31 = 2 * (z * x * sinA2 + y * sinA * cosA);
mat.m32 = 2 * (z * y * sinA2 - x * sinA * cosA);
mat.m33 = 1 - 2 * (x2 + y2) * sinA2;
mat.m14 = mat.m24 = mat.m34 = 0;
mat.m41 = mat.m42 = mat.m43 = 0;
mat.m44 = 1;
}
this.multRight(mat);
};
CanvasMatrix4.prototype.multRight = function(mat)
{
var m11 = (this.m11 * mat.m11 + this.m12 * mat.m21 +
this.m13 * mat.m31 + this.m14 * mat.m41);
var m12 = (this.m11 * mat.m12 + this.m12 * mat.m22 +
this.m13 * mat.m32 + this.m14 * mat.m42);
var m13 = (this.m11 * mat.m13 + this.m12 * mat.m23 +
this.m13 * mat.m33 + this.m14 * mat.m43);
var m14 = (this.m11 * mat.m14 + this.m12 * mat.m24 +
this.m13 * mat.m34 + this.m14 * mat.m44);
var m21 = (this.m21 * mat.m11 + this.m22 * mat.m21 +
this.m23 * mat.m31 + this.m24 * mat.m41);
var m22 = (this.m21 * mat.m12 + this.m22 * mat.m22 +
this.m23 * mat.m32 + this.m24 * mat.m42);
var m23 = (this.m21 * mat.m13 + this.m22 * mat.m23 +
this.m23 * mat.m33 + this.m24 * mat.m43);
var m24 = (this.m21 * mat.m14 + this.m22 * mat.m24 +
this.m23 * mat.m34 + this.m24 * mat.m44);
var m31 = (this.m31 * mat.m11 + this.m32 * mat.m21 +
this.m33 * mat.m31 + this.m34 * mat.m41);
var m32 = (this.m31 * mat.m12 + this.m32 * mat.m22 +
this.m33 * mat.m32 + this.m34 * mat.m42);
var m33 = (this.m31 * mat.m13 + this.m32 * mat.m23 +
this.m33 * mat.m33 + this.m34 * mat.m43);
var m34 = (this.m31 * mat.m14 + this.m32 * mat.m24 +
this.m33 * mat.m34 + this.m34 * mat.m44);
var m41 = (this.m41 * mat.m11 + this.m42 * mat.m21 +
this.m43 * mat.m31 + this.m44 * mat.m41);
var m42 = (this.m41 * mat.m12 + this.m42 * mat.m22 +
this.m43 * mat.m32 + this.m44 * mat.m42);
var m43 = (this.m41 * mat.m13 + this.m42 * mat.m23 +
this.m43 * mat.m33 + this.m44 * mat.m43);
var m44 = (this.m41 * mat.m14 + this.m42 * mat.m24 +
this.m43 * mat.m34 + this.m44 * mat.m44);
this.m11 = m11;
this.m12 = m12;
this.m13 = m13;
this.m14 = m14;
this.m21 = m21;
this.m22 = m22;
this.m23 = m23;
this.m24 = m24;
this.m31 = m31;
this.m32 = m32;
this.m33 = m33;
this.m34 = m34;
this.m41 = m41;
this.m42 = m42;
this.m43 = m43;
this.m44 = m44;
};
CanvasMatrix4.prototype.multLeft = function(mat)
{
var m11 = (mat.m11 * this.m11 + mat.m12 * this.m21 +
mat.m13 * this.m31 + mat.m14 * this.m41);
var m12 = (mat.m11 * this.m12 + mat.m12 * this.m22 +
mat.m13 * this.m32 + mat.m14 * this.m42);
var m13 = (mat.m11 * this.m13 + mat.m12 * this.m23 +
mat.m13 * this.m33 + mat.m14 * this.m43);
var m14 = (mat.m11 * this.m14 + mat.m12 * this.m24 +
mat.m13 * this.m34 + mat.m14 * this.m44);
var m21 = (mat.m21 * this.m11 + mat.m22 * this.m21 +
mat.m23 * this.m31 + mat.m24 * this.m41);
var m22 = (mat.m21 * this.m12 + mat.m22 * this.m22 +
mat.m23 * this.m32 + mat.m24 * this.m42);
var m23 = (mat.m21 * this.m13 + mat.m22 * this.m23 +
mat.m23 * this.m33 + mat.m24 * this.m43);
var m24 = (mat.m21 * this.m14 + mat.m22 * this.m24 +
mat.m23 * this.m34 + mat.m24 * this.m44);
var m31 = (mat.m31 * this.m11 + mat.m32 * this.m21 +
mat.m33 * this.m31 + mat.m34 * this.m41);
var m32 = (mat.m31 * this.m12 + mat.m32 * this.m22 +
mat.m33 * this.m32 + mat.m34 * this.m42);
var m33 = (mat.m31 * this.m13 + mat.m32 * this.m23 +
mat.m33 * this.m33 + mat.m34 * this.m43);
var m34 = (mat.m31 * this.m14 + mat.m32 * this.m24 +
mat.m33 * this.m34 + mat.m34 * this.m44);
var m41 = (mat.m41 * this.m11 + mat.m42 * this.m21 +
mat.m43 * this.m31 + mat.m44 * this.m41);
var m42 = (mat.m41 * this.m12 + mat.m42 * this.m22 +
mat.m43 * this.m32 + mat.m44 * this.m42);
var m43 = (mat.m41 * this.m13 + mat.m42 * this.m23 +
mat.m43 * this.m33 + mat.m44 * this.m43);
var m44 = (mat.m41 * this.m14 + mat.m42 * this.m24 +
mat.m43 * this.m34 + mat.m44 * this.m44);
this.m11 = m11;
this.m12 = m12;
this.m13 = m13;
this.m14 = m14;
this.m21 = m21;
this.m22 = m22;
this.m23 = m23;
this.m24 = m24;
this.m31 = m31;
this.m32 = m32;
this.m33 = m33;
this.m34 = m34;
this.m41 = m41;
this.m42 = m42;
this.m43 = m43;
this.m44 = m44;
};
CanvasMatrix4.prototype.ortho = function(left, right, bottom, top, near, far)
{
var tx = (left + right) / (left - right);
var ty = (top + bottom) / (bottom - top);
var tz = (far + near) / (near - far);
var matrix = new CanvasMatrix4();
matrix.m11 = 2 / (right - left);
matrix.m12 = 0;
matrix.m13 = 0;
matrix.m14 = 0;
matrix.m21 = 0;
matrix.m22 = 2 / (top - bottom);
matrix.m23 = 0;
matrix.m24 = 0;
matrix.m31 = 0;
matrix.m32 = 0;
matrix.m33 = -2 / (far - near);
matrix.m34 = 0;
matrix.m41 = tx;
matrix.m42 = ty;
matrix.m43 = tz;
matrix.m44 = 1;
this.multRight(matrix);
};
CanvasMatrix4.prototype.frustum = function(left, right, bottom, top, near, far)
{
var matrix = new CanvasMatrix4();
var A = (right + left) / (right - left);
var B = (top + bottom) / (top - bottom);
var C = -(far + near) / (far - near);
var D = -(2 * far * near) / (far - near);
matrix.m11 = (2 * near) / (right - left);
matrix.m12 = 0;
matrix.m13 = 0;
matrix.m14 = 0;
matrix.m21 = 0;
matrix.m22 = 2 * near / (top - bottom);
matrix.m23 = 0;
matrix.m24 = 0;
matrix.m31 = A;
matrix.m32 = B;
matrix.m33 = C;
matrix.m34 = -1;
matrix.m41 = 0;
matrix.m42 = 0;
matrix.m43 = D;
matrix.m44 = 0;
this.multRight(matrix);
};
CanvasMatrix4.prototype.perspective = function(fovy, aspect, zNear, zFar)
{
var top = Math.tan(fovy * Math.PI / 360) * zNear;
var bottom = -top;
var left = aspect * bottom;
var right = aspect * top;
this.frustum(left, right, bottom, top, zNear, zFar);
};
CanvasMatrix4.prototype.lookat = function(eyex, eyey, eyez, centerx, centery, centerz, upx, upy, upz)
{
var matrix = new CanvasMatrix4();
// Make rotation matrix
// Z vector
var zx = eyex - centerx;
var zy = eyey - centery;
var zz = eyez - centerz;
var mag = Math.sqrt(zx * zx + zy * zy + zz * zz);
if (mag) {
zx /= mag;
zy /= mag;
zz /= mag;
}
// Y vector
var yx = upx;
var yy = upy;
var yz = upz;
// X vector = Y cross Z
xx =  yy * zz - yz * zy;
xy = -yx * zz + yz * zx;
xz =  yx * zy - yy * zx;
// Recompute Y = Z cross X
yx = zy * xz - zz * xy;
yy = -zx * xz + zz * xx;
yx = zx * xy - zy * xx;
// cross product gives area of parallelogram, which is < 1.0 for
// non-perpendicular unit-length vectors; so normalize x, y here
mag = Math.sqrt(xx * xx + xy * xy + xz * xz);
if (mag) {
xx /= mag;
xy /= mag;
xz /= mag;
}
mag = Math.sqrt(yx * yx + yy * yy + yz * yz);
if (mag) {
yx /= mag;
yy /= mag;
yz /= mag;
}
matrix.m11 = xx;
matrix.m12 = xy;
matrix.m13 = xz;
matrix.m14 = 0;
matrix.m21 = yx;
matrix.m22 = yy;
matrix.m23 = yz;
matrix.m24 = 0;
matrix.m31 = zx;
matrix.m32 = zy;
matrix.m33 = zz;
matrix.m34 = 0;
matrix.m41 = 0;
matrix.m42 = 0;
matrix.m43 = 0;
matrix.m44 = 1;
matrix.translate(-eyex, -eyey, -eyez);
this.multRight(matrix);
};
// Support functions
CanvasMatrix4.prototype._determinant2x2 = function(a, b, c, d)
{
return a * d - b * c;
};
CanvasMatrix4.prototype._determinant3x3 = function(a1, a2, a3, b1, b2, b3, c1, c2, c3)
{
return a1 * this._determinant2x2(b2, b3, c2, c3) -
b1 * this._determinant2x2(a2, a3, c2, c3) +
c1 * this._determinant2x2(a2, a3, b2, b3);
};
CanvasMatrix4.prototype._determinant4x4 = function()
{
var a1 = this.m11;
var b1 = this.m12;
var c1 = this.m13;
var d1 = this.m14;
var a2 = this.m21;
var b2 = this.m22;
var c2 = this.m23;
var d2 = this.m24;
var a3 = this.m31;
var b3 = this.m32;
var c3 = this.m33;
var d3 = this.m34;
var a4 = this.m41;
var b4 = this.m42;
var c4 = this.m43;
var d4 = this.m44;
return a1 * this._determinant3x3(b2, b3, b4, c2, c3, c4, d2, d3, d4) -
b1 * this._determinant3x3(a2, a3, a4, c2, c3, c4, d2, d3, d4) +
c1 * this._determinant3x3(a2, a3, a4, b2, b3, b4, d2, d3, d4) -
d1 * this._determinant3x3(a2, a3, a4, b2, b3, b4, c2, c3, c4);
};
CanvasMatrix4.prototype._makeAdjoint = function()
{
var a1 = this.m11;
var b1 = this.m12;
var c1 = this.m13;
var d1 = this.m14;
var a2 = this.m21;
var b2 = this.m22;
var c2 = this.m23;
var d2 = this.m24;
var a3 = this.m31;
var b3 = this.m32;
var c3 = this.m33;
var d3 = this.m34;
var a4 = this.m41;
var b4 = this.m42;
var c4 = this.m43;
var d4 = this.m44;
// Row column labeling reversed since we transpose rows & columns
this.m11  =   this._determinant3x3(b2, b3, b4, c2, c3, c4, d2, d3, d4);
this.m21  = - this._determinant3x3(a2, a3, a4, c2, c3, c4, d2, d3, d4);
this.m31  =   this._determinant3x3(a2, a3, a4, b2, b3, b4, d2, d3, d4);
this.m41  = - this._determinant3x3(a2, a3, a4, b2, b3, b4, c2, c3, c4);
this.m12  = - this._determinant3x3(b1, b3, b4, c1, c3, c4, d1, d3, d4);
this.m22  =   this._determinant3x3(a1, a3, a4, c1, c3, c4, d1, d3, d4);
this.m32  = - this._determinant3x3(a1, a3, a4, b1, b3, b4, d1, d3, d4);
this.m42  =   this._determinant3x3(a1, a3, a4, b1, b3, b4, c1, c3, c4);
this.m13  =   this._determinant3x3(b1, b2, b4, c1, c2, c4, d1, d2, d4);
this.m23  = - this._determinant3x3(a1, a2, a4, c1, c2, c4, d1, d2, d4);
this.m33  =   this._determinant3x3(a1, a2, a4, b1, b2, b4, d1, d2, d4);
this.m43  = - this._determinant3x3(a1, a2, a4, b1, b2, b4, c1, c2, c4);
this.m14  = - this._determinant3x3(b1, b2, b3, c1, c2, c3, d1, d2, d3);
this.m24  =   this._determinant3x3(a1, a2, a3, c1, c2, c3, d1, d2, d3);
this.m34  = - this._determinant3x3(a1, a2, a3, b1, b2, b3, d1, d2, d3);
this.m44  =   this._determinant3x3(a1, a2, a3, b1, b2, b3, c1, c2, c3);
};</script>
<script>
rglwidgetClass = function() {
this.canvas = null;
this.userMatrix = new CanvasMatrix4();
this.types = [];
this.prMatrix = new CanvasMatrix4();
this.mvMatrix = new CanvasMatrix4();
this.vp = null;
this.prmvMatrix = null;
this.origs = null;
this.gl = null;
this.scene = null;
};
(function() {
this.multMV = function(M, v) {
return [ M.m11 * v[0] + M.m12 * v[1] + M.m13 * v[2] + M.m14 * v[3],
M.m21 * v[0] + M.m22 * v[1] + M.m23 * v[2] + M.m24 * v[3],
M.m31 * v[0] + M.m32 * v[1] + M.m33 * v[2] + M.m34 * v[3],
M.m41 * v[0] + M.m42 * v[1] + M.m43 * v[2] + M.m44 * v[3]
];
};
this.vlen = function(v) {
return Math.sqrt(this.dotprod(v, v));
};
this.dotprod = function(a, b) {
return a[0]*b[0] + a[1]*b[1] + a[2]*b[2];
};
this.xprod = function(a, b) {
return [a[1]*b[2] - a[2]*b[1],
a[2]*b[0] - a[0]*b[2],
a[0]*b[1] - a[1]*b[0]];
};
this.cbind = function(a, b) {
return a.map(function(currentValue, index, array) {
return currentValue.concat(b[index]);
});
};
this.swap = function(a, i, j) {
var temp = a[i];
a[i] = a[j];
a[j] = temp;
};
this.flatten = function(a) {
return [].concat.apply([], a);
};
/* set element of 1d or 2d array as if it was flattened.  Column major, zero based! */
this.setElement = function(a, i, value) {
if (Array.isArray(a[0])) {
var dim = a.length,
col = Math.floor(i/dim),
row = i % dim;
a[row][col] = value;
} else {
a[i] = value;
}
};
this.transpose = function(a) {
var newArray = [],
n = a.length,
m = a[0].length,
i;
for(i = 0; i < m; i++){
newArray.push([]);
}
for(i = 0; i < n; i++){
for(var j = 0; j < m; j++){
newArray[j].push(a[i][j]);
}
}
return newArray;
};
this.sumsq = function(x) {
var result = 0, i;
for (i=0; i < x.length; i++)
result += x[i]*x[i];
return result;
};
this.toCanvasMatrix4 = function(mat) {
if (mat instanceof CanvasMatrix4)
return mat;
var result = new CanvasMatrix4();
mat = this.flatten(this.transpose(mat));
result.load(mat);
return result;
};
this.stringToRgb = function(s) {
s = s.replace("#", "");
var bigint = parseInt(s, 16);
return [((bigint >> 16) & 255)/255,
((bigint >> 8) & 255)/255,
(bigint & 255)/255];
};
this.componentProduct = function(x, y) {
if (typeof y === "undefined") {
this.alertOnce("Bad arg to componentProduct");
}
var result = new Float32Array(3), i;
for (i = 0; i<3; i++)
result[i] = x[i]*y[i];
return result;
};
this.getPowerOfTwo = function(value) {
var pow = 1;
while(pow<value) {
pow *= 2;
}
return pow;
};
this.unique = function(arr) {
arr = [].concat(arr);
return arr.filter(function(value, index, self) {
return self.indexOf(value) === index;
});
};
this.repeatToLen = function(arr, len) {
arr = [].concat(arr);
while (arr.length < len/2)
arr = arr.concat(arr);
return arr.concat(arr.slice(0, len - arr.length));
};
this.alertOnce = function(msg) {
if (typeof this.alerted !== "undefined")
return;
this.alerted = true;
alert(msg);
};
this.f_is_lit = 1;
this.f_is_smooth = 2;
this.f_has_texture = 4;
this.f_is_indexed = 8;
this.f_depth_sort = 16;
this.f_fixed_quads = 32;
this.f_is_transparent = 64;
this.f_is_lines = 128;
this.f_sprites_3d = 256;
this.f_sprite_3d = 512;
this.f_is_subscene = 1024;
this.f_is_clipplanes = 2048;
this.whichList = function(id) {
var obj = this.getObj(id),
flags = obj.flags;
if (obj.type === "light")
return "lights";
if (flags & this.f_is_subscene)
return "subscenes";
if (flags & this.f_is_clipplanes)
return "clipplanes";
if (flags & this.f_is_transparent)
return "transparent";
return "opaque";
};
this.getObj = function(id) {
if (typeof id !== "number") {
this.alertOnce("getObj id is "+typeof id);
}
return this.scene.objects[id];
};
this.getIdsByType = function(type, subscene) {
var
result = [], i, self = this;
if (typeof subscene === "undefined") {
Object.keys(this.scene.objects).forEach(
function(key) {
key = parseInt(key, 10);
if (self.getObj(key).type === type)
result.push(key);
});
} else {
ids = this.getObj(subscene).objects;
for (i=0; i < ids.length; i++) {
if (this.getObj(ids[i]).type === type) {
result.push(ids[i]);
}
}
}
return result;
};
this.getMaterial = function(id, property) {
var obj = this.getObj(id),
mat = obj.material[property];
if (typeof mat === "undefined")
mat = this.scene.material[property];
return mat;
};
this.inSubscene = function(id, subscene) {
return this.getObj(subscene).objects.indexOf(id) > -1;
};
this.addToSubscene = function(id, subscene) {
var thelist,
thesub = this.getObj(subscene),
ids = [id],
obj = this.getObj(id), i;
if (typeof obj.newIds !== "undefined") {
ids = ids.concat(obj.newIds);
}
for (i = 0; i < ids.length; i++) {
id = ids[i];
if (thesub.objects.indexOf(id) == -1) {
thelist = this.whichList(id);
thesub.objects.push(id);
thesub[thelist].push(id);
}
}
};
this.delFromSubscene = function(id, subscene) {
var thelist,
thesub = this.getObj(subscene),
obj = this.getObj(id),
ids = [id], i;
if (typeof obj.newIds !== "undefined")
ids = ids.concat(obj.newIds);
for (j=0; j<ids.length;j++) {
id = ids[j];
i = thesub.objects.indexOf(id);
if (i > -1) {
thesub.objects.splice(i, 1);
thelist = this.whichList(id);
i = thesub[thelist].indexOf(id);
thesub[thelist].splice(i, 1);
}
}
};
this.setSubsceneEntries = function(ids, subsceneid) {
var sub = this.getObj(subsceneid);
sub.objects = ids;
this.initSubscene(subsceneid);
};
this.getSubsceneEntries = function(subscene) {
return this.getObj(subscene).objects;
};
this.getChildSubscenes = function(subscene) {
return this.getObj(subscene).subscenes;
};
this.startDrawing = function() {
var value = this.drawing;
this.drawing = true;
return value;
}
this.stopDrawing = function(saved) {
this.drawing = saved;
if (!saved && this.gl && this.gl.isContextLost())
this.restartCanvas();
}
this.getVertexShader = function(id) {
var obj = this.getObj(id),
flags = obj.flags,
type = obj.type,
is_lit = flags & this.f_is_lit,
has_texture = flags & this.f_has_texture,
fixed_quads = flags & this.f_fixed_quads,
sprites_3d = flags & this.f_sprites_3d,
sprite_3d = flags & this.f_sprite_3d,
nclipplanes = this.countClipplanes(),
result;
if (type === "clipplanes" || sprites_3d) return;
result = "  /* ****** "+type+" object "+id+" vertex shader ****** */\n"+
"  attribute vec3 aPos;\n"+
"  attribute vec4 aCol;\n"+
" uniform mat4 mvMatrix;\n"+
" uniform mat4 prMatrix;\n"+
" varying vec4 vCol;\n"+
" varying vec4 vPosition;\n";
if ((is_lit && !fixed_quads) || sprite_3d)
result = result + "  attribute vec3 aNorm;\n"+
" uniform mat4 normMatrix;\n"+
" varying vec3 vNormal;\n";
if (has_texture || type === "text")
result = result + " attribute vec2 aTexcoord;\n"+
" varying vec2 vTexcoord;\n";
if (type === "text")
result = result + "  uniform vec2 textScale;\n";
if (fixed_quads)
result = result + "  attribute vec2 aOfs;\n";
else if (sprite_3d)
result = result + "  uniform vec3 uOrig;\n"+
" uniform float uSize;\n"+
" uniform mat4 usermat;\n";
result = result + "  void main(void) {\n";
if (nclipplanes || (!fixed_quads && !sprite_3d))
result = result + "    vPosition = mvMatrix * vec4(aPos, 1.);\n";
if (!fixed_quads && !sprite_3d)
result = result + "    gl_Position = prMatrix * vPosition;\n";
if (type == "points") {
var size = this.getMaterial(id, "size");
result = result + "    gl_PointSize = "+size.toFixed(1)+";\n";
}
result = result + "    vCol = aCol;\n";
if (is_lit && !fixed_quads && !sprite_3d)
result = result + "    vNormal = normalize((normMatrix * vec4(aNorm, 1.)).xyz);\n";
if (has_texture || type === "text")
result = result + "    vTexcoord = aTexcoord;\n";
if (type == "text")
result = result + "    vec4 pos = prMatrix * mvMatrix * vec4(aPos, 1.);\n"+
"   pos = pos/pos.w;\n"+
"   gl_Position = pos + vec4(aOfs*textScale, 0.,0.);\n";
if (type == "sprites")
result = result + "    vec4 pos = mvMatrix * vec4(aPos, 1.);\n"+
"   pos = pos/pos.w + vec4(aOfs, 0., 0.);\n"+
"   gl_Position = prMatrix*pos;\n";
if (sprite_3d)
result = result + "    vNormal = normalize((normMatrix * vec4(aNorm, 1.)).xyz);\n"+
"   vec4 pos = mvMatrix * vec4(uOrig, 1.);\n"+
"   vPosition = pos/pos.w + vec4(uSize*(vec4(aPos, 1.)*usermat).xyz,0.);\n"+
"   gl_Position = prMatrix * vPosition;\n";
result = result + "  }\n";
return result;
};
this.getFragmentShader = function(id) {
var obj = this.getObj(id),
flags = obj.flags,
type = obj.type,
is_lit = flags & this.f_is_lit,
has_texture = flags & this.f_has_texture,
fixed_quads = flags & this.f_fixed_quads,
sprites_3d = flags & this.f_sprites_3d,
nclipplanes = this.countClipplanes(), i,
texture_format, nlights,
result;
if (type === "clipplanes" || sprites_3d) return;
if (has_texture)
texture_format = this.getMaterial(id, "textype");
result = "/* ****** "+type+" object "+id+" fragment shader ****** */\n"+
"#ifdef GL_ES\n"+
"  precision highp float;\n"+
"#endif\n"+
"  varying vec4 vCol; // carries alpha\n"+
"  varying vec4 vPosition;\n";
if (has_texture || type === "text")
result = result + "  varying vec2 vTexcoord;\n"+
" uniform sampler2D uSampler;\n";
if (is_lit && !fixed_quads)
result = result + "  varying vec3 vNormal;\n";
for (i = 0; i < nclipplanes; i++)
result = result + "  uniform vec4 vClipplane"+i+";\n";
if (is_lit) {
nlights = this.countLights();
if (nlights)
result = result + "  uniform mat4 mvMatrix;\n";
else
is_lit = false;
}
if (is_lit) {
result = result + "    uniform vec3 emission;\n"+
"   uniform float shininess;\n";
for (i=0; i < nlights; i++) {
result = result + "    uniform vec3 ambient" + i + ";\n"+
"   uniform vec3 specular" + i +"; // light*material\n"+
"   uniform vec3 diffuse" + i + ";\n"+
"   uniform vec3 lightDir" + i + ";\n"+
"   uniform bool viewpoint" + i + ";\n"+
"   uniform bool finite" + i + ";\n";
}
}
result = result + "  void main(void) {\n";
for (i=0; i < nclipplanes;i++)
result = result + "    if (dot(vPosition, vClipplane"+i+") < 0.0) discard;\n";
if (is_lit) {
result = result + "    vec3 eye = normalize(-vPosition.xyz);\n"+
"   vec3 lightdir;\n"+
"   vec4 colDiff;\n"+
"   vec3 halfVec;\n"+
"   vec4 lighteffect = vec4(emission, 0.);\n"+
"   vec3 col;\n"+
"   float nDotL;\n";
if (fixed_quads) {
result = result +   "    vec3 n = vec3(0., 0., 1.);\n";
}
else {
result = result +   "    vec3 n = normalize(vNormal);\n"+
"   n = -faceforward(n, n, eye);\n";
}
for (i=0; i < nlights; i++) {
result = result + "   colDiff = vec4(vCol.rgb * diffuse" + i + ", vCol.a);\n"+
"   lightdir = lightDir" + i + ";\n"+
"   if (!viewpoint" + i +")\n"+
"     lightdir = (mvMatrix * vec4(lightdir, 1.)).xyz;\n"+
"   if (!finite" + i + ") {\n"+
"     halfVec = normalize(lightdir + eye);\n"+
"   } else {\n"+
"     lightdir = normalize(lightdir - vPosition.xyz);\n"+
"     halfVec = normalize(lightdir + eye);\n"+
"   }\n"+
"    col = ambient" + i + ";\n"+
"   nDotL = dot(n, lightdir);\n"+
"   col = col + max(nDotL, 0.) * colDiff.rgb;\n"+
"   col = col + pow(max(dot(halfVec, n), 0.), shininess) * specular" + i + ";\n"+
"   lighteffect = lighteffect + vec4(col, colDiff.a);\n";
}
} else {
result = result +   "   vec4 colDiff = vCol;\n"+
"    vec4 lighteffect = colDiff;\n";
}
if ((has_texture && texture_format === "rgba") || type === "text")
result = result +   "    vec4 textureColor = lighteffect*texture2D(uSampler, vTexcoord);\n";
if (has_texture) {
result = result + {
rgb:            "   vec4 textureColor = lighteffect*vec4(texture2D(uSampler, vTexcoord).rgb, 1.);\n",
alpha:          "   vec4 textureColor = texture2D(uSampler, vTexcoord);\n"+
"   float luminance = dot(vec3(1.,1.,1.), textureColor.rgb)/3.;\n"+
"   textureColor =  vec4(lighteffect.rgb, lighteffect.a*luminance);\n",
luminance:      "   vec4 textureColor = vec4(lighteffect.rgb*dot(texture2D(uSampler, vTexcoord).rgb, vec3(1.,1.,1.))/3., lighteffect.a);\n",
"luminance.alpha":"    vec4 textureColor = texture2D(uSampler, vTexcoord);\n"+
"   float luminance = dot(vec3(1.,1.,1.),textureColor.rgb)/3.;\n"+
"   textureColor = vec4(lighteffect.rgb*luminance, lighteffect.a*textureColor.a);\n"
}[texture_format]+
"   gl_FragColor = textureColor;\n";
} else if (type === "text") {
result = result +   "    if (textureColor.a < 0.1)\n"+
"     discard;\n"+
"   else\n"+
"     gl_FragColor = textureColor;\n";
} else
result = result +   "   gl_FragColor = lighteffect;\n";
result = result + "  }\n";
return result;
};
this.getShader = function(shaderType, code) {
var gl = this.gl, shader;
shader = gl.createShader(shaderType);
gl.shaderSource(shader, code);
gl.compileShader(shader);
if (!gl.getShaderParameter(shader, gl.COMPILE_STATUS) && !gl.isContextLost())
alert(gl.getShaderInfoLog(shader));
return shader;
};
this.handleLoadedTexture = function(texture, textureCanvas) { 
var gl = this.gl || this.initGL();
gl.pixelStorei(gl.UNPACK_FLIP_Y_WEBGL, true);
gl.bindTexture(gl.TEXTURE_2D, texture);
gl.texImage2D(gl.TEXTURE_2D, 0, gl.RGBA, gl.RGBA, gl.UNSIGNED_BYTE, textureCanvas);
gl.texParameteri(gl.TEXTURE_2D, gl.TEXTURE_MAG_FILTER, gl.LINEAR);
gl.texParameteri(gl.TEXTURE_2D, gl.TEXTURE_MIN_FILTER, gl.LINEAR_MIPMAP_NEAREST);
gl.generateMipmap(gl.TEXTURE_2D);
gl.bindTexture(gl.TEXTURE_2D, null);
};
this.loadImageToTexture = function(uri, texture) {
var canvas = this.textureCanvas,
ctx = canvas.getContext("2d"),
image = new Image(),
self = this;
image.onload = function() {
var w = image.width,
h = image.height,
canvasX = self.getPowerOfTwo(w),
canvasY = self.getPowerOfTwo(h),
gl = self.gl || self.initGL(),
maxTexSize = gl.getParameter(gl.MAX_TEXTURE_SIZE);
if (maxTexSize > 4096) maxTexSize = 4096;
while (canvasX > 1 && canvasY > 1 && (canvasX > maxTexSize || canvasY > maxTexSize)) {
canvasX /= 2;
canvasY /= 2;
}
canvas.width = canvasX;
canvas.height = canvasY;
ctx.imageSmoothingEnabled = true;
ctx.drawImage(image, 0, 0, canvasX, canvasY);
self.handleLoadedTexture(texture, canvas);
self.drawScene();
};
image.src = uri;
};
this.drawTextToCanvas = function(text, cex, family, font) {
var canvasX, canvasY,
textY,
scaling = 20,
textColour = "white",
backgroundColour = "rgba(0,0,0,0)",
canvas = this.textureCanvas,
ctx = canvas.getContext("2d"),
i, textHeights = [], widths = [], offset = 0, offsets = [],
fontStrings = [],
getFontString = function(i) {
textHeights[i] = scaling*cex[i];
var fontString = textHeights[i] + "px",
family0 = family[i],
font0 = font[i];
if (family0 === "sans")
family0 = "sans-serif";
else if (family0 === "mono")
family0 = "monospace";
fontString = fontString + " " + family0;
if (font0 === 2 || font0 === 4)
fontString = "bold " + fontString;
if (font0 === 3 || font0 === 4)
fontString = "italic " + fontString;
return fontString;
};
cex = this.repeatToLen(cex, text.length);
family = this.repeatToLen(family, text.length);
font = this.repeatToLen(font, text.length);
canvasX = 1;
for (i = 0; i < text.length; i++)  {
ctx.font = fontStrings[i] = getFontString(i);
widths[i] = ctx.measureText(text[i]).width;
offset = offsets[i] = offset + 2*textHeights[i];
canvasX = (widths[i] > canvasX) ? widths[i] : canvasX;
}
canvasX = this.getPowerOfTwo(canvasX);
canvasY = this.getPowerOfTwo(offset);
canvas.width = canvasX;
canvas.height = canvasY;
ctx.fillStyle = backgroundColour;
ctx.fillRect(0, 0, ctx.canvas.width, ctx.canvas.height);
ctx.textBaseline = "alphabetic";
for(i = 0; i < text.length; i++) {
textY = offsets[i];
ctx.font = fontStrings[i];
ctx.fillStyle = textColour;
ctx.textAlign = "left";
ctx.fillText(text[i], 0,  textY);
}
return {canvasX:canvasX, canvasY:canvasY,
widths:widths, textHeights:textHeights,
offsets:offsets};
};
this.setViewport = function(id) {
var gl = this.gl || this.initGL(),
vp = this.getObj(id).par3d.viewport,
x = vp.x*this.canvas.width,
y = vp.y*this.canvas.height,
width = vp.width*this.canvas.width,
height = vp.height*this.canvas.height;
this.vp = {x:x, y:y, width:width, height:height};
gl.viewport(x, y, width, height);
gl.scissor(x, y, width, height);
gl.enable(gl.SCISSOR_TEST);
};
this.setprMatrix = function(id) {
var subscene = this.getObj(id),
embedding = subscene.embeddings.projection;
if (embedding === "replace")
this.prMatrix.makeIdentity();
else
this.setprMatrix(subscene.parent);
if (embedding === "inherit")
return;
// This is based on the Frustum::enclose code from geom.cpp
var bbox = subscene.par3d.bbox,
scale = subscene.par3d.scale,
ranges = [(bbox[1]-bbox[0])*scale[0]/2,
(bbox[3]-bbox[2])*scale[1]/2,
(bbox[5]-bbox[4])*scale[2]/2],
radius = Math.sqrt(this.sumsq(ranges))*1.1; // A bit bigger to handle labels
if (radius <= 0) radius = 1;
var observer = subscene.par3d.observer,
distance = observer[2],
FOV = subscene.par3d.FOV, ortho = FOV === 0,
t = ortho ? 1 : Math.tan(FOV*Math.PI/360),
near = distance - radius,
far = distance + radius,
hlen = t*near,
aspect = this.vp.width/this.vp.height,
z = subscene.par3d.zoom;
if (ortho) {
if (aspect > 1)
this.prMatrix.ortho(-hlen*aspect*z, hlen*aspect*z,
-hlen*z, hlen*z, near, far);
else
this.prMatrix.ortho(-hlen*z, hlen*z,
-hlen*z/aspect, hlen*z/aspect,
near, far);
} else {
if (aspect > 1)
this.prMatrix.frustum(-hlen*aspect*z, hlen*aspect*z,
-hlen*z, hlen*z, near, far);
else
this.prMatrix.frustum(-hlen*z, hlen*z,
-hlen*z/aspect, hlen*z/aspect,
near, far);
}
};
this.setmvMatrix = function(id) {
var observer = this.getObj(id).par3d.observer;
this.mvMatrix.makeIdentity();
this.setmodelMatrix(id);
this.mvMatrix.translate(-observer[0], -observer[1], -observer[2]);
};
this.setmodelMatrix = function(id) {
var subscene = this.getObj(id),
embedding = subscene.embeddings.model;
if (embedding !== "inherit") {
var scale = subscene.par3d.scale,
bbox = subscene.par3d.bbox,
center = [(bbox[0]+bbox[1])/2,
(bbox[2]+bbox[3])/2,
(bbox[4]+bbox[5])/2];
this.mvMatrix.translate(-center[0], -center[1], -center[2]);
this.mvMatrix.scale(scale[0], scale[1], scale[2]);
this.mvMatrix.multRight( subscene.par3d.userMatrix );
}
if (embedding !== "replace")
this.setmodelMatrix(subscene.parent);
};
this.setnormMatrix = function(subsceneid) {
var self = this,
recurse = function(id) {
var sub = self.getObj(id),
embedding = sub.embeddings.model;
if (embedding !== "inherit") {
var scale = sub.par3d.scale;
self.normMatrix.scale(1/scale[0], 1/scale[1], 1/scale[2]);
self.normMatrix.multRight(sub.par3d.userMatrix);
}
if (embedding !== "replace")
recurse(sub.parent);
};
self.normMatrix.makeIdentity();
recurse(subsceneid);
};
this.setprmvMatrix = function() {
this.prmvMatrix = new CanvasMatrix4( this.mvMatrix );
this.prmvMatrix.multRight( this.prMatrix );
};
this.countClipplanes = function() {
return this.countObjs("clipplanes");
};
this.countLights = function() {
return this.countObjs("light");
};
this.countObjs = function(type) {
var self = this,
bound = 0;
Object.keys(this.scene.objects).forEach(
function(key) {
if (self.getObj(parseInt(key, 10)).type === type)
bound = bound + 1;
});
return bound;
};
this.initSubscene = function(id) {
var sub = this.getObj(id),
i, obj;
if (sub.type !== "subscene")
return;
sub.par3d.userMatrix = this.toCanvasMatrix4(sub.par3d.userMatrix);
sub.par3d.listeners = [].concat(sub.par3d.listeners);
sub.backgroundId = undefined;
sub.subscenes = [];
sub.clipplanes = [];
sub.transparent = [];
sub.opaque = [];
sub.lights = [];
for (i=0; i < sub.objects.length; i++) {
obj = this.getObj(sub.objects[i]);
if (typeof obj === "undefined") {
sub.objects.splice(i, 1);
i--;
} else if (obj.type === "background")
sub.backgroundId = obj.id;
else
sub[this.whichList(obj.id)].push(obj.id);
}
};
this.copyObj = function(id, reuse) {
var obj = this.getObj(id),
prev = document.getElementById(reuse);
if (prev !== null) {
prev = prev.rglinstance;
var
prevobj = prev.getObj(id),
fields = ["flags", "type",
"colors", "vertices", "centers",
"normals", "offsets",
"texts", "cex", "family", "font", "adj",
"material",
"radii",
"texcoords",
"userMatrix", "ids",
"dim",
"par3d", "userMatrix",
"viewpoint", "finite"],
i;
for (i = 0; i < fields.length; i++) {
if (typeof prevobj[fields[i]] !== "undefined")
obj[fields[i]] = prevobj[fields[i]];
}
} else
console.warn("copyObj failed");
};
this.planeUpdateTriangles = function(id, bbox) {
var perms = [[0,0,1], [1,2,2], [2,1,0]],
x, xrow, elem, A, d, nhits, i, j, k, u, v, w, intersect, which, v0, v2, vx, reverse,
face1 = [], face2 = [], normals = [],
obj = this.getObj(id),
nPlanes = obj.normals.length;
obj.bbox = bbox;
obj.vertices = [];
obj.initialized = false;
for (elem = 0; elem < nPlanes; elem++) {
//    Vertex Av = normal.getRecycled(elem);
x = [];
A = obj.normals[elem];
d = obj.offsets[elem][0];
nhits = 0;
for (i=0; i<3; i++)
for (j=0; j<2; j++)
for (k=0; k<2; k++) {
u = perms[0][i];
v = perms[1][i];
w = perms[2][i];
if (A[w] !== 0.0) {
intersect = -(d + A[u]*bbox[j+2*u] + A[v]*bbox[k+2*v])/A[w];
if (bbox[2*w] < intersect && intersect < bbox[1+2*w]) {
xrow = [];
xrow[u] = bbox[j+2*u];
xrow[v] = bbox[k+2*v];
xrow[w] = intersect;
x.push(xrow);
face1[nhits] = j + 2*u;
face2[nhits] = k + 2*v;
nhits++;
}
}
}
if (nhits > 3) {
/* Re-order the intersections so the triangles work */
for (i=0; i<nhits-2; i++) {
which = 0; /* initialize to suppress warning */
for (j=i+1; j<nhits; j++) {
if (face1[i] == face1[j] || face1[i] == face2[j] ||
face2[i] == face1[j] || face2[i] == face2[j] ) {
which = j;
break;
}
}
if (which > i+1) {
this.swap(x, i+1, which);
this.swap(face1, i+1, which);
this.swap(face2, i+1, which);
}
}
}
if (nhits >= 3) {
/* Put in order so that the normal points out the FRONT of the faces */
v0 = [x[0][0] - x[1][0] , x[0][1] - x[1][1], x[0][2] - x[1][2]];
v2 = [x[2][0] - x[1][0] , x[2][1] - x[1][1], x[2][2] - x[1][2]];
/* cross-product */
vx = this.xprod(v0, v2);
reverse = this.dotprod(vx, A) > 0;
for (i=0; i<nhits-2; i++) {
obj.vertices.push(x[0]);
normals.push(A);
for (j=1; j<3; j++) {
obj.vertices.push(x[i + (reverse ? 3-j : j)]);
normals.push(A);
}
}
}
}
obj.pnormals = normals;
};
this.initObj = function(id) {
var obj = this.getObj(id),
flags = obj.flags,
type = obj.type,
is_indexed = flags & this.f_is_indexed,
is_lit = flags & this.f_is_lit,
has_texture = flags & this.f_has_texture,
fixed_quads = flags & this.f_fixed_quads,
depth_sort = flags & this.f_depth_sort,
sprites_3d = flags & this.f_sprites_3d,
sprite_3d = flags & this.f_sprite_3d,
gl = this.gl || this.initGL(),
texinfo, drawtype, nclipplanes, f, frowsize, nrows,
i,j,v, mat, uri, matobj;
if (typeof id !== "number") {
this.alertOnce("initObj id is "+typeof id);
}
obj.initialized = true;
if (type === "bboxdeco" || type === "subscene")
return;
if (type === "light") {
obj.ambient = new Float32Array(obj.colors[0].slice(0,3));
obj.diffuse = new Float32Array(obj.colors[1].slice(0,3));
obj.specular = new Float32Array(obj.colors[2].slice(0,3));
obj.lightDir = new Float32Array(obj.vertices[0]);
return;
}
if (type === "clipplanes") {
obj.vClipplane = this.flatten(this.cbind(obj.normals, obj.offsets));
return;
}
if (type == "background" && typeof obj.ids !== "undefined") {
obj.quad = this.flatten([].concat(obj.ids));
return;
}
if (typeof obj.vertices === "undefined")
obj.vertices = [];
v = obj.vertices;
obj.vertexCount = v.length;
if (!obj.vertexCount) return;
if (!sprites_3d) {
if (gl.isContextLost()) return;
obj.prog = gl.createProgram();
gl.attachShader(obj.prog, this.getShader( gl.VERTEX_SHADER,
this.getVertexShader(id) ));
gl.attachShader(obj.prog, this.getShader( gl.FRAGMENT_SHADER,
this.getFragmentShader(id) ));
//  Force aPos to location 0, aCol to location 1
gl.bindAttribLocation(obj.prog, 0, "aPos");
gl.bindAttribLocation(obj.prog, 1, "aCol");
gl.linkProgram(obj.prog);
var linked = gl.getProgramParameter(obj.prog, gl.LINK_STATUS);
if (!linked) {
// An error occurred while linking
var lastError = gl.getProgramInfoLog(obj.prog);
console.warn("Error in program linking:" + lastError);
gl.deleteProgram(obj.prog);
return;
}
}
if (type === "text") {
texinfo = this.drawTextToCanvas(obj.texts,
this.flatten(obj.cex),
this.flatten(obj.family),
this.flatten(obj.family));
}
if (fixed_quads && !sprites_3d) {
obj.ofsLoc = gl.getAttribLocation(obj.prog, "aOfs");
}
if (sprite_3d) {
obj.origLoc = gl.getUniformLocation(obj.prog, "uOrig");
obj.sizeLoc = gl.getUniformLocation(obj.prog, "uSize");
obj.usermatLoc = gl.getUniformLocation(obj.prog, "usermat");
}
if (has_texture || type == "text") {
obj.texture = gl.createTexture();
obj.texLoc = gl.getAttribLocation(obj.prog, "aTexcoord");
obj.sampler = gl.getUniformLocation(obj.prog, "uSampler");
}
if (has_texture) {
mat = obj.material;
if (typeof mat.uri !== "undefined")
uri = mat.uri;
else if (typeof mat.uriElementId === "undefined") {
matobj = this.getObj(mat.uriId);
if (typeof matobj !== "undefined") {
uri = matobj.material.uri;
} else {
uri = "";
}
} else
uri = document.getElementById(mat.uriElementId).rglinstance.getObj(mat.uriId).material.uri;
this.loadImageToTexture(uri, obj.texture);
}
if (type === "text") {
this.handleLoadedTexture(obj.texture, this.textureCanvas);
}
var stride = 3, nc, cofs, nofs, radofs, oofs, tofs, vnew, v1;
nc = obj.colorCount = obj.colors.length;
if (nc > 1) {
cofs = stride;
stride = stride + 4;
v = this.cbind(v, obj.colors);
} else {
cofs = -1;
obj.onecolor = this.flatten(obj.colors);
}
if (typeof obj.normals !== "undefined") {
nofs = stride;
stride = stride + 3;
v = this.cbind(v, typeof obj.pnormals !== "undefined" ? obj.pnormals : obj.normals);
} else
nofs = -1;
if (typeof obj.radii !== "undefined") {
radofs = stride;
stride = stride + 1;
if (obj.radii.length === v.length) {
v = this.cbind(v, obj.radii);
} else if (obj.radii.length === 1) {
v = v.map(function(row, i, arr) { return row.concat(obj.radii[0]);});
}
} else
radofs = -1;
if (type == "sprites" && !sprites_3d) {
tofs = stride;
stride += 2;
oofs = stride;
stride += 2;
vnew = new Array(4*v.length);
var size = obj.radii, s = size[0]/2;
for (i=0; i < v.length; i++) {
if (size.length > 1)
s = size[i]/2;
vnew[4*i]  = v[i].concat([0,0,-s,-s]);
vnew[4*i+1]= v[i].concat([1,0, s,-s]);
vnew[4*i+2]= v[i].concat([1,1, s, s]);
vnew[4*i+3]= v[i].concat([0,1,-s, s]);
}
v = vnew;
obj.vertexCount = v.length;
} else if (type === "text") {
tofs = stride;
stride += 2;
oofs = stride;
stride += 2;
vnew = new Array(4*v.length);
for (i=0; i < v.length; i++) {
vnew[4*i]  = v[i].concat([0,-0.5]).concat(obj.adj[0]);
vnew[4*i+1]= v[i].concat([1,-0.5]).concat(obj.adj[0]);
vnew[4*i+2]= v[i].concat([1, 1.5]).concat(obj.adj[0]);
vnew[4*i+3]= v[i].concat([0, 1.5]).concat(obj.adj[0]);
for (j=0; j < 4; j++) {
v1 = vnew[4*i+j];
v1[tofs+2] = 2*(v1[tofs]-v1[tofs+2])*texinfo.widths[i];
v1[tofs+3] = 2*(v1[tofs+1]-v1[tofs+3])*texinfo.textHeights[i];
v1[tofs] *= texinfo.widths[i]/texinfo.canvasX;
v1[tofs+1] = 1.0-(texinfo.offsets[i] -
v1[tofs+1]*texinfo.textHeights[i])/texinfo.canvasY;
vnew[4*i+j] = v1;
}
}
v = vnew;
obj.vertexCount = v.length;
} else if (typeof obj.texcoords !== "undefined") {
tofs = stride;
stride += 2;
oofs = -1;
v = this.cbind(v, obj.texcoords);
} else {
tofs = -1;
oofs = -1;
}
if (stride !== v[0].length) {
this.alertOnce("problem in stride calculation");
}
obj.vOffsets = {vofs:0, cofs:cofs, nofs:nofs, radofs:radofs, oofs:oofs, tofs:tofs, stride:stride};
obj.values = new Float32Array(this.flatten(v));
if (sprites_3d) {
obj.userMatrix = new CanvasMatrix4(obj.userMatrix);
obj.objects = this.flatten([].concat(obj.ids));
is_lit = false;
}
if (is_lit && !fixed_quads) {
obj.normLoc = gl.getAttribLocation(obj.prog, "aNorm");
}
nclipplanes = this.countClipplanes();
if (nclipplanes && !sprites_3d) {
obj.clipLoc = [];
for (i=0; i < nclipplanes; i++)
obj.clipLoc[i] = gl.getUniformLocation(obj.prog,"vClipplane" + i);
}
if (is_lit) {
obj.emissionLoc = gl.getUniformLocation(obj.prog, "emission");
obj.emission = new Float32Array(this.stringToRgb(this.getMaterial(id, "emission")));
obj.shininessLoc = gl.getUniformLocation(obj.prog, "shininess");
obj.shininess = this.getMaterial(id, "shininess");
obj.nlights = this.countLights();
obj.ambientLoc = [];
obj.ambient = new Float32Array(this.stringToRgb(this.getMaterial(id, "ambient")));
obj.specularLoc = [];
obj.specular = new Float32Array(this.stringToRgb(this.getMaterial(id, "specular")));
obj.diffuseLoc = [];
obj.lightDirLoc = [];
obj.viewpointLoc = [];
obj.finiteLoc = [];
for (i=0; i < obj.nlights; i++) {
obj.ambientLoc[i] = gl.getUniformLocation(obj.prog, "ambient" + i);
obj.specularLoc[i] = gl.getUniformLocation(obj.prog, "specular" + i);
obj.diffuseLoc[i] = gl.getUniformLocation(obj.prog, "diffuse" + i);
obj.lightDirLoc[i] = gl.getUniformLocation(obj.prog, "lightDir" + i);
obj.viewpointLoc[i] = gl.getUniformLocation(obj.prog, "viewpoint" + i);
obj.finiteLoc[i] = gl.getUniformLocation(obj.prog, "finite" + i);
}
}
if (is_indexed) {
if ((type === "quads" || type === "text" ||
type === "sprites") && !sprites_3d) {
nrows = Math.floor(obj.vertexCount/4);
f = Array(6*nrows);
for (i=0; i < nrows; i++) {
f[6*i] = 4*i;
f[6*i+1] = 4*i + 1;
f[6*i+2] = 4*i + 2;
f[6*i+3] = 4*i;
f[6*i+4] = 4*i + 2;
f[6*i+5] = 4*i + 3;
}
frowsize = 6;
} else if (type === "triangles") {
nrows = Math.floor(obj.vertexCount/3);
f = Array(3*nrows);
for (i=0; i < f.length; i++) {
f[i] = i;
}
frowsize = 3;
} else if (type === "spheres") {
nrows = obj.vertexCount;
f = Array(nrows);
for (i=0; i < f.length; i++) {
f[i] = i;
}
frowsize = 1;
} else if (type === "surface") {
var dim = obj.dim[0],
nx = dim[0],
nz = dim[1];
f = [];
for (j=0; j<nx-1; j++) {
for (i=0; i<nz-1; i++) {
f.push(j + nx*i,
j + nx*(i+1),
j + 1 + nx*(i+1),
j + nx*i,
j + 1 + nx*(i+1),
j + 1 + nx*i);
}
}
frowsize = 6;
}
obj.f = new Uint16Array(f);
if (depth_sort) {
drawtype = "DYNAMIC_DRAW";
} else {
drawtype = "STATIC_DRAW";
}
}
if (type !== "spheres" && !sprites_3d) {
obj.buf = gl.createBuffer();
gl.bindBuffer(gl.ARRAY_BUFFER, obj.buf);
gl.bufferData(gl.ARRAY_BUFFER, obj.values, gl.STATIC_DRAW); //
}
if (is_indexed && type !== "spheres" && !sprites_3d) {
obj.ibuf = gl.createBuffer();
gl.bindBuffer(gl.ELEMENT_ARRAY_BUFFER, obj.ibuf);
gl.bufferData(gl.ELEMENT_ARRAY_BUFFER, obj.f, gl[drawtype]);
}
if (!sprites_3d) {
obj.mvMatLoc = gl.getUniformLocation(obj.prog, "mvMatrix");
obj.prMatLoc = gl.getUniformLocation(obj.prog, "prMatrix");
}
if (type === "text") {
obj.textScaleLoc = gl.getUniformLocation(obj.prog, "textScale");
}
if (is_lit && !sprites_3d) {
obj.normMatLoc = gl.getUniformLocation(obj.prog, "normMatrix");
}
};
this.setDepthTest = function(id) {
var gl = this.gl || this.initGL(),
tests = {never: gl.NEVER,
less:  gl.LESS,
equal: gl.EQUAL,
lequal:gl.LEQUAL,
greater: gl.GREATER,
notequal: gl.NOTEQUAL,
gequal: gl.GEQUAL,
always: gl.ALWAYS},
test = tests[this.getMaterial(id, "depth_test")];
gl.depthFunc(test);
};
this.mode4type = {points : "POINTS",
linestrip : "LINE_STRIP",
abclines : "LINES",
lines : "LINES",
sprites : "TRIANGLES",
planes : "TRIANGLES",
text : "TRIANGLES",
quads : "TRIANGLES",
surface : "TRIANGLES",
triangles : "TRIANGLES"};
this.drawObj = function(id, subsceneid) {
var obj = this.getObj(id),
subscene = this.getObj(subsceneid),
flags = obj.flags,
type = obj.type,
is_indexed = flags & this.f_is_indexed,
is_lit = flags & this.f_is_lit,
has_texture = flags & this.f_has_texture,
fixed_quads = flags & this.f_fixed_quads,
depth_sort = flags & this.f_depth_sort,
sprites_3d = flags & this.f_sprites_3d,
sprite_3d = flags & this.f_sprite_3d,
is_lines = flags & this.f_is_lines,
gl = this.gl || this.initGL(),
sphereMV, baseofs, ofs, sscale, i, count, light,
faces;
if (typeof id !== "number") {
this.alertOnce("drawObj id is "+typeof id);
}
if (type === "planes") {
if (obj.bbox !== subscene.par3d.bbox || !obj.initialized) {
this.planeUpdateTriangles(id, subscene.par3d.bbox);
}
}
if (!obj.initialized)
this.initObj(id);
if (type === "clipplanes") {
count = obj.offsets.length;
var IMVClip = [];
for (i=0; i < count; i++) {
IMVClip[i] = this.multMV(this.invMatrix, obj.vClipplane.slice(4*i, 4*(i+1)));
}
obj.IMVClip = IMVClip;
return;
}
if (type === "light" || type === "bboxdeco" || !obj.vertexCount)
return;
this.setDepthTest(id);
if (sprites_3d) {
var norigs = obj.vertices.length,
savenorm = new CanvasMatrix4(this.normMatrix);
this.origs = obj.vertices;
this.usermat = new Float32Array(obj.userMatrix.getAsArray());
this.radii = obj.radii;
this.normMatrix = subscene.spriteNormmat;
for (this.iOrig=0; this.iOrig < norigs; this.iOrig++) {
for (i=0; i < obj.objects.length; i++) {
this.drawObj(obj.objects[i], subsceneid);
}
}
this.normMatrix = savenorm;
return;
} else {
gl.useProgram(obj.prog);
}
if (sprite_3d) {
gl.uniform3fv(obj.origLoc, new Float32Array(this.origs[this.iOrig]));
if (this.radii.length > 1) {
gl.uniform1f(obj.sizeLoc, this.radii[this.iOrig][0]);
} else {
gl.uniform1f(obj.sizeLoc, this.radii[0][0]);
}
gl.uniformMatrix4fv(obj.usermatLoc, false, this.usermat);
}
if (type === "spheres") {
gl.bindBuffer(gl.ARRAY_BUFFER, this.sphere.buf);
} else {
gl.bindBuffer(gl.ARRAY_BUFFER, obj.buf);
}
if (is_indexed && type !== "spheres") {
gl.bindBuffer(gl.ELEMENT_ARRAY_BUFFER, obj.ibuf);
} else if (type === "spheres") {
gl.bindBuffer(gl.ELEMENT_ARRAY_BUFFER, this.sphere.ibuf);
}
gl.uniformMatrix4fv( obj.prMatLoc, false, new Float32Array(this.prMatrix.getAsArray()) );
gl.uniformMatrix4fv( obj.mvMatLoc, false, new Float32Array(this.mvMatrix.getAsArray()) );
var clipcheck = 0,
clipplaneids = subscene.clipplanes,
clip, j;
for (i=0; i < clipplaneids.length; i++) {
clip = this.getObj(clipplaneids[i]);
for (j=0; j < clip.offsets.length; j++) {
gl.uniform4fv(obj.clipLoc[clipcheck + j], clip.IMVClip[j]);
}
clipcheck += clip.offsets.length;
}
if (typeof obj.clipLoc !== "undefined")
for (i=clipcheck; i < obj.clipLoc.length; i++)
gl.uniform4f(obj.clipLoc[i], 0,0,0,0);
if (is_lit) {
gl.uniformMatrix4fv( obj.normMatLoc, false, new Float32Array(this.normMatrix.getAsArray()) );
gl.uniform3fv( obj.emissionLoc, obj.emission);
gl.uniform1f( obj.shininessLoc, obj.shininess);
for (i=0; i < subscene.lights.length; i++) {
light = this.getObj(subscene.lights[i]);
gl.uniform3fv( obj.ambientLoc[i], this.componentProduct(light.ambient, obj.ambient));
gl.uniform3fv( obj.specularLoc[i], this.componentProduct(light.specular, obj.specular));
gl.uniform3fv( obj.diffuseLoc[i], light.diffuse);
gl.uniform3fv( obj.lightDirLoc[i], light.lightDir);
gl.uniform1i( obj.viewpointLoc[i], light.viewpoint);
gl.uniform1i( obj.finiteLoc[i], light.finite);
}
for (i=subscene.lights.length; i < obj.nlights; i++) {
gl.uniform3f( obj.ambientLoc[i], 0,0,0);
gl.uniform3f( obj.specularLoc[i], 0,0,0);
gl.uniform3f( obj.diffuseLoc[i], 0,0,0);
}
}
if (type === "text") {
gl.uniform2f( obj.textScaleLoc, 0.75/this.vp.width, 0.75/this.vp.height);
}
gl.enableVertexAttribArray( this.posLoc );
var nc = obj.colorCount;
count = obj.vertexCount;
if (depth_sort) {
var nfaces = obj.centers.length,
frowsize, z, w;
if (sprites_3d) frowsize = 1;
else if (type === "triangles") frowsize = 3;
else frowsize = 6;
var depths = new Float32Array(nfaces);
faces = new Array(nfaces);
for(i=0; i<nfaces; i++) {
z = this.prmvMatrix.m13*obj.centers[3*i] +
this.prmvMatrix.m23*obj.centers[3*i+1] +
this.prmvMatrix.m33*obj.centers[3*i+2] +
this.prmvMatrix.m43;
w = this.prmvMatrix.m14*obj.centers[3*i] +
this.prmvMatrix.m24*obj.centers[3*i+1] +
this.prmvMatrix.m34*obj.centers[3*i+2] +
this.prmvMatrix.m44;
depths[i] = z/w;
faces[i] = i;
}
var depthsort = function(i,j) { return depths[j] - depths[i]; };
faces.sort(depthsort);
if (type !== "spheres") {
var f = new Uint16Array(obj.f.length);
for (i=0; i<nfaces; i++) {
for (j=0; j<frowsize; j++) {
f[frowsize*i + j] = obj.f[frowsize*faces[i] + j];
}
}
gl.bufferData(gl.ELEMENT_ARRAY_BUFFER, f, gl.DYNAMIC_DRAW);
}
}
if (type === "spheres") {
subscene = this.getObj(subsceneid);
var scale = subscene.par3d.scale,
scount = count;
gl.vertexAttribPointer(this.posLoc,  3, gl.FLOAT, false, 4*this.sphere.vOffsets.stride,  0);
gl.enableVertexAttribArray(obj.normLoc );
gl.vertexAttribPointer(obj.normLoc,  3, gl.FLOAT, false, 4*this.sphere.vOffsets.stride,  0);
gl.disableVertexAttribArray( this.colLoc );
var sphereNorm = new CanvasMatrix4();
sphereNorm.scale(scale[0], scale[1], scale[2]);
sphereNorm.multRight(this.normMatrix);
gl.uniformMatrix4fv( obj.normMatLoc, false, new Float32Array(sphereNorm.getAsArray()) );
if (nc == 1) {
gl.vertexAttrib4fv( this.colLoc, new Float32Array(obj.onecolor));
}
if (has_texture) {
gl.enableVertexAttribArray( obj.texLoc );
gl.vertexAttribPointer(obj.texLoc, 2, gl.FLOAT, false, 4*this.sphere.vOffsets.stride, 4*this.sphere.vOffsets.tofs);
gl.activeTexture(gl.TEXTURE0);
gl.bindTexture(gl.TEXTURE_2D, obj.texture);
gl.uniform1i( obj.sampler, 0);
}
for (i = 0; i < scount; i++) {
sphereMV = new CanvasMatrix4();
if (depth_sort) {
baseofs = faces[i]*obj.vOffsets.stride;
} else {
baseofs = i*obj.vOffsets.stride;
}
ofs = baseofs + obj.vOffsets.radofs;
sscale = obj.values[ofs];
sphereMV.scale(sscale/scale[0], sscale/scale[1], sscale/scale[2]);
sphereMV.translate(obj.values[baseofs],
obj.values[baseofs+1],
obj.values[baseofs+2]);
sphereMV.multRight(this.mvMatrix);
gl.uniformMatrix4fv( obj.mvMatLoc, false, new Float32Array(sphereMV.getAsArray()) );
if (nc > 1) {
ofs = baseofs + obj.vOffsets.cofs;
gl.vertexAttrib4f( this.colLoc, obj.values[ofs],
obj.values[ofs+1],
obj.values[ofs+2],
obj.values[ofs+3] );
}
gl.drawElements(gl.TRIANGLES, this.sphere.sphereCount, gl.UNSIGNED_SHORT, 0);
}
return;
} else {
if (obj.colorCount === 1) {
gl.disableVertexAttribArray( this.colLoc );
gl.vertexAttrib4fv( this.colLoc, new Float32Array(obj.onecolor));
} else {
gl.enableVertexAttribArray( this.colLoc );
gl.vertexAttribPointer(this.colLoc, 4, gl.FLOAT, false, 4*obj.vOffsets.stride, 4*obj.vOffsets.cofs);
}
}
if (is_lit && obj.vOffsets.nofs > 0) {
gl.enableVertexAttribArray( obj.normLoc );
gl.vertexAttribPointer(obj.normLoc, 3, gl.FLOAT, false, 4*obj.vOffsets.stride, 4*obj.vOffsets.nofs);
}
if (has_texture || type === "text") {
gl.enableVertexAttribArray( obj.texLoc );
gl.vertexAttribPointer(obj.texLoc, 2, gl.FLOAT, false, 4*obj.vOffsets.stride, 4*obj.vOffsets.tofs);
gl.activeTexture(gl.TEXTURE0);
gl.bindTexture(gl.TEXTURE_2D, obj.texture);
gl.uniform1i( obj.sampler, 0);
}
if (fixed_quads) {
gl.enableVertexAttribArray( obj.ofsLoc );
gl.vertexAttribPointer(obj.ofsLoc, 2, gl.FLOAT, false, 4*obj.vOffsets.stride, 4*obj.vOffsets.oofs);
}
var mode = this.mode4type[type];
if (type === "sprites" || type === "text" || type === "quads") {
count = count * 6/4;
} else if (type === "surface") {
count = obj.f.length;
}
if (is_lines) {
gl.lineWidth( this.getMaterial(id, "lwd") );
}
gl.vertexAttribPointer(this.posLoc,  3, gl.FLOAT, false, 4*obj.vOffsets.stride,  4*obj.vOffsets.vofs);
if (is_indexed) {
gl.drawElements(gl[mode], count, gl.UNSIGNED_SHORT, 0);
} else {
gl.drawArrays(gl[mode], 0, count);
}
};
this.drawBackground = function(id, subsceneid) {
var gl = this.gl || this.initGL(),
obj = this.getObj(id),
bg, i;
if (!obj.initialized)
this.initObj(id);
if (obj.colors.length) {
bg = obj.colors[0];
gl.clearColor(bg[0], bg[1], bg[2], bg[3]);
gl.depthMask(true);
gl.clear(gl.COLOR_BUFFER_BIT | gl.DEPTH_BUFFER_BIT);
}
if (typeof obj.quad !== "undefined") {
this.prMatrix.makeIdentity();
this.mvMatrix.makeIdentity();
gl.disable(gl.BLEND);
gl.disable(gl.DEPTH_TEST);
gl.depthMask(false);
for (i=0; i < obj.quad.length; i++)
this.drawObj(obj.quad[i], subsceneid);
}
};
this.drawSubscene = function(subsceneid) {
var gl = this.gl || this.initGL(),
obj = this.getObj(subsceneid),
objects = this.scene.objects,
subids = obj.objects,
subscene_has_faces = false,
subscene_needs_sorting = false,
flags, i;
if (obj.par3d.skipRedraw)
return;
for (i=0; i < subids.length; i++) {
flags = objects[subids[i]].flags;
if (typeof flags !== "undefined") {
subscene_has_faces |= (flags & this.f_is_lit)
& !(flags & this.f_fixed_quads);
subscene_needs_sorting |= (flags & this.f_depth_sort);
}
}
this.setViewport(subsceneid);
if (typeof obj.backgroundId !== "undefined")
this.drawBackground(obj.backgroundId, subsceneid);
if (subids.length) {
this.setprMatrix(subsceneid);
this.setmvMatrix(subsceneid);
if (subscene_has_faces) {
this.setnormMatrix(subsceneid);
if ((obj.flags & this.f_sprites_3d) &&
typeof obj.spriteNormmat === "undefined") {
obj.spriteNormmat = new CanvasMatrix4(this.normMatrix);
}
}
if (subscene_needs_sorting)
this.setprmvMatrix();
gl.enable(gl.DEPTH_TEST);
gl.depthMask(true);
gl.disable(gl.BLEND);
var clipids = obj.clipplanes;
if (typeof clipids === "undefined") {
console.warn("bad clipids");
}
if (clipids.length > 0) {
this.invMatrix = new CanvasMatrix4(this.mvMatrix);
this.invMatrix.invert();
for (i = 0; i < clipids.length; i++)
this.drawObj(clipids[i], subsceneid);
}
subids = obj.opaque;
if (subids.length > 0) {
for (i = 0; i < subids.length; i++) {
this.drawObj(subids[i], subsceneid);
}
}
subids = obj.transparent;
if (subids.length > 0) {
gl.depthMask(false);
gl.blendFuncSeparate(gl.SRC_ALPHA, gl.ONE_MINUS_SRC_ALPHA,
gl.ONE, gl.ONE);
gl.enable(gl.BLEND);
for (i = 0; i < subids.length; i++) {
this.drawObj(subids[i], subsceneid);
}
}
subids = obj.subscenes;
for (i = 0; i < subids.length; i++) {
this.drawSubscene(subids[i]);
}
}
};
this.relMouseCoords = function(event) {
var totalOffsetX = 0,
totalOffsetY = 0,
currentElement = this.canvas;
do {
totalOffsetX += currentElement.offsetLeft;
totalOffsetY += currentElement.offsetTop;
currentElement = currentElement.offsetParent;
}
while(currentElement);
var canvasX = event.pageX - totalOffsetX,
canvasY = event.pageY - totalOffsetY;
return {x:canvasX, y:canvasY};
};
this.setMouseHandlers = function() {
var self = this, activeSubscene, handler,
handlers = {}, drag = 0;
handlers.rotBase = 0;
this.screenToVector = function(x, y) {
var viewport = this.getObj(activeSubscene).par3d.viewport,
width = viewport.width*this.canvas.width,
height = viewport.height*this.canvas.height,
radius = Math.max(width, height)/2.0,
cx = width/2.0,
cy = height/2.0,
px = (x-cx)/radius,
py = (y-cy)/radius,
plen = Math.sqrt(px*px+py*py);
if (plen > 1.e-6) {
px = px/plen;
py = py/plen;
}
var angle = (Math.SQRT2 - plen)/Math.SQRT2*Math.PI/2,
z = Math.sin(angle),
zlen = Math.sqrt(1.0 - z*z);
px = px * zlen;
py = py * zlen;
return [px, py, z];
};
handlers.trackballdown = function(x,y) {
var activeSub = this.getObj(activeSubscene),
activeModel = this.getObj(this.useid(activeSub.id, "model")),
i, l = activeModel.par3d.listeners;
handlers.rotBase = this.screenToVector(x, y);
this.saveMat = [];
for (i = 0; i < l.length; i++) {
activeSub = this.getObj(l[i]);
activeSub.saveMat = new CanvasMatrix4(activeSub.par3d.userMatrix);
}
};
handlers.trackballmove = function(x,y) {
var rotCurrent = this.screenToVector(x,y),
rotBase = handlers.rotBase,
dot = rotBase[0]*rotCurrent[0] +
rotBase[1]*rotCurrent[1] +
rotBase[2]*rotCurrent[2],
angle = Math.acos( dot/this.vlen(rotBase)/this.vlen(rotCurrent) )*180.0/Math.PI,
axis = this.xprod(rotBase, rotCurrent),
objects = this.scene.objects,
activeSub = this.getObj(activeSubscene),
activeModel = this.getObj(this.useid(activeSub.id, "model")),
l = activeModel.par3d.listeners,
i;
for (i = 0; i < l.length; i++) {
activeSub = this.getObj(l[i]);
activeSub.par3d.userMatrix.load(objects[l[i]].saveMat);
activeSub.par3d.userMatrix.rotate(angle, axis[0], axis[1], axis[2]);
}
this.drawScene();
};
handlers.trackballend = 0;
handlers.axisdown = function(x,y) {
handlers.rotBase = this.screenToVector(x, this.canvas.height/2);
var activeSub = this.getObj(activeSubscene),
activeModel = this.getObj(this.useid(activeSub.id, "model")),
i, l = activeModel.par3d.listeners;
for (i = 0; i < l.length; i++) {
activeSub = this.getObj(l[i]);
activeSub.saveMat = new CanvasMatrix4(activeSub.par3d.userMatrix);
}
};
handlers.axismove = function(x,y) {
var rotCurrent = this.screenToVector(x, this.canvas.height/2),
rotBase = handlers.rotBase,
angle = (rotCurrent[0] - rotBase[0])*180/Math.PI,
rotMat = new CanvasMatrix4();
rotMat.rotate(angle, handlers.axis[0], handlers.axis[1], handlers.axis[2]);
var activeSub = this.getObj(activeSubscene),
activeModel = this.getObj(this.useid(activeSub.id, "model")),
i, l = activeModel.par3d.listeners;
for (i = 0; i < l.length; i++) {
activeSub = this.getObj(l[i]);
activeSub.par3d.userMatrix.load(activeSub.saveMat);
activeSub.par3d.userMatrix.multLeft(rotMat);
}
this.drawScene();
};
handlers.axisend = 0;
handlers.y0zoom = 0;
handlers.zoom0 = 0;
handlers.zoomdown = function(x, y) {
var activeSub = this.getObj(activeSubscene),
activeProjection = this.getObj(this.useid(activeSub.id, "projection")),
i, l = activeProjection.par3d.listeners;
handlers.y0zoom = y;
for (i = 0; i < l.length; i++) {
activeSub = this.getObj(l[i]);
activeSub.zoom0 = Math.log(activeSub.par3d.zoom);
}
};
handlers.zoommove = function(x, y) {
var activeSub = this.getObj(activeSubscene),
activeProjection = this.getObj(this.useid(activeSub.id, "projection")),
i, l = activeProjection.par3d.listeners;
for (i = 0; i < l.length; i++) {
activeSub = this.getObj(l[i]);
activeSub.par3d.zoom = Math.exp(activeSub.zoom0 + (y-handlers.y0zoom)/this.canvas.height);
}
this.drawScene();
};
handlers.zoomend = 0;
handlers.y0fov = 0;
handlers.fovdown = function(x, y) {
handlers.y0fov = y;
var activeSub = this.getObj(activeSubscene),
activeProjection = this.getObj(this.useid(activeSub.id, "projection")),
i, l = activeProjection.par3d.listeners;
for (i = 0; i < l.length; i++) {
activeSub = this.getObj(l[i]);
activeSub.fov0 = activeSub.par3d.FOV;
}
};
handlers.fovmove = function(x, y) {
var activeSub = this.getObj(activeSubscene),
activeProjection = this.getObj(this.useid(activeSub.id, "projection")),
i, l = activeProjection.par3d.listeners;
for (i = 0; i < l.length; i++) {
activeSub = this.getObj(l[i]);
activeSub.par3d.FOV = Math.max(1, Math.min(179, activeSub.fov0 +
180*(y-handlers.y0fov)/this.canvas.height));
}
this.drawScene();
};
handlers.fovend = 0;
this.canvas.onmousedown = function ( ev ){
if (!ev.which) // Use w3c defns in preference to MS
switch (ev.button) {
case 0: ev.which = 1; break;
case 1:
case 4: ev.which = 2; break;
case 2: ev.which = 3;
}
drag = ["left", "middle", "right"][ev.which-1];
var coords = self.relMouseCoords(ev);
coords.y = self.canvas.height-coords.y;
activeSubscene = self.whichSubscene(coords);
var sub = self.getObj(activeSubscene), f;
handler = sub.par3d.mouseMode[drag];
switch (handler) {
case "xAxis":
handler = "axis";
handlers.axis = [1.0, 0.0, 0.0];
break;
case "yAxis":
handler = "axis";
handlers.axis = [0.0, 1.0, 0.0];
break;
case "zAxis":
handler = "axis";
handlers.axis = [0.0, 0.0, 1.0];
break;
}
f = handlers[handler + "down"];
if (f) {
coords = self.translateCoords(activeSubscene, coords);
f.call(self, coords.x, coords.y);
ev.preventDefault();
}
};
this.canvas.onmouseup = function ( ev ){
if ( drag === 0 ) return;
var f = handlers[handler + "up"];
if (f)
f();
drag = 0;
};
this.canvas.onmouseout = this.canvas.onmouseup;
this.canvas.onmousemove = function ( ev ) {
if ( drag === 0 ) return;
var f = handlers[handler + "move"];
if (f) {
var coords = self.relMouseCoords(ev);
coords.y = self.canvas.height - coords.y;
coords = self.translateCoords(activeSubscene, coords);
f.call(self, coords.x, coords.y);
}
};
handlers.wheelHandler = function(ev) {
var del = 1.02, i;
if (ev.shiftKey) del = 1.002;
var ds = ((ev.detail || ev.wheelDelta) > 0) ? del : (1 / del);
if (typeof activeSubscene === "undefined")
activeSubscene = self.scene.rootSubscene;
var activeSub = self.getObj(activeSubscene),
activeProjection = self.getObj(self.useid(activeSub.id, "projection")),
l = activeProjection.par3d.listeners;
for (i = 0; i < l.length; i++) {
activeSub = self.getObj(l[i]);
activeSub.par3d.zoom *= ds;
}
self.drawScene();
ev.preventDefault();
};
this.canvas.addEventListener("DOMMouseScroll", handlers.wheelHandler, false);
this.canvas.addEventListener("mousewheel", handlers.wheelHandler, false);
};
this.useid = function(subsceneid, type) {
var sub = this.getObj(subsceneid);
if (sub.embeddings[type] === "inherit")
return(this.useid(sub.parent, type));
else
return subsceneid;
};
this.inViewport = function(coords, subsceneid) {
var viewport = this.getObj(subsceneid).par3d.viewport,
x0 = coords.x - viewport.x*this.canvas.width,
y0 = coords.y - viewport.y*this.canvas.height;
return 0 <= x0 && x0 <= viewport.width*this.canvas.width &&
0 <= y0 && y0 <= viewport.height*this.canvas.height;
};
this.whichSubscene = function(coords) {
var self = this,
recurse = function(subsceneid) {
var subscenes = self.getChildSubscenes(subsceneid), i, id;
for (i=0; i < subscenes.length; i++) {
id = recurse(subscenes[i]);
if (typeof(id) !== "undefined")
return(id);
}
if (self.inViewport(coords, subsceneid))
return(subsceneid);
else
return undefined;
},
rootid = this.scene.rootSubscene,
result = recurse(rootid);
if (typeof(result) === "undefined")
result = rootid;
return result;
};
this.translateCoords = function(subsceneid, coords) {
var viewport = this.getObj(subsceneid).par3d.viewport;
return {x: coords.x - viewport.x*this.canvas.width,
y: coords.y - viewport.y*this.canvas.height};
};
this.initSphere = function() {
var verts = this.scene.sphereVerts, 
reuse = verts.reuse, result;
if (typeof reuse !== "undefined") {
var prev = document.getElementById(reuse).rglinstance.sphere;
result = {values: prev.values, vOffsets: prev.vOffsets, it: prev.it};
} else 
result = {values: new Float32Array(this.flatten(this.cbind(this.transpose(verts.vb),
this.transpose(verts.texcoords)))),
it: new Uint16Array(this.flatten(this.transpose(verts.it))),
vOffsets: {vofs:0, cofs:-1, nofs:-1, radofs:-1, oofs:-1, 
tofs:3, stride:5}};
result.sphereCount = result.it.length;
this.sphere = result;
};
this.initSphereGL = function() {
var gl = this.gl || this.initGL(), sphere = this.sphere;
if (gl.isContextLost()) return;
sphere.buf = gl.createBuffer();
gl.bindBuffer(gl.ARRAY_BUFFER, sphere.buf);
gl.bufferData(gl.ARRAY_BUFFER, sphere.values, gl.STATIC_DRAW);
sphere.ibuf = gl.createBuffer();
gl.bindBuffer(gl.ELEMENT_ARRAY_BUFFER, sphere.ibuf);
gl.bufferData(gl.ELEMENT_ARRAY_BUFFER, sphere.it, gl.STATIC_DRAW);
return;
};
this.initialize = function(el, x) {
this.textureCanvas = document.createElement("canvas");
this.textureCanvas.style.display = "block";
this.scene = x;
this.normMatrix = new CanvasMatrix4();
this.saveMat = {};
this.distance = null;
this.posLoc = 0;
this.colLoc = 1;
if (el) {
el.rglinstance = this;
this.el = el;
this.webGLoptions = el.rglinstance.scene.webGLoptions;
this.initCanvas();
}
};
this.restartCanvas = function() {
var newcanvas = document.createElement("canvas");
newcanvas.width = this.el.width;
newcanvas.height = this.el.height;
newcanvas.addEventListener("webglcontextrestored",
this.onContextRestored, false);
newcanvas.addEventListener("webglcontextlost",
this.onContextLost, false);            
while (this.el.firstChild) {
this.el.removeChild(this.el.firstChild);
}
this.el.appendChild(newcanvas);
this.canvas = newcanvas;
this.gl = null;      	
}
this.initCanvas = function() {
this.restartCanvas();
var objs = this.scene.objects,
self = this;
Object.keys(objs).forEach(function(key){
var id = parseInt(key, 10),
obj = self.getObj(id);
if (typeof obj.reuse !== "undefined")
self.copyObj(id, obj.reuse);
});
Object.keys(objs).forEach(function(key){
self.initSubscene(parseInt(key, 10));
});
this.setMouseHandlers();      
this.initSphere();
this.onContextRestored = function(event) {
self.initGL();
self.drawScene();
console.log("restored context for "+self.scene.rootSubscene);
}
this.onContextLost = function(event) {
if (!self.drawing)
self.restartCanvas();
event.preventDefault();
}
this.initGL0();
lazyLoadScene = function() {
if (self.isInBrowserViewport()) {
if (!self.gl) {
self.initGL();
}
self.drawScene();
}
}
window.addEventListener("DOMContentLoaded", lazyLoadScene, false);
window.addEventListener("load", lazyLoadScene, false);
window.addEventListener("resize", lazyLoadScene, false);
window.addEventListener("scroll", lazyLoadScene, false);
};
/* this is only used by writeWebGL; rglwidget has
no debug element and does the drawing in rglwidget.js */
this.start = function() {
if (typeof this.prefix !== "undefined") {
this.debugelement = document.getElementById(this.prefix + "debug");
this.debug("");
}
this.drag = 0;
this.drawScene();
};
this.debug = function(msg, img) {
if (typeof this.debugelement !== "undefined" && this.debugelement !== null) {
this.debugelement.innerHTML = msg;
if (typeof img !== "undefined") {
this.debugelement.insertBefore(img, this.debugelement.firstChild);
}
} else if (msg !== "")
alert(msg);
};
this.getSnapshot = function() {
var img;
if (typeof this.scene.snapshot !== "undefined") {
img = document.createElement("img");
img.src = this.scene.snapshot;
img.alt = "Snapshot";
}
return img;
};
this.initGL0 = function() {
if (!window.WebGLRenderingContext){
alert("Your browser does not support WebGL. See http://get.webgl.org");
return;
}
};
this.isInBrowserViewport = function() {
var rect = this.canvas.getBoundingClientRect(),
windHeight = (window.innerHeight || document.documentElement.clientHeight),
windWidth = (window.innerWidth || document.documentElement.clientWidth);
return (
rect.top >= -windHeight &&
rect.left >= -windWidth &&
rect.bottom <= 2*windHeight &&
rect.right <= 2*windWidth);
}
this.initGL = function() {
var self = this;
if (this.gl) {
if (!this.drawing && this.gl.isContextLost())
this.restartCanvas();
else
return this.gl;
}
// if (!this.isInBrowserViewport()) return; Return what??? At this point we know this.gl is null.
this.canvas.addEventListener("webglcontextrestored",
this.onContextRestored, false);
this.canvas.addEventListener("webglcontextlost",
this.onContextLost, false);      
this.gl = this.canvas.getContext("webgl", this.webGLoptions) ||
this.canvas.getContext("experimental-webgl", this.webGLoptions);
var save = this.startDrawing();
this.initSphereGL(); 
Object.keys(this.scene.objects).forEach(function(key){
self.initObj(parseInt(key, 10));
});
this.stopDrawing(save);
return this.gl;
};
this.resize = function(el) {
this.canvas.width = el.width;
this.canvas.height = el.height;
};
this.drawScene = function() {
var gl = this.gl || this.initGL(),
save = this.startDrawing();
gl.enable(gl.DEPTH_TEST);
gl.depthFunc(gl.LEQUAL);
gl.clearDepth(1.0);
gl.clearColor(1,1,1,1);
gl.depthMask(true); // Must be true before clearing depth buffer
gl.clear(gl.COLOR_BUFFER_BIT | gl.DEPTH_BUFFER_BIT);
this.drawSubscene(this.scene.rootSubscene);
this.stopDrawing(save);
};
this.subsetSetter = function(el, control) {
if (typeof control.subscenes === "undefined" ||
control.subscenes === null)
control.subscenes = this.scene.rootSubscene;
var value = Math.round(control.value),
subscenes = [].concat(control.subscenes),
i, j, entries, subsceneid,
ismissing = function(x) {
return control.fullset.indexOf(x) < 0;
},
tointeger = function(x) {
return parseInt(x, 10);
};
for (i=0; i < subscenes.length; i++) {
subsceneid = subscenes[i];
if (typeof this.getObj(subsceneid) === "undefined")
this.alertOnce("typeof object is undefined");
entries = this.getObj(subsceneid).objects;
entries = entries.filter(ismissing);
if (control.accumulate) {
for (j=0; j<=value; j++)
entries = entries.concat(control.subsets[j]);
} else {
entries = entries.concat(control.subsets[value]);
}
entries = entries.map(tointeger);
this.setSubsceneEntries(this.unique(entries), subsceneid);
}
};
this.propertySetter = function(el, control)  {
var value = control.value,
values = [].concat(control.values),
svals = [].concat(control.param),
direct = values[0] === null,
entries = [].concat(control.entries),
ncol = entries.length,
nrow = values.length/ncol,
properties = this.repeatToLen(control.properties, ncol),
objids = this.repeatToLen(control.objids, ncol),
property = properties[0], objid = objids[0],
obj = this.getObj(objid),
propvals, i, v1, v2, p, entry, gl, needsBinding,
newprop, newid,
getPropvals = function() {
if (property === "userMatrix")
return obj.userMatrix.getAsArray();
else
return obj[property];
};
if (direct && typeof value === "undefined")
return;
if (control.interp) {
values = values.slice(0, ncol).concat(values).
concat(values.slice(ncol*(nrow-1), ncol*nrow));
svals = [-Infinity].concat(svals).concat(Infinity);
for (i = 1; i < svals.length; i++) {
if (value <= svals[i]) {
if (svals[i] === Infinity)
p = 1;
else
p = (svals[i] - value)/(svals[i] - svals[i-1]);
break;
}
}
} else if (!direct) {
value = Math.round(value);
}
propvals = getPropvals();
for (j=0; j<entries.length; j++) {
entry = entries[j];
newprop = properties[j];
newid = objids[j];
if (newprop != property || newid != objid) {
property = newprop;
objid = newid;
obj = this.getObj(objid);
propvals = getPropvals();
}
if (control.interp) {
v1 = values[ncol*(i-1) + j];
v2 = values[ncol*i + j];
this.setElement(propvals, entry, p*v1 + (1-p)*v2);
} else if (!direct) {
this.setElement(propvals, entry, values[ncol*value + j]);
} else {
this.setElement(propvals, entry, value[j]);
}
}
needsBinding = [];
for (j=0; j < entries.length; j++) {
if (properties[j] === "values" &&
needsBinding.indexOf(objids[j]) === -1) {
needsBinding.push(objids[j]);
}
}
for (j=0; j < needsBinding.length; j++) {
gl = this.gl || this.initGL();
obj = this.getObj(needsBinding[j]);
gl.bindBuffer(gl.ARRAY_BUFFER, obj.buf);
gl.bufferData(gl.ARRAY_BUFFER, obj.values, gl.STATIC_DRAW);
}
};
this.vertexSetter = function(el, control)  {
var svals = [].concat(control.param),
j, k, p, propvals, stride, ofs, obj,
attrib,
ofss    = {x:"vofs", y:"vofs", z:"vofs",
red:"cofs", green:"cofs", blue:"cofs",
alpha:"cofs", radii:"radofs",
nx:"nofs", ny:"nofs", nz:"nofs",
ox:"oofs", oy:"oofs", oz:"oofs",
ts:"tofs", tt:"tofs"},
pos     = {x:0, y:1, z:2,
red:0, green:1, blue:2,
alpha:3,radii:0,
nx:0, ny:1, nz:2,
ox:0, oy:1, oz:2,
ts:0, tt:1},
values = control.values,
direct = values === null,
ncol,
interp = control.interp,
vertices = [].concat(control.vertices),
attributes = [].concat(control.attributes),
value = control.value;
ncol = Math.max(vertices.length, attributes.length);
if (!ncol)
return;
vertices = this.repeatToLen(vertices, ncol);
attributes = this.repeatToLen(attributes, ncol);
if (direct)
interp = false;
/* JSON doesn't pass Infinity */
svals[0] = -Infinity;
svals[svals.length - 1] = Infinity;
for (j = 1; j < svals.length; j++) {
if (value <= svals[j]) {
if (interp) {
if (svals[j] === Infinity)
p = 1;
else
p = (svals[j] - value)/(svals[j] - svals[j-1]);
} else {
if (svals[j] - value > value - svals[j-1])
j = j - 1;
}
break;
}
}
obj = this.getObj(control.objid);
propvals = obj.values;
for (k=0; k<ncol; k++) {
attrib = attributes[k];
vertex = vertices[k];
ofs = obj.vOffsets[ofss[attrib]];
if (ofs < 0)
this.alertOnce("Attribute '"+attrib+"' not found in object "+control.objid);
else {
stride = obj.vOffsets.stride;
ofs = vertex*stride + ofs + pos[attrib];
if (direct) {
propvals[ofs] = value;
} else if (interp) {
propvals[ofs] = p*values[j-1][k] + (1-p)*values[j][k];
} else {
propvals[ofs] = values[j][k];
}
}
}
if (typeof obj.buf !== "undefined") {
var gl = this.gl || this.initGL();
gl.bindBuffer(gl.ARRAY_BUFFER, obj.buf);
gl.bufferData(gl.ARRAY_BUFFER, propvals, gl.STATIC_DRAW);
}
};
this.ageSetter = function(el, control) {
var objids = [].concat(control.objids),
nobjs = objids.length,
time = control.value,
births = [].concat(control.births),
ages = [].concat(control.ages),
steps = births.length,
j = Array(steps),
p = Array(steps),
i, k, age, j0, propvals, stride, ofs, objid, obj,
attrib, dim,
attribs = ["colors", "alpha", "radii", "vertices",
"normals", "origins", "texcoords",
"x", "y", "z",
"red", "green", "blue"],
ofss    = ["cofs", "cofs", "radofs", "vofs",
"nofs", "oofs", "tofs",
"vofs", "vofs", "vofs",
"cofs", "cofs", "cofs"],
dims    = [3,1,1,3,
3,2,2,
1,1,1,
1,1,1],
pos     = [0,3,0,0,
0,0,0,
0,1,2,
0,1,2];
/* Infinity doesn't make it through JSON */
ages[0] = -Infinity;
ages[ages.length-1] = Infinity;
for (i = 0; i < steps; i++) {
if (births[i] !== null) {  // NA in R becomes null
age = time - births[i];
for (j0 = 1; age > ages[j0]; j0++);
if (ages[j0] == Infinity)
p[i] = 1;
else if (ages[j0] > ages[j0-1])
p[i] = (ages[j0] - age)/(ages[j0] - ages[j0-1]);
else
p[i] = 0;
j[i] = j0;
}
}
for (l = 0; l < nobjs; l++) {
objid = objids[l];
obj = this.getObj(objid);
propvals = obj.values;
stride = obj.vOffsets.stride;
for (k = 0; k < attribs.length; k++) {
attrib = control[attribs[k]];
if (typeof attrib !== "undefined") {
ofs = obj.vOffsets[ofss[k]];
if (ofs >= 0) {
dim = dims[k];
ofs = ofs + pos[k];
for (i = 0; i < steps; i++) {
if (births[i] !== null) {
for (d=0; d < dim; d++) {
propvals[i*stride + ofs + d] = p[i]*attrib[dim*(j[i]-1) + d] + (1-p[i])*attrib[dim*j[i] + d];
}
}
}
} else
this.alertOnce("\'"+attribs[k]+"\' property not found in object "+objid);
}
}
obj.values = propvals;
if (typeof obj.buf !== "undefined") {
gl = this.gl || this.initGL();
gl.bindBuffer(gl.ARRAY_BUFFER, obj.buf);
gl.bufferData(gl.ARRAY_BUFFER, obj.values, gl.STATIC_DRAW);
}
}
};
this.oldBridge = function(el, control) {
var attrname, global = window[control.prefix + "rgl"];
if (typeof global !== "undefined")
for (attrname in global)
this[attrname] = global[attrname];
window[control.prefix + "rgl"] = this;
};
this.Player = function(el, control) {
var
self = this,
components = [].concat(control.components),
Tick = function() { /* "this" will be a timer */
var i,
nominal = this.value,
slider = this.Slider,
labels = this.outputLabels,
output = this.Output,
step;
if (typeof slider !== "undefined" && nominal != slider.value)
slider.value = nominal;
if (typeof output !== "undefined") {
step = Math.round((nominal - output.sliderMin)/output.sliderStep);
if (labels !== null) {
output.innerHTML = labels[step];
} else {
step = step*output.sliderStep + output.sliderMin;
output.innerHTML = step.toPrecision(output.outputPrecision);
}
}
for (i=0; i < this.actions.length; i++) {
this.actions[i].value = nominal;
}
self.applyControls(el, this.actions, false);
self.drawScene();
},
OnSliderInput = function() { /* "this" will be the slider */
this.rgltimer.value = Number(this.value);
this.rgltimer.Tick();
},
addSlider = function(min, max, step, value) {
var slider = document.createElement("input");
slider.type = "range";
slider.min = min;
slider.max = max;
slider.step = step;
slider.value = value;
slider.oninput = OnSliderInput;
slider.sliderActions = control.actions;
slider.sliderScene = this;
slider.className = "rgl-slider";
slider.id = el.id + "-slider";
el.rgltimer.Slider = slider;
slider.rgltimer = el.rgltimer;
el.appendChild(slider);
},
addLabel = function(labels, min, step, precision) {
var output = document.createElement("output");
output.sliderMin = min;
output.sliderStep = step;
output.outputPrecision = precision;
output.className = "rgl-label";
output.id = el.id + "-label";
el.rgltimer.Output = output;
el.rgltimer.outputLabels = labels;
el.appendChild(output);
},
addButton = function(label) {
var button = document.createElement("input"),
onclicks = {Reverse: function() { this.rgltimer.reverse();},
Play: function() { this.rgltimer.play();
this.value = this.rgltimer.enabled ? "Pause" : "Play"; },
Slower: function() { this.rgltimer.slower(); },
Faster: function() { this.rgltimer.faster(); },
Reset: function() { this.rgltimer.reset(); }};
button.rgltimer = el.rgltimer;
button.type = "button";
button.value = label;
if (label === "Play")
button.rgltimer.PlayButton = button;
button.onclick = onclicks[label];
button.className = "rgl-button";
button.id = el.id + "-" + label;
el.appendChild(button);
};
if (typeof control.reinit !== "undefined" && control.reinit !== null) {
control.actions.reinit = control.reinit;
}
el.rgltimer = new rgltimerClass(Tick, control.start, control.interval, control.stop, control.value, control.rate, control.loop, control.actions);
for (var i=0; i < components.length; i++) {
switch(components[i]) {
case "Slider": addSlider(control.start, control.stop,
control.step, control.value);
break;
case "Label": addLabel(control.labels, control.start,
control.step, control.precision);
break;
default:
addButton(components[i]);
}
}
el.rgltimer.Tick();
};
this.applyControls = function(el, x, draw) {
var self = this, reinit = x.reinit, i, control, type;
for (i = 0; i < x.length; i++) {
control = x[i];
type = control.type;
self[type](el, control);
}
if (typeof reinit !== "undefined" && reinit !== null) {
reinit = [].concat(reinit);
for (i = 0; i < reinit.length; i++)
self.getObj(reinit[i]).initialized = false;
}
if (typeof draw === "undefined" || draw)
self.drawScene();
};
this.sceneChangeHandler = function(message) {
var self = document.getElementById(message.elementId).rglinstance,
objs = message.objects, mat = message.material,
root = message.rootSubscene,
initSubs = message.initSubscenes,
redraw = message.redrawScene,
skipRedraw = message.skipRedraw,
deletes, subs, allsubs = [], i,j;
if (typeof message.delete !== "undefined") {
deletes = [].concat(message.delete);
if (typeof message.delfromSubscenes !== "undefined")
subs = [].concat(message.delfromSubscenes);
else
subs = [];
for (i = 0; i < deletes.length; i++) {
for (j = 0; j < subs.length; j++) {
self.delFromSubscene(deletes[i], subs[j]);
}
delete self.scene.objects[deletes[i]];
}
}
if (typeof objs !== "undefined") {
Object.keys(objs).forEach(function(key){
key = parseInt(key, 10);
self.scene.objects[key] = objs[key];
self.initObj(key);
var obj = self.getObj(key),
subs = [].concat(obj.inSubscenes), k;
allsubs = allsubs.concat(subs);
for (k = 0; k < subs.length; k++)
self.addToSubscene(key, subs[k]);
});
}
if (typeof mat !== "undefined") {
self.scene.material = mat;
}
if (typeof root !== "undefined") {
self.scene.rootSubscene = root;
}
if (typeof initSubs !== "undefined")
allsubs = allsubs.concat(initSubs);
allsubs = self.unique(allsubs);
for (i = 0; i < allsubs.length; i++) {
self.initSubscene(allsubs[i]);
}
if (typeof skipRedraw !== "undefined") {
root = self.getObj(self.scene.rootSubscene);
root.par3d.skipRedraw = skipRedraw;
}
if (redraw)
self.drawScene();
};
}).call(rglwidgetClass.prototype);
rgltimerClass = function(Tick, startTime, interval, stopTime, value, rate, loop, actions) {
this.enabled = false;
this.timerId = 0;
this.startTime = startTime;         /* nominal start time in seconds */
this.value = value;                 /* current nominal time */
this.interval = interval;           /* seconds between updates */
this.stopTime = stopTime;           /* nominal stop time */
this.rate = rate;                   /* nominal units per second */
this.loop = loop;                   /* "none", "cycle", or "oscillate" */
this.realStart = undefined;         /* real world start time */
this.multiplier = 1;                /* multiplier for fast-forward
or reverse */
this.actions = actions;
this.Tick = Tick;
};
(function() {
this.play = function() {
if (this.enabled) {
this.enabled = false;
window.clearInterval(this.timerId);
this.timerId = 0;
return;
}
var tick = function(self) {
var now = new Date();
self.value = self.multiplier*self.rate*(now - self.realStart)/1000 + self.startTime;
if (self.value > self.stopTime || self.value < self.startTime) {
if (!self.loop) {
self.reset();
} else {
var cycle = self.stopTime - self.startTime,
newval = (self.value - self.startTime) % cycle + self.startTime;
if (newval < self.startTime) {
newval += cycle;
}
self.realStart += (self.value - newval)*1000/self.multiplier/self.rate;
self.value = newval;
}
}
if (typeof self.Tick !== "undefined") {
self.Tick(self.value);
}
};
this.realStart = new Date() - 1000*(this.value - this.startTime)/this.rate/this.multiplier;
this.timerId = window.setInterval(tick, 1000*this.interval, this);
this.enabled = true;
};
this.reset = function() {
this.value = this.startTime;
this.newmultiplier(1);
if (typeof this.Tick !== "undefined") {
this.Tick(this.value);
}
if (this.enabled)
this.play();  /* really pause... */
if (typeof this.PlayButton !== "undefined")
this.PlayButton.value = "Play";
};
this.faster = function() {
this.newmultiplier(Math.SQRT2*this.multiplier);
};
this.slower = function() {
this.newmultiplier(this.multiplier/Math.SQRT2);
};
this.reverse = function() {
this.newmultiplier(-this.multiplier);
};
this.newmultiplier = function(newmult) {
if (newmult != this.multiplier) {
this.realStart += 1000*(this.value - this.startTime)/this.rate*(1/this.multiplier - 1/newmult);
this.multiplier = newmult;
}
};
}).call(rgltimerClass.prototype);</script>
<div id="testgldiv" class="rglWebGL"></div>
<script type="text/javascript">
var testgldiv = document.getElementById("testgldiv"),
testglrgl = new rglwidgetClass();
testgldiv.width = 673;
testgldiv.height = 481;
testglrgl.initialize(testgldiv,
{"material":{"color":"#000000","alpha":1,"lit":true,"ambient":"#000000","specular":"#FFFFFF","emission":"#000000","shininess":50,"smooth":true,"front":"filled","back":"filled","size":3,"lwd":1,"fog":false,"point_antialias":false,"line_antialias":false,"texture":null,"textype":"rgb","texmipmap":false,"texminfilter":"linear","texmagfilter":"linear","texenvmap":false,"depth_mask":true,"depth_test":"less"},"rootSubscene":1,"objects":{"7":{"id":7,"type":"spheres","material":{},"vertices":[[-0.6572132,-0.6011992,1.735446],[-0.6293955,-0.3823225,1.550033],[-2.779397,-0.2412383,-0.1464566],[-0.3117707,-0.1357593,-2.363019],[1.974489,-1.134402,-0.7544022],[-0.05613753,0.1638257,-2.786],[3.002674,-0.3627592,0.3348874],[-2.055329,0.9438949,-1.465181],[-2.287408,1.797241,-1.983526],[-0.5263812,1.49277,-0.1620126],[-0.5092055,1.683585,-0.3238945],[2.24781,-0.3753827,-0.683474],[2.047823,-0.484464,-0.6832207],[2.148542,-0.2951097,-0.8017395],[3.89979,0.6472914,-0.8279481],[3.954123,0.7206101,-0.7333815],[3.592972,0.5488912,-0.4211349],[-3.856284,-0.4228272,-0.2967519],[-4.254033,-0.2068406,0.688414],[-4.234221,-0.4662555,-0.2792875],[-1.904168,0.1567959,-2.119838],[2.184851,-1.168771,-1.014217],[1.863383,-0.9624448,-0.9064645],[2.888994,-0.1631284,0.6808261],[2.245919,-1.044406,-0.8738121],[-3.573968,-0.4536156,-0.1212038],[-2.651255,-0.8303288,2.046371],[-3.385706,-0.4538647,1.378599],[1.372957,-0.1365448,3.5],[-0.0009899207,0.4020936,3.219072],[2.669126,1.352901,4.379677],[-2.420593,0.4117647,0.2336399]],"colors":[[0,0,1,1]],"radii":[[0.108661]],"centers":[[-0.6572132,-0.6011992,1.735446],[-0.6293955,-0.3823225,1.550033],[-2.779397,-0.2412383,-0.1464566],[-0.3117707,-0.1357593,-2.363019],[1.974489,-1.134402,-0.7544022],[-0.05613753,0.1638257,-2.786],[3.002674,-0.3627592,0.3348874],[-2.055329,0.9438949,-1.465181],[-2.287408,1.797241,-1.983526],[-0.5263812,1.49277,-0.1620126],[-0.5092055,1.683585,-0.3238945],[2.24781,-0.3753827,-0.683474],[2.047823,-0.484464,-0.6832207],[2.148542,-0.2951097,-0.8017395],[3.89979,0.6472914,-0.8279481],[3.954123,0.7206101,-0.7333815],[3.592972,0.5488912,-0.4211349],[-3.856284,-0.4228272,-0.2967519],[-4.254033,-0.2068406,0.688414],[-4.234221,-0.4662555,-0.2792875],[-1.904168,0.1567959,-2.119838],[2.184851,-1.168771,-1.014217],[1.863383,-0.9624448,-0.9064645],[2.888994,-0.1631284,0.6808261],[2.245919,-1.044406,-0.8738121],[-3.573968,-0.4536156,-0.1212038],[-2.651255,-0.8303288,2.046371],[-3.385706,-0.4538647,1.378599],[1.372957,-0.1365448,3.5],[-0.0009899207,0.4020936,3.219072],[2.669126,1.352901,4.379677],[-2.420593,0.4117647,0.2336399]],"ignoreExtent":false,"flags":3},"9":{"id":9,"type":"text","material":{"lit":false},"vertices":[[-0.1499548,2.445477,5.739757]],"colors":[[0,0,0,1]],"texts":[["3D plot of the first three Principal Components of the mtcars dataset"]],"cex":[[1]],"adj":[[0.5,0.5]],"centers":[[-0.1499548,2.445477,5.739757]],"family":[["sans"]],"font":[[1]],"ignoreExtent":true,"flags":40},"10":{"id":10,"type":"text","material":{"lit":false},"vertices":[[-0.1499548,-2.06292,-4.716415]],"colors":[[0,0,0,1]],"texts":[["Click on and drag it around or zoom in or out to see it in different angles"]],"cex":[[1]],"adj":[[0.5,0.5]],"centers":[[-0.1499548,-2.06292,-4.716415]],"family":[["sans"]],"font":[[1]],"ignoreExtent":true,"flags":40},"11":{"id":11,"type":"text","material":{"lit":false},"vertices":[[-0.1499548,-1.817007,-4.146079]],"colors":[[0,0,0,1]],"texts":[["First PC (~60%)"]],"cex":[[1]],"adj":[[0.5,0.5]],"centers":[[-0.1499548,-1.817007,-4.146079]],"family":[["sans"]],"font":[[1]],"ignoreExtent":true,"flags":40},"12":{"id":12,"type":"text","material":{"lit":false},"vertices":[[-5.790812,0.3142351,-4.146079]],"colors":[[0,0,0,1]],"texts":[["Third PC (~6%)"]],"cex":[[1]],"adj":[[0.5,0.5]],"centers":[[-5.790812,0.3142351,-4.146079]],"family":[["sans"]],"font":[[1]],"ignoreExtent":true,"flags":40},"13":{"id":13,"type":"text","material":{"lit":false},"vertices":[[-5.790812,-1.817007,0.796839]],"colors":[[0,0,0,1]],"texts":[["Second PC (~24%)"]],"cex":[[1]],"adj":[[0.5,0.5]],"centers":[[-5.790812,-1.817007,0.796839]],"family":[["sans"]],"font":[[1]],"ignoreExtent":true,"flags":40},"14":{"id":14,"type":"text","material":{"lit":false},"vertices":[[-0.6572132,-0.6011992,1.735446],[-0.6293955,-0.3823225,1.550033],[-2.779397,-0.2412383,-0.1464566],[-0.3117707,-0.1357593,-2.363019],[1.974489,-1.134402,-0.7544022],[-0.05613753,0.1638257,-2.786],[3.002674,-0.3627592,0.3348874],[-2.055329,0.9438949,-1.465181],[-2.287408,1.797241,-1.983526],[-0.5263812,1.49277,-0.1620126],[-0.5092055,1.683585,-0.3238945],[2.24781,-0.3753827,-0.683474],[2.047823,-0.484464,-0.6832207],[2.148542,-0.2951097,-0.8017395],[3.89979,0.6472914,-0.8279481],[3.954123,0.7206101,-0.7333815],[3.592972,0.5488912,-0.4211349],[-3.856284,-0.4228272,-0.2967519],[-4.254033,-0.2068406,0.688414],[-4.234221,-0.4662555,-0.2792875],[-1.904168,0.1567959,-2.119838],[2.184851,-1.168771,-1.014217],[1.863383,-0.9624448,-0.9064645],[2.888994,-0.1631284,0.6808261],[2.245919,-1.044406,-0.8738121],[-3.573968,-0.4536156,-0.1212038],[-2.651255,-0.8303288,2.046371],[-3.385706,-0.4538647,1.378599],[1.372957,-0.1365448,3.5],[-0.0009899207,0.4020936,3.219072],[2.669126,1.352901,4.379677],[-2.420593,0.4117647,0.2336399]],"colors":[[0,0,0,1]],"texts":[["Mazda RX4"],["Mazda RX4 Wag"],["Datsun 710"],["Hornet 4 Drive"],["Hornet Sportabout"],["Valiant"],["Duster 360"],["Merc 240D"],["Merc 230"],["Merc 280"],["Merc 280C"],["Merc 450SE"],["Merc 450SL"],["Merc 450SLC"],["Cadillac Fleetwood"],["Lincoln Continental"],["Chrysler Imperial"],["Fiat 128"],["Honda Civic"],["Toyota Corolla"],["Toyota Corona"],["Dodge Challenger"],["AMC Javelin"],["Camaro Z28"],["Pontiac Firebird"],["Fiat X1-9"],["Porsche 914-2"],["Lotus Europa"],["Ford Pantera L"],["Ferrari Dino"],["Maserati Bora"],["Volvo 142E"]],"cex":[[0.7]],"adj":[[1,1]],"centers":[[-0.6572132,-0.6011992,1.735446],[-0.6293955,-0.3823225,1.550033],[-2.779397,-0.2412383,-0.1464566],[-0.3117707,-0.1357593,-2.363019],[1.974489,-1.134402,-0.7544022],[-0.05613753,0.1638257,-2.786],[3.002674,-0.3627592,0.3348874],[-2.055329,0.9438949,-1.465181],[-2.287408,1.797241,-1.983526],[-0.5263812,1.49277,-0.1620126],[-0.5092055,1.683585,-0.3238945],[2.24781,-0.3753827,-0.683474],[2.047823,-0.484464,-0.6832207],[2.148542,-0.2951097,-0.8017395],[3.89979,0.6472914,-0.8279481],[3.954123,0.7206101,-0.7333815],[3.592972,0.5488912,-0.4211349],[-3.856284,-0.4228272,-0.2967519],[-4.254033,-0.2068406,0.688414],[-4.234221,-0.4662555,-0.2792875],[-1.904168,0.1567959,-2.119838],[2.184851,-1.168771,-1.014217],[1.863383,-0.9624448,-0.9064645],[2.888994,-0.1631284,0.6808261],[2.245919,-1.044406,-0.8738121],[-3.573968,-0.4536156,-0.1212038],[-2.651255,-0.8303288,2.046371],[-3.385706,-0.4538647,1.378599],[1.372957,-0.1365448,3.5],[-0.0009899207,0.4020936,3.219072],[2.669126,1.352901,4.379677],[-2.420593,0.4117647,0.2336399]],"family":[["sans"]],"font":[[1]],"ignoreExtent":false,"flags":40},"5":{"id":5,"type":"light","vertices":[[0,0,1]],"colors":[[1,1,1,1],[1,1,1,1],[1,1,1,1]],"viewpoint":true,"finite":false},"4":{"id":4,"type":"background","material":{"fog":true},"colors":[[0.2980392,0.2980392,0.2980392,1]],"centers":[[0,0,0]],"sphere":false,"fogtype":"none","flags":0},"6":{"id":6,"type":"background","material":{"lit":false,"back":"lines"},"colors":[[1,1,1,1]],"centers":[[0,0,0]],"sphere":false,"fogtype":"none","flags":0},"8":{"id":8,"type":"bboxdeco","material":{"front":"lines","back":"lines"},"vertices":[[-4,"NA","NA"],[-2,"NA","NA"],[0,"NA","NA"],[2,"NA","NA"],[4,"NA","NA"],["NA",-1,"NA"],["NA",-0.5,"NA"],["NA",0,"NA"],["NA",0.5,"NA"],["NA",1,"NA"],["NA",1.5,"NA"],["NA","NA",-2],["NA","NA",0],["NA","NA",2],["NA","NA",4]],"colors":[[0,0,0,1]],"draw_front":true,"newIds":[22,23,24,25,26,27,28]},"1":{"id":1,"type":"subscene","par3d":{"antialias":8,"FOV":30,"ignoreExtent":false,"listeners":1,"mouseMode":{"left":"trackball","right":"zoom","middle":"fov","wheel":"pull"},"observer":[0,0,28.49808],"modelMatrix":[[1,0,0,0.1499548],[0,0.3420202,0.9396926,-0.8562584],[0,-0.9396926,0.3420202,-28.47533],[0,0,0,1]],"projMatrix":[[2.665751,0,0,0],[0,3.732051,0,0],[0,0,-3.863703,-102.7323],[0,0,-1,0]],"skipRedraw":false,"userMatrix":[[1,0,0,0],[0,0.3420201,0.9396926,0],[0,-0.9396926,0.3420201,0],[0,0,0,1]],"scale":[1,1,1],"viewport":{"x":0,"y":0,"width":1,"height":1},"zoom":1,"bbox":[-4.362694,4.062784,-1.277432,1.905902,-2.89466,4.488338],"windowRect":[0,45,672,525],"family":"sans","font":1,"cex":1,"useFreeType":true,"fontname":"/Library/Frameworks/R.framework/Versions/3.2/Resources/library/rgl/fonts/FreeSans.ttf","maxClipPlanes":6},"embeddings":{"viewport":"replace","projection":"replace","model":"replace"},"objects":[6,8,7,9,10,11,12,13,14,5,22,23,24,25,26,27,28],"subscenes":[],"flags":1195},"22":{"id":22,"type":"lines","material":{"lit":false,"front":"lines","back":"lines"},"vertices":[[-4,-1.325182,-3.005405],[4,-1.325182,-3.005405],[-4,-1.325182,-3.005405],[-4,-1.407153,-3.195518],[-2,-1.325182,-3.005405],[-2,-1.407153,-3.195518],[0,-1.325182,-3.005405],[0,-1.407153,-3.195518],[2,-1.325182,-3.005405],[2,-1.407153,-3.195518],[4,-1.325182,-3.005405],[4,-1.407153,-3.195518]],"colors":[[0,0,0,1]],"centers":[[0,-1.325182,-3.005405],[-4,-1.366167,-3.100461],[-2,-1.366167,-3.100461],[0,-1.366167,-3.100461],[2,-1.366167,-3.100461],[4,-1.366167,-3.100461]],"ignoreExtent":true,"origId":8,"flags":128},"23":{"id":23,"type":"text","material":{"lit":false,"front":"lines","back":"lines"},"vertices":[[-4,-1.571095,-3.575742],[-2,-1.571095,-3.575742],[0,-1.571095,-3.575742],[2,-1.571095,-3.575742],[4,-1.571095,-3.575742]],"colors":[[0,0,0,1]],"texts":[["-4"],["-2"],["0"],["2"],["4"]],"cex":[[1]],"adj":[[0.5,0.5]],"centers":[[-4,-1.571095,-3.575742],[-2,-1.571095,-3.575742],[0,-1.571095,-3.575742],[2,-1.571095,-3.575742],[4,-1.571095,-3.575742]],"family":[["sans"]],"font":[[1]],"ignoreExtent":true,"origId":8,"flags":40},"24":{"id":24,"type":"lines","material":{"lit":false,"front":"lines","back":"lines"},"vertices":[[-4.489076,-1,-3.005405],[-4.489076,1.5,-3.005405],[-4.489076,-1,-3.005405],[-4.706032,-1,-3.195518],[-4.489076,-0.5,-3.005405],[-4.706032,-0.5,-3.195518],[-4.489076,0,-3.005405],[-4.706032,0,-3.195518],[-4.489076,0.5,-3.005405],[-4.706032,0.5,-3.195518],[-4.489076,1,-3.005405],[-4.706032,1,-3.195518],[-4.489076,1.5,-3.005405],[-4.706032,1.5,-3.195518]],"colors":[[0,0,0,1]],"centers":[[-4.489076,0.25,-3.005405],[-4.597554,-1,-3.100461],[-4.597554,-0.5,-3.100461],[-4.597554,0,-3.100461],[-4.597554,0.5,-3.100461],[-4.597554,1,-3.100461],[-4.597554,1.5,-3.100461]],"ignoreExtent":true,"origId":8,"flags":128},"25":{"id":25,"type":"text","material":{"lit":false,"front":"lines","back":"lines"},"vertices":[[-5.139944,-1,-3.575742],[-5.139944,-0.5,-3.575742],[-5.139944,0,-3.575742],[-5.139944,0.5,-3.575742],[-5.139944,1,-3.575742],[-5.139944,1.5,-3.575742]],"colors":[[0,0,0,1]],"texts":[["-1"],["-0.5"],["0"],["0.5"],["1"],["1.5"]],"cex":[[1]],"adj":[[0.5,0.5]],"centers":[[-5.139944,-1,-3.575742],[-5.139944,-0.5,-3.575742],[-5.139944,0,-3.575742],[-5.139944,0.5,-3.575742],[-5.139944,1,-3.575742],[-5.139944,1.5,-3.575742]],"family":[["sans"]],"font":[[1]],"ignoreExtent":true,"origId":8,"flags":40},"26":{"id":26,"type":"lines","material":{"lit":false,"front":"lines","back":"lines"},"vertices":[[-4.489076,-1.325182,-2],[-4.489076,-1.325182,4],[-4.489076,-1.325182,-2],[-4.706032,-1.407153,-2],[-4.489076,-1.325182,0],[-4.706032,-1.407153,0],[-4.489076,-1.325182,2],[-4.706032,-1.407153,2],[-4.489076,-1.325182,4],[-4.706032,-1.407153,4]],"colors":[[0,0,0,1]],"centers":[[-4.489076,-1.325182,1],[-4.597554,-1.366167,-2],[-4.597554,-1.366167,0],[-4.597554,-1.366167,2],[-4.597554,-1.366167,4]],"ignoreExtent":true,"origId":8,"flags":128},"27":{"id":27,"type":"text","material":{"lit":false,"front":"lines","back":"lines"},"vertices":[[-5.139944,-1.571095,-2],[-5.139944,-1.571095,0],[-5.139944,-1.571095,2],[-5.139944,-1.571095,4]],"colors":[[0,0,0,1]],"texts":[["-2"],["0"],["2"],["4"]],"cex":[[1]],"adj":[[0.5,0.5]],"centers":[[-5.139944,-1.571095,-2],[-5.139944,-1.571095,0],[-5.139944,-1.571095,2],[-5.139944,-1.571095,4]],"family":[["sans"]],"font":[[1]],"ignoreExtent":true,"origId":8,"flags":40},"28":{"id":28,"type":"lines","material":{"lit":false,"front":"lines","back":"lines"},"vertices":[[-4.489076,-1.325182,-3.005405],[-4.489076,1.953652,-3.005405],[-4.489076,-1.325182,4.599083],[-4.489076,1.953652,4.599083],[-4.489076,-1.325182,-3.005405],[-4.489076,-1.325182,4.599083],[-4.489076,1.953652,-3.005405],[-4.489076,1.953652,4.599083],[-4.489076,-1.325182,-3.005405],[4.189167,-1.325182,-3.005405],[-4.489076,-1.325182,4.599083],[4.189167,-1.325182,4.599083],[-4.489076,1.953652,-3.005405],[4.189167,1.953652,-3.005405],[-4.489076,1.953652,4.599083],[4.189167,1.953652,4.599083],[4.189167,-1.325182,-3.005405],[4.189167,1.953652,-3.005405],[4.189167,-1.325182,4.599083],[4.189167,1.953652,4.599083],[4.189167,-1.325182,-3.005405],[4.189167,-1.325182,4.599083],[4.189167,1.953652,-3.005405],[4.189167,1.953652,4.599083]],"colors":[[0,0,0,1]],"centers":[[-4.489076,0.3142351,-3.005405],[-4.489076,0.3142351,4.599083],[-4.489076,-1.325182,0.796839],[-4.489076,1.953652,0.796839],[-0.1499548,-1.325182,-3.005405],[-0.1499548,-1.325182,4.599083],[-0.1499548,1.953652,-3.005405],[-0.1499548,1.953652,4.599083],[4.189167,0.3142351,-3.005405],[4.189167,0.3142351,4.599083],[4.189167,-1.325182,0.796839],[4.189167,1.953652,0.796839]],"ignoreExtent":true,"origId":8,"flags":128}},"snapshot":"data:image/png;base64,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","width":673,"height":481,"sphereVerts":{"vb":[[0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0.07465783,0.1464466,0.2126075,0.2705981,0.3181896,0.3535534,0.3753303,0.3826834,0.3753303,0.3535534,0.3181896,0.2705981,0.2126075,0.1464466,0.07465783,0,0,0.1379497,0.2705981,0.3928475,0.5,0.5879378,0.6532815,0.6935199,0.7071068,0.6935199,0.6532815,0.5879378,0.5,0.3928475,0.2705981,0.1379497,0,0,0.18024,0.3535534,0.51328,0.6532815,0.7681778,0.8535534,0.9061274,0.9238795,0.9061274,0.8535534,0.7681778,0.6532815,0.51328,0.3535534,0.18024,0,0,0.1950903,0.3826834,0.5555702,0.7071068,0.8314696,0.9238795,0.9807853,1,0.9807853,0.9238795,0.8314696,0.7071068,0.5555702,0.3826834,0.1950903,0,0,0.18024,0.3535534,0.51328,0.6532815,0.7681778,0.8535534,0.9061274,0.9238795,0.9061274,0.8535534,0.7681778,0.6532815,0.51328,0.3535534,0.18024,0,0,0.1379497,0.2705981,0.3928475,0.5,0.5879378,0.6532815,0.6935199,0.7071068,0.6935199,0.6532815,0.5879378,0.5,0.3928475,0.2705981,0.1379497,0,0,0.07465783,0.1464466,0.2126075,0.2705981,0.3181896,0.3535534,0.3753303,0.3826834,0.3753303,0.3535534,0.3181896,0.2705981,0.2126075,0.1464466,0.07465783,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,-0,-0.07465783,-0.1464466,-0.2126075,-0.2705981,-0.3181896,-0.3535534,-0.3753303,-0.3826834,-0.3753303,-0.3535534,-0.3181896,-0.2705981,-0.2126075,-0.1464466,-0.07465783,-0,-0,-0.1379497,-0.2705981,-0.3928475,-0.5,-0.5879378,-0.6532815,-0.6935199,-0.7071068,-0.6935199,-0.6532815,-0.5879378,-0.5,-0.3928475,-0.2705981,-0.1379497,-0,-0,-0.18024,-0.3535534,-0.51328,-0.6532815,-0.7681778,-0.8535534,-0.9061274,-0.9238795,-0.9061274,-0.8535534,-0.7681778,-0.6532815,-0.51328,-0.3535534,-0.18024,-0,-0,-0.1950903,-0.3826834,-0.5555702,-0.7071068,-0.8314696,-0.9238795,-0.9807853,-1,-0.9807853,-0.9238795,-0.8314696,-0.7071068,-0.5555702,-0.3826834,-0.1950903,-0,-0,-0.18024,-0.3535534,-0.51328,-0.6532815,-0.7681778,-0.8535534,-0.9061274,-0.9238795,-0.9061274,-0.8535534,-0.7681778,-0.6532815,-0.51328,-0.3535534,-0.18024,-0,-0,-0.1379497,-0.2705981,-0.3928475,-0.5,-0.5879378,-0.6532815,-0.6935199,-0.7071068,-0.6935199,-0.6532815,-0.5879378,-0.5,-0.3928475,-0.2705981,-0.1379497,-0,-0,-0.07465783,-0.1464466,-0.2126075,-0.2705981,-0.3181896,-0.3535534,-0.3753303,-0.3826834,-0.3753303,-0.3535534,-0.3181896,-0.2705981,-0.2126075,-0.1464466,-0.07465783,-0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0],[-1,-0.9807853,-0.9238795,-0.8314696,-0.7071068,-0.5555702,-0.3826834,-0.1950903,0,0.1950903,0.3826834,0.5555702,0.7071068,0.8314696,0.9238795,0.9807853,1,-1,-0.9807853,-0.9238795,-0.8314696,-0.7071068,-0.5555702,-0.3826834,-0.1950903,0,0.1950903,0.3826834,0.5555702,0.7071068,0.8314696,0.9238795,0.9807853,1,-1,-0.9807853,-0.9238795,-0.8314696,-0.7071068,-0.5555702,-0.3826834,-0.1950903,0,0.1950903,0.3826834,0.5555702,0.7071068,0.8314696,0.9238795,0.9807853,1,-1,-0.9807853,-0.9238795,-0.8314696,-0.7071068,-0.5555702,-0.3826834,-0.1950903,0,0.1950903,0.3826834,0.5555702,0.7071068,0.8314696,0.9238795,0.9807853,1,-1,-0.9807853,-0.9238795,-0.8314696,-0.7071068,-0.5555702,-0.3826834,-0.1950903,0,0.1950903,0.3826834,0.5555702,0.7071068,0.8314696,0.9238795,0.9807853,1,-1,-0.9807853,-0.9238795,-0.8314696,-0.7071068,-0.5555702,-0.3826834,-0.1950903,0,0.1950903,0.3826834,0.5555702,0.7071068,0.8314696,0.9238795,0.9807853,1,-1,-0.9807853,-0.9238795,-0.8314696,-0.7071068,-0.5555702,-0.3826834,-0.1950903,0,0.1950903,0.3826834,0.5555702,0.7071068,0.8314696,0.9238795,0.9807853,1,-1,-0.9807853,-0.9238795,-0.8314696,-0.7071068,-0.5555702,-0.3826834,-0.1950903,0,0.1950903,0.3826834,0.5555702,0.7071068,0.8314696,0.9238795,0.9807853,1,-1,-0.9807853,-0.9238795,-0.8314696,-0.7071068,-0.5555702,-0.3826834,-0.1950903,0,0.1950903,0.3826834,0.5555702,0.7071068,0.8314696,0.9238795,0.9807853,1,-1,-0.9807853,-0.9238795,-0.8314696,-0.7071068,-0.5555702,-0.3826834,-0.1950903,0,0.1950903,0.3826834,0.5555702,0.7071068,0.8314696,0.9238795,0.9807853,1,-1,-0.9807853,-0.9238795,-0.8314696,-0.7071068,-0.5555702,-0.3826834,-0.1950903,0,0.1950903,0.3826834,0.5555702,0.7071068,0.8314696,0.9238795,0.9807853,1,-1,-0.9807853,-0.9238795,-0.8314696,-0.7071068,-0.5555702,-0.3826834,-0.1950903,0,0.1950903,0.3826834,0.5555702,0.7071068,0.8314696,0.9238795,0.9807853,1,-1,-0.9807853,-0.9238795,-0.8314696,-0.7071068,-0.5555702,-0.3826834,-0.1950903,0,0.1950903,0.3826834,0.5555702,0.7071068,0.8314696,0.9238795,0.9807853,1,-1,-0.9807853,-0.9238795,-0.8314696,-0.7071068,-0.5555702,-0.3826834,-0.1950903,0,0.1950903,0.3826834,0.5555702,0.7071068,0.8314696,0.9238795,0.9807853,1,-1,-0.9807853,-0.9238795,-0.8314696,-0.7071068,-0.5555702,-0.3826834,-0.1950903,0,0.1950903,0.3826834,0.5555702,0.7071068,0.8314696,0.9238795,0.9807853,1,-1,-0.9807853,-0.9238795,-0.8314696,-0.7071068,-0.5555702,-0.3826834,-0.1950903,0,0.1950903,0.3826834,0.5555702,0.7071068,0.8314696,0.9238795,0.9807853,1,-1,-0.9807853,-0.9238795,-0.8314696,-0.7071068,-0.5555702,-0.3826834,-0.1950903,0,0.1950903,0.3826834,0.5555702,0.7071068,0.8314696,0.9238795,0.9807853,1],[0,0.1950903,0.3826834,0.5555702,0.7071068,0.8314696,0.9238795,0.9807853,1,0.9807853,0.9238795,0.8314696,0.7071068,0.5555702,0.3826834,0.1950903,0,0,0.18024,0.3535534,0.51328,0.6532815,0.7681778,0.8535534,0.9061274,0.9238795,0.9061274,0.8535534,0.7681778,0.6532815,0.51328,0.3535534,0.18024,0,0,0.1379497,0.2705981,0.3928475,0.5,0.5879378,0.6532815,0.6935199,0.7071068,0.6935199,0.6532815,0.5879378,0.5,0.3928475,0.2705981,0.1379497,0,0,0.07465783,0.1464466,0.2126075,0.2705981,0.3181896,0.3535534,0.3753303,0.3826834,0.3753303,0.3535534,0.3181896,0.2705981,0.2126075,0.1464466,0.07465783,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,-0,-0.07465783,-0.1464466,-0.2126075,-0.2705981,-0.3181896,-0.3535534,-0.3753303,-0.3826834,-0.3753303,-0.3535534,-0.3181896,-0.2705981,-0.2126075,-0.1464466,-0.07465783,-0,-0,-0.1379497,-0.2705981,-0.3928475,-0.5,-0.5879378,-0.6532815,-0.6935199,-0.7071068,-0.6935199,-0.6532815,-0.5879378,-0.5,-0.3928475,-0.2705981,-0.1379497,-0,-0,-0.18024,-0.3535534,-0.51328,-0.6532815,-0.7681778,-0.8535534,-0.9061274,-0.9238795,-0.9061274,-0.8535534,-0.7681778,-0.6532815,-0.51328,-0.3535534,-0.18024,-0,-0,-0.1950903,-0.3826834,-0.5555702,-0.7071068,-0.8314696,-0.9238795,-0.9807853,-1,-0.9807853,-0.9238795,-0.8314696,-0.7071068,-0.5555702,-0.3826834,-0.1950903,-0,-0,-0.18024,-0.3535534,-0.51328,-0.6532815,-0.7681778,-0.8535534,-0.9061274,-0.9238795,-0.9061274,-0.8535534,-0.7681778,-0.6532815,-0.51328,-0.3535534,-0.18024,-0,-0,-0.1379497,-0.2705981,-0.3928475,-0.5,-0.5879378,-0.6532815,-0.6935199,-0.7071068,-0.6935199,-0.6532815,-0.5879378,-0.5,-0.3928475,-0.2705981,-0.1379497,-0,-0,-0.07465783,-0.1464466,-0.2126075,-0.2705981,-0.3181896,-0.3535534,-0.3753303,-0.3826834,-0.3753303,-0.3535534,-0.3181896,-0.2705981,-0.2126075,-0.1464466,-0.07465783,-0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0.07465783,0.1464466,0.2126075,0.2705981,0.3181896,0.3535534,0.3753303,0.3826834,0.3753303,0.3535534,0.3181896,0.2705981,0.2126075,0.1464466,0.07465783,0,0,0.1379497,0.2705981,0.3928475,0.5,0.5879378,0.6532815,0.6935199,0.7071068,0.6935199,0.6532815,0.5879378,0.5,0.3928475,0.2705981,0.1379497,0,0,0.18024,0.3535534,0.51328,0.6532815,0.7681778,0.8535534,0.9061274,0.9238795,0.9061274,0.8535534,0.7681778,0.6532815,0.51328,0.3535534,0.18024,0,0,0.1950903,0.3826834,0.5555702,0.7071068,0.8314696,0.9238795,0.9807853,1,0.9807853,0.9238795,0.8314696,0.7071068,0.5555702,0.3826834,0.1950903,0]],"it":[[0,1,2,3,4,5,6,7,8,9,10,11,12,13,14,15,17,18,19,20,21,22,23,24,25,26,27,28,29,30,31,32,34,35,36,37,38,39,40,41,42,43,44,45,46,47,48,49,51,52,53,54,55,56,57,58,59,60,61,62,63,64,65,66,68,69,70,71,72,73,74,75,76,77,78,79,80,81,82,83,85,86,87,88,89,90,91,92,93,94,95,96,97,98,99,100,102,103,104,105,106,107,108,109,110,111,112,113,114,115,116,117,119,120,121,122,123,124,125,126,127,128,129,130,131,132,133,134,136,137,138,139,140,141,142,143,144,145,146,147,148,149,150,151,153,154,155,156,157,158,159,160,161,162,163,164,165,166,167,168,170,171,172,173,174,175,176,177,178,179,180,181,182,183,184,185,187,188,189,190,191,192,193,194,195,196,197,198,199,200,201,202,204,205,206,207,208,209,210,211,212,213,214,215,216,217,218,219,221,222,223,224,225,226,227,228,229,230,231,232,233,234,235,236,238,239,240,241,242,243,244,245,246,247,248,249,250,251,252,253,255,256,257,258,259,260,261,262,263,264,265,266,267,268,269,270,0,1,2,3,4,5,6,7,8,9,10,11,12,13,14,15,17,18,19,20,21,22,23,24,25,26,27,28,29,30,31,32,34,35,36,37,38,39,40,41,42,43,44,45,46,47,48,49,51,52,53,54,55,56,57,58,59,60,61,62,63,64,65,66,68,69,70,71,72,73,74,75,76,77,78,79,80,81,82,83,85,86,87,88,89,90,91,92,93,94,95,96,97,98,99,100,102,103,104,105,106,107,108,109,110,111,112,113,114,115,116,117,119,120,121,122,123,124,125,126,127,128,129,130,131,132,133,134,136,137,138,139,140,141,142,143,144,145,146,147,148,149,150,151,153,154,155,156,157,158,159,160,161,162,163,164,165,166,167,168,170,171,172,173,174,175,176,177,178,179,180,181,182,183,184,185,187,188,189,190,191,192,193,194,195,196,197,198,199,200,201,202,204,205,206,207,208,209,210,211,212,213,214,215,216,217,218,219,221,222,223,224,225,226,227,228,229,230,231,232,233,234,235,236,238,239,240,241,242,243,244,245,246,247,248,249,250,251,252,253,255,256,257,258,259,260,261,262,263,264,265,266,267,268,269,270],[17,18,19,20,21,22,23,24,25,26,27,28,29,30,31,32,34,35,36,37,38,39,40,41,42,43,44,45,46,47,48,49,51,52,53,54,55,56,57,58,59,60,61,62,63,64,65,66,68,69,70,71,72,73,74,75,76,77,78,79,80,81,82,83,85,86,87,88,89,90,91,92,93,94,95,96,97,98,99,100,102,103,104,105,106,107,108,109,110,111,112,113,114,115,116,117,119,120,121,122,123,124,125,126,127,128,129,130,131,132,133,134,136,137,138,139,140,141,142,143,144,145,146,147,148,149,150,151,153,154,155,156,157,158,159,160,161,162,163,164,165,166,167,168,170,171,172,173,174,175,176,177,178,179,180,181,182,183,184,185,187,188,189,190,191,192,193,194,195,196,197,198,199,200,201,202,204,205,206,207,208,209,210,211,212,213,214,215,216,217,218,219,221,222,223,224,225,226,227,228,229,230,231,232,233,234,235,236,238,239,240,241,242,243,244,245,246,247,248,249,250,251,252,253,255,256,257,258,259,260,261,262,263,264,265,266,267,268,269,270,272,273,274,275,276,277,278,279,280,281,282,283,284,285,286,287,18,19,20,21,22,23,24,25,26,27,28,29,30,31,32,33,35,36,37,38,39,40,41,42,43,44,45,46,47,48,49,50,52,53,54,55,56,57,58,59,60,61,62,63,64,65,66,67,69,70,71,72,73,74,75,76,77,78,79,80,81,82,83,84,86,87,88,89,90,91,92,93,94,95,96,97,98,99,100,101,103,104,105,106,107,108,109,110,111,112,113,114,115,116,117,118,120,121,122,123,124,125,126,127,128,129,130,131,132,133,134,135,137,138,139,140,141,142,143,144,145,146,147,148,149,150,151,152,154,155,156,157,158,159,160,161,162,163,164,165,166,167,168,169,171,172,173,174,175,176,177,178,179,180,181,182,183,184,185,186,188,189,190,191,192,193,194,195,196,197,198,199,200,201,202,203,205,206,207,208,209,210,211,212,213,214,215,216,217,218,219,220,222,223,224,225,226,227,228,229,230,231,232,233,234,235,236,237,239,240,241,242,243,244,245,246,247,248,249,250,251,252,253,254,256,257,258,259,260,261,262,263,264,265,266,267,268,269,270,271,273,274,275,276,277,278,279,280,281,282,283,284,285,286,287,288],[18,19,20,21,22,23,24,25,26,27,28,29,30,31,32,33,35,36,37,38,39,40,41,42,43,44,45,46,47,48,49,50,52,53,54,55,56,57,58,59,60,61,62,63,64,65,66,67,69,70,71,72,73,74,75,76,77,78,79,80,81,82,83,84,86,87,88,89,90,91,92,93,94,95,96,97,98,99,100,101,103,104,105,106,107,108,109,110,111,112,113,114,115,116,117,118,120,121,122,123,124,125,126,127,128,129,130,131,132,133,134,135,137,138,139,140,141,142,143,144,145,146,147,148,149,150,151,152,154,155,156,157,158,159,160,161,162,163,164,165,166,167,168,169,171,172,173,174,175,176,177,178,179,180,181,182,183,184,185,186,188,189,190,191,192,193,194,195,196,197,198,199,200,201,202,203,205,206,207,208,209,210,211,212,213,214,215,216,217,218,219,220,222,223,224,225,226,227,228,229,230,231,232,233,234,235,236,237,239,240,241,242,243,244,245,246,247,248,249,250,251,252,253,254,256,257,258,259,260,261,262,263,264,265,266,267,268,269,270,271,273,274,275,276,277,278,279,280,281,282,283,284,285,286,287,288,1,2,3,4,5,6,7,8,9,10,11,12,13,14,15,16,18,19,20,21,22,23,24,25,26,27,28,29,30,31,32,33,35,36,37,38,39,40,41,42,43,44,45,46,47,48,49,50,52,53,54,55,56,57,58,59,60,61,62,63,64,65,66,67,69,70,71,72,73,74,75,76,77,78,79,80,81,82,83,84,86,87,88,89,90,91,92,93,94,95,96,97,98,99,100,101,103,104,105,106,107,108,109,110,111,112,113,114,115,116,117,118,120,121,122,123,124,125,126,127,128,129,130,131,132,133,134,135,137,138,139,140,141,142,143,144,145,146,147,148,149,150,151,152,154,155,156,157,158,159,160,161,162,163,164,165,166,167,168,169,171,172,173,174,175,176,177,178,179,180,181,182,183,184,185,186,188,189,190,191,192,193,194,195,196,197,198,199,200,201,202,203,205,206,207,208,209,210,211,212,213,214,215,216,217,218,219,220,222,223,224,225,226,227,228,229,230,231,232,233,234,235,236,237,239,240,241,242,243,244,245,246,247,248,249,250,251,252,253,254,256,257,258,259,260,261,262,263,264,265,266,267,268,269,270,271]],"primitivetype":"triangle","material":null,"normals":null,"texcoords":[[0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0.0625,0.0625,0.0625,0.0625,0.0625,0.0625,0.0625,0.0625,0.0625,0.0625,0.0625,0.0625,0.0625,0.0625,0.0625,0.0625,0.0625,0.125,0.125,0.125,0.125,0.125,0.125,0.125,0.125,0.125,0.125,0.125,0.125,0.125,0.125,0.125,0.125,0.125,0.1875,0.1875,0.1875,0.1875,0.1875,0.1875,0.1875,0.1875,0.1875,0.1875,0.1875,0.1875,0.1875,0.1875,0.1875,0.1875,0.1875,0.25,0.25,0.25,0.25,0.25,0.25,0.25,0.25,0.25,0.25,0.25,0.25,0.25,0.25,0.25,0.25,0.25,0.3125,0.3125,0.3125,0.3125,0.3125,0.3125,0.3125,0.3125,0.3125,0.3125,0.3125,0.3125,0.3125,0.3125,0.3125,0.3125,0.3125,0.375,0.375,0.375,0.375,0.375,0.375,0.375,0.375,0.375,0.375,0.375,0.375,0.375,0.375,0.375,0.375,0.375,0.4375,0.4375,0.4375,0.4375,0.4375,0.4375,0.4375,0.4375,0.4375,0.4375,0.4375,0.4375,0.4375,0.4375,0.4375,0.4375,0.4375,0.5,0.5,0.5,0.5,0.5,0.5,0.5,0.5,0.5,0.5,0.5,0.5,0.5,0.5,0.5,0.5,0.5,0.5625,0.5625,0.5625,0.5625,0.5625,0.5625,0.5625,0.5625,0.5625,0.5625,0.5625,0.5625,0.5625,0.5625,0.5625,0.5625,0.5625,0.625,0.625,0.625,0.625,0.625,0.625,0.625,0.625,0.625,0.625,0.625,0.625,0.625,0.625,0.625,0.625,0.625,0.6875,0.6875,0.6875,0.6875,0.6875,0.6875,0.6875,0.6875,0.6875,0.6875,0.6875,0.6875,0.6875,0.6875,0.6875,0.6875,0.6875,0.75,0.75,0.75,0.75,0.75,0.75,0.75,0.75,0.75,0.75,0.75,0.75,0.75,0.75,0.75,0.75,0.75,0.8125,0.8125,0.8125,0.8125,0.8125,0.8125,0.8125,0.8125,0.8125,0.8125,0.8125,0.8125,0.8125,0.8125,0.8125,0.8125,0.8125,0.875,0.875,0.875,0.875,0.875,0.875,0.875,0.875,0.875,0.875,0.875,0.875,0.875,0.875,0.875,0.875,0.875,0.9375,0.9375,0.9375,0.9375,0.9375,0.9375,0.9375,0.9375,0.9375,0.9375,0.9375,0.9375,0.9375,0.9375,0.9375,0.9375,0.9375,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1],[0,0.0625,0.125,0.1875,0.25,0.3125,0.375,0.4375,0.5,0.5625,0.625,0.6875,0.75,0.8125,0.875,0.9375,1,0,0.0625,0.125,0.1875,0.25,0.3125,0.375,0.4375,0.5,0.5625,0.625,0.6875,0.75,0.8125,0.875,0.9375,1,0,0.0625,0.125,0.1875,0.25,0.3125,0.375,0.4375,0.5,0.5625,0.625,0.6875,0.75,0.8125,0.875,0.9375,1,0,0.0625,0.125,0.1875,0.25,0.3125,0.375,0.4375,0.5,0.5625,0.625,0.6875,0.75,0.8125,0.875,0.9375,1,0,0.0625,0.125,0.1875,0.25,0.3125,0.375,0.4375,0.5,0.5625,0.625,0.6875,0.75,0.8125,0.875,0.9375,1,0,0.0625,0.125,0.1875,0.25,0.3125,0.375,0.4375,0.5,0.5625,0.625,0.6875,0.75,0.8125,0.875,0.9375,1,0,0.0625,0.125,0.1875,0.25,0.3125,0.375,0.4375,0.5,0.5625,0.625,0.6875,0.75,0.8125,0.875,0.9375,1,0,0.0625,0.125,0.1875,0.25,0.3125,0.375,0.4375,0.5,0.5625,0.625,0.6875,0.75,0.8125,0.875,0.9375,1,0,0.0625,0.125,0.1875,0.25,0.3125,0.375,0.4375,0.5,0.5625,0.625,0.6875,0.75,0.8125,0.875,0.9375,1,0,0.0625,0.125,0.1875,0.25,0.3125,0.375,0.4375,0.5,0.5625,0.625,0.6875,0.75,0.8125,0.875,0.9375,1,0,0.0625,0.125,0.1875,0.25,0.3125,0.375,0.4375,0.5,0.5625,0.625,0.6875,0.75,0.8125,0.875,0.9375,1,0,0.0625,0.125,0.1875,0.25,0.3125,0.375,0.4375,0.5,0.5625,0.625,0.6875,0.75,0.8125,0.875,0.9375,1,0,0.0625,0.125,0.1875,0.25,0.3125,0.375,0.4375,0.5,0.5625,0.625,0.6875,0.75,0.8125,0.875,0.9375,1,0,0.0625,0.125,0.1875,0.25,0.3125,0.375,0.4375,0.5,0.5625,0.625,0.6875,0.75,0.8125,0.875,0.9375,1,0,0.0625,0.125,0.1875,0.25,0.3125,0.375,0.4375,0.5,0.5625,0.625,0.6875,0.75,0.8125,0.875,0.9375,1,0,0.0625,0.125,0.1875,0.25,0.3125,0.375,0.4375,0.5,0.5625,0.625,0.6875,0.75,0.8125,0.875,0.9375,1,0,0.0625,0.125,0.1875,0.25,0.3125,0.375,0.4375,0.5,0.5625,0.625,0.6875,0.75,0.8125,0.875,0.9375,1]]}});
testglrgl.prefix = "testgl";
</script>
<p id="testgldebug">
You must enable Javascript to view this page properly.</p>
<script>testglrgl.start();</script>

Next, we could use the results of the PCA in a supervised learning work (in regression or classification). We could also continue and perform clustering analysis using the results of the PCA to create specific segments of cars that are similar in some ways.


***

In this article we saw what PCA is in the context of unsupervised statistical learning approaches and how one can perform PCA in a dataset and then briefly interpret its results. As mentioned, PCA can also be used to perform regression models using the principal component scores as features and the use of PCA can lead to a more robust model since most of the important factors in a dataset are concentrated in its first principal components. 























