---
layout: post
title:  "Multiple Regression Analysis in R - First Steps"
date: 2015-03-12
excerpt: "In this example we'll extend the concept of linear regression to include multiple predictors."
image: "/images/multipleregression1.jpg"
permalink: /blog/2015/03/12/Multiple-Linear-Regression-First-Steps
categories: [Tutorial, R, Statistics]
---


{% include advertisements.html %}

In our previous study example, we looked at the Simple Linear Regression model. We loaded the **Prestige** dataset and used *income* as our response variable and *education* as the predictor. We generated three models regressing Income onto Education (with some transformations applied) and had strong indications that the linear model was not the most appropriate for the dataset.



In this example we'll extend the concept of linear regression to include multiple predictors. **Prestige** will continue to be our dataset of choice and can be found in the car package ```library(car)```.


```r
# Load the package that contains the full dataset.
library(car)
library(corrplot) # We'll use corrplot later on in this example too.
library(visreg) # This library will allow us to show multivariate graphs.
library(rgl)
library(knitr)
library(scatterplot3d)
```

Load the Prestige dataset once again:


```r
# Inspect and summarize the data.
head(Prestige,5)
```

```
##                     education income women prestige census type
## gov.administrators      13.11  12351 11.16     68.8   1113 prof
## general.managers        12.26  25879  4.02     69.1   1130 prof
## accountants             12.77   9271 15.70     63.4   1171 prof
## purchasing.officers     11.42   8865  9.11     56.8   1175 prof
## chemists                14.62   8403 11.68     73.5   2111 prof
```

```r
str(Prestige)
```

```
## 'data.frame':	102 obs. of  6 variables:
##  $ education: num  13.1 12.3 12.8 11.4 14.6 ...
##  $ income   : int  12351 25879 9271 8865 8403 11030 8258 14163 11377 11023 ...
##  $ women    : num  11.16 4.02 15.7 9.11 11.68 ...
##  $ prestige : num  68.8 69.1 63.4 56.8 73.5 77.6 72.6 78.1 73.1 68.8 ...
##  $ census   : int  1113 1130 1171 1175 2111 2113 2133 2141 2143 2153 ...
##  $ type     : Factor w/ 3 levels "bc","prof","wc": 2 2 2 2 2 2 2 2 2 2 ...
```

```r
summary(Prestige)
```

```
##    education          income          women           prestige    
##  Min.   : 6.380   Min.   :  611   Min.   : 0.000   Min.   :14.80  
##  1st Qu.: 8.445   1st Qu.: 4106   1st Qu.: 3.592   1st Qu.:35.23  
##  Median :10.540   Median : 5930   Median :13.600   Median :43.60  
##  Mean   :10.738   Mean   : 6798   Mean   :28.979   Mean   :46.83  
##  3rd Qu.:12.648   3rd Qu.: 8187   3rd Qu.:52.203   3rd Qu.:59.27  
##  Max.   :15.970   Max.   :25879   Max.   :97.510   Max.   :87.20  
##      census       type   
##  Min.   :1113   bc  :44  
##  1st Qu.:3120   prof:31  
##  Median :5135   wc  :23  
##  Mean   :5402   NA's: 4  
##  3rd Qu.:8312            
##  Max.   :9517
```



If you recall from our previous example, the Prestige dataset is a data frame with 102 rows and 6 columns. Each row is an observations that relate to an occupation. The columns relate to predictors such as average years of education, percentage of women in the occupation, prestige of the occupation, etc.

For our multiple linear regression example, we'll use more than one predictor. Our response variable will continue to be *Income* but now we will include *women*, *prestige* and *education* as our list of predictor variables. Remember that Education refers to the average number of years of education that exists in each profession. The women variable refers to the percentage of women in the profession and the prestige variable refers to a prestige score for each occupation (given by a metric called Pineo-Porter), from a social survey conducted in the mid-1960s.


{% include advertisements.html %}



```r
# Let's subset the data to capture income, education, women and prestige.
newdata = Prestige[,c(1:4)]
summary(newdata)
```

```
##    education          income          women           prestige    
##  Min.   : 6.380   Min.   :  611   Min.   : 0.000   Min.   :14.80  
##  1st Qu.: 8.445   1st Qu.: 4106   1st Qu.: 3.592   1st Qu.:35.23  
##  Median :10.540   Median : 5930   Median :13.600   Median :43.60  
##  Mean   :10.738   Mean   : 6798   Mean   :28.979   Mean   :46.83  
##  3rd Qu.:12.648   3rd Qu.: 8187   3rd Qu.:52.203   3rd Qu.:59.27  
##  Max.   :15.970   Max.   :25879   Max.   :97.510   Max.   :87.20
```

Our new dataset contains the four variables to be used in our model. It is now easy for us to plot them using the *plot* function:


```r
# Plot matrix of all variables.
plot(newdata, pch=16, col="blue", main="Matrix Scatterplot of Income, Education, Women and Prestige")
```

<span class="image fit"><img src="{{ "/images/MultipleLinearRegression_files/figure-html/unnamed-chunk-4-1.png" | absolute_url }}" alt="" /></span>

The matrix plot above allows us to vizualise the relationship among all variables in one single image. For example, we can see how income and education are related (see first column, second row top to bottom graph).

Another interesting example is the relationship between income and percentage of women (third column left to right second row top to bottom graph). Here we can see that as the percentage of women increases, average income in the profession declines.





Also from the matrix plot, note how prestige seems to have a similar pattern relative to education when plotted against income (fourth column left to right second row top to bottom graph).

To keep within the objectives of this study example, we'll start by fitting a linear regression on this dataset and see how well it models the observed data. We'll add all other predictors and give each of them a separate slope coefficient. We want to estimate the relationship and fit a *plane* (note that in a multi-dimensional setting, with two or more predictors and one response, the least squares regression line becomes a plane) that explains this relationship.


{% include advertisements.html %}


For our multiple linear regression example, we want to solve the following equation:

$$ Income = B0 + B1 * Education + B2 * Prestige + B3 * Women $$

The model will estimate the value of the *intercept (B0)* and each predictor's *slope (B1) for education*, *(B2) for prestige* and *(B3) for women*. The *intercept* is the average expected income value for the average value across all predictors. The value for each *slope*  estimate will be the average increase in income associated with a one-unit increase in each predictor value, holding the others constant. We want our model to fit a line or plane across the observed relationship in a way that the line/plane created is as close as possible to all data points.


<script async data-uid="d4f8348a0d" src="https://thoughtful-builder-4808.ck.page/d4f8348a0d/index.js"></script>


Let's start by using R **lm** function. The **lm** function is used to fit linear models. For more details, see: https://stat.ethz.ch/R-manual/R-devel/library/stats/html/lm.html. Here we are using *Least Squares* approach again.



```r
set.seed(1)

# Center predictors.
education.c = scale(newdata$education, center=TRUE, scale=FALSE)
prestige.c = scale(newdata$prestige, center=TRUE, scale=FALSE)
women.c = scale(newdata$women, center=TRUE, scale=FALSE)

# bind these new variables into newdata and display a summary.
new.c.vars = cbind(education.c, prestige.c, women.c)
newdata = cbind(newdata, new.c.vars)
names(newdata)[5:7] = c("education.c", "prestige.c", "women.c" )
summary(newdata)
```

```
##    education          income          women           prestige    
##  Min.   : 6.380   Min.   :  611   Min.   : 0.000   Min.   :14.80  
##  1st Qu.: 8.445   1st Qu.: 4106   1st Qu.: 3.592   1st Qu.:35.23  
##  Median :10.540   Median : 5930   Median :13.600   Median :43.60  
##  Mean   :10.738   Mean   : 6798   Mean   :28.979   Mean   :46.83  
##  3rd Qu.:12.648   3rd Qu.: 8187   3rd Qu.:52.203   3rd Qu.:59.27  
##  Max.   :15.970   Max.   :25879   Max.   :97.510   Max.   :87.20  
##   education.c       prestige.c         women.c      
##  Min.   :-4.358   Min.   :-32.033   Min.   :-28.98  
##  1st Qu.:-2.293   1st Qu.:-11.608   1st Qu.:-25.39  
##  Median :-0.198   Median : -3.233   Median :-15.38  
##  Mean   : 0.000   Mean   :  0.000   Mean   :  0.00  
##  3rd Qu.: 1.909   3rd Qu.: 12.442   3rd Qu.: 23.22  
##  Max.   : 5.232   Max.   : 40.367   Max.   : 68.53
```

```r
# fit a linear model and run a summary of its results.
mod1 = lm(income ~ education.c + prestige.c + women.c, data=newdata)
summary(mod1)
```

```
## 
## Call:
## lm(formula = income ~ education.c + prestige.c + women.c, data = newdata)
## 
## Residuals:
##     Min      1Q  Median      3Q     Max 
## -7715.3  -929.7  -231.2   689.7 14391.8 
## 
## Coefficients:
##             Estimate Std. Error t value Pr(>|t|)    
## (Intercept) 6797.902    254.934  26.665  < 2e-16 ***
## education.c  177.199    187.632   0.944    0.347    
## prestige.c   141.435     29.910   4.729 7.58e-06 ***
## women.c      -50.896      8.556  -5.948 4.19e-08 ***
## ---
## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
## 
## Residual standard error: 2575 on 98 degrees of freedom
## Multiple R-squared:  0.6432,	Adjusted R-squared:  0.6323 
## F-statistic: 58.89 on 3 and 98 DF,  p-value: < 2.2e-16
```


The result of the model is shown above.

Similar to our previous simple linear regression example, note we created a centered version of all predictor variables each ending with a **.c** in their names. These new variables were centered on their mean. This transformation was applied on each variable so we could have a meaningful interpretation of the intercept estimates. Centering allows us to say that the estimated income is **$6,798** when we consider the average number of years of education, the average percent of women and the average prestige from the dataset.


{% include advertisements.html %}


From the model output and the scatterplot we can make some interesting observations:

- For any given level of education and prestige in a profession, improving one percentage point of women in a given profession will see the average income decline by **$-50.9**. Similarly, for any given level of education and percent of women, seeing an improvement in prestige by one point in a given profession will lead to an an extra **$141.4** in average income.

- Note also our Adjusted R-squared value (we're now looking at adjusted R-square as a more appropriate metric of variability as the adjusted R-squared increases only if the new term added ends up improving the model more than would be expected by chance). In this model, we arrived in a larger R-squared number of **0.6322843** (compared to roughly **0.37** from our last simple linear regression exercise).

- Recall from our previous simple linear regression exmaple that our centered education predictor variable had a significant p-value (close to zero). But from the multiple regression model output above, education no longer displays a significant p-value. Here, education represents the average effect while holding the other variables women and prestige constant. From the matrix scatterplot shown above, we can see the pattern income takes when regressed on education and prestige. Note how closely aligned their pattern is with each other. So in essence, when they are put together in the model, education is no longer significant after adjusting for prestige. When we have two or more predictor variables strongly correlated, we face a problem of *collinearity* (the predictors are *collinear*).





Let's validate this situation with a correlation plot:


```r
# Plot a correlation graph
newdatacor = cor(newdata[1:4])
corrplot(newdatacor, method = "number")
```

<span class="image fit"><img src="{{ "/images/MultipleLinearRegression_files/figure-html/unnamed-chunk-6-1.png" | absolute_url }}" alt="" /></span>





The correlation matrix shown above highlights the situation we encoutered with the model output. Notice that the correlation between education and prestige is very high at **0.85**. This reveals each profession's level of education is strongly aligned to each profession's level of prestige. So in essence, education's high p-value indicates that women and prestige are related to income, but there is no evidence that education is associated with income, at least not when these other two predictors are also considered in the model.


{% include advertisements.html %}


- The model output can also help answer whether there is a relationship between the response and the predictors used. We can use the value of our F-Statistic to test whether all our coefficients are equal to zero (testing for the null hypothesis which means). The F-Statistic value from our model is **58.89** on **3** and **98** degrees of freedom. So assuming that the number of data points is appropriate and given that the p-values returned are low, we have some evidence that at least one of the predictors is associated with income.

- Given that we have indications that at least one of the predictors is associated with income, and based on the fact that education here has a high p-value, we can consider removing education from the model and see how the model fit changes (we are not going to run a variable selection procedure such as *forward*, *backward* or *mixed selection* in this example):


```r
# fit a linear model excluding the variable education
mod2 = lm(income ~ prestige.c + women.c, data=newdata)
summary(mod2)
```

```
## 
## Call:
## lm(formula = income ~ prestige.c + women.c, data = newdata)
## 
## Residuals:
##     Min      1Q  Median      3Q     Max 
## -7620.9 -1008.7  -240.4   873.1 14180.0 
## 
## Coefficients:
##             Estimate Std. Error t value Pr(>|t|)    
## (Intercept) 6797.902    254.795  26.680  < 2e-16 ***
## prestige.c   165.875     14.988  11.067  < 2e-16 ***
## women.c      -48.385      8.128  -5.953 4.02e-08 ***
## ---
## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
## 
## Residual standard error: 2573 on 99 degrees of freedom
## Multiple R-squared:   0.64,	Adjusted R-squared:  0.6327 
## F-statistic: 87.98 on 2 and 99 DF,  p-value: < 2.2e-16
```





The model excluding education has in fact improved our F-Statistic from **58.89** to **87.98** but no substantial improvement was achieved in residual standard error and adjusted R-square value. This is possibly due to the presence of outlier points in the data. 

Let's plot this last model's residuals: 


```r
# Plot model residuals.
plot(mod2, pch=16, which=1)
```

<span class="image fit"><img src="{{ "/images/MultipleLinearRegression_files/figure-html/unnamed-chunk-8-1.png" | absolute_url }}" alt="" /></span>

Note how the residuals plot of this last model shows some important points still lying far away from the middle area of the graph. 





Let's visualize a three-dimensional interactive graph with both predictors and the target variable:



```r
newdat <- expand.grid(prestige.c=seq(-35,45,by=5),women.c=seq(-25,70,by=5))
newdat$pp <- predict(mod2,newdata=newdat)
with(newdata,plot3d(prestige.c,women.c,income, col="blue", size=1, type="s", main="3D Linear Model Fit"))
with(newdat,surface3d(unique(prestige.c),unique(women.c),pp,
                      alpha=0.3,front="line", back="line"))
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
{"material":{"color":"#000000","alpha":1,"lit":true,"ambient":"#000000","specular":"#FFFFFF","emission":"#000000","shininess":50,"smooth":true,"front":"filled","back":"filled","size":3,"lwd":1,"fog":false,"point_antialias":false,"line_antialias":false,"texture":null,"textype":"rgb","texmipmap":false,"texminfilter":"linear","texmagfilter":"linear","texenvmap":false,"depth_mask":true,"depth_test":"less"},"rootSubscene":1,"objects":{"7":{"id":7,"type":"spheres","material":{},"vertices":[[21.96667,-17.81902,12351],[22.26667,-24.95902,25879],[16.56667,-13.27902,9271],[9.966666,-19.86902,8865],[26.66667,-17.29902,8403],[30.76667,-23.84902,11030],[25.76667,-3.32902,8258],[31.26667,-26.28902,14163],[26.26667,-27.94902,11377],[21.96667,-28.03902,11023],[15.16667,-27.06902,5902],[13.16667,-21.14902,7059],[6.966667,-13.64902,8425],[15.36667,28.33098,8049],[28.06667,19.30098,7405],[8.266666,25.79098,6336],[35.46667,-23.84902,19263],[11.26667,48.12098,6112],[11.46667,5.91098,9593],[25.96667,-24.83902,4686],[37.76667,-9.38902,12480],[12.76667,54.80098,5648],[19.26667,17.82098,8034],[40.36666,-18.41902,25308],[19.86667,-24.65902,14558],[21.56667,-22.06902,17498],[17.86667,67.14098,4614],[-11.93333,47.16098,3485],[25.26667,53.68098,5092],[22.46667,-4.26902,10432],[20.66667,47.06098,5180],[10.36667,-7.949019,6197],[10.76667,-17.82902,7562],[7.266667,-20.84902,8206],[-0.8333333,68.53098,4036],[-4.933333,66.99098,3148],[2.566667,39.26098,4348],[-4.533333,62.78098,2448],[0.8666667,46.94098,4330],[-15.93333,-17.60902,4761],[-14.13333,54.21098,3016],[-8.133333,63.88098,2901],[-10.73333,-21.35902,5511],[-9.633333,23.29098,3739],[-8.733334,67.16098,3161],[-17.43333,18.08098,4741],[4.266667,27.12098,5052],[-11.13333,10.19098,6259],[-11.23333,34.25098,4075],[-5.333333,-11.93902,7482],[-6.633333,-25.81902,8780],[-20.33333,38.84098,2594],[-32.03333,-21.97902,918],[-23.53333,-25.28902,2370],[0.4666667,-15.88902,8131],[0.2666667,-4.53902,6992],[4.266667,-5.09902,7956],[-3.333333,-28.97902,8895],[4.766667,-27.32902,8891],[-17.13333,23.02098,3116],[-26.63333,-13.46902,3930],[8.066667,-22.96902,7869],[-20.93333,67.55098,611],[-26.03333,40.33098,3000],[-29.53333,4.590981,3472],[-26.73333,1.10098,3582],[-2.733333,-25.37902,3643],[-25.33333,-1.22902,1656],[-11.53333,-28.97902,6860],[-7.933333,4.320981,4199],[-21.63333,-11.71902,5134],[-12.03333,-11.71902,5134],[-23.63333,43.26098,1890],[-13.53333,2.38098,4443],[-18.03333,10.50098,3485],[-4.333333,-27.47902,8043],[-2.633333,-24.69902,6686],[-10.93333,-26.67902,6565],[-5.033333,-23.80902,6477],[-10.93333,-15.35902,5811],[-3.133333,-23.19902,6573],[3.966667,45.56098,3942],[-9.633333,-26.05902,5449],[-18.63333,61.69098,2847],[-8.733334,-28.16902,5795],[3.466667,-28.19902,7716],[-19.53333,-28.97902,4696],[-5.933333,-27.63902,8316],[3.366667,-27.98902,7147],[4.266667,-28.32902,8880],[-7.933333,-28.41902,5299],[-10.63333,-28.45902,5959],[-16.93333,-26.51902,4549],[-3.933333,-28.36902,6928],[-20.33333,-27.88902,3910],[19.26667,-28.39902,14032],[2.066667,-28.97902,8845],[-10.93333,-19.50902,5562],[-21.73333,-25.38902,4224],[-20.73333,-28.97902,4753],[-4.633333,-15.39902,6462],[-11.63333,41.89098,3617]],"colors":[[0,0,1,1]],"radii":[[243.1443]],"centers":[[21.96667,-17.81902,12351],[22.26667,-24.95902,25879],[16.56667,-13.27902,9271],[9.966666,-19.86902,8865],[26.66667,-17.29902,8403],[30.76667,-23.84902,11030],[25.76667,-3.32902,8258],[31.26667,-26.28902,14163],[26.26667,-27.94902,11377],[21.96667,-28.03902,11023],[15.16667,-27.06902,5902],[13.16667,-21.14902,7059],[6.966667,-13.64902,8425],[15.36667,28.33098,8049],[28.06667,19.30098,7405],[8.266666,25.79098,6336],[35.46667,-23.84902,19263],[11.26667,48.12098,6112],[11.46667,5.91098,9593],[25.96667,-24.83902,4686],[37.76667,-9.38902,12480],[12.76667,54.80098,5648],[19.26667,17.82098,8034],[40.36666,-18.41902,25308],[19.86667,-24.65902,14558],[21.56667,-22.06902,17498],[17.86667,67.14098,4614],[-11.93333,47.16098,3485],[25.26667,53.68098,5092],[22.46667,-4.26902,10432],[20.66667,47.06098,5180],[10.36667,-7.949019,6197],[10.76667,-17.82902,7562],[7.266667,-20.84902,8206],[-0.8333333,68.53098,4036],[-4.933333,66.99098,3148],[2.566667,39.26098,4348],[-4.533333,62.78098,2448],[0.8666667,46.94098,4330],[-15.93333,-17.60902,4761],[-14.13333,54.21098,3016],[-8.133333,63.88098,2901],[-10.73333,-21.35902,5511],[-9.633333,23.29098,3739],[-8.733334,67.16098,3161],[-17.43333,18.08098,4741],[4.266667,27.12098,5052],[-11.13333,10.19098,6259],[-11.23333,34.25098,4075],[-5.333333,-11.93902,7482],[-6.633333,-25.81902,8780],[-20.33333,38.84098,2594],[-32.03333,-21.97902,918],[-23.53333,-25.28902,2370],[0.4666667,-15.88902,8131],[0.2666667,-4.53902,6992],[4.266667,-5.09902,7956],[-3.333333,-28.97902,8895],[4.766667,-27.32902,8891],[-17.13333,23.02098,3116],[-26.63333,-13.46902,3930],[8.066667,-22.96902,7869],[-20.93333,67.55098,611],[-26.03333,40.33098,3000],[-29.53333,4.590981,3472],[-26.73333,1.10098,3582],[-2.733333,-25.37902,3643],[-25.33333,-1.22902,1656],[-11.53333,-28.97902,6860],[-7.933333,4.320981,4199],[-21.63333,-11.71902,5134],[-12.03333,-11.71902,5134],[-23.63333,43.26098,1890],[-13.53333,2.38098,4443],[-18.03333,10.50098,3485],[-4.333333,-27.47902,8043],[-2.633333,-24.69902,6686],[-10.93333,-26.67902,6565],[-5.033333,-23.80902,6477],[-10.93333,-15.35902,5811],[-3.133333,-23.19902,6573],[3.966667,45.56098,3942],[-9.633333,-26.05902,5449],[-18.63333,61.69098,2847],[-8.733334,-28.16902,5795],[3.466667,-28.19902,7716],[-19.53333,-28.97902,4696],[-5.933333,-27.63902,8316],[3.366667,-27.98902,7147],[4.266667,-28.32902,8880],[-7.933333,-28.41902,5299],[-10.63333,-28.45902,5959],[-16.93333,-26.51902,4549],[-3.933333,-28.36902,6928],[-20.33333,-27.88902,3910],[19.26667,-28.39902,14032],[2.066667,-28.97902,8845],[-10.93333,-19.50902,5562],[-21.73333,-25.38902,4224],[-20.73333,-28.97902,4753],[-4.633333,-15.39902,6462],[-11.63333,41.89098,3617]],"ignoreExtent":false,"flags":3},"9":{"id":9,"type":"text","material":{"lit":false},"vertices":[[4.166666,87.23504,30725.82]],"colors":[[0,0,0,1]],"texts":[["3D Linear Model Fit"]],"cex":[[1]],"adj":[[0.5,0.5]],"centers":[[4.166666,87.23504,30725.82]],"family":[["sans"]],"font":[[1]],"ignoreExtent":true,"flags":40},"10":{"id":10,"type":"text","material":{"lit":false},"vertices":[[4.166666,-47.68307,-4235.823]],"colors":[[0,0,0,1]],"texts":[["prestige.c"]],"cex":[[1]],"adj":[[0.5,0.5]],"centers":[[4.166666,-47.68307,-4235.823]],"family":[["sans"]],"font":[[1]],"ignoreExtent":true,"flags":40},"11":{"id":11,"type":"text","material":{"lit":false},"vertices":[[-45.92087,19.77598,-4235.823]],"colors":[[0,0,0,1]],"texts":[["women.c"]],"cex":[[1]],"adj":[[0.5,0.5]],"centers":[[-45.92087,19.77598,-4235.823]],"family":[["sans"]],"font":[[1]],"ignoreExtent":true,"flags":40},"12":{"id":12,"type":"text","material":{"lit":false},"vertices":[[-45.92087,-47.68307,13245]],"colors":[[0,0,0,1]],"texts":[["income"]],"cex":[[1]],"adj":[[0.5,0.5]],"centers":[[-45.92087,-47.68307,13245]],"family":[["sans"]],"font":[[1]],"ignoreExtent":true,"flags":40},"13":{"id":13,"type":"surface","material":{"alpha":0.2980392,"front":"lines","back":"lines"},"vertices":[[-35,-25,2201.902],[-30,-25,3031.276],[-25,-25,3860.65],[-20,-25,4690.024],[-15,-25,5519.398],[-10,-25,6348.772],[-5,-25,7178.146],[0,-25,8007.521],[5,-25,8836.895],[10,-25,9666.269],[15,-25,10495.64],[20,-25,11325.02],[25,-25,12154.39],[30,-25,12983.76],[35,-25,13813.14],[40,-25,14642.51],[45,-25,15471.89],[-35,-20,1959.979],[-30,-20,2789.353],[-25,-20,3618.727],[-20,-20,4448.101],[-15,-20,5277.475],[-10,-20,6106.849],[-5,-20,6936.223],[0,-20,7765.597],[5,-20,8594.971],[10,-20,9424.345],[15,-20,10253.72],[20,-20,11083.09],[25,-20,11912.47],[30,-20,12741.84],[35,-20,13571.21],[40,-20,14400.59],[45,-20,15229.96],[-35,-15,1718.055],[-30,-15,2547.429],[-25,-15,3376.803],[-20,-15,4206.177],[-15,-15,5035.551],[-10,-15,5864.925],[-5,-15,6694.299],[0,-15,7523.673],[5,-15,8353.047],[10,-15,9182.421],[15,-15,10011.79],[20,-15,10841.17],[25,-15,11670.54],[30,-15,12499.92],[35,-15,13329.29],[40,-15,14158.67],[45,-15,14988.04],[-35,-10,1476.131],[-30,-10,2305.505],[-25,-10,3134.879],[-20,-10,3964.253],[-15,-10,4793.627],[-10,-10,5623.001],[-5,-10,6452.375],[0,-10,7281.75],[5,-10,8111.124],[10,-10,8940.497],[15,-10,9769.871],[20,-10,10599.25],[25,-10,11428.62],[30,-10,12257.99],[35,-10,13087.37],[40,-10,13916.74],[45,-10,14746.12],[-35,-5,1234.208],[-30,-5,2063.582],[-25,-5,2892.956],[-20,-5,3722.33],[-15,-5,4551.704],[-10,-5,5381.078],[-5,-5,6210.452],[0,-5,7039.826],[5,-5,7869.2],[10,-5,8698.573],[15,-5,9527.947],[20,-5,10357.32],[25,-5,11186.7],[30,-5,12016.07],[35,-5,12845.44],[40,-5,13674.82],[45,-5,14504.19],[-35,0,992.284],[-30,0,1821.658],[-25,0,2651.032],[-20,0,3480.406],[-15,0,4309.78],[-10,0,5139.154],[-5,0,5968.528],[0,0,6797.902],[5,0,7627.276],[10,0,8456.65],[15,0,9286.024],[20,0,10115.4],[25,0,10944.77],[30,0,11774.15],[35,0,12603.52],[40,0,13432.89],[45,0,14262.27],[-35,5,750.3604],[-30,5,1579.734],[-25,5,2409.108],[-20,5,3238.482],[-15,5,4067.856],[-10,5,4897.23],[-5,5,5726.604],[0,5,6555.979],[5,5,7385.352],[10,5,8214.727],[15,5,9044.101],[20,5,9873.475],[25,5,10702.85],[30,5,11532.22],[35,5,12361.6],[40,5,13190.97],[45,5,14020.34],[-35,10,508.4367],[-30,10,1337.811],[-25,10,2167.185],[-20,10,2996.559],[-15,10,3825.933],[-10,10,4655.307],[-5,10,5484.681],[0,10,6314.055],[5,10,7143.429],[10,10,7972.803],[15,10,8802.177],[20,10,9631.551],[25,10,10460.92],[30,10,11290.3],[35,10,12119.67],[40,10,12949.05],[45,10,13778.42],[-35,15,266.513],[-30,15,1095.887],[-25,15,1925.261],[-20,15,2754.635],[-15,15,3584.009],[-10,15,4413.383],[-5,15,5242.757],[0,15,6072.131],[5,15,6901.505],[10,15,7730.879],[15,15,8560.253],[20,15,9389.627],[25,15,10219],[30,15,11048.38],[35,15,11877.75],[40,15,12707.12],[45,15,13536.5],[-35,20,24.58932],[-30,20,853.9633],[-25,20,1683.337],[-20,20,2512.711],[-15,20,3342.085],[-10,20,4171.459],[-5,20,5000.833],[0,20,5830.207],[5,20,6659.581],[10,20,7488.955],[15,20,8318.329],[20,20,9147.703],[25,20,9977.077],[30,20,10806.45],[35,20,11635.83],[40,20,12465.2],[45,20,13294.57],[-35,25,-217.3344],[-30,25,612.0397],[-25,25,1441.414],[-20,25,2270.788],[-15,25,3100.162],[-10,25,3929.536],[-5,25,4758.91],[0,25,5588.284],[5,25,6417.658],[10,25,7247.032],[15,25,8076.406],[20,25,8905.779],[25,25,9735.153],[30,25,10564.53],[35,25,11393.9],[40,25,12223.28],[45,25,13052.65],[-35,30,-459.258],[-30,30,370.116],[-25,30,1199.49],[-20,30,2028.864],[-15,30,2858.238],[-10,30,3687.612],[-5,30,4516.986],[0,30,5346.36],[5,30,6175.734],[10,30,7005.108],[15,30,7834.482],[20,30,8663.855],[25,30,9493.229],[30,30,10322.6],[35,30,11151.98],[40,30,11981.35],[45,30,12810.73],[-35,35,-701.1817],[-30,35,128.1923],[-25,35,957.5663],[-20,35,1786.94],[-15,35,2616.314],[-10,35,3445.688],[-5,35,4275.062],[0,35,5104.436],[5,35,5933.81],[10,35,6763.184],[15,35,7592.558],[20,35,8421.933],[25,35,9251.307],[30,35,10080.68],[35,35,10910.05],[40,35,11739.43],[45,35,12568.8],[-35,40,-943.1053],[-30,40,-113.7314],[-25,40,715.6426],[-20,40,1545.017],[-15,40,2374.391],[-10,40,3203.765],[-5,40,4033.139],[0,40,4862.513],[5,40,5691.887],[10,40,6521.261],[15,40,7350.635],[20,40,8180.008],[25,40,9009.383],[30,40,9838.757],[35,40,10668.13],[40,40,11497.5],[45,40,12326.88],[-35,45,-1185.029],[-30,45,-355.6551],[-25,45,473.7189],[-20,45,1303.093],[-15,45,2132.467],[-10,45,2961.841],[-5,45,3791.215],[0,45,4620.589],[5,45,5449.963],[10,45,6279.337],[15,45,7108.711],[20,45,7938.085],[25,45,8767.459],[30,45,9596.833],[35,45,10426.21],[40,45,11255.58],[45,45,12084.96],[-35,50,-1426.953],[-30,50,-597.5787],[-25,50,231.7953],[-20,50,1061.169],[-15,50,1890.543],[-10,50,2719.917],[-5,50,3549.291],[0,50,4378.665],[5,50,5208.039],[10,50,6037.413],[15,50,6866.787],[20,50,7696.161],[25,50,8525.535],[30,50,9354.909],[35,50,10184.28],[40,50,11013.66],[45,50,11843.03],[-35,55,-1668.876],[-30,55,-839.5024],[-25,55,-10.1284],[-20,55,819.2456],[-15,55,1648.62],[-10,55,2477.994],[-5,55,3307.368],[0,55,4136.742],[5,55,4966.116],[10,55,5795.49],[15,55,6624.864],[20,55,7454.237],[25,55,8283.611],[30,55,9112.985],[35,55,9942.359],[40,55,10771.73],[45,55,11601.11],[-35,60,-1910.8],[-30,60,-1081.426],[-25,60,-252.0521],[-20,60,577.3219],[-15,60,1406.696],[-10,60,2236.07],[-5,60,3065.444],[0,60,3894.818],[5,60,4724.192],[10,60,5553.566],[15,60,6382.94],[20,60,7212.314],[25,60,8041.688],[30,60,8871.062],[35,60,9700.436],[40,60,10529.81],[45,60,11359.18],[-35,65,-2152.724],[-30,65,-1323.35],[-25,65,-493.9757],[-20,65,335.3983],[-15,65,1164.772],[-10,65,1994.146],[-5,65,2823.52],[0,65,3652.894],[5,65,4482.268],[10,65,5311.642],[15,65,6141.016],[20,65,6970.39],[25,65,7799.764],[30,65,8629.138],[35,65,9458.512],[40,65,10287.89],[45,65,11117.26],[-35,70,-2394.647],[-30,70,-1565.273],[-25,70,-735.8994],[-20,70,93.47457],[-15,70,922.8486],[-10,70,1752.223],[-5,70,2581.596],[0,70,3410.97],[5,70,4240.345],[10,70,5069.719],[15,70,5899.092],[20,70,6728.466],[25,70,7557.84],[30,70,8387.215],[35,70,9216.589],[40,70,10045.96],[45,70,10875.34]],"normals":[[-0.9599769,0.2800197,0.005787358],[-0.9599769,0.2800197,0.005787358],[-0.9599768,0.2800197,0.005787358],[-0.9599769,0.2800199,0.005787358],[-0.9599767,0.2800199,0.005787357],[-0.9599767,0.2800199,0.005787357],[-0.9599767,0.2800199,0.005787357],[-0.9599767,0.2800199,0.005787357],[-0.9599767,0.2800199,0.005787357],[-0.9599767,0.2800199,0.005787357],[-0.9599767,0.2800199,0.005787357],[-0.9599767,0.2800199,0.005787357],[-0.9599767,0.2800199,0.005787357],[-0.9599767,0.2800199,0.005787357],[-0.9599767,0.2800199,0.005787357],[-0.9599767,0.2800199,0.005787357],[-0.9599767,0.2800199,0.005787357],[-0.9599768,0.2800198,0.005787358],[-0.9599769,0.2800198,0.005787358],[-0.9599768,0.2800198,0.005787358],[-0.9599769,0.2800198,0.005787359],[-0.9599767,0.2800199,0.005787358],[-0.9599767,0.2800199,0.005787357],[-0.9599767,0.2800199,0.005787357],[-0.9599767,0.2800199,0.005787357],[-0.9599767,0.2800199,0.005787357],[-0.9599767,0.2800199,0.005787357],[-0.9599767,0.2800199,0.005787357],[-0.9599767,0.2800199,0.005787357],[-0.9599767,0.2800199,0.005787357],[-0.9599767,0.2800199,0.005787357],[-0.9599767,0.2800199,0.005787357],[-0.9599767,0.2800199,0.005787357],[-0.9599767,0.2800199,0.005787357],[-0.9599768,0.2800198,0.005787358],[-0.9599769,0.2800198,0.005787358],[-0.9599769,0.2800198,0.005787358],[-0.9599768,0.2800197,0.005787358],[-0.9599768,0.2800197,0.005787358],[-0.9599769,0.2800196,0.005787358],[-0.9599769,0.2800196,0.005787358],[-0.9599769,0.2800196,0.005787358],[-0.9599769,0.2800197,0.005787358],[-0.9599767,0.2800199,0.005787358],[-0.9599767,0.2800199,0.005787357],[-0.9599767,0.2800199,0.005787357],[-0.9599767,0.2800199,0.005787357],[-0.9599767,0.2800199,0.005787357],[-0.9599767,0.2800199,0.005787357],[-0.9599767,0.2800199,0.005787357],[-0.9599767,0.2800199,0.005787357],[-0.9599768,0.2800197,0.005787357],[-0.9599769,0.2800198,0.005787358],[-0.9599769,0.2800198,0.005787358],[-0.9599769,0.2800198,0.005787358],[-0.9599768,0.2800197,0.005787358],[-0.9599769,0.2800197,0.005787358],[-0.9599769,0.2800197,0.005787358],[-0.9599769,0.2800197,0.005787358],[-0.9599768,0.2800198,0.005787359],[-0.9599767,0.2800199,0.005787359],[-0.9599767,0.2800199,0.005787357],[-0.9599767,0.2800199,0.005787357],[-0.9599767,0.2800199,0.005787357],[-0.9599767,0.2800199,0.005787357],[-0.9599767,0.2800199,0.005787357],[-0.9599767,0.2800199,0.005787357],[-0.9599767,0.2800199,0.005787357],[-0.9599768,0.2800198,0.005787357],[-0.9599769,0.2800198,0.005787358],[-0.9599769,0.2800198,0.005787358],[-0.9599769,0.2800199,0.005787358],[-0.9599767,0.2800199,0.005787358],[-0.9599767,0.2800199,0.005787357],[-0.9599767,0.2800199,0.005787357],[-0.9599767,0.2800199,0.005787357],[-0.9599769,0.2800198,0.005787359],[-0.9599769,0.2800196,0.005787359],[-0.9599769,0.2800196,0.005787359],[-0.9599769,0.2800198,0.005787359],[-0.9599767,0.2800199,0.005787357],[-0.9599767,0.2800199,0.005787357],[-0.9599767,0.2800199,0.005787357],[-0.9599767,0.2800199,0.005787357],[-0.9599767,0.2800199,0.005787357],[-0.9599768,0.2800198,0.005787358],[-0.9599769,0.2800198,0.005787358],[-0.9599769,0.2800197,0.005787358],[-0.9599769,0.2800198,0.005787358],[-0.9599768,0.2800198,0.005787358],[-0.9599768,0.2800197,0.005787357],[-0.9599769,0.2800196,0.005787358],[-0.9599769,0.2800197,0.005787358],[-0.9599769,0.2800197,0.005787358],[-0.9599769,0.2800195,0.005787358],[-0.9599769,0.2800195,0.00578736],[-0.9599769,0.2800195,0.005787359],[-0.9599769,0.2800194,0.005787358],[-0.9599769,0.2800194,0.005787358],[-0.9599769,0.2800194,0.005787358],[-0.9599769,0.2800196,0.005787359],[-0.9599768,0.2800197,0.005787359],[-0.9599768,0.2800198,0.005787357],[-0.9599769,0.2800198,0.005787358],[-0.9599769,0.2800198,0.005787358],[-0.9599769,0.2800198,0.005787358],[-0.9599769,0.2800198,0.005787358],[-0.9599769,0.2800197,0.005787358],[-0.9599769,0.2800197,0.005787358],[-0.9599768,0.2800197,0.005787359],[-0.9599769,0.2800197,0.005787357],[-0.9599769,0.2800198,0.005787357],[-0.9599769,0.2800198,0.005787359],[-0.9599769,0.2800196,0.00578736],[-0.959977,0.2800194,0.005787359],[-0.959977,0.2800194,0.005787359],[-0.959977,0.2800194,0.005787359],[-0.9599769,0.2800195,0.00578736],[-0.9599769,0.2800196,0.005787361],[-0.9599768,0.2800198,0.005787358],[-0.9599768,0.2800198,0.005787358],[-0.9599769,0.2800198,0.005787358],[-0.9599769,0.2800198,0.005787358],[-0.9599769,0.2800198,0.005787358],[-0.9599769,0.2800199,0.005787358],[-0.9599767,0.2800199,0.005787357],[-0.9599769,0.2800199,0.005787359],[-0.9599769,0.2800198,0.005787358],[-0.9599769,0.2800198,0.005787358],[-0.9599767,0.2800199,0.005787357],[-0.9599767,0.2800199,0.005787357],[-0.9599767,0.2800199,0.005787357],[-0.9599767,0.2800199,0.005787357],[-0.9599767,0.2800199,0.005787357],[-0.9599769,0.2800198,0.005787359],[-0.9599768,0.2800197,0.00578736],[-0.9599768,0.2800198,0.005787358],[-0.9599769,0.2800198,0.005787358],[-0.9599769,0.2800198,0.005787358],[-0.9599769,0.2800198,0.005787358],[-0.9599768,0.2800198,0.005787358],[-0.9599768,0.2800197,0.005787358],[-0.9599769,0.2800197,0.005787358],[-0.9599767,0.2800199,0.005787358],[-0.9599767,0.2800199,0.005787357],[-0.9599767,0.2800199,0.005787357],[-0.9599767,0.2800199,0.005787357],[-0.9599767,0.2800199,0.005787357],[-0.9599767,0.2800199,0.005787357],[-0.9599767,0.2800199,0.005787357],[-0.9599767,0.2800199,0.005787357],[-0.9599767,0.2800199,0.005787357],[-0.9599767,0.2800199,0.005787357],[-0.9599768,0.2800198,0.005787357],[-0.9599769,0.2800198,0.005787358],[-0.9599768,0.2800198,0.005787358],[-0.9599769,0.2800198,0.005787358],[-0.9599769,0.2800198,0.005787358],[-0.9599768,0.2800197,0.005787358],[-0.9599768,0.2800197,0.005787359],[-0.9599768,0.2800197,0.005787358],[-0.9599769,0.2800196,0.005787358],[-0.9599769,0.2800196,0.005787358],[-0.9599769,0.2800197,0.005787358],[-0.9599767,0.2800199,0.005787358],[-0.9599767,0.2800199,0.005787357],[-0.9599767,0.2800199,0.005787357],[-0.9599767,0.2800199,0.005787357],[-0.9599767,0.2800199,0.005787357],[-0.9599767,0.2800199,0.005787357],[-0.9599768,0.2800197,0.005787357],[-0.9599769,0.2800198,0.005787358],[-0.9599769,0.2800198,0.005787358],[-0.9599769,0.2800197,0.005787358],[-0.9599769,0.2800198,0.005787358],[-0.9599769,0.2800199,0.005787358],[-0.9599769,0.2800199,0.005787359],[-0.9599768,0.2800197,0.005787358],[-0.9599769,0.2800197,0.005787358],[-0.9599769,0.2800197,0.005787358],[-0.9599768,0.2800198,0.005787359],[-0.9599767,0.2800199,0.005787359],[-0.9599767,0.2800199,0.005787357],[-0.9599767,0.2800199,0.005787357],[-0.9599767,0.2800199,0.005787357],[-0.9599767,0.2800199,0.005787357],[-0.9599767,0.2800199,0.005787357],[-0.9599768,0.2800198,0.005787357],[-0.9599769,0.2800198,0.005787358],[-0.9599769,0.2800198,0.005787358],[-0.9599769,0.2800198,0.005787358],[-0.9599769,0.2800198,0.005787358],[-0.9599768,0.2800198,0.005787358],[-0.9599769,0.2800198,0.005787358],[-0.9599767,0.2800199,0.005787358],[-0.9599767,0.2800199,0.005787357],[-0.9599767,0.2800199,0.005787357],[-0.9599769,0.2800198,0.005787359],[-0.9599769,0.2800196,0.005787359],[-0.9599769,0.2800194,0.005787358],[-0.9599769,0.2800196,0.005787359],[-0.9599769,0.2800198,0.005787359],[-0.9599767,0.2800199,0.005787357],[-0.9599767,0.2800199,0.005787357],[-0.9599768,0.2800197,0.005787358],[-0.9599769,0.2800198,0.005787358],[-0.9599769,0.2800198,0.005787358],[-0.9599768,0.2800198,0.005787358],[-0.9599769,0.2800198,0.005787358],[-0.9599769,0.2800197,0.005787358],[-0.9599768,0.2800197,0.005787358],[-0.9599768,0.2800197,0.005787358],[-0.9599769,0.2800196,0.005787358],[-0.9599769,0.2800196,0.005787358],[-0.9599769,0.2800196,0.005787358],[-0.9599769,0.2800196,0.005787357],[-0.959977,0.2800195,0.005787358],[-0.9599769,0.2800195,0.00578736],[-0.9599769,0.2800195,0.005787359],[-0.9599769,0.2800194,0.005787358],[-0.9599769,0.2800194,0.005787358],[-0.9599768,0.2800198,0.005787358],[-0.9599769,0.2800198,0.005787358],[-0.9599769,0.2800198,0.005787358],[-0.9599769,0.2800198,0.005787358],[-0.9599769,0.2800198,0.005787358],[-0.9599769,0.2800198,0.005787358],[-0.9599769,0.2800198,0.005787358],[-0.9599768,0.2800197,0.005787358],[-0.9599769,0.2800197,0.005787358],[-0.9599769,0.2800197,0.005787358],[-0.9599768,0.2800197,0.005787358],[-0.9599769,0.2800198,0.005787357],[-0.9599767,0.2800199,0.005787357],[-0.9599769,0.2800198,0.005787359],[-0.9599769,0.2800196,0.00578736],[-0.959977,0.2800194,0.005787359],[-0.959977,0.2800194,0.005787359],[-0.9599768,0.2800198,0.005787357],[-0.9599769,0.2800198,0.005787358],[-0.9599769,0.2800198,0.005787358],[-0.9599769,0.2800198,0.005787358],[-0.9599769,0.2800198,0.005787358],[-0.9599769,0.2800198,0.005787358],[-0.9599769,0.2800199,0.005787358],[-0.9599767,0.2800199,0.005787358],[-0.9599767,0.2800199,0.005787357],[-0.9599767,0.2800199,0.005787357],[-0.9599769,0.2800199,0.005787359],[-0.9599769,0.2800198,0.005787358],[-0.9599769,0.2800198,0.005787358],[-0.9599767,0.2800199,0.005787357],[-0.9599767,0.2800199,0.005787357],[-0.9599767,0.2800199,0.005787357],[-0.9599767,0.2800199,0.005787357],[-0.9599768,0.2800197,0.005787358],[-0.9599769,0.2800198,0.005787358],[-0.9599769,0.2800198,0.005787358],[-0.9599769,0.2800198,0.005787358],[-0.9599769,0.2800198,0.005787358],[-0.9599769,0.2800197,0.005787358],[-0.9599768,0.2800197,0.005787358],[-0.9599768,0.2800197,0.005787358],[-0.9599769,0.2800196,0.005787358],[-0.9599769,0.2800196,0.005787358],[-0.9599769,0.2800197,0.005787358],[-0.9599767,0.2800199,0.005787358],[-0.9599767,0.2800199,0.005787357],[-0.9599767,0.2800199,0.005787357],[-0.9599767,0.2800199,0.005787357],[-0.9599767,0.2800199,0.005787357],[-0.9599767,0.2800199,0.005787357],[-0.9599768,0.2800197,0.005787358],[-0.9599769,0.2800198,0.005787358],[-0.9599769,0.2800198,0.005787358],[-0.9599769,0.2800198,0.005787358],[-0.9599769,0.2800198,0.005787358],[-0.9599769,0.2800198,0.005787358],[-0.9599769,0.2800198,0.005787358],[-0.9599768,0.2800197,0.005787358],[-0.9599769,0.2800197,0.005787358],[-0.9599769,0.2800197,0.005787358],[-0.9599768,0.2800197,0.005787359],[-0.9599768,0.2800197,0.005787358],[-0.9599769,0.2800197,0.005787358],[-0.9599767,0.2800199,0.005787358],[-0.9599767,0.2800199,0.005787357],[-0.9599767,0.2800199,0.005787357],[-0.9599767,0.2800199,0.005787357],[-0.9599768,0.2800198,0.005787358],[-0.9599769,0.2800198,0.005787358],[-0.9599769,0.2800198,0.005787358],[-0.9599769,0.2800198,0.005787358],[-0.9599768,0.2800198,0.005787358],[-0.9599769,0.2800198,0.005787358],[-0.9599769,0.2800198,0.005787358],[-0.9599769,0.2800199,0.005787358],[-0.9599767,0.2800199,0.005787358],[-0.9599767,0.2800199,0.005787357],[-0.9599769,0.2800199,0.005787359],[-0.9599768,0.2800197,0.005787358],[-0.9599768,0.2800198,0.005787359],[-0.9599767,0.2800199,0.005787359],[-0.9599767,0.2800199,0.005787357],[-0.9599767,0.2800199,0.005787357],[-0.9599767,0.2800199,0.005787357],[-0.9599768,0.2800198,0.005787358],[-0.9599769,0.2800198,0.005787358],[-0.9599769,0.2800198,0.005787358],[-0.9599769,0.2800198,0.005787358],[-0.9599769,0.2800198,0.005787358],[-0.9599769,0.2800198,0.005787358],[-0.9599769,0.2800198,0.005787358],[-0.9599768,0.2800198,0.005787358],[-0.9599768,0.2800197,0.005787358],[-0.9599769,0.2800197,0.005787358],[-0.9599767,0.2800199,0.005787358],[-0.9599767,0.2800199,0.005787357],[-0.9599769,0.2800198,0.005787359],[-0.9599769,0.2800196,0.005787359],[-0.9599769,0.2800194,0.005787358],[-0.9599769,0.2800194,0.005787358],[-0.9599769,0.2800194,0.005787358],[-0.9599767,0.2800199,0.005787357],[-0.9599768,0.2800198,0.005787358],[-0.9599768,0.2800198,0.005787358],[-0.9599768,0.2800198,0.005787357],[-0.9599768,0.2800198,0.005787358],[-0.9599768,0.2800198,0.005787358],[-0.9599768,0.2800199,0.005787357],[-0.9599769,0.2800198,0.005787358],[-0.9599769,0.2800195,0.005787358],[-0.9599769,0.2800196,0.005787359],[-0.9599768,0.2800198,0.005787358],[-0.9599767,0.2800199,0.005787357],[-0.9599769,0.2800197,0.005787358],[-0.959977,0.2800192,0.005787359],[-0.9599771,0.2800189,0.00578736],[-0.9599771,0.2800189,0.00578736],[-0.9599771,0.2800189,0.00578736]],"colors":[[0,0,0,0.2980392]],"dim":[[17,20]],"centers":[[-32.5,-22.5,2495.627],[-27.5,-22.5,3325.001],[-22.5,-22.5,4154.375],[-17.5,-22.5,4983.75],[-12.5,-22.5,5813.123],[-7.5,-22.5,6642.498],[-2.5,-22.5,7471.872],[2.5,-22.5,8301.246],[7.5,-22.5,9130.62],[12.5,-22.5,9959.993],[17.5,-22.5,10789.37],[22.5,-22.5,11618.74],[27.5,-22.5,12448.12],[32.5,-22.5,13277.49],[37.5,-22.5,14106.86],[42.5,-22.5,14936.24],[-32.5,-17.5,2253.704],[-27.5,-17.5,3083.078],[-22.5,-17.5,3912.452],[-17.5,-17.5,4741.826],[-12.5,-17.5,5571.2],[-7.5,-17.5,6400.574],[-2.5,-17.5,7229.947],[2.5,-17.5,8059.322],[7.5,-17.5,8888.696],[12.5,-17.5,9718.07],[17.5,-17.5,10547.44],[22.5,-17.5,11376.82],[27.5,-17.5,12206.19],[32.5,-17.5,13035.57],[37.5,-17.5,13864.94],[42.5,-17.5,14694.31],[-32.5,-12.5,2011.78],[-27.5,-12.5,2841.154],[-22.5,-12.5,3670.528],[-17.5,-12.5,4499.902],[-12.5,-12.5,5329.276],[-7.5,-12.5,6158.65],[-2.5,-12.5,6988.024],[2.5,-12.5,7817.398],[7.5,-12.5,8646.772],[12.5,-12.5,9476.146],[17.5,-12.5,10305.52],[22.5,-12.5,11134.89],[27.5,-12.5,11964.27],[32.5,-12.5,12793.64],[37.5,-12.5,13623.02],[42.5,-12.5,14452.39],[-32.5,-7.5,1769.856],[-27.5,-7.5,2599.23],[-22.5,-7.5,3428.604],[-17.5,-7.5,4257.979],[-12.5,-7.5,5087.353],[-7.5,-7.5,5916.727],[-2.5,-7.5,6746.101],[2.5,-7.5,7575.475],[7.5,-7.5,8404.849],[12.5,-7.5,9234.223],[17.5,-7.5,10063.6],[22.5,-7.5,10892.97],[27.5,-7.5,11722.34],[32.5,-7.5,12551.72],[37.5,-7.5,13381.09],[42.5,-7.5,14210.47],[-32.5,-2.5,1527.933],[-27.5,-2.5,2357.307],[-22.5,-2.5,3186.681],[-17.5,-2.5,4016.055],[-12.5,-2.5,4845.429],[-7.5,-2.5,5674.803],[-2.5,-2.5,6504.177],[2.5,-2.5,7333.551],[7.5,-2.5,8162.925],[12.5,-2.5,8992.299],[17.5,-2.5,9821.673],[22.5,-2.5,10651.05],[27.5,-2.5,11480.42],[32.5,-2.5,12309.79],[37.5,-2.5,13139.17],[42.5,-2.5,13968.54],[-32.5,2.5,1286.009],[-27.5,2.5,2115.383],[-22.5,2.5,2944.757],[-17.5,2.5,3774.131],[-12.5,2.5,4603.505],[-7.5,2.5,5432.879],[-2.5,2.5,6262.253],[2.5,2.5,7091.627],[7.5,2.5,7921.001],[12.5,2.5,8750.376],[17.5,2.5,9579.75],[22.5,2.5,10409.12],[27.5,2.5,11238.5],[32.5,2.5,12067.87],[37.5,2.5,12897.25],[42.5,2.5,13726.62],[-32.5,7.5,1044.086],[-27.5,7.5,1873.459],[-22.5,7.5,2702.833],[-17.5,7.5,3532.208],[-12.5,7.5,4361.582],[-7.5,7.5,5190.956],[-2.5,7.5,6020.33],[2.5,7.5,6849.703],[7.5,7.5,7679.078],[12.5,7.5,8508.452],[17.5,7.5,9337.825],[22.5,7.5,10167.2],[27.5,7.5,10996.57],[32.5,7.5,11825.95],[37.5,7.5,12655.32],[42.5,7.5,13484.7],[-32.5,12.5,802.1618],[-27.5,12.5,1631.536],[-22.5,12.5,2460.91],[-17.5,12.5,3290.284],[-12.5,12.5,4119.658],[-7.5,12.5,4949.032],[-2.5,12.5,5778.406],[2.5,12.5,6607.779],[7.5,12.5,7437.154],[12.5,12.5,8266.528],[17.5,12.5,9095.902],[22.5,12.5,9925.276],[27.5,12.5,10754.65],[32.5,12.5,11584.02],[37.5,12.5,12413.4],[42.5,12.5,13242.77],[-32.5,17.5,560.2382],[-27.5,17.5,1389.612],[-22.5,17.5,2218.986],[-17.5,17.5,3048.36],[-12.5,17.5,3877.734],[-7.5,17.5,4707.108],[-2.5,17.5,5536.482],[2.5,17.5,6365.856],[7.5,17.5,7195.23],[12.5,17.5,8024.604],[17.5,17.5,8853.978],[22.5,17.5,9683.353],[27.5,17.5,10512.73],[32.5,17.5,11342.1],[37.5,17.5,12171.47],[42.5,17.5,13000.85],[-32.5,22.5,318.3145],[-27.5,22.5,1147.688],[-22.5,22.5,1977.062],[-17.5,22.5,2806.437],[-12.5,22.5,3635.811],[-7.5,22.5,4465.185],[-2.5,22.5,5294.559],[2.5,22.5,6123.933],[7.5,22.5,6953.306],[12.5,22.5,7782.681],[17.5,22.5,8612.055],[22.5,22.5,9441.429],[27.5,22.5,10270.8],[32.5,22.5,11100.18],[37.5,22.5,11929.55],[42.5,22.5,12758.92],[-32.5,27.5,76.39082],[-27.5,27.5,905.7648],[-22.5,27.5,1735.139],[-17.5,27.5,2564.513],[-12.5,27.5,3393.887],[-7.5,27.5,4223.261],[-2.5,27.5,5052.635],[2.5,27.5,5882.009],[7.5,27.5,6711.383],[12.5,27.5,7540.757],[17.5,27.5,8370.131],[22.5,27.5,9199.505],[27.5,27.5,10028.88],[32.5,27.5,10858.25],[37.5,27.5,11687.63],[42.5,27.5,12517],[-32.5,32.5,-165.5329],[-27.5,32.5,663.8411],[-22.5,32.5,1493.215],[-17.5,32.5,2322.589],[-12.5,32.5,3151.963],[-7.5,32.5,3981.337],[-2.5,32.5,4810.711],[2.5,32.5,5640.085],[7.5,32.5,6469.459],[12.5,32.5,7298.833],[17.5,32.5,8128.207],[22.5,32.5,8957.582],[27.5,32.5,9786.955],[32.5,32.5,10616.33],[37.5,32.5,11445.7],[42.5,32.5,12275.08],[-32.5,37.5,-407.4565],[-27.5,37.5,421.9174],[-22.5,37.5,1251.292],[-17.5,37.5,2080.666],[-12.5,37.5,2910.039],[-7.5,37.5,3739.414],[-2.5,37.5,4568.787],[2.5,37.5,5398.161],[7.5,37.5,6227.535],[12.5,37.5,7056.91],[17.5,37.5,7886.283],[22.5,37.5,8715.657],[27.5,37.5,9545.032],[32.5,37.5,10374.41],[37.5,37.5,11203.78],[42.5,37.5,12033.15],[-32.5,42.5,-649.3802],[-27.5,42.5,179.9938],[-22.5,42.5,1009.368],[-17.5,42.5,1838.742],[-12.5,42.5,2668.116],[-7.5,42.5,3497.49],[-2.5,42.5,4326.864],[2.5,42.5,5156.238],[7.5,42.5,5985.611],[12.5,42.5,6814.986],[17.5,42.5,7644.359],[22.5,42.5,8473.734],[27.5,42.5,9303.108],[32.5,42.5,10132.48],[37.5,42.5,10961.86],[42.5,42.5,11791.23],[-32.5,47.5,-891.3038],[-27.5,47.5,-61.9299],[-22.5,47.5,767.4441],[-17.5,47.5,1596.818],[-12.5,47.5,2426.192],[-7.5,47.5,3255.566],[-2.5,47.5,4084.94],[2.5,47.5,4914.314],[7.5,47.5,5743.688],[12.5,47.5,6573.062],[17.5,47.5,7402.436],[22.5,47.5,8231.81],[27.5,47.5,9061.185],[32.5,47.5,9890.559],[37.5,47.5,10719.93],[42.5,47.5,11549.31],[-32.5,52.5,-1133.228],[-27.5,52.5,-303.8536],[-22.5,52.5,525.5204],[-17.5,52.5,1354.894],[-12.5,52.5,2184.269],[-7.5,52.5,3013.643],[-2.5,52.5,3843.017],[2.5,52.5,4672.39],[7.5,52.5,5501.765],[12.5,52.5,6331.138],[17.5,52.5,7160.513],[22.5,52.5,7989.886],[27.5,52.5,8819.261],[32.5,52.5,9648.634],[37.5,52.5,10478.01],[42.5,52.5,11307.38],[-32.5,57.5,-1375.151],[-27.5,57.5,-545.7772],[-22.5,57.5,283.5967],[-17.5,57.5,1112.971],[-12.5,57.5,1942.345],[-7.5,57.5,2771.719],[-2.5,57.5,3601.093],[2.5,57.5,4430.467],[7.5,57.5,5259.841],[12.5,57.5,6089.215],[17.5,57.5,6918.589],[22.5,57.5,7747.962],[27.5,57.5,8577.337],[32.5,57.5,9406.711],[37.5,57.5,10236.08],[42.5,57.5,11065.46],[-32.5,62.5,-1617.075],[-27.5,62.5,-787.7009],[-22.5,62.5,41.67309],[-17.5,62.5,871.0471],[-12.5,62.5,1700.421],[-7.5,62.5,2529.795],[-2.5,62.5,3359.169],[2.5,62.5,4188.543],[7.5,62.5,5017.917],[12.5,62.5,5847.291],[17.5,62.5,6676.665],[22.5,62.5,7506.039],[27.5,62.5,8335.413],[32.5,62.5,9164.786],[37.5,62.5,9994.161],[42.5,62.5,10823.54],[-32.5,67.5,-1858.999],[-27.5,67.5,-1029.625],[-22.5,67.5,-200.2506],[-17.5,67.5,629.1234],[-12.5,67.5,1458.497],[-7.5,67.5,2287.871],[-2.5,67.5,3117.245],[2.5,67.5,3946.619],[7.5,67.5,4775.993],[12.5,67.5,5605.367],[17.5,67.5,6434.741],[22.5,67.5,7264.115],[27.5,67.5,8093.489],[32.5,67.5,8922.863],[37.5,67.5,9752.238],[42.5,67.5,10581.61]],"ignoreExtent":false,"flags":91},"5":{"id":5,"type":"light","vertices":[[0,0,1]],"colors":[[1,1,1,1],[1,1,1,1],[1,1,1,1]],"viewpoint":true,"finite":false},"4":{"id":4,"type":"background","material":{"fog":true},"colors":[[0.2980392,0.2980392,0.2980392,1]],"centers":[[0,0,0]],"sphere":false,"fogtype":"none","flags":0},"6":{"id":6,"type":"background","material":{"lit":false,"back":"lines"},"colors":[[1,1,1,1]],"centers":[[0,0,0]],"sphere":false,"fogtype":"none","flags":0},"8":{"id":8,"type":"bboxdeco","material":{"front":"lines","back":"lines"},"vertices":[[-20,"NA","NA"],[0,"NA","NA"],[20,"NA","NA"],[40,"NA","NA"],["NA",-20,"NA"],["NA",0,"NA"],["NA",20,"NA"],["NA",40,"NA"],["NA",60,"NA"],["NA","NA",0],["NA","NA",5000],["NA","NA",10000],["NA","NA",15000],["NA","NA",20000],["NA","NA",25000]],"colors":[[0,0,0,1]],"draw_front":true,"newIds":[21,22,23,24,25,26,27]},"1":{"id":1,"type":"subscene","par3d":{"antialias":8,"FOV":30,"ignoreExtent":false,"listeners":1,"mouseMode":{"left":"trackball","right":"zoom","middle":"fov","wheel":"pull"},"observer":[0,0,67529.3],"modelMatrix":[[201.4993,0,0,-1007.497],[0,51.17001,0.542538,-7496.754],[0,-140.5884,0.1974677,-67109.3],[0,0,0,1]],"projMatrix":[[2.665751,0,0,0],[0,3.732051,0,0],[0,0,-3.863703,-243435.3],[0,0,-1,0]],"skipRedraw":false,"userMatrix":[[1,0,0,0],[0,0.3420201,0.9396926,0],[0,-0.9396926,0.3420201,0],[0,0,0,1]],"scale":[201.4993,149.6111,0.5773569],"viewport":{"x":0,"y":0,"width":1,"height":1},"zoom":1,"bbox":[-35,45,-30.60419,70.15616,-2394.647,26300.13],"windowRect":[0,45,672,525],"family":"sans","font":1,"cex":1,"useFreeType":true,"fontname":"/Library/Frameworks/R.framework/Versions/3.2/Resources/library/rgl/fonts/FreeSans.ttf","maxClipPlanes":6},"embeddings":{"viewport":"replace","projection":"replace","model":"replace"},"objects":[6,8,7,9,10,11,12,13,5,21,22,23,24,25,26,27],"subscenes":[],"flags":1275},"21":{"id":21,"type":"lines","material":{"lit":false,"front":"lines","back":"lines"},"vertices":[[-20,-32.1156,-2825.069],[40,-32.1156,-2825.069],[-20,-32.1156,-2825.069],[-20,-34.71018,-3563.96],[0,-32.1156,-2825.069],[0,-34.71018,-3563.96],[20,-32.1156,-2825.069],[20,-34.71018,-3563.96],[40,-32.1156,-2825.069],[40,-34.71018,-3563.96]],"colors":[[0,0,0,1]],"centers":[[10,-32.1156,-2825.069],[-20,-33.41289,-3194.514],[0,-33.41289,-3194.514],[20,-33.41289,-3194.514],[40,-33.41289,-3194.514]],"ignoreExtent":true,"origId":8,"flags":128},"22":{"id":22,"type":"text","material":{"lit":false,"front":"lines","back":"lines"},"vertices":[[-20,-39.89934,-5041.741],[0,-39.89934,-5041.741],[20,-39.89934,-5041.741],[40,-39.89934,-5041.741]],"colors":[[0,0,0,1]],"texts":[["-20"],["0"],["20"],["40"]],"cex":[[1]],"adj":[[0.5,0.5]],"centers":[[-20,-39.89934,-5041.741],[0,-39.89934,-5041.741],[20,-39.89934,-5041.741],[40,-39.89934,-5041.741]],"family":[["sans"]],"font":[[1]],"ignoreExtent":true,"origId":8,"flags":40},"23":{"id":23,"type":"lines","material":{"lit":false,"front":"lines","back":"lines"},"vertices":[[-36.2,-20,-2825.069],[-36.2,60,-2825.069],[-36.2,-20,-2825.069],[-38.26,-20,-3563.96],[-36.2,0,-2825.069],[-38.26,0,-3563.96],[-36.2,20,-2825.069],[-38.26,20,-3563.96],[-36.2,40,-2825.069],[-38.26,40,-3563.96],[-36.2,60,-2825.069],[-38.26,60,-3563.96]],"colors":[[0,0,0,1]],"centers":[[-36.2,20,-2825.069],[-37.23,-20,-3194.514],[-37.23,0,-3194.514],[-37.23,20,-3194.514],[-37.23,40,-3194.514],[-37.23,60,-3194.514]],"ignoreExtent":true,"origId":8,"flags":128},"24":{"id":24,"type":"text","material":{"lit":false,"front":"lines","back":"lines"},"vertices":[[-42.38,-20,-5041.741],[-42.38,0,-5041.741],[-42.38,20,-5041.741],[-42.38,40,-5041.741],[-42.38,60,-5041.741]],"colors":[[0,0,0,1]],"texts":[["-20"],["0"],["20"],["40"],["60"]],"cex":[[1]],"adj":[[0.5,0.5]],"centers":[[-42.38,-20,-5041.741],[-42.38,0,-5041.741],[-42.38,20,-5041.741],[-42.38,40,-5041.741],[-42.38,60,-5041.741]],"family":[["sans"]],"font":[[1]],"ignoreExtent":true,"origId":8,"flags":40},"25":{"id":25,"type":"lines","material":{"lit":false,"front":"lines","back":"lines"},"vertices":[[-36.2,-32.1156,0],[-36.2,-32.1156,25000],[-36.2,-32.1156,0],[-38.26,-34.71018,0],[-36.2,-32.1156,5000],[-38.26,-34.71018,5000],[-36.2,-32.1156,10000],[-38.26,-34.71018,10000],[-36.2,-32.1156,15000],[-38.26,-34.71018,15000],[-36.2,-32.1156,20000],[-38.26,-34.71018,20000],[-36.2,-32.1156,25000],[-38.26,-34.71018,25000]],"colors":[[0,0,0,1]],"centers":[[-36.2,-32.1156,12500],[-37.23,-33.41289,0],[-37.23,-33.41289,5000],[-37.23,-33.41289,10000],[-37.23,-33.41289,15000],[-37.23,-33.41289,20000],[-37.23,-33.41289,25000]],"ignoreExtent":true,"origId":8,"flags":128},"26":{"id":26,"type":"text","material":{"lit":false,"front":"lines","back":"lines"},"vertices":[[-42.38,-39.89934,0],[-42.38,-39.89934,5000],[-42.38,-39.89934,10000],[-42.38,-39.89934,15000],[-42.38,-39.89934,20000],[-42.38,-39.89934,25000]],"colors":[[0,0,0,1]],"texts":[["0"],["5000"],["10000"],["15000"],["20000"],["25000"]],"cex":[[1]],"adj":[[0.5,0.5]],"centers":[[-42.38,-39.89934,0],[-42.38,-39.89934,5000],[-42.38,-39.89934,10000],[-42.38,-39.89934,15000],[-42.38,-39.89934,20000],[-42.38,-39.89934,25000]],"family":[["sans"]],"font":[[1]],"ignoreExtent":true,"origId":8,"flags":40},"27":{"id":27,"type":"lines","material":{"lit":false,"front":"lines","back":"lines"},"vertices":[[-36.2,-32.1156,-2825.069],[-36.2,71.66756,-2825.069],[-36.2,-32.1156,26730.55],[-36.2,71.66756,26730.55],[-36.2,-32.1156,-2825.069],[-36.2,-32.1156,26730.55],[-36.2,71.66756,-2825.069],[-36.2,71.66756,26730.55],[-36.2,-32.1156,-2825.069],[46.2,-32.1156,-2825.069],[-36.2,-32.1156,26730.55],[46.2,-32.1156,26730.55],[-36.2,71.66756,-2825.069],[46.2,71.66756,-2825.069],[-36.2,71.66756,26730.55],[46.2,71.66756,26730.55],[46.2,-32.1156,-2825.069],[46.2,71.66756,-2825.069],[46.2,-32.1156,26730.55],[46.2,71.66756,26730.55],[46.2,-32.1156,-2825.069],[46.2,-32.1156,26730.55],[46.2,71.66756,-2825.069],[46.2,71.66756,26730.55]],"colors":[[0,0,0,1]],"centers":[[-36.2,19.77598,-2825.069],[-36.2,19.77598,26730.55],[-36.2,-32.1156,11952.74],[-36.2,71.66756,11952.74],[5,-32.1156,-2825.069],[5,-32.1156,26730.55],[5,71.66756,-2825.069],[5,71.66756,26730.55],[46.2,19.77598,-2825.069],[46.2,19.77598,26730.55],[46.2,-32.1156,11952.74],[46.2,71.66756,11952.74]],"ignoreExtent":true,"origId":8,"flags":128}},"snapshot":"data:image/png;base64,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","width":673,"height":481,"sphereVerts":{"vb":[[0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0.07465783,0.1464466,0.2126075,0.2705981,0.3181896,0.3535534,0.3753303,0.3826834,0.3753303,0.3535534,0.3181896,0.2705981,0.2126075,0.1464466,0.07465783,0,0,0.1379497,0.2705981,0.3928475,0.5,0.5879378,0.6532815,0.6935199,0.7071068,0.6935199,0.6532815,0.5879378,0.5,0.3928475,0.2705981,0.1379497,0,0,0.18024,0.3535534,0.51328,0.6532815,0.7681778,0.8535534,0.9061274,0.9238795,0.9061274,0.8535534,0.7681778,0.6532815,0.51328,0.3535534,0.18024,0,0,0.1950903,0.3826834,0.5555702,0.7071068,0.8314696,0.9238795,0.9807853,1,0.9807853,0.9238795,0.8314696,0.7071068,0.5555702,0.3826834,0.1950903,0,0,0.18024,0.3535534,0.51328,0.6532815,0.7681778,0.8535534,0.9061274,0.9238795,0.9061274,0.8535534,0.7681778,0.6532815,0.51328,0.3535534,0.18024,0,0,0.1379497,0.2705981,0.3928475,0.5,0.5879378,0.6532815,0.6935199,0.7071068,0.6935199,0.6532815,0.5879378,0.5,0.3928475,0.2705981,0.1379497,0,0,0.07465783,0.1464466,0.2126075,0.2705981,0.3181896,0.3535534,0.3753303,0.3826834,0.3753303,0.3535534,0.3181896,0.2705981,0.2126075,0.1464466,0.07465783,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,-0,-0.07465783,-0.1464466,-0.2126075,-0.2705981,-0.3181896,-0.3535534,-0.3753303,-0.3826834,-0.3753303,-0.3535534,-0.3181896,-0.2705981,-0.2126075,-0.1464466,-0.07465783,-0,-0,-0.1379497,-0.2705981,-0.3928475,-0.5,-0.5879378,-0.6532815,-0.6935199,-0.7071068,-0.6935199,-0.6532815,-0.5879378,-0.5,-0.3928475,-0.2705981,-0.1379497,-0,-0,-0.18024,-0.3535534,-0.51328,-0.6532815,-0.7681778,-0.8535534,-0.9061274,-0.9238795,-0.9061274,-0.8535534,-0.7681778,-0.6532815,-0.51328,-0.3535534,-0.18024,-0,-0,-0.1950903,-0.3826834,-0.5555702,-0.7071068,-0.8314696,-0.9238795,-0.9807853,-1,-0.9807853,-0.9238795,-0.8314696,-0.7071068,-0.5555702,-0.3826834,-0.1950903,-0,-0,-0.18024,-0.3535534,-0.51328,-0.6532815,-0.7681778,-0.8535534,-0.9061274,-0.9238795,-0.9061274,-0.8535534,-0.7681778,-0.6532815,-0.51328,-0.3535534,-0.18024,-0,-0,-0.1379497,-0.2705981,-0.3928475,-0.5,-0.5879378,-0.6532815,-0.6935199,-0.7071068,-0.6935199,-0.6532815,-0.5879378,-0.5,-0.3928475,-0.2705981,-0.1379497,-0,-0,-0.07465783,-0.1464466,-0.2126075,-0.2705981,-0.3181896,-0.3535534,-0.3753303,-0.3826834,-0.3753303,-0.3535534,-0.3181896,-0.2705981,-0.2126075,-0.1464466,-0.07465783,-0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0],[-1,-0.9807853,-0.9238795,-0.8314696,-0.7071068,-0.5555702,-0.3826834,-0.1950903,0,0.1950903,0.3826834,0.5555702,0.7071068,0.8314696,0.9238795,0.9807853,1,-1,-0.9807853,-0.9238795,-0.8314696,-0.7071068,-0.5555702,-0.3826834,-0.1950903,0,0.1950903,0.3826834,0.5555702,0.7071068,0.8314696,0.9238795,0.9807853,1,-1,-0.9807853,-0.9238795,-0.8314696,-0.7071068,-0.5555702,-0.3826834,-0.1950903,0,0.1950903,0.3826834,0.5555702,0.7071068,0.8314696,0.9238795,0.9807853,1,-1,-0.9807853,-0.9238795,-0.8314696,-0.7071068,-0.5555702,-0.3826834,-0.1950903,0,0.1950903,0.3826834,0.5555702,0.7071068,0.8314696,0.9238795,0.9807853,1,-1,-0.9807853,-0.9238795,-0.8314696,-0.7071068,-0.5555702,-0.3826834,-0.1950903,0,0.1950903,0.3826834,0.5555702,0.7071068,0.8314696,0.9238795,0.9807853,1,-1,-0.9807853,-0.9238795,-0.8314696,-0.7071068,-0.5555702,-0.3826834,-0.1950903,0,0.1950903,0.3826834,0.5555702,0.7071068,0.8314696,0.9238795,0.9807853,1,-1,-0.9807853,-0.9238795,-0.8314696,-0.7071068,-0.5555702,-0.3826834,-0.1950903,0,0.1950903,0.3826834,0.5555702,0.7071068,0.8314696,0.9238795,0.9807853,1,-1,-0.9807853,-0.9238795,-0.8314696,-0.7071068,-0.5555702,-0.3826834,-0.1950903,0,0.1950903,0.3826834,0.5555702,0.7071068,0.8314696,0.9238795,0.9807853,1,-1,-0.9807853,-0.9238795,-0.8314696,-0.7071068,-0.5555702,-0.3826834,-0.1950903,0,0.1950903,0.3826834,0.5555702,0.7071068,0.8314696,0.9238795,0.9807853,1,-1,-0.9807853,-0.9238795,-0.8314696,-0.7071068,-0.5555702,-0.3826834,-0.1950903,0,0.1950903,0.3826834,0.5555702,0.7071068,0.8314696,0.9238795,0.9807853,1,-1,-0.9807853,-0.9238795,-0.8314696,-0.7071068,-0.5555702,-0.3826834,-0.1950903,0,0.1950903,0.3826834,0.5555702,0.7071068,0.8314696,0.9238795,0.9807853,1,-1,-0.9807853,-0.9238795,-0.8314696,-0.7071068,-0.5555702,-0.3826834,-0.1950903,0,0.1950903,0.3826834,0.5555702,0.7071068,0.8314696,0.9238795,0.9807853,1,-1,-0.9807853,-0.9238795,-0.8314696,-0.7071068,-0.5555702,-0.3826834,-0.1950903,0,0.1950903,0.3826834,0.5555702,0.7071068,0.8314696,0.9238795,0.9807853,1,-1,-0.9807853,-0.9238795,-0.8314696,-0.7071068,-0.5555702,-0.3826834,-0.1950903,0,0.1950903,0.3826834,0.5555702,0.7071068,0.8314696,0.9238795,0.9807853,1,-1,-0.9807853,-0.9238795,-0.8314696,-0.7071068,-0.5555702,-0.3826834,-0.1950903,0,0.1950903,0.3826834,0.5555702,0.7071068,0.8314696,0.9238795,0.9807853,1,-1,-0.9807853,-0.9238795,-0.8314696,-0.7071068,-0.5555702,-0.3826834,-0.1950903,0,0.1950903,0.3826834,0.5555702,0.7071068,0.8314696,0.9238795,0.9807853,1,-1,-0.9807853,-0.9238795,-0.8314696,-0.7071068,-0.5555702,-0.3826834,-0.1950903,0,0.1950903,0.3826834,0.5555702,0.7071068,0.8314696,0.9238795,0.9807853,1],[0,0.1950903,0.3826834,0.5555702,0.7071068,0.8314696,0.9238795,0.9807853,1,0.9807853,0.9238795,0.8314696,0.7071068,0.5555702,0.3826834,0.1950903,0,0,0.18024,0.3535534,0.51328,0.6532815,0.7681778,0.8535534,0.9061274,0.9238795,0.9061274,0.8535534,0.7681778,0.6532815,0.51328,0.3535534,0.18024,0,0,0.1379497,0.2705981,0.3928475,0.5,0.5879378,0.6532815,0.6935199,0.7071068,0.6935199,0.6532815,0.5879378,0.5,0.3928475,0.2705981,0.1379497,0,0,0.07465783,0.1464466,0.2126075,0.2705981,0.3181896,0.3535534,0.3753303,0.3826834,0.3753303,0.3535534,0.3181896,0.2705981,0.2126075,0.1464466,0.07465783,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,-0,-0.07465783,-0.1464466,-0.2126075,-0.2705981,-0.3181896,-0.3535534,-0.3753303,-0.3826834,-0.3753303,-0.3535534,-0.3181896,-0.2705981,-0.2126075,-0.1464466,-0.07465783,-0,-0,-0.1379497,-0.2705981,-0.3928475,-0.5,-0.5879378,-0.6532815,-0.6935199,-0.7071068,-0.6935199,-0.6532815,-0.5879378,-0.5,-0.3928475,-0.2705981,-0.1379497,-0,-0,-0.18024,-0.3535534,-0.51328,-0.6532815,-0.7681778,-0.8535534,-0.9061274,-0.9238795,-0.9061274,-0.8535534,-0.7681778,-0.6532815,-0.51328,-0.3535534,-0.18024,-0,-0,-0.1950903,-0.3826834,-0.5555702,-0.7071068,-0.8314696,-0.9238795,-0.9807853,-1,-0.9807853,-0.9238795,-0.8314696,-0.7071068,-0.5555702,-0.3826834,-0.1950903,-0,-0,-0.18024,-0.3535534,-0.51328,-0.6532815,-0.7681778,-0.8535534,-0.9061274,-0.9238795,-0.9061274,-0.8535534,-0.7681778,-0.6532815,-0.51328,-0.3535534,-0.18024,-0,-0,-0.1379497,-0.2705981,-0.3928475,-0.5,-0.5879378,-0.6532815,-0.6935199,-0.7071068,-0.6935199,-0.6532815,-0.5879378,-0.5,-0.3928475,-0.2705981,-0.1379497,-0,-0,-0.07465783,-0.1464466,-0.2126075,-0.2705981,-0.3181896,-0.3535534,-0.3753303,-0.3826834,-0.3753303,-0.3535534,-0.3181896,-0.2705981,-0.2126075,-0.1464466,-0.07465783,-0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0.07465783,0.1464466,0.2126075,0.2705981,0.3181896,0.3535534,0.3753303,0.3826834,0.3753303,0.3535534,0.3181896,0.2705981,0.2126075,0.1464466,0.07465783,0,0,0.1379497,0.2705981,0.3928475,0.5,0.5879378,0.6532815,0.6935199,0.7071068,0.6935199,0.6532815,0.5879378,0.5,0.3928475,0.2705981,0.1379497,0,0,0.18024,0.3535534,0.51328,0.6532815,0.7681778,0.8535534,0.9061274,0.9238795,0.9061274,0.8535534,0.7681778,0.6532815,0.51328,0.3535534,0.18024,0,0,0.1950903,0.3826834,0.5555702,0.7071068,0.8314696,0.9238795,0.9807853,1,0.9807853,0.9238795,0.8314696,0.7071068,0.5555702,0.3826834,0.1950903,0]],"it":[[0,1,2,3,4,5,6,7,8,9,10,11,12,13,14,15,17,18,19,20,21,22,23,24,25,26,27,28,29,30,31,32,34,35,36,37,38,39,40,41,42,43,44,45,46,47,48,49,51,52,53,54,55,56,57,58,59,60,61,62,63,64,65,66,68,69,70,71,72,73,74,75,76,77,78,79,80,81,82,83,85,86,87,88,89,90,91,92,93,94,95,96,97,98,99,100,102,103,104,105,106,107,108,109,110,111,112,113,114,115,116,117,119,120,121,122,123,124,125,126,127,128,129,130,131,132,133,134,136,137,138,139,140,141,142,143,144,145,146,147,148,149,150,151,153,154,155,156,157,158,159,160,161,162,163,164,165,166,167,168,170,171,172,173,174,175,176,177,178,179,180,181,182,183,184,185,187,188,189,190,191,192,193,194,195,196,197,198,199,200,201,202,204,205,206,207,208,209,210,211,212,213,214,215,216,217,218,219,221,222,223,224,225,226,227,228,229,230,231,232,233,234,235,236,238,239,240,241,242,243,244,245,246,247,248,249,250,251,252,253,255,256,257,258,259,260,261,262,263,264,265,266,267,268,269,270,0,1,2,3,4,5,6,7,8,9,10,11,12,13,14,15,17,18,19,20,21,22,23,24,25,26,27,28,29,30,31,32,34,35,36,37,38,39,40,41,42,43,44,45,46,47,48,49,51,52,53,54,55,56,57,58,59,60,61,62,63,64,65,66,68,69,70,71,72,73,74,75,76,77,78,79,80,81,82,83,85,86,87,88,89,90,91,92,93,94,95,96,97,98,99,100,102,103,104,105,106,107,108,109,110,111,112,113,114,115,116,117,119,120,121,122,123,124,125,126,127,128,129,130,131,132,133,134,136,137,138,139,140,141,142,143,144,145,146,147,148,149,150,151,153,154,155,156,157,158,159,160,161,162,163,164,165,166,167,168,170,171,172,173,174,175,176,177,178,179,180,181,182,183,184,185,187,188,189,190,191,192,193,194,195,196,197,198,199,200,201,202,204,205,206,207,208,209,210,211,212,213,214,215,216,217,218,219,221,222,223,224,225,226,227,228,229,230,231,232,233,234,235,236,238,239,240,241,242,243,244,245,246,247,248,249,250,251,252,253,255,256,257,258,259,260,261,262,263,264,265,266,267,268,269,270],[17,18,19,20,21,22,23,24,25,26,27,28,29,30,31,32,34,35,36,37,38,39,40,41,42,43,44,45,46,47,48,49,51,52,53,54,55,56,57,58,59,60,61,62,63,64,65,66,68,69,70,71,72,73,74,75,76,77,78,79,80,81,82,83,85,86,87,88,89,90,91,92,93,94,95,96,97,98,99,100,102,103,104,105,106,107,108,109,110,111,112,113,114,115,116,117,119,120,121,122,123,124,125,126,127,128,129,130,131,132,133,134,136,137,138,139,140,141,142,143,144,145,146,147,148,149,150,151,153,154,155,156,157,158,159,160,161,162,163,164,165,166,167,168,170,171,172,173,174,175,176,177,178,179,180,181,182,183,184,185,187,188,189,190,191,192,193,194,195,196,197,198,199,200,201,202,204,205,206,207,208,209,210,211,212,213,214,215,216,217,218,219,221,222,223,224,225,226,227,228,229,230,231,232,233,234,235,236,238,239,240,241,242,243,244,245,246,247,248,249,250,251,252,253,255,256,257,258,259,260,261,262,263,264,265,266,267,268,269,270,272,273,274,275,276,277,278,279,280,281,282,283,284,285,286,287,18,19,20,21,22,23,24,25,26,27,28,29,30,31,32,33,35,36,37,38,39,40,41,42,43,44,45,46,47,48,49,50,52,53,54,55,56,57,58,59,60,61,62,63,64,65,66,67,69,70,71,72,73,74,75,76,77,78,79,80,81,82,83,84,86,87,88,89,90,91,92,93,94,95,96,97,98,99,100,101,103,104,105,106,107,108,109,110,111,112,113,114,115,116,117,118,120,121,122,123,124,125,126,127,128,129,130,131,132,133,134,135,137,138,139,140,141,142,143,144,145,146,147,148,149,150,151,152,154,155,156,157,158,159,160,161,162,163,164,165,166,167,168,169,171,172,173,174,175,176,177,178,179,180,181,182,183,184,185,186,188,189,190,191,192,193,194,195,196,197,198,199,200,201,202,203,205,206,207,208,209,210,211,212,213,214,215,216,217,218,219,220,222,223,224,225,226,227,228,229,230,231,232,233,234,235,236,237,239,240,241,242,243,244,245,246,247,248,249,250,251,252,253,254,256,257,258,259,260,261,262,263,264,265,266,267,268,269,270,271,273,274,275,276,277,278,279,280,281,282,283,284,285,286,287,288],[18,19,20,21,22,23,24,25,26,27,28,29,30,31,32,33,35,36,37,38,39,40,41,42,43,44,45,46,47,48,49,50,52,53,54,55,56,57,58,59,60,61,62,63,64,65,66,67,69,70,71,72,73,74,75,76,77,78,79,80,81,82,83,84,86,87,88,89,90,91,92,93,94,95,96,97,98,99,100,101,103,104,105,106,107,108,109,110,111,112,113,114,115,116,117,118,120,121,122,123,124,125,126,127,128,129,130,131,132,133,134,135,137,138,139,140,141,142,143,144,145,146,147,148,149,150,151,152,154,155,156,157,158,159,160,161,162,163,164,165,166,167,168,169,171,172,173,174,175,176,177,178,179,180,181,182,183,184,185,186,188,189,190,191,192,193,194,195,196,197,198,199,200,201,202,203,205,206,207,208,209,210,211,212,213,214,215,216,217,218,219,220,222,223,224,225,226,227,228,229,230,231,232,233,234,235,236,237,239,240,241,242,243,244,245,246,247,248,249,250,251,252,253,254,256,257,258,259,260,261,262,263,264,265,266,267,268,269,270,271,273,274,275,276,277,278,279,280,281,282,283,284,285,286,287,288,1,2,3,4,5,6,7,8,9,10,11,12,13,14,15,16,18,19,20,21,22,23,24,25,26,27,28,29,30,31,32,33,35,36,37,38,39,40,41,42,43,44,45,46,47,48,49,50,52,53,54,55,56,57,58,59,60,61,62,63,64,65,66,67,69,70,71,72,73,74,75,76,77,78,79,80,81,82,83,84,86,87,88,89,90,91,92,93,94,95,96,97,98,99,100,101,103,104,105,106,107,108,109,110,111,112,113,114,115,116,117,118,120,121,122,123,124,125,126,127,128,129,130,131,132,133,134,135,137,138,139,140,141,142,143,144,145,146,147,148,149,150,151,152,154,155,156,157,158,159,160,161,162,163,164,165,166,167,168,169,171,172,173,174,175,176,177,178,179,180,181,182,183,184,185,186,188,189,190,191,192,193,194,195,196,197,198,199,200,201,202,203,205,206,207,208,209,210,211,212,213,214,215,216,217,218,219,220,222,223,224,225,226,227,228,229,230,231,232,233,234,235,236,237,239,240,241,242,243,244,245,246,247,248,249,250,251,252,253,254,256,257,258,259,260,261,262,263,264,265,266,267,268,269,270,271]],"primitivetype":"triangle","material":null,"normals":null,"texcoords":[[0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0.0625,0.0625,0.0625,0.0625,0.0625,0.0625,0.0625,0.0625,0.0625,0.0625,0.0625,0.0625,0.0625,0.0625,0.0625,0.0625,0.0625,0.125,0.125,0.125,0.125,0.125,0.125,0.125,0.125,0.125,0.125,0.125,0.125,0.125,0.125,0.125,0.125,0.125,0.1875,0.1875,0.1875,0.1875,0.1875,0.1875,0.1875,0.1875,0.1875,0.1875,0.1875,0.1875,0.1875,0.1875,0.1875,0.1875,0.1875,0.25,0.25,0.25,0.25,0.25,0.25,0.25,0.25,0.25,0.25,0.25,0.25,0.25,0.25,0.25,0.25,0.25,0.3125,0.3125,0.3125,0.3125,0.3125,0.3125,0.3125,0.3125,0.3125,0.3125,0.3125,0.3125,0.3125,0.3125,0.3125,0.3125,0.3125,0.375,0.375,0.375,0.375,0.375,0.375,0.375,0.375,0.375,0.375,0.375,0.375,0.375,0.375,0.375,0.375,0.375,0.4375,0.4375,0.4375,0.4375,0.4375,0.4375,0.4375,0.4375,0.4375,0.4375,0.4375,0.4375,0.4375,0.4375,0.4375,0.4375,0.4375,0.5,0.5,0.5,0.5,0.5,0.5,0.5,0.5,0.5,0.5,0.5,0.5,0.5,0.5,0.5,0.5,0.5,0.5625,0.5625,0.5625,0.5625,0.5625,0.5625,0.5625,0.5625,0.5625,0.5625,0.5625,0.5625,0.5625,0.5625,0.5625,0.5625,0.5625,0.625,0.625,0.625,0.625,0.625,0.625,0.625,0.625,0.625,0.625,0.625,0.625,0.625,0.625,0.625,0.625,0.625,0.6875,0.6875,0.6875,0.6875,0.6875,0.6875,0.6875,0.6875,0.6875,0.6875,0.6875,0.6875,0.6875,0.6875,0.6875,0.6875,0.6875,0.75,0.75,0.75,0.75,0.75,0.75,0.75,0.75,0.75,0.75,0.75,0.75,0.75,0.75,0.75,0.75,0.75,0.8125,0.8125,0.8125,0.8125,0.8125,0.8125,0.8125,0.8125,0.8125,0.8125,0.8125,0.8125,0.8125,0.8125,0.8125,0.8125,0.8125,0.875,0.875,0.875,0.875,0.875,0.875,0.875,0.875,0.875,0.875,0.875,0.875,0.875,0.875,0.875,0.875,0.875,0.9375,0.9375,0.9375,0.9375,0.9375,0.9375,0.9375,0.9375,0.9375,0.9375,0.9375,0.9375,0.9375,0.9375,0.9375,0.9375,0.9375,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1],[0,0.0625,0.125,0.1875,0.25,0.3125,0.375,0.4375,0.5,0.5625,0.625,0.6875,0.75,0.8125,0.875,0.9375,1,0,0.0625,0.125,0.1875,0.25,0.3125,0.375,0.4375,0.5,0.5625,0.625,0.6875,0.75,0.8125,0.875,0.9375,1,0,0.0625,0.125,0.1875,0.25,0.3125,0.375,0.4375,0.5,0.5625,0.625,0.6875,0.75,0.8125,0.875,0.9375,1,0,0.0625,0.125,0.1875,0.25,0.3125,0.375,0.4375,0.5,0.5625,0.625,0.6875,0.75,0.8125,0.875,0.9375,1,0,0.0625,0.125,0.1875,0.25,0.3125,0.375,0.4375,0.5,0.5625,0.625,0.6875,0.75,0.8125,0.875,0.9375,1,0,0.0625,0.125,0.1875,0.25,0.3125,0.375,0.4375,0.5,0.5625,0.625,0.6875,0.75,0.8125,0.875,0.9375,1,0,0.0625,0.125,0.1875,0.25,0.3125,0.375,0.4375,0.5,0.5625,0.625,0.6875,0.75,0.8125,0.875,0.9375,1,0,0.0625,0.125,0.1875,0.25,0.3125,0.375,0.4375,0.5,0.5625,0.625,0.6875,0.75,0.8125,0.875,0.9375,1,0,0.0625,0.125,0.1875,0.25,0.3125,0.375,0.4375,0.5,0.5625,0.625,0.6875,0.75,0.8125,0.875,0.9375,1,0,0.0625,0.125,0.1875,0.25,0.3125,0.375,0.4375,0.5,0.5625,0.625,0.6875,0.75,0.8125,0.875,0.9375,1,0,0.0625,0.125,0.1875,0.25,0.3125,0.375,0.4375,0.5,0.5625,0.625,0.6875,0.75,0.8125,0.875,0.9375,1,0,0.0625,0.125,0.1875,0.25,0.3125,0.375,0.4375,0.5,0.5625,0.625,0.6875,0.75,0.8125,0.875,0.9375,1,0,0.0625,0.125,0.1875,0.25,0.3125,0.375,0.4375,0.5,0.5625,0.625,0.6875,0.75,0.8125,0.875,0.9375,1,0,0.0625,0.125,0.1875,0.25,0.3125,0.375,0.4375,0.5,0.5625,0.625,0.6875,0.75,0.8125,0.875,0.9375,1,0,0.0625,0.125,0.1875,0.25,0.3125,0.375,0.4375,0.5,0.5625,0.625,0.6875,0.75,0.8125,0.875,0.9375,1,0,0.0625,0.125,0.1875,0.25,0.3125,0.375,0.4375,0.5,0.5625,0.625,0.6875,0.75,0.8125,0.875,0.9375,1,0,0.0625,0.125,0.1875,0.25,0.3125,0.375,0.4375,0.5,0.5625,0.625,0.6875,0.75,0.8125,0.875,0.9375,1]]}});
testglrgl.prefix = "testgl";
</script>
<p id="testgldebug">
You must enable Javascript to view this page properly.</p>
<script>testglrgl.start();</script>


Note from the 3D graph above (you can interact with the plot by cicking and dragging its surface around to change the viewing angle) how this view more clearly highlights the pattern existent across prestige and women relative to income. Also, this interactive view allows us to more clearly see those three or four outlier points as well as how well our last linear model fit the data.


{% include advertisements.html %}


At this stage we could try a few different transformations on both the predictors and the response variable to see how this would improve the model fit. For now, let’s apply a logarithmic transformation with the *log* function on the income variable (the log function here transforms using the natural log. If base 10 is desired log10 is the function to be used). Also, we could try to square both predictors. Let’s apply these suggested transformations directly into the model function and see what happens with both the model fit and the model accuracy.


```r
# fit a model excluding the variable education,  log the income variable.
mod3 = lm(log(income) ~ prestige.c + I(prestige.c^2) + women.c + I(women.c^2) , data=newdata)
summary(mod3)
```

```
## 
## Call:
## lm(formula = log(income) ~ prestige.c + I(prestige.c^2) + women.c + 
##     I(women.c^2), data = newdata)
## 
## Residuals:
##      Min       1Q   Median       3Q      Max 
## -1.01614 -0.10973  0.00966  0.14479  0.80844 
## 
## Coefficients:
##                   Estimate Std. Error t value Pr(>|t|)    
## (Intercept)      8.809e+00  5.944e-02 148.188  < 2e-16 ***
## prestige.c       2.518e-02  1.787e-03  14.096  < 2e-16 ***
## I(prestige.c^2) -2.605e-04  9.396e-05  -2.773  0.00666 ** 
## women.c         -6.306e-03  1.476e-03  -4.271 4.53e-05 ***
## I(women.c^2)    -7.194e-05  4.014e-05  -1.792  0.07620 .  
## ---
## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
## 
## Residual standard error: 0.293 on 97 degrees of freedom
## Multiple R-squared:  0.7643,	Adjusted R-squared:  0.7546 
## F-statistic: 78.64 on 4 and 97 DF,  p-value: < 2.2e-16
```

```r
# Plot model residuals.
plot(mod3, pch=16, which=1)
```

![](https://github.com/FelipeRego/feliperego.github.io/raw/master/images/MultipleLinearRegression_files/figure-html/unnamed-chunk-9-1.png)<!-- -->




```r
newdat2 <- expand.grid(prestige.c=seq(-35,45,by=5),women.c=seq(-25,70,by=5))
newdat2$pp <- predict(mod3,newdata=newdat2)
with(newdata,plot3d(prestige.c,women.c,log(income), col="blue", size=1, type="s", main="3D Quadratic Model Fit with Log of Income"))
with(newdat2,surface3d(unique(prestige.c),unique(women.c),pp,
                      alpha=0.3,front="line", back="line"))
```

<div id="testgl2div" class="rglWebGL"></div>
<script type="text/javascript">
var testgl2div = document.getElementById("testgl2div"),
testgl2rgl = new rglwidgetClass();
testgl2div.width = 673;
testgl2div.height = 481;
testgl2rgl.initialize(testgl2div,
{"material":{"color":"#000000","alpha":1,"lit":true,"ambient":"#000000","specular":"#FFFFFF","emission":"#000000","shininess":50,"smooth":true,"front":"filled","back":"filled","size":3,"lwd":1,"fog":false,"point_antialias":false,"line_antialias":false,"texture":null,"textype":"rgb","texmipmap":false,"texminfilter":"linear","texmagfilter":"linear","texenvmap":false,"depth_mask":true,"depth_test":"less"},"rootSubscene":1,"objects":{"28":{"id":28,"type":"spheres","material":{},"vertices":[[21.96667,-17.81902,9.421493],[22.26667,-24.95902,10.16119],[16.56667,-13.27902,9.134646],[9.966666,-19.86902,9.089867],[26.66667,-17.29902,9.036345],[30.76667,-23.84902,9.308374],[25.76667,-3.32902,9.018938],[31.26667,-26.28902,9.558388],[26.26667,-27.94902,9.339349],[21.96667,-28.03902,9.307739],[15.16667,-27.06902,8.683046],[13.16667,-21.14902,8.862059],[6.966667,-13.64902,9.038959],[15.36667,28.33098,8.993303],[28.06667,19.30098,8.909911],[8.266666,25.79098,8.754003],[35.46667,-23.84902,9.865941],[11.26667,48.12098,8.718009],[11.46667,5.91098,9.168789],[25.96667,-24.83902,8.452334],[37.76667,-9.38902,9.431883],[12.76667,54.80098,8.639057],[19.26667,17.82098,8.991438],[40.36666,-18.41902,10.13888],[19.86667,-24.65902,9.585896],[21.56667,-22.06902,9.769842],[17.86667,67.14098,8.436851],[-11.93333,47.16098,8.156223],[25.26667,53.68098,8.535426],[22.46667,-4.26902,9.252633],[20.66667,47.06098,8.552561],[10.36667,-7.949019,8.73182],[10.76667,-17.82902,8.930891],[7.266667,-20.84902,9.012621],[-0.8333333,68.53098,8.303009],[-4.933333,66.99098,8.054523],[2.566667,39.26098,8.377471],[-4.533333,62.78098,7.803027],[0.8666667,46.94098,8.373322],[-15.93333,-17.60902,8.468213],[-14.13333,54.21098,8.011686],[-8.133333,63.88098,7.972811],[-10.73333,-21.35902,8.614501],[-9.633333,23.29098,8.226574],[-8.733334,67.16098,8.058643],[-17.43333,18.08098,8.464004],[4.266667,27.12098,8.527539],[-11.13333,10.19098,8.741776],[-11.23333,34.25098,8.312626],[-5.333333,-11.93902,8.920256],[-6.633333,-25.81902,9.080232],[-20.33333,38.84098,7.860956],[-32.03333,-21.97902,6.822197],[-23.53333,-25.28902,7.770645],[0.4666667,-15.88902,9.003439],[0.2666667,-4.53902,8.852522],[4.266667,-5.09902,8.981682],[-3.333333,-28.97902,9.093245],[4.766667,-27.32902,9.092794],[-17.13333,23.02098,8.044306],[-26.63333,-13.46902,8.276395],[8.066667,-22.96902,8.970686],[-20.93333,67.55098,6.415097],[-26.03333,40.33098,8.006368],[-29.53333,4.590981,8.152486],[-26.73333,1.10098,8.183677],[-2.733333,-25.37902,8.200562],[-25.33333,-1.22902,7.41216],[-11.53333,-28.97902,8.833463],[-7.933333,4.320981,8.342602],[-21.63333,-11.71902,8.54364],[-12.03333,-11.71902,8.54364],[-23.63333,43.26098,7.544332],[-13.53333,2.38098,8.399085],[-18.03333,10.50098,8.156223],[-4.333333,-27.47902,8.992558],[-2.633333,-24.69902,8.807771],[-10.93333,-26.67902,8.789508],[-5.033333,-23.80902,8.776012],[-10.93333,-15.35902,8.667508],[-3.133333,-23.19902,8.790726],[3.966667,45.56098,8.279444],[-9.633333,-26.05902,8.603188],[-18.63333,61.69098,7.954021],[-8.733334,-28.16902,8.664751],[3.466667,-28.19902,8.951052],[-19.53333,-28.97902,8.454467],[-5.933333,-27.63902,9.025937],[3.366667,-27.98902,8.874448],[4.266667,-28.32902,9.091557],[-7.933333,-28.41902,8.575274],[-10.63333,-28.45902,8.692658],[-16.93333,-26.51902,8.422663],[-3.933333,-28.36902,8.843327],[-20.33333,-27.88902,8.271293],[19.26667,-28.39902,9.549096],[2.066667,-28.97902,9.087607],[-10.93333,-19.50902,8.623713],[-21.73333,-25.38902,8.348537],[-20.73333,-28.97902,8.466531],[-4.633333,-15.39902,8.773694],[-11.63333,41.89098,8.1934]],"colors":[[0,0,1,1]],"radii":[[1.169203]],"centers":[[21.96667,-17.81902,9.421493],[22.26667,-24.95902,10.16119],[16.56667,-13.27902,9.134646],[9.966666,-19.86902,9.089867],[26.66667,-17.29902,9.036345],[30.76667,-23.84902,9.308374],[25.76667,-3.32902,9.018938],[31.26667,-26.28902,9.558388],[26.26667,-27.94902,9.339349],[21.96667,-28.03902,9.307739],[15.16667,-27.06902,8.683046],[13.16667,-21.14902,8.862059],[6.966667,-13.64902,9.038959],[15.36667,28.33098,8.993303],[28.06667,19.30098,8.909911],[8.266666,25.79098,8.754003],[35.46667,-23.84902,9.865941],[11.26667,48.12098,8.718009],[11.46667,5.91098,9.168789],[25.96667,-24.83902,8.452334],[37.76667,-9.38902,9.431883],[12.76667,54.80098,8.639057],[19.26667,17.82098,8.991438],[40.36666,-18.41902,10.13888],[19.86667,-24.65902,9.585896],[21.56667,-22.06902,9.769842],[17.86667,67.14098,8.436851],[-11.93333,47.16098,8.156223],[25.26667,53.68098,8.535426],[22.46667,-4.26902,9.252633],[20.66667,47.06098,8.552561],[10.36667,-7.949019,8.73182],[10.76667,-17.82902,8.930891],[7.266667,-20.84902,9.012621],[-0.8333333,68.53098,8.303009],[-4.933333,66.99098,8.054523],[2.566667,39.26098,8.377471],[-4.533333,62.78098,7.803027],[0.8666667,46.94098,8.373322],[-15.93333,-17.60902,8.468213],[-14.13333,54.21098,8.011686],[-8.133333,63.88098,7.972811],[-10.73333,-21.35902,8.614501],[-9.633333,23.29098,8.226574],[-8.733334,67.16098,8.058643],[-17.43333,18.08098,8.464004],[4.266667,27.12098,8.527539],[-11.13333,10.19098,8.741776],[-11.23333,34.25098,8.312626],[-5.333333,-11.93902,8.920256],[-6.633333,-25.81902,9.080232],[-20.33333,38.84098,7.860956],[-32.03333,-21.97902,6.822197],[-23.53333,-25.28902,7.770645],[0.4666667,-15.88902,9.003439],[0.2666667,-4.53902,8.852522],[4.266667,-5.09902,8.981682],[-3.333333,-28.97902,9.093245],[4.766667,-27.32902,9.092794],[-17.13333,23.02098,8.044306],[-26.63333,-13.46902,8.276395],[8.066667,-22.96902,8.970686],[-20.93333,67.55098,6.415097],[-26.03333,40.33098,8.006368],[-29.53333,4.590981,8.152486],[-26.73333,1.10098,8.183677],[-2.733333,-25.37902,8.200562],[-25.33333,-1.22902,7.41216],[-11.53333,-28.97902,8.833463],[-7.933333,4.320981,8.342602],[-21.63333,-11.71902,8.54364],[-12.03333,-11.71902,8.54364],[-23.63333,43.26098,7.544332],[-13.53333,2.38098,8.399085],[-18.03333,10.50098,8.156223],[-4.333333,-27.47902,8.992558],[-2.633333,-24.69902,8.807771],[-10.93333,-26.67902,8.789508],[-5.033333,-23.80902,8.776012],[-10.93333,-15.35902,8.667508],[-3.133333,-23.19902,8.790726],[3.966667,45.56098,8.279444],[-9.633333,-26.05902,8.603188],[-18.63333,61.69098,7.954021],[-8.733334,-28.16902,8.664751],[3.466667,-28.19902,8.951052],[-19.53333,-28.97902,8.454467],[-5.933333,-27.63902,9.025937],[3.366667,-27.98902,8.874448],[4.266667,-28.32902,9.091557],[-7.933333,-28.41902,8.575274],[-10.63333,-28.45902,8.692658],[-16.93333,-26.51902,8.422663],[-3.933333,-28.36902,8.843327],[-20.33333,-27.88902,8.271293],[19.26667,-28.39902,9.549096],[2.066667,-28.97902,9.087607],[-10.93333,-19.50902,8.623713],[-21.73333,-25.38902,8.348537],[-20.73333,-28.97902,8.466531],[-4.633333,-15.39902,8.773694],[-11.63333,41.89098,8.1934]],"ignoreExtent":false,"flags":3},"30":{"id":30,"type":"text","material":{"lit":false},"vertices":[[4.166666,87.23503,10.87975]],"colors":[[0,0,0,1]],"texts":[["3D Quadratic Model Fit with Log of Income"]],"cex":[[1]],"adj":[[0.5,0.5]],"centers":[[4.166666,87.23503,10.87975]],"family":[["sans"]],"font":[[1]],"ignoreExtent":true,"flags":40},"31":{"id":31,"type":"text","material":{"lit":false},"vertices":[[4.166666,-47.68306,5.696534]],"colors":[[0,0,0,1]],"texts":[["prestige.c"]],"cex":[[1]],"adj":[[0.5,0.5]],"centers":[[4.166666,-47.68306,5.696534]],"family":[["sans"]],"font":[[1]],"ignoreExtent":true,"flags":40},"32":{"id":32,"type":"text","material":{"lit":false},"vertices":[[-45.92086,19.77598,5.696534]],"colors":[[0,0,0,1]],"texts":[["women.c"]],"cex":[[1]],"adj":[[0.5,0.5]],"centers":[[-45.92086,19.77598,5.696534]],"family":[["sans"]],"font":[[1]],"ignoreExtent":true,"flags":40},"33":{"id":33,"type":"text","material":{"lit":false},"vertices":[[-45.92086,-47.68306,8.288142]],"colors":[[0,0,0,1]],"texts":[["log(income)"]],"cex":[[1]],"adj":[[0.5,0.5]],"centers":[[-45.92086,-47.68306,8.288142]],"family":[["sans"]],"font":[[1]],"ignoreExtent":true,"flags":40},"34":{"id":34,"type":"surface","material":{"alpha":0.2980392,"front":"lines","back":"lines"},"vertices":[[-35,-25,7.720649],[-30,-25,7.931245],[-25,-25,8.128816],[-20,-25,8.313358],[-15,-25,8.484875],[-10,-25,8.643363],[-5,-25,8.788825],[0,-25,8.921261],[5,-25,9.040668],[10,-25,9.14705],[15,-25,9.240404],[20,-25,9.320731],[25,-25,9.388032],[30,-25,9.442305],[35,-25,9.483551],[40,-25,9.51177],[45,-25,9.526963],[-35,-20,7.705303],[-30,-20,7.9159],[-25,-20,8.11347],[-20,-20,8.298013],[-15,-20,8.469529],[-10,-20,8.628018],[-5,-20,8.77348],[0,-20,8.905915],[5,-20,9.025324],[10,-20,9.131704],[15,-20,9.225059],[20,-20,9.305387],[25,-20,9.372686],[30,-20,9.42696],[35,-20,9.468206],[40,-20,9.496426],[45,-20,9.511618],[-35,-15,7.686361],[-30,-15,7.896958],[-25,-15,8.094528],[-20,-15,8.279071],[-15,-15,8.450587],[-10,-15,8.609076],[-5,-15,8.754539],[0,-15,8.886973],[5,-15,9.006381],[10,-15,9.112762],[15,-15,9.206117],[20,-15,9.286444],[25,-15,9.353745],[30,-15,9.408017],[35,-15,9.449264],[40,-15,9.477484],[45,-15,9.492676],[-35,-10,7.663822],[-30,-10,7.874419],[-25,-10,8.071989],[-20,-10,8.256532],[-15,-10,8.428048],[-10,-10,8.586537],[-5,-10,8.731999],[0,-10,8.864434],[5,-10,8.983842],[10,-10,9.090223],[15,-10,9.183578],[20,-10,9.263905],[25,-10,9.331205],[30,-10,9.385479],[35,-10,9.426724],[40,-10,9.454945],[45,-10,9.470137],[-35,-5,7.637686],[-30,-5,7.848283],[-25,-5,8.045853],[-20,-5,8.230396],[-15,-5,8.401912],[-10,-5,8.560401],[-5,-5,8.705863],[0,-5,8.838298],[5,-5,8.957706],[10,-5,9.064088],[15,-5,9.157441],[20,-5,9.237769],[25,-5,9.305069],[30,-5,9.359343],[35,-5,9.400589],[40,-5,9.428808],[45,-5,9.444],[-35,0,7.607953],[-30,0,7.81855],[-25,0,8.01612],[-20,0,8.200663],[-15,0,8.372179],[-10,0,8.530668],[-5,0,8.67613],[0,0,8.808565],[5,0,8.927973],[10,0,9.034354],[15,0,9.127708],[20,0,9.208035],[25,0,9.275336],[30,0,9.32961],[35,0,9.370855],[40,0,9.399076],[45,0,9.414268],[-35,5,7.574623],[-30,5,7.78522],[-25,5,7.98279],[-20,5,8.167333],[-15,5,8.338849],[-10,5,8.497338],[-5,5,8.6428],[0,5,8.775235],[5,5,8.894643],[10,5,9.001024],[15,5,9.094378],[20,5,9.174706],[25,5,9.242006],[30,5,9.29628],[35,5,9.337526],[40,5,9.365746],[45,5,9.380938],[-35,10,7.537696],[-30,10,7.748293],[-25,10,7.945863],[-20,10,8.130406],[-15,10,8.301922],[-10,10,8.460411],[-5,10,8.605873],[0,10,8.738308],[5,10,8.857717],[10,10,8.964098],[15,10,9.057452],[20,10,9.137779],[25,10,9.205079],[30,10,9.259353],[35,10,9.300599],[40,10,9.328818],[45,10,9.34401],[-35,15,7.497172],[-30,15,7.707769],[-25,15,7.905339],[-20,15,8.089882],[-15,15,8.261398],[-10,15,8.419888],[-5,15,8.56535],[0,15,8.697784],[5,15,8.817192],[10,15,8.923573],[15,15,9.016928],[20,15,9.097255],[25,15,9.164556],[30,15,9.218829],[35,15,9.260076],[40,15,9.288295],[45,15,9.303487],[-35,20,7.453052],[-30,20,7.663649],[-25,20,7.861218],[-20,20,8.045761],[-15,20,8.217278],[-10,20,8.375767],[-5,20,8.521229],[0,20,8.653664],[5,20,8.773071],[10,20,8.879453],[15,20,8.972807],[20,20,9.053134],[25,20,9.120435],[30,20,9.174708],[35,20,9.215955],[40,20,9.244174],[45,20,9.259366],[-35,25,7.405334],[-30,25,7.615931],[-25,25,7.813501],[-20,25,7.998044],[-15,25,8.169559],[-10,25,8.328049],[-5,25,8.473511],[0,25,8.605946],[5,25,8.725354],[10,25,8.831736],[15,25,8.92509],[20,25,9.005417],[25,25,9.072717],[30,25,9.12699],[35,25,9.168237],[40,25,9.196456],[45,25,9.211648],[-35,30,7.354019],[-30,30,7.564616],[-25,30,7.762186],[-20,30,7.946729],[-15,30,8.118245],[-10,30,8.276734],[-5,30,8.422196],[0,30,8.554631],[5,30,8.674039],[10,30,8.78042],[15,30,8.873775],[20,30,8.954102],[25,30,9.021402],[30,30,9.075676],[35,30,9.116922],[40,30,9.145142],[45,30,9.160334],[-35,35,7.299108],[-30,35,7.509705],[-25,35,7.707274],[-20,35,7.891817],[-15,35,8.063334],[-10,35,8.221823],[-5,35,8.367285],[0,35,8.49972],[5,35,8.619127],[10,35,8.725509],[15,35,8.818863],[20,35,8.89919],[25,35,8.966491],[30,35,9.020764],[35,35,9.06201],[40,35,9.09023],[45,35,9.105422],[-35,40,7.240599],[-30,40,7.451196],[-25,40,7.648766],[-20,40,7.833309],[-15,40,8.004825],[-10,40,8.163314],[-5,40,8.308776],[0,40,8.441211],[5,40,8.560619],[10,40,8.667001],[15,40,8.760354],[20,40,8.840682],[25,40,8.907982],[30,40,8.962255],[35,40,9.003502],[40,40,9.031721],[45,40,9.046913],[-35,45,7.178493],[-30,45,7.38909],[-25,45,7.58666],[-20,45,7.771203],[-15,45,7.942719],[-10,45,8.101209],[-5,45,8.246671],[0,45,8.379106],[5,45,8.498513],[10,45,8.604895],[15,45,8.698249],[20,45,8.778576],[25,45,8.845877],[30,45,8.90015],[35,45,8.941396],[40,45,8.969616],[45,45,8.984808],[-35,50,7.112791],[-30,50,7.323388],[-25,50,7.520958],[-20,50,7.705501],[-15,50,7.877017],[-10,50,8.035506],[-5,50,8.180968],[0,50,8.313403],[5,50,8.432811],[10,50,8.539192],[15,50,8.632546],[20,50,8.712873],[25,50,8.780174],[30,50,8.834447],[35,50,8.875693],[40,50,8.903913],[45,50,8.919106],[-35,55,7.043491],[-30,55,7.254088],[-25,55,7.451658],[-20,55,7.636201],[-15,55,7.807717],[-10,55,7.966207],[-5,55,8.111669],[0,55,8.244103],[5,55,8.363512],[10,55,8.469893],[15,55,8.563247],[20,55,8.643575],[25,55,8.710875],[30,55,8.765148],[35,55,8.806395],[40,55,8.834614],[45,55,8.849806],[-35,60,6.970595],[-30,60,7.181192],[-25,60,7.378762],[-20,60,7.563305],[-15,60,7.734821],[-10,60,7.89331],[-5,60,8.038772],[0,60,8.171207],[5,60,8.290615],[10,60,8.396996],[15,60,8.490351],[20,60,8.570678],[25,60,8.637979],[30,60,8.692251],[35,60,8.733498],[40,60,8.761717],[45,60,8.77691],[-35,65,6.894102],[-30,65,7.104699],[-25,65,7.302269],[-20,65,7.486812],[-15,65,7.658328],[-10,65,7.816817],[-5,65,7.962279],[0,65,8.094714],[5,65,8.214122],[10,65,8.320503],[15,65,8.413857],[20,65,8.494184],[25,65,8.561485],[30,65,8.615758],[35,65,8.657004],[40,65,8.685224],[45,65,8.700417],[-35,70,6.814012],[-30,70,7.024609],[-25,70,7.222178],[-20,70,7.406722],[-15,70,7.578238],[-10,70,7.736726],[-5,70,7.882188],[0,70,8.014624],[5,70,8.134031],[10,70,8.240413],[15,70,8.333767],[20,70,8.414094],[25,70,8.481395],[30,70,8.535668],[35,70,8.576915],[40,70,8.605134],[45,70,8.620326]],"normals":[[-0.04208186,0.003066334,0.9991096],[-0.04078247,0.003066499,0.9991633],[-0.03818317,0.003066839,0.9992661],[-0.03558313,0.003067157,0.9993621],[-0.03298235,0.003067383,0.9994513],[-0.03038086,0.003067541,0.9995338],[-0.02777883,0.003067773,0.9996094],[-0.0251762,0.003068032,0.9996784],[-0.02257301,0.003068223,0.9997405],[-0.01996941,0.00306844,0.9997959],[-0.01736545,0.00306859,0.9998446],[-0.01476108,0.003068671,0.9998864],[-0.01215637,0.003068778,0.9999215],[-0.009551457,0.003068817,0.9999497],[-0.006946352,0.003068836,0.9999712],[-0.004341105,0.003068929,0.9999859],[-0.003038474,0.003068991,0.9999908],[-0.04208181,0.003425688,0.9991083],[-0.04078242,0.003425872,0.9991623],[-0.03818312,0.003426224,0.999265],[-0.03558309,0.003426553,0.9993609],[-0.03298233,0.003426835,0.9994501],[-0.03038085,0.003427071,0.9995325],[-0.02777878,0.00342733,0.9996083],[-0.02517615,0.003427614,0.9996772],[-0.02257299,0.003427851,0.9997394],[-0.01996938,0.003428041,0.9997947],[-0.01736543,0.003428208,0.9998434],[-0.01476106,0.003428351,0.9998852],[-0.01215636,0.003428472,0.9999203],[-0.009551445,0.003428568,0.9999485],[-0.006946368,0.003428618,0.9999701],[-0.004341099,0.003428645,0.9999847],[-0.003038423,0.003428661,0.9999896],[-0.04208169,0.004144392,0.9991056],[-0.04078231,0.004144614,0.9991595],[-0.03818302,0.00414504,0.9992622],[-0.03558299,0.004145438,0.9993582],[-0.03298227,0.004145809,0.9994473],[-0.03038079,0.004146151,0.9995298],[-0.02777868,0.004146465,0.9996055],[-0.02517603,0.004146774,0.9996744],[-0.02257292,0.004147056,0.9997366],[-0.01996935,0.004147262,0.999792],[-0.01736535,0.004147463,0.9998406],[-0.01476102,0.004147661,0.9998825],[-0.01215635,0.004147783,0.9999176],[-0.009551397,0.004147901,0.9999459],[-0.006946374,0.004148014,0.9999673],[-0.004341112,0.00414805,0.999982],[-0.003038366,0.004148046,0.9999868],[-0.04208155,0.004863136,0.9991024],[-0.04078217,0.00486341,0.9991563],[-0.03818291,0.00486391,0.999259],[-0.03558288,0.004864365,0.999355],[-0.03298214,0.004864824,0.9994442],[-0.03038069,0.004865249,0.9995266],[-0.02777859,0.004865617,0.9996023],[-0.02517595,0.004865929,0.9996713],[-0.02257288,0.004866184,0.9997334],[-0.01996929,0.004866454,0.9997888],[-0.01736527,0.004866714,0.9998374],[-0.01476098,0.004866918,0.9998792],[-0.01215633,0.004867088,0.9999143],[-0.009551366,0.004867202,0.9999426],[-0.006946326,0.004867306,0.9999641],[-0.004341097,0.004867426,0.9999788],[-0.003038356,0.004867473,0.9999835],[-0.0420814,0.005581873,0.9990987],[-0.040782,0.005582175,0.9991525],[-0.03818278,0.005582748,0.9992552],[-0.03558274,0.005583284,0.9993512],[-0.03298199,0.005583783,0.9994404],[-0.03038057,0.005584244,0.9995228],[-0.02777848,0.005584666,0.9995986],[-0.02517588,0.005585051,0.9996675],[-0.02257282,0.005585399,0.9997297],[-0.01996919,0.005585708,0.9997851],[-0.01736521,0.00558598,0.9998336],[-0.01476092,0.005586214,0.9998755],[-0.01215628,0.005586409,0.9999105],[-0.009551354,0.005586567,0.9999388],[-0.006946275,0.005586687,0.9999603],[-0.004341058,0.00558677,0.999975],[-0.003038345,0.005586797,0.9999799],[-0.04208123,0.006300578,0.9990943],[-0.04078184,0.006300907,0.9991482],[-0.03818259,0.006301566,0.999251],[-0.03558259,0.006302195,0.999347],[-0.03298187,0.006302733,0.9994361],[-0.03038044,0.006303231,0.9995186],[-0.02777836,0.006303708,0.9995942],[-0.02517575,0.006304166,0.9996632],[-0.02257269,0.006304605,0.9997254],[-0.01996913,0.006304955,0.9997808],[-0.01736513,0.006305237,0.9998294],[-0.01476085,0.0063055,0.9998712],[-0.01215626,0.006305698,0.9999062],[-0.009551314,0.006305853,0.9999346],[-0.006946247,0.006305989,0.9999561],[-0.004341039,0.006306082,0.9999708],[-0.003038332,0.006306112,0.9999756],[-0.04208104,0.007019298,0.9990896],[-0.04078165,0.007019675,0.9991434],[-0.0381824,0.007020374,0.9992461],[-0.03558241,0.007021036,0.9993421],[-0.03298172,0.007021699,0.9994313],[-0.0303803,0.007022302,0.9995138],[-0.02777823,0.007022833,0.9995894],[-0.02517563,0.007023294,0.9996584],[-0.02257258,0.007023683,0.9997206],[-0.01996906,0.007024048,0.999776],[-0.01736505,0.007024389,0.9998245],[-0.01476076,0.007024708,0.9998664],[-0.0121562,0.007025002,0.9999015],[-0.009551291,0.0070252,0.9999298],[-0.006946213,0.007025328,0.9999512],[-0.004340994,0.007025455,0.9999659],[-0.003038318,0.007025512,0.9999707],[-0.04208079,0.007737982,0.9990843],[-0.04078144,0.007738399,0.9991382],[-0.03818223,0.007739195,0.9992408],[-0.03558221,0.007739938,0.9993368],[-0.03298152,0.007740629,0.999426],[-0.03038014,0.007741268,0.9995085],[-0.02777809,0.007741854,0.9995841],[-0.02517552,0.007742388,0.9996532],[-0.02257249,0.007742869,0.9997153],[-0.01996895,0.007743298,0.9997707],[-0.01736496,0.007743674,0.9998193],[-0.01476066,0.007743999,0.9998611],[-0.01215611,0.00774427,0.9998962],[-0.009551264,0.007744489,0.9999245],[-0.006946175,0.007744655,0.9999459],[-0.004340947,0.00774477,0.9999607],[-0.003038302,0.007744807,0.9999655],[-0.04208057,0.008456654,0.9990785],[-0.0407812,0.00845711,0.9991323],[-0.03818198,0.008458003,0.999235],[-0.03558201,0.008458828,0.999331],[-0.03298135,0.008459546,0.9994202],[-0.03037996,0.008460223,0.9995027],[-0.02777793,0.008460863,0.9995784],[-0.02517535,0.008461469,0.9996473],[-0.02257233,0.008462043,0.9997095],[-0.01996883,0.008462536,0.9997648],[-0.01736485,0.008462947,0.9998134],[-0.0147606,0.008463278,0.9998553],[-0.01215606,0.008463526,0.9998903],[-0.009551208,0.008463742,0.9999186],[-0.006946135,0.008463924,0.9999401],[-0.004340922,0.008464049,0.9999548],[-0.003038284,0.008464089,0.9999596],[-0.0420803,0.009175337,0.9990722],[-0.04078095,0.009175832,0.9991261],[-0.03818173,0.009176763,0.9992287],[-0.03558179,0.009177644,0.9993247],[-0.03298115,0.009178487,0.9994139],[-0.03037977,0.009179257,0.9994963],[-0.02777774,0.009179951,0.999572],[-0.02517519,0.009180561,0.9996409],[-0.02257219,0.009181083,0.9997031],[-0.01996871,0.009181569,0.9997585],[-0.01736475,0.009182015,0.9998071],[-0.0147605,0.009182423,0.999849],[-0.01215599,0.009182793,0.999884],[-0.009551148,0.009183077,0.9999123],[-0.006946092,0.009183275,0.9999337],[-0.004340895,0.009183409,0.9999484],[-0.003038264,0.009183453,0.9999533],[-0.04207999,0.009893983,0.9990653],[-0.04078066,0.009894516,0.9991192],[-0.03818149,0.00989552,0.9992219],[-0.03558153,0.009896459,0.9993178],[-0.0329809,0.009897354,0.999407],[-0.03037956,0.009898183,0.9994895],[-0.02777756,0.009898933,0.9995651],[-0.02517504,0.009899617,0.9996341],[-0.02257206,0.009900232,0.9996963],[-0.01996857,0.009900779,0.9997516],[-0.01736463,0.009901259,0.9998002],[-0.01476038,0.009901674,0.999842],[-0.01215588,0.009902023,0.9998771],[-0.009551083,0.009902302,0.9999053],[-0.006946045,0.009902516,0.9999269],[-0.004340864,0.00990266,0.9999416],[-0.003038244,0.009902707,0.9999464],[-0.04207971,0.01061261,0.9990579],[-0.04078036,0.01061318,0.9991118],[-0.0381812,0.01061429,0.9992145],[-0.03558128,0.01061531,0.9993105],[-0.03298066,0.01061623,0.9993997],[-0.03037933,0.01061709,0.999482],[-0.02777735,0.0106179,0.9995578],[-0.02517483,0.01061866,0.9996267],[-0.02257187,0.01061936,0.9996889],[-0.01996842,0.01061997,0.9997442],[-0.0173645,0.01062049,0.9997928],[-0.01476029,0.01062091,0.9998347],[-0.01215581,0.01062124,0.9998698],[-0.009550989,0.01062154,0.9998981],[-0.006945993,0.01062179,0.9999195],[-0.004340856,0.01062192,0.9999342],[-0.003038221,0.01062195,0.999939],[-0.04207939,0.01133127,0.99905],[-0.04078004,0.01133189,0.9991039],[-0.0381809,0.01133305,0.9992067],[-0.03558102,0.01133414,0.9993026],[-0.03298041,0.01133515,0.9993918],[-0.03037909,0.01133608,0.9994742],[-0.02777713,0.01133694,0.9995499],[-0.02517463,0.0113377,0.9996188],[-0.02257169,0.01133836,0.999681],[-0.01996824,0.01133899,0.9997363],[-0.01736436,0.01133956,0.9997849],[-0.0147602,0.01134004,0.9998268],[-0.01215571,0.01134046,0.9998618],[-0.009550889,0.0113408,0.9998901],[-0.006945938,0.01134105,0.9999116],[-0.004340846,0.01134121,0.9999263],[-0.003038197,0.01134127,0.9999312],[-0.04207902,0.01204987,0.9990417],[-0.0407797,0.01205052,0.9990955],[-0.03818059,0.01205174,0.9991983],[-0.03558071,0.0120529,0.9992942],[-0.03298013,0.01205399,0.9993833],[-0.03037885,0.01205497,0.9994658],[-0.0277769,0.01205587,0.9995415],[-0.02517444,0.0120567,0.9996104],[-0.02257152,0.01205745,0.9996725],[-0.01996805,0.01205812,0.999728],[-0.01736422,0.01205871,0.9997766],[-0.01476008,0.01205921,0.9998184],[-0.01215559,0.01205963,0.9998534],[-0.009550809,0.01205998,0.9998817],[-0.006945879,0.01206024,0.9999032],[-0.004340809,0.01206041,0.9999179],[-0.003038172,0.01206047,0.9999228],[-0.0420786,0.01276842,0.9990327],[-0.04077934,0.01276911,0.9990866],[-0.03818025,0.01277042,0.9991893],[-0.03558036,0.01277166,0.9992852],[-0.03297985,0.0127728,0.9993744],[-0.0303786,0.01277383,0.9994569],[-0.02777665,0.01277479,0.9995326],[-0.02517419,0.01277569,0.9996015],[-0.0225713,0.01277653,0.9996636],[-0.0199679,0.01277724,0.999719],[-0.01736406,0.01277784,0.9997676],[-0.01475994,0.01277837,0.9998094],[-0.01215548,0.01277882,0.9998446],[-0.009550699,0.0127792,0.9998728],[-0.00694584,0.01277948,0.9998943],[-0.004340795,0.01277962,0.999909],[-0.003038145,0.01277966,0.9999138],[-0.0420782,0.013487,0.9990233],[-0.04077896,0.01348773,0.9990773],[-0.0381799,0.01348912,0.9991799],[-0.03558002,0.0134904,0.9992758],[-0.03297954,0.0134916,0.999365],[-0.03037832,0.01349274,0.9994475],[-0.02777639,0.01349377,0.9995232],[-0.02517396,0.01349468,0.999592],[-0.02257106,0.0134955,0.9996542],[-0.01996771,0.01349627,0.9997095],[-0.01736392,0.01349692,0.9997581],[-0.01475981,0.01349746,0.9998],[-0.01215534,0.01349796,0.9998351],[-0.009550609,0.01349834,0.9998633],[-0.006945799,0.01349861,0.9998848],[-0.004340753,0.01349883,0.9998995],[-0.003038116,0.01349892,0.9999043],[-0.0420778,0.01420554,0.9990134],[-0.04077853,0.01420632,0.9990672],[-0.03817952,0.01420779,0.9991699],[-0.03557969,0.01420914,0.9992659],[-0.03297919,0.01421041,0.9993551],[-0.03037798,0.01421162,0.9994375],[-0.02777612,0.01421271,0.9995132],[-0.02517375,0.01421365,0.9995821],[-0.02257084,0.01421451,0.9996443],[-0.01996748,0.0142153,0.9996997],[-0.01736377,0.01421599,0.9997482],[-0.01475966,0.01421658,0.99979],[-0.0121552,0.01421708,0.9998251],[-0.009550538,0.01421748,0.9998534],[-0.006945706,0.01421781,0.9998749],[-0.004340686,0.01421804,0.9998895],[-0.003038133,0.01421811,0.9998943],[-0.04207736,0.01492408,0.9990029],[-0.0407781,0.01492488,0.9990568],[-0.03817912,0.01492642,0.9991595],[-0.03557933,0.01492785,0.9992554],[-0.03297883,0.01492918,0.9993446],[-0.03037764,0.01493041,0.999427],[-0.02777586,0.01493153,0.9995027],[-0.0251735,0.01493256,0.9995716],[-0.0225706,0.0149335,0.9996337],[-0.0199673,0.0149343,0.9996891],[-0.01736357,0.01493503,0.9997378],[-0.0147595,0.01493568,0.9997796],[-0.01215507,0.0149362,0.9998146],[-0.009550414,0.01493665,0.9998429],[-0.006945611,0.014937,0.9998644],[-0.004340664,0.01493719,0.9998791],[-0.003038197,0.01493724,0.9998839],[-0.04207692,0.01564257,0.998992],[-0.04077766,0.0156434,0.9990458],[-0.0381787,0.015645,0.9991485],[-0.03557894,0.0156465,0.9992444],[-0.03297845,0.0156479,0.9993336],[-0.0303773,0.01564921,0.999416],[-0.02777556,0.01565039,0.9994917],[-0.0251732,0.01565148,0.9995606],[-0.02257035,0.01565247,0.9996228],[-0.0199671,0.01565334,0.9996781],[-0.01736335,0.0156541,0.9997267],[-0.01475934,0.01565475,0.9997686],[-0.01215496,0.01565528,0.9998035],[-0.009550308,0.01565567,0.9998319],[-0.006945533,0.01565598,0.9998533],[-0.004340617,0.01565624,0.9998681],[-0.003038163,0.01565634,0.9998729],[-0.04207671,0.01600177,0.9989862],[-0.04077744,0.01600263,0.9990402],[-0.03817848,0.01600428,0.9991428],[-0.03557875,0.01600582,0.9992388],[-0.03297826,0.01600727,0.999328],[-0.03037713,0.01600864,0.9994104],[-0.02777541,0.01600987,0.9994861],[-0.02517305,0.01601098,0.999555],[-0.02257022,0.01601197,0.999617],[-0.01996699,0.01601286,0.9996724],[-0.01736325,0.01601364,0.9997211],[-0.01475926,0.01601431,0.9997629],[-0.01215491,0.01601482,0.9997979],[-0.009550278,0.01601518,0.9998261],[-0.006945494,0.01601547,0.9998477],[-0.004340568,0.01601576,0.9998624],[-0.003038098,0.01601588,0.9998672]],"colors":[[0,0,0,0.2980392]],"dim":[[17,20]],"centers":[[-32.5,-22.5,7.818274],[-27.5,-22.5,8.022358],[-22.5,-22.5,8.213414],[-17.5,-22.5,8.391443],[-12.5,-22.5,8.556446],[-7.5,-22.5,8.708422],[-2.5,-22.5,8.84737],[2.5,-22.5,8.973291],[7.5,-22.5,9.086186],[12.5,-22.5,9.186054],[17.5,-22.5,9.272895],[22.5,-22.5,9.346708],[27.5,-22.5,9.407495],[32.5,-22.5,9.455256],[37.5,-22.5,9.489988],[42.5,-22.5,9.511694],[-32.5,-17.5,7.80113],[-27.5,-17.5,8.005214],[-22.5,-17.5,8.196271],[-17.5,-17.5,8.3743],[-12.5,-17.5,8.539303],[-7.5,-17.5,8.691278],[-2.5,-17.5,8.830227],[2.5,-17.5,8.956149],[7.5,-17.5,9.069043],[12.5,-17.5,9.16891],[17.5,-17.5,9.255751],[22.5,-17.5,9.329565],[27.5,-17.5,9.390352],[32.5,-17.5,9.438112],[37.5,-17.5,9.472845],[42.5,-17.5,9.494551],[-32.5,-12.5,7.78039],[-27.5,-12.5,7.984474],[-22.5,-12.5,8.17553],[-17.5,-12.5,8.353559],[-12.5,-12.5,8.518562],[-7.5,-12.5,8.670538],[-2.5,-12.5,8.809486],[2.5,-12.5,8.935408],[7.5,-12.5,9.048302],[12.5,-12.5,9.14817],[17.5,-12.5,9.235011],[22.5,-12.5,9.308825],[27.5,-12.5,9.369612],[32.5,-12.5,9.417371],[37.5,-12.5,9.452104],[42.5,-12.5,9.47381],[-32.5,-7.5,7.756053],[-27.5,-7.5,7.960136],[-22.5,-7.5,8.151193],[-17.5,-7.5,8.329222],[-12.5,-7.5,8.494225],[-7.5,-7.5,8.6462],[-2.5,-7.5,8.785149],[2.5,-7.5,8.91107],[7.5,-7.5,9.023965],[12.5,-7.5,9.123832],[17.5,-7.5,9.210673],[22.5,-7.5,9.284487],[27.5,-7.5,9.345274],[32.5,-7.5,9.393034],[37.5,-7.5,9.427767],[42.5,-7.5,9.449472],[-32.5,-2.5,7.728118],[-27.5,-2.5,7.932201],[-22.5,-2.5,8.123259],[-17.5,-2.5,8.301288],[-12.5,-2.5,8.46629],[-7.5,-2.5,8.618265],[-2.5,-2.5,8.757214],[2.5,-2.5,8.883136],[7.5,-2.5,8.99603],[12.5,-2.5,9.095898],[17.5,-2.5,9.182738],[22.5,-2.5,9.256553],[27.5,-2.5,9.317339],[32.5,-2.5,9.365099],[37.5,-2.5,9.399832],[42.5,-2.5,9.421538],[-32.5,2.5,7.696587],[-27.5,2.5,7.90067],[-22.5,2.5,8.091726],[-17.5,2.5,8.269755],[-12.5,2.5,8.434758],[-7.5,2.5,8.586734],[-2.5,2.5,8.725682],[2.5,2.5,8.851604],[7.5,2.5,8.964499],[12.5,2.5,9.064366],[17.5,2.5,9.151207],[22.5,2.5,9.225021],[27.5,2.5,9.285809],[32.5,2.5,9.333569],[37.5,2.5,9.368301],[42.5,2.5,9.390007],[-32.5,7.5,7.661458],[-27.5,7.5,7.865542],[-22.5,7.5,8.056599],[-17.5,7.5,8.234628],[-12.5,7.5,8.39963],[-7.5,7.5,8.551605],[-2.5,7.5,8.690554],[2.5,7.5,8.816476],[7.5,7.5,8.929371],[12.5,7.5,9.029238],[17.5,7.5,9.116079],[22.5,7.5,9.189893],[27.5,7.5,9.25068],[32.5,7.5,9.298439],[37.5,7.5,9.333173],[42.5,7.5,9.354877],[-32.5,12.5,7.622733],[-27.5,12.5,7.826817],[-22.5,12.5,8.017873],[-17.5,12.5,8.195902],[-12.5,12.5,8.360905],[-7.5,12.5,8.51288],[-2.5,12.5,8.651829],[2.5,12.5,8.77775],[7.5,12.5,8.890645],[12.5,12.5,8.990513],[17.5,12.5,9.077353],[22.5,12.5,9.151167],[27.5,12.5,9.211954],[32.5,12.5,9.259714],[37.5,12.5,9.294447],[42.5,12.5,9.316153],[-32.5,17.5,7.58041],[-27.5,17.5,7.784493],[-22.5,17.5,7.97555],[-17.5,17.5,8.15358],[-12.5,17.5,8.318583],[-7.5,17.5,8.470558],[-2.5,17.5,8.609507],[2.5,17.5,8.735428],[7.5,17.5,8.848322],[12.5,17.5,8.948191],[17.5,17.5,9.03503],[22.5,17.5,9.108845],[27.5,17.5,9.169632],[32.5,17.5,9.217392],[37.5,17.5,9.252125],[42.5,17.5,9.27383],[-32.5,22.5,7.534491],[-27.5,22.5,7.738575],[-22.5,22.5,7.929631],[-17.5,22.5,8.10766],[-12.5,22.5,8.272663],[-7.5,22.5,8.424639],[-2.5,22.5,8.563587],[2.5,22.5,8.689508],[7.5,22.5,8.802402],[12.5,22.5,8.902271],[17.5,22.5,8.989112],[22.5,22.5,9.062925],[27.5,22.5,9.123713],[32.5,22.5,9.171473],[37.5,22.5,9.206205],[42.5,22.5,9.227911],[-32.5,27.5,7.484975],[-27.5,27.5,7.689059],[-22.5,27.5,7.880115],[-17.5,27.5,8.058145],[-12.5,27.5,8.223146],[-7.5,27.5,8.375122],[-2.5,27.5,8.514071],[2.5,27.5,8.639992],[7.5,27.5,8.752888],[12.5,27.5,8.852755],[17.5,27.5,8.939596],[22.5,27.5,9.013409],[27.5,27.5,9.074196],[32.5,27.5,9.121956],[37.5,27.5,9.156689],[42.5,27.5,9.178394],[-32.5,32.5,7.431862],[-27.5,32.5,7.635945],[-22.5,32.5,7.827002],[-17.5,32.5,8.005032],[-12.5,32.5,8.170034],[-7.5,32.5,8.32201],[-2.5,32.5,8.460958],[2.5,32.5,8.58688],[7.5,32.5,8.699774],[12.5,32.5,8.799642],[17.5,32.5,8.886482],[22.5,32.5,8.960297],[27.5,32.5,9.021083],[32.5,32.5,9.068844],[37.5,32.5,9.103576],[42.5,32.5,9.125282],[-32.5,37.5,7.375152],[-27.5,37.5,7.579235],[-22.5,37.5,7.770291],[-17.5,37.5,7.948321],[-12.5,37.5,8.113324],[-7.5,37.5,8.2653],[-2.5,37.5,8.404248],[2.5,37.5,8.530169],[7.5,37.5,8.643064],[12.5,37.5,8.742931],[17.5,37.5,8.829772],[22.5,37.5,8.903586],[27.5,37.5,8.964373],[32.5,37.5,9.012133],[37.5,37.5,9.046865],[42.5,37.5,9.068571],[-32.5,42.5,7.314845],[-27.5,42.5,7.518928],[-22.5,42.5,7.709985],[-17.5,42.5,7.888014],[-12.5,42.5,8.053017],[-7.5,42.5,8.204992],[-2.5,42.5,8.343941],[2.5,42.5,8.469862],[7.5,42.5,8.582756],[12.5,42.5,8.682625],[17.5,42.5,8.769465],[22.5,42.5,8.843279],[27.5,42.5,8.904066],[32.5,42.5,8.951826],[37.5,42.5,8.986559],[42.5,42.5,9.008265],[-32.5,47.5,7.25094],[-27.5,47.5,7.455024],[-22.5,47.5,7.646081],[-17.5,47.5,7.82411],[-12.5,47.5,7.989113],[-7.5,47.5,8.141088],[-2.5,47.5,8.280037],[2.5,47.5,8.405958],[7.5,47.5,8.518853],[12.5,47.5,8.61872],[17.5,47.5,8.705561],[22.5,47.5,8.779375],[27.5,47.5,8.840162],[32.5,47.5,8.887921],[37.5,47.5,8.922655],[42.5,47.5,8.944361],[-32.5,52.5,7.183439],[-27.5,52.5,7.387523],[-22.5,52.5,7.578579],[-17.5,52.5,7.756609],[-12.5,52.5,7.921612],[-7.5,52.5,8.073587],[-2.5,52.5,8.212536],[2.5,52.5,8.338457],[7.5,52.5,8.451352],[12.5,52.5,8.551219],[17.5,52.5,8.638061],[22.5,52.5,8.711874],[27.5,52.5,8.772661],[32.5,52.5,8.820421],[37.5,52.5,8.855154],[42.5,52.5,8.87686],[-32.5,57.5,7.112342],[-27.5,57.5,7.316425],[-22.5,57.5,7.507482],[-17.5,57.5,7.685511],[-12.5,57.5,7.850514],[-7.5,57.5,8.002489],[-2.5,57.5,8.141438],[2.5,57.5,8.26736],[7.5,57.5,8.380254],[12.5,57.5,8.480122],[17.5,57.5,8.566962],[22.5,57.5,8.640777],[27.5,57.5,8.701563],[32.5,57.5,8.749323],[37.5,57.5,8.784056],[42.5,57.5,8.805761],[-32.5,62.5,7.037647],[-27.5,62.5,7.24173],[-22.5,62.5,7.432787],[-17.5,62.5,7.610816],[-12.5,62.5,7.775819],[-7.5,62.5,7.927794],[-2.5,62.5,8.066743],[2.5,62.5,8.192665],[7.5,62.5,8.305559],[12.5,62.5,8.405427],[17.5,62.5,8.492268],[22.5,62.5,8.566082],[27.5,62.5,8.626868],[32.5,62.5,8.674628],[37.5,62.5,8.70936],[42.5,62.5,8.731067],[-32.5,67.5,6.959355],[-27.5,67.5,7.163439],[-22.5,67.5,7.354495],[-17.5,67.5,7.532525],[-12.5,67.5,7.697527],[-7.5,67.5,7.849503],[-2.5,67.5,7.988451],[2.5,67.5,8.114372],[7.5,67.5,8.227267],[12.5,67.5,8.327135],[17.5,67.5,8.413976],[22.5,67.5,8.48779],[27.5,67.5,8.548576],[32.5,67.5,8.596336],[37.5,67.5,8.631069],[42.5,67.5,8.652775]],"ignoreExtent":false,"flags":91},"5":{"id":5,"reuse":"testgldiv"},"6":{"id":6,"reuse":"testgldiv"},"29":{"id":29,"type":"bboxdeco","material":{"front":"lines","back":"lines"},"vertices":[[-20,"NA","NA"],[0,"NA","NA"],[20,"NA","NA"],[40,"NA","NA"],["NA",-20,"NA"],["NA",0,"NA"],["NA",20,"NA"],["NA",40,"NA"],["NA",60,"NA"],["NA","NA",7],["NA","NA",8],["NA","NA",9],["NA","NA",10]],"colors":[[0,0,0,1]],"draw_front":true,"newIds":[42,43,44,45,46,47,48]},"1":{"id":1,"type":"subscene","par3d":{"antialias":8,"FOV":30,"ignoreExtent":false,"listeners":1,"mouseMode":{"left":"trackball","right":"zoom","middle":"fov","wheel":"pull"},"observer":[0,0,314.5009],"modelMatrix":[[0.9689531,0,0,-4.844765],[0,0.2460616,17.59739,-150.7158],[0,-0.6760486,6.404928,-354.2163],[0,0,0,1]],"projMatrix":[[2.665751,0,0,0],[0,3.732051,0,0],[0,0,-3.863704,-1133.739],[0,0,-1,0]],"skipRedraw":false,"userMatrix":[[1,0,0,0],[0,0.3420201,0.9396926,0],[0,-0.9396926,0.3420201,0],[0,0,0,1]],"scale":[0.9689531,0.7194359,18.72676],"viewport":{"x":0,"y":0,"width":1,"height":1},"zoom":1,"bbox":[-35,45,-30.60419,70.15615,6.352662,10.22362],"windowRect":[0,45,672,525],"family":"sans","font":1,"cex":1,"useFreeType":true,"fontname":"/Library/Frameworks/R.framework/Versions/3.2/Resources/library/rgl/fonts/FreeSans.ttf","maxClipPlanes":6},"embeddings":{"viewport":"replace","projection":"replace","model":"replace"},"objects":[6,29,28,30,31,32,33,34,5,42,43,44,45,46,47,48],"subscenes":[],"flags":1275},"42":{"id":42,"type":"lines","material":{"lit":false,"front":"lines","back":"lines"},"vertices":[[-20,-32.11559,6.294598],[40,-32.11559,6.294598],[-20,-32.11559,6.294598],[-20,-34.71017,6.194921],[0,-32.11559,6.294598],[0,-34.71017,6.194921],[20,-32.11559,6.294598],[20,-34.71017,6.194921],[40,-32.11559,6.294598],[40,-34.71017,6.194921]],"colors":[[0,0,0,1]],"centers":[[10,-32.11559,6.294598],[-20,-33.41288,6.244759],[0,-33.41288,6.244759],[20,-33.41288,6.244759],[40,-33.41288,6.244759]],"ignoreExtent":true,"origId":29,"flags":128},"43":{"id":43,"type":"text","material":{"lit":false,"front":"lines","back":"lines"},"vertices":[[-20,-39.89933,5.995566],[0,-39.89933,5.995566],[20,-39.89933,5.995566],[40,-39.89933,5.995566]],"colors":[[0,0,0,1]],"texts":[["-20"],["0"],["20"],["40"]],"cex":[[1]],"adj":[[0.5,0.5]],"centers":[[-20,-39.89933,5.995566],[0,-39.89933,5.995566],[20,-39.89933,5.995566],[40,-39.89933,5.995566]],"family":[["sans"]],"font":[[1]],"ignoreExtent":true,"origId":29,"flags":40},"44":{"id":44,"type":"lines","material":{"lit":false,"front":"lines","back":"lines"},"vertices":[[-36.2,-20,6.294598],[-36.2,60,6.294598],[-36.2,-20,6.294598],[-38.26,-20,6.194921],[-36.2,0,6.294598],[-38.26,0,6.194921],[-36.2,20,6.294598],[-38.26,20,6.194921],[-36.2,40,6.294598],[-38.26,40,6.194921],[-36.2,60,6.294598],[-38.26,60,6.194921]],"colors":[[0,0,0,1]],"centers":[[-36.2,20,6.294598],[-37.23,-20,6.244759],[-37.23,0,6.244759],[-37.23,20,6.244759],[-37.23,40,6.244759],[-37.23,60,6.244759]],"ignoreExtent":true,"origId":29,"flags":128},"45":{"id":45,"type":"text","material":{"lit":false,"front":"lines","back":"lines"},"vertices":[[-42.38,-20,5.995566],[-42.38,0,5.995566],[-42.38,20,5.995566],[-42.38,40,5.995566],[-42.38,60,5.995566]],"colors":[[0,0,0,1]],"texts":[["-20"],["0"],["20"],["40"],["60"]],"cex":[[1]],"adj":[[0.5,0.5]],"centers":[[-42.38,-20,5.995566],[-42.38,0,5.995566],[-42.38,20,5.995566],[-42.38,40,5.995566],[-42.38,60,5.995566]],"family":[["sans"]],"font":[[1]],"ignoreExtent":true,"origId":29,"flags":40},"46":{"id":46,"type":"lines","material":{"lit":false,"front":"lines","back":"lines"},"vertices":[[-36.2,-32.11559,7],[-36.2,-32.11559,10],[-36.2,-32.11559,7],[-38.26,-34.71017,7],[-36.2,-32.11559,8],[-38.26,-34.71017,8],[-36.2,-32.11559,9],[-38.26,-34.71017,9],[-36.2,-32.11559,10],[-38.26,-34.71017,10]],"colors":[[0,0,0,1]],"centers":[[-36.2,-32.11559,8.5],[-37.23,-33.41288,7],[-37.23,-33.41288,8],[-37.23,-33.41288,9],[-37.23,-33.41288,10]],"ignoreExtent":true,"origId":29,"flags":128},"47":{"id":47,"type":"text","material":{"lit":false,"front":"lines","back":"lines"},"vertices":[[-42.38,-39.89933,7],[-42.38,-39.89933,8],[-42.38,-39.89933,9],[-42.38,-39.89933,10]],"colors":[[0,0,0,1]],"texts":[["7"],["8"],["9"],["10"]],"cex":[[1]],"adj":[[0.5,0.5]],"centers":[[-42.38,-39.89933,7],[-42.38,-39.89933,8],[-42.38,-39.89933,9],[-42.38,-39.89933,10]],"family":[["sans"]],"font":[[1]],"ignoreExtent":true,"origId":29,"flags":40},"48":{"id":48,"type":"lines","material":{"lit":false,"front":"lines","back":"lines"},"vertices":[[-36.2,-32.11559,6.294598],[-36.2,71.66756,6.294598],[-36.2,-32.11559,10.28169],[-36.2,71.66756,10.28169],[-36.2,-32.11559,6.294598],[-36.2,-32.11559,10.28169],[-36.2,71.66756,6.294598],[-36.2,71.66756,10.28169],[-36.2,-32.11559,6.294598],[46.2,-32.11559,6.294598],[-36.2,-32.11559,10.28169],[46.2,-32.11559,10.28169],[-36.2,71.66756,6.294598],[46.2,71.66756,6.294598],[-36.2,71.66756,10.28169],[46.2,71.66756,10.28169],[46.2,-32.11559,6.294598],[46.2,71.66756,6.294598],[46.2,-32.11559,10.28169],[46.2,71.66756,10.28169],[46.2,-32.11559,6.294598],[46.2,-32.11559,10.28169],[46.2,71.66756,6.294598],[46.2,71.66756,10.28169]],"colors":[[0,0,0,1]],"centers":[[-36.2,19.77598,6.294598],[-36.2,19.77598,10.28169],[-36.2,-32.11559,8.288142],[-36.2,71.66756,8.288142],[5,-32.11559,6.294598],[5,-32.11559,10.28169],[5,71.66756,6.294598],[5,71.66756,10.28169],[46.2,19.77598,6.294598],[46.2,19.77598,10.28169],[46.2,-32.11559,8.288142],[46.2,71.66756,8.288142]],"ignoreExtent":true,"origId":29,"flags":128}},"snapshot":"data:image/png;base64,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","width":673,"height":481,"sphereVerts":{"reuse":"testgldiv"}});
testgl2rgl.prefix = "testgl2";
</script>
<p id="testgl2debug">
You must enable Javascript to view this page properly.</p>
<script>testgl2rgl.start();</script>

By transforming both the predictors and the target variable, we achieve an improved model fit. Note how the adjusted R-square has jumped to **0.7545965**. Most predictors’ p-values are significant. Here, the squared women.c predictor yields a weak p-value (maybe an indication that in the presence of other predictors, it is not relevant to include and we could exclude it from the model.)





Let’s go on and remove the squared women.c variable from the model to see how it changes:


```r
# fit a model excluding the variable education,  log the income variable.
mod4 = lm(log(income) ~ prestige.c + I(prestige.c^2) + women.c , data=newdata)
summary(mod4)
```

```
## 
## Call:
## lm(formula = log(income) ~ prestige.c + I(prestige.c^2) + women.c, 
##     data = newdata)
## 
## Residuals:
##      Min       1Q   Median       3Q      Max 
## -1.12302 -0.09650  0.02764  0.13913  0.78303 
## 
## Coefficients:
##                   Estimate Std. Error t value Pr(>|t|)    
## (Intercept)      8.729e+00  4.023e-02 216.995  < 2e-16 ***
## prestige.c       2.499e-02  1.803e-03  13.858  < 2e-16 ***
## I(prestige.c^2) -2.351e-04  9.392e-05  -2.503    0.014 *  
## women.c         -8.365e-03  9.376e-04  -8.922 2.64e-14 ***
## ---
## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
## 
## Residual standard error: 0.2963 on 98 degrees of freedom
## Multiple R-squared:  0.7565,	Adjusted R-squared:  0.7491 
## F-statistic: 101.5 on 3 and 98 DF,  p-value: < 2.2e-16
```

```r
# Plot model residuals.
plot(mod4, pch=16, which=1)
```

![](https://github.com/FelipeRego/feliperego.github.io/raw/master/images/MultipleLinearRegression_files/figure-html/unnamed-chunk-10-1.png)<!-- -->



```r
newdat3 <- expand.grid(prestige.c=seq(-35,45,by=5),women.c=seq(-25,70,by=5))
newdat3$pp <- predict(mod4,newdata=newdat3)
with(newdata,plot3d(prestige.c,women.c,log(income), col="blue", size=1, type="s", main="3D Quadratic Model Fit with Log of Income excl. Women^2"))
with(newdat3,surface3d(unique(prestige.c),unique(women.c),pp,
                      alpha=0.3,front="line", back="line"))
```

<div id="testgl3div" class="rglWebGL"></div>
<script type="text/javascript">
var testgl3div = document.getElementById("testgl3div"),
testgl3rgl = new rglwidgetClass();
testgl3div.width = 673;
testgl3div.height = 481;
testgl3rgl.initialize(testgl3div,
{"material":{"color":"#000000","alpha":1,"lit":true,"ambient":"#000000","specular":"#FFFFFF","emission":"#000000","shininess":50,"smooth":true,"front":"filled","back":"filled","size":3,"lwd":1,"fog":false,"point_antialias":false,"line_antialias":false,"texture":null,"textype":"rgb","texmipmap":false,"texminfilter":"linear","texmagfilter":"linear","texenvmap":false,"depth_mask":true,"depth_test":"less"},"rootSubscene":1,"objects":{"49":{"id":49,"type":"spheres","material":{},"vertices":[[21.96667,-17.81902,9.421493],[22.26667,-24.95902,10.16119],[16.56667,-13.27902,9.134646],[9.966666,-19.86902,9.089867],[26.66667,-17.29902,9.036345],[30.76667,-23.84902,9.308374],[25.76667,-3.32902,9.018938],[31.26667,-26.28902,9.558388],[26.26667,-27.94902,9.339349],[21.96667,-28.03902,9.307739],[15.16667,-27.06902,8.683046],[13.16667,-21.14902,8.862059],[6.966667,-13.64902,9.038959],[15.36667,28.33098,8.993303],[28.06667,19.30098,8.909911],[8.266666,25.79098,8.754003],[35.46667,-23.84902,9.865941],[11.26667,48.12098,8.718009],[11.46667,5.91098,9.168789],[25.96667,-24.83902,8.452334],[37.76667,-9.38902,9.431883],[12.76667,54.80098,8.639057],[19.26667,17.82098,8.991438],[40.36666,-18.41902,10.13888],[19.86667,-24.65902,9.585896],[21.56667,-22.06902,9.769842],[17.86667,67.14098,8.436851],[-11.93333,47.16098,8.156223],[25.26667,53.68098,8.535426],[22.46667,-4.26902,9.252633],[20.66667,47.06098,8.552561],[10.36667,-7.949019,8.73182],[10.76667,-17.82902,8.930891],[7.266667,-20.84902,9.012621],[-0.8333333,68.53098,8.303009],[-4.933333,66.99098,8.054523],[2.566667,39.26098,8.377471],[-4.533333,62.78098,7.803027],[0.8666667,46.94098,8.373322],[-15.93333,-17.60902,8.468213],[-14.13333,54.21098,8.011686],[-8.133333,63.88098,7.972811],[-10.73333,-21.35902,8.614501],[-9.633333,23.29098,8.226574],[-8.733334,67.16098,8.058643],[-17.43333,18.08098,8.464004],[4.266667,27.12098,8.527539],[-11.13333,10.19098,8.741776],[-11.23333,34.25098,8.312626],[-5.333333,-11.93902,8.920256],[-6.633333,-25.81902,9.080232],[-20.33333,38.84098,7.860956],[-32.03333,-21.97902,6.822197],[-23.53333,-25.28902,7.770645],[0.4666667,-15.88902,9.003439],[0.2666667,-4.53902,8.852522],[4.266667,-5.09902,8.981682],[-3.333333,-28.97902,9.093245],[4.766667,-27.32902,9.092794],[-17.13333,23.02098,8.044306],[-26.63333,-13.46902,8.276395],[8.066667,-22.96902,8.970686],[-20.93333,67.55098,6.415097],[-26.03333,40.33098,8.006368],[-29.53333,4.590981,8.152486],[-26.73333,1.10098,8.183677],[-2.733333,-25.37902,8.200562],[-25.33333,-1.22902,7.41216],[-11.53333,-28.97902,8.833463],[-7.933333,4.320981,8.342602],[-21.63333,-11.71902,8.54364],[-12.03333,-11.71902,8.54364],[-23.63333,43.26098,7.544332],[-13.53333,2.38098,8.399085],[-18.03333,10.50098,8.156223],[-4.333333,-27.47902,8.992558],[-2.633333,-24.69902,8.807771],[-10.93333,-26.67902,8.789508],[-5.033333,-23.80902,8.776012],[-10.93333,-15.35902,8.667508],[-3.133333,-23.19902,8.790726],[3.966667,45.56098,8.279444],[-9.633333,-26.05902,8.603188],[-18.63333,61.69098,7.954021],[-8.733334,-28.16902,8.664751],[3.466667,-28.19902,8.951052],[-19.53333,-28.97902,8.454467],[-5.933333,-27.63902,9.025937],[3.366667,-27.98902,8.874448],[4.266667,-28.32902,9.091557],[-7.933333,-28.41902,8.575274],[-10.63333,-28.45902,8.692658],[-16.93333,-26.51902,8.422663],[-3.933333,-28.36902,8.843327],[-20.33333,-27.88902,8.271293],[19.26667,-28.39902,9.549096],[2.066667,-28.97902,9.087607],[-10.93333,-19.50902,8.623713],[-21.73333,-25.38902,8.348537],[-20.73333,-28.97902,8.466531],[-4.633333,-15.39902,8.773694],[-11.63333,41.89098,8.1934]],"colors":[[0,0,1,1]],"radii":[[1.169203]],"centers":[[21.96667,-17.81902,9.421493],[22.26667,-24.95902,10.16119],[16.56667,-13.27902,9.134646],[9.966666,-19.86902,9.089867],[26.66667,-17.29902,9.036345],[30.76667,-23.84902,9.308374],[25.76667,-3.32902,9.018938],[31.26667,-26.28902,9.558388],[26.26667,-27.94902,9.339349],[21.96667,-28.03902,9.307739],[15.16667,-27.06902,8.683046],[13.16667,-21.14902,8.862059],[6.966667,-13.64902,9.038959],[15.36667,28.33098,8.993303],[28.06667,19.30098,8.909911],[8.266666,25.79098,8.754003],[35.46667,-23.84902,9.865941],[11.26667,48.12098,8.718009],[11.46667,5.91098,9.168789],[25.96667,-24.83902,8.452334],[37.76667,-9.38902,9.431883],[12.76667,54.80098,8.639057],[19.26667,17.82098,8.991438],[40.36666,-18.41902,10.13888],[19.86667,-24.65902,9.585896],[21.56667,-22.06902,9.769842],[17.86667,67.14098,8.436851],[-11.93333,47.16098,8.156223],[25.26667,53.68098,8.535426],[22.46667,-4.26902,9.252633],[20.66667,47.06098,8.552561],[10.36667,-7.949019,8.73182],[10.76667,-17.82902,8.930891],[7.266667,-20.84902,9.012621],[-0.8333333,68.53098,8.303009],[-4.933333,66.99098,8.054523],[2.566667,39.26098,8.377471],[-4.533333,62.78098,7.803027],[0.8666667,46.94098,8.373322],[-15.93333,-17.60902,8.468213],[-14.13333,54.21098,8.011686],[-8.133333,63.88098,7.972811],[-10.73333,-21.35902,8.614501],[-9.633333,23.29098,8.226574],[-8.733334,67.16098,8.058643],[-17.43333,18.08098,8.464004],[4.266667,27.12098,8.527539],[-11.13333,10.19098,8.741776],[-11.23333,34.25098,8.312626],[-5.333333,-11.93902,8.920256],[-6.633333,-25.81902,9.080232],[-20.33333,38.84098,7.860956],[-32.03333,-21.97902,6.822197],[-23.53333,-25.28902,7.770645],[0.4666667,-15.88902,9.003439],[0.2666667,-4.53902,8.852522],[4.266667,-5.09902,8.981682],[-3.333333,-28.97902,9.093245],[4.766667,-27.32902,9.092794],[-17.13333,23.02098,8.044306],[-26.63333,-13.46902,8.276395],[8.066667,-22.96902,8.970686],[-20.93333,67.55098,6.415097],[-26.03333,40.33098,8.006368],[-29.53333,4.590981,8.152486],[-26.73333,1.10098,8.183677],[-2.733333,-25.37902,8.200562],[-25.33333,-1.22902,7.41216],[-11.53333,-28.97902,8.833463],[-7.933333,4.320981,8.342602],[-21.63333,-11.71902,8.54364],[-12.03333,-11.71902,8.54364],[-23.63333,43.26098,7.544332],[-13.53333,2.38098,8.399085],[-18.03333,10.50098,8.156223],[-4.333333,-27.47902,8.992558],[-2.633333,-24.69902,8.807771],[-10.93333,-26.67902,8.789508],[-5.033333,-23.80902,8.776012],[-10.93333,-15.35902,8.667508],[-3.133333,-23.19902,8.790726],[3.966667,45.56098,8.279444],[-9.633333,-26.05902,8.603188],[-18.63333,61.69098,7.954021],[-8.733334,-28.16902,8.664751],[3.466667,-28.19902,8.951052],[-19.53333,-28.97902,8.454467],[-5.933333,-27.63902,9.025937],[3.366667,-27.98902,8.874448],[4.266667,-28.32902,9.091557],[-7.933333,-28.41902,8.575274],[-10.63333,-28.45902,8.692658],[-16.93333,-26.51902,8.422663],[-3.933333,-28.36902,8.843327],[-20.33333,-27.88902,8.271293],[19.26667,-28.39902,9.549096],[2.066667,-28.97902,9.087607],[-10.93333,-19.50902,8.623713],[-21.73333,-25.38902,8.348537],[-20.73333,-28.97902,8.466531],[-4.633333,-15.39902,8.773694],[-11.63333,41.89098,8.1934]],"ignoreExtent":false,"flags":3},"51":{"id":51,"type":"text","material":{"lit":false},"vertices":[[4.166666,87.23503,10.87975]],"colors":[[0,0,0,1]],"texts":[["3D Quadratic Model Fit with Log of Income excl. Women^2"]],"cex":[[1]],"adj":[[0.5,0.5]],"centers":[[4.166666,87.23503,10.87975]],"family":[["sans"]],"font":[[1]],"ignoreExtent":true,"flags":40},"52":{"id":52,"type":"text","material":{"lit":false},"vertices":[[4.166666,-47.68306,5.696534]],"colors":[[0,0,0,1]],"texts":[["prestige.c"]],"cex":[[1]],"adj":[[0.5,0.5]],"centers":[[4.166666,-47.68306,5.696534]],"family":[["sans"]],"font":[[1]],"ignoreExtent":true,"flags":40},"53":{"id":53,"type":"text","material":{"lit":false},"vertices":[[-45.92086,19.77598,5.696534]],"colors":[[0,0,0,1]],"texts":[["women.c"]],"cex":[[1]],"adj":[[0.5,0.5]],"centers":[[-45.92086,19.77598,5.696534]],"family":[["sans"]],"font":[[1]],"ignoreExtent":true,"flags":40},"54":{"id":54,"type":"text","material":{"lit":false},"vertices":[[-45.92086,-47.68306,8.288142]],"colors":[[0,0,0,1]],"texts":[["log(income)"]],"cex":[[1]],"adj":[[0.5,0.5]],"centers":[[-45.92086,-47.68306,8.288142]],"family":[["sans"]],"font":[[1]],"ignoreExtent":true,"flags":40},"55":{"id":55,"type":"surface","material":{"alpha":0.2980392,"front":"lines","back":"lines"},"vertices":[[-35,-25,7.775837],[-30,-25,7.977198],[-25,-25,8.166804],[-20,-25,8.344659],[-15,-25,8.510759],[-10,-25,8.665107],[-5,-25,8.8077],[0,-25,8.938541],[5,-25,9.057629],[10,-25,9.164963],[15,-25,9.260544],[20,-25,9.344371],[25,-25,9.416446],[30,-25,9.476767],[35,-25,9.525334],[40,-25,9.562149],[45,-25,9.58721],[-35,-20,7.73401],[-30,-20,7.93537],[-25,-20,8.124977],[-20,-20,8.302832],[-15,-20,8.468932],[-10,-20,8.62328],[-5,-20,8.765873],[0,-20,8.896714],[5,-20,9.015801],[10,-20,9.123136],[15,-20,9.218717],[20,-20,9.302544],[25,-20,9.374619],[30,-20,9.434939],[35,-20,9.483507],[40,-20,9.520321],[45,-20,9.545382],[-35,-15,7.692183],[-30,-15,7.893543],[-25,-15,8.08315],[-20,-15,8.261003],[-15,-15,8.427104],[-10,-15,8.581451],[-5,-15,8.724046],[0,-15,8.854886],[5,-15,8.973974],[10,-15,9.081308],[15,-15,9.176889],[20,-15,9.260716],[25,-15,9.33279],[30,-15,9.393112],[35,-15,9.44168],[40,-15,9.478494],[45,-15,9.503555],[-35,-10,7.650355],[-30,-10,7.851716],[-25,-10,8.041323],[-20,-10,8.219176],[-15,-10,8.385277],[-10,-10,8.539624],[-5,-10,8.682219],[0,-10,8.813059],[5,-10,8.932146],[10,-10,9.039481],[15,-10,9.135061],[20,-10,9.218889],[25,-10,9.290963],[30,-10,9.351284],[35,-10,9.399852],[40,-10,9.436666],[45,-10,9.461728],[-35,-5,7.608528],[-30,-5,7.809888],[-25,-5,7.999495],[-20,-5,8.177349],[-15,-5,8.34345],[-10,-5,8.497797],[-5,-5,8.640391],[0,-5,8.771232],[5,-5,8.890319],[10,-5,8.997653],[15,-5,9.093234],[20,-5,9.177062],[25,-5,9.249136],[30,-5,9.309457],[35,-5,9.358025],[40,-5,9.394839],[45,-5,9.4199],[-35,0,7.5667],[-30,0,7.768061],[-25,0,7.957668],[-20,0,8.135522],[-15,0,8.301622],[-10,0,8.45597],[-5,0,8.598563],[0,0,8.729404],[5,0,8.848492],[10,0,8.955826],[15,0,9.051407],[20,0,9.135235],[25,0,9.207309],[30,0,9.26763],[35,0,9.316197],[40,0,9.353012],[45,0,9.378073],[-35,5,7.524873],[-30,5,7.726233],[-25,5,7.915841],[-20,5,8.093695],[-15,5,8.259795],[-10,5,8.414143],[-5,5,8.556736],[0,5,8.687577],[5,5,8.806664],[10,5,8.913999],[15,5,9.00958],[20,5,9.093407],[25,5,9.165482],[30,5,9.225802],[35,5,9.27437],[40,5,9.311185],[45,5,9.336246],[-35,10,7.483046],[-30,10,7.684406],[-25,10,7.874013],[-20,10,8.051867],[-15,10,8.217967],[-10,10,8.372314],[-5,10,8.514909],[0,10,8.64575],[5,10,8.764837],[10,10,8.872171],[15,10,8.967752],[20,10,9.051579],[25,10,9.123654],[30,10,9.183975],[35,10,9.232543],[40,10,9.269357],[45,10,9.294418],[-35,15,7.441218],[-30,15,7.642579],[-25,15,7.832186],[-20,15,8.010039],[-15,15,8.17614],[-10,15,8.330487],[-5,15,8.473082],[0,15,8.603922],[5,15,8.72301],[10,15,8.830344],[15,15,8.925924],[20,15,9.009752],[25,15,9.081826],[30,15,9.142147],[35,15,9.190715],[40,15,9.22753],[45,15,9.252591],[-35,20,7.399391],[-30,20,7.600751],[-25,20,7.790358],[-20,20,7.968212],[-15,20,8.134313],[-10,20,8.28866],[-5,20,8.431254],[0,20,8.562095],[5,20,8.681182],[10,20,8.788516],[15,20,8.884097],[20,20,8.967925],[25,20,9.039999],[30,20,9.10032],[35,20,9.148888],[40,20,9.185702],[45,20,9.210763],[-35,25,7.357563],[-30,25,7.558924],[-25,25,7.748531],[-20,25,7.926385],[-15,25,8.092485],[-10,25,8.246833],[-5,25,8.389426],[0,25,8.520267],[5,25,8.639355],[10,25,8.746689],[15,25,8.84227],[20,25,8.926098],[25,25,8.998172],[30,25,9.058493],[35,25,9.10706],[40,25,9.143875],[45,25,9.168936],[-35,30,7.315736],[-30,30,7.517097],[-25,30,7.706704],[-20,30,7.884557],[-15,30,8.050658],[-10,30,8.205006],[-5,30,8.347599],[0,30,8.47844],[5,30,8.597528],[10,30,8.704862],[15,30,8.800443],[20,30,8.88427],[25,30,8.956345],[30,30,9.016665],[35,30,9.065233],[40,30,9.102048],[45,30,9.127109],[-35,35,7.273909],[-30,35,7.475269],[-25,35,7.664876],[-20,35,7.84273],[-15,35,8.008831],[-10,35,8.163177],[-5,35,8.305772],[0,35,8.436613],[5,35,8.5557],[10,35,8.663034],[15,35,8.758615],[20,35,8.842443],[25,35,8.914517],[30,35,8.974838],[35,35,9.023406],[40,35,9.06022],[45,35,9.085281],[-35,40,7.232081],[-30,40,7.433442],[-25,40,7.623049],[-20,40,7.800903],[-15,40,7.967003],[-10,40,8.12135],[-5,40,8.263945],[0,40,8.394785],[5,40,8.513873],[10,40,8.621207],[15,40,8.716787],[20,40,8.800615],[25,40,8.872689],[30,40,8.933011],[35,40,8.981578],[40,40,9.018393],[45,40,9.043454],[-35,45,7.190254],[-30,45,7.391614],[-25,45,7.581222],[-20,45,7.759075],[-15,45,7.925176],[-10,45,8.079523],[-5,45,8.222117],[0,45,8.352958],[5,45,8.472045],[10,45,8.579379],[15,45,8.67496],[20,45,8.758788],[25,45,8.830862],[30,45,8.891183],[35,45,8.939751],[40,45,8.976565],[45,45,9.001626],[-35,50,7.148427],[-30,50,7.349787],[-25,50,7.539394],[-20,50,7.717248],[-15,50,7.883348],[-10,50,8.037696],[-5,50,8.18029],[0,50,8.311131],[5,50,8.430218],[10,50,8.537552],[15,50,8.633133],[20,50,8.716961],[25,50,8.789035],[30,50,8.849356],[35,50,8.897923],[40,50,8.934738],[45,50,8.959799],[-35,55,7.106599],[-30,55,7.30796],[-25,55,7.497567],[-20,55,7.67542],[-15,55,7.841521],[-10,55,7.995868],[-5,55,8.138462],[0,55,8.269303],[5,55,8.388391],[10,55,8.495725],[15,55,8.591306],[20,55,8.675133],[25,55,8.747208],[30,55,8.807528],[35,55,8.856096],[40,55,8.892911],[45,55,8.917972],[-35,60,7.064772],[-30,60,7.266132],[-25,60,7.455739],[-20,60,7.633593],[-15,60,7.799694],[-10,60,7.954041],[-5,60,8.096635],[0,60,8.227476],[5,60,8.346563],[10,60,8.453897],[15,60,8.549479],[20,60,8.633306],[25,60,8.70538],[30,60,8.765701],[35,60,8.814269],[40,60,8.851083],[45,60,8.876144],[-35,65,7.022944],[-30,65,7.224305],[-25,65,7.413912],[-20,65,7.591766],[-15,65,7.757866],[-10,65,7.912214],[-5,65,8.054808],[0,65,8.185648],[5,65,8.304736],[10,65,8.41207],[15,65,8.50765],[20,65,8.591478],[25,65,8.663552],[30,65,8.723874],[35,65,8.772442],[40,65,8.809256],[45,65,8.834317],[-35,70,6.981117],[-30,70,7.182477],[-25,70,7.372085],[-20,70,7.549938],[-15,70,7.716039],[-10,70,7.870386],[-5,70,8.01298],[0,70,8.143821],[5,70,8.262908],[10,70,8.370242],[15,70,8.465823],[20,70,8.549651],[25,70,8.621725],[30,70,8.682046],[35,70,8.730614],[40,70,8.767428],[45,70,8.79249]],"normals":[[-0.04023807,0.008358421,0.9991552],[-0.03906547,0.00835881,0.9992017],[-0.03672003,0.00835953,0.9992907],[-0.03437393,0.008360204,0.9993741],[-0.03202719,0.008360857,0.9994521],[-0.02967992,0.008361463,0.9995245],[-0.02733225,0.008362022,0.9995915],[-0.02498414,0.008362536,0.9996529],[-0.02263551,0.008363005,0.9997089],[-0.02028661,0.008363427,0.9997593],[-0.01793727,0.008363802,0.9998042],[-0.01558773,0.008364132,0.9998436],[-0.01323793,0.008364415,0.9998775],[-0.01088782,0.008364651,0.9999058],[-0.008537578,0.008364891,0.9999287],[-0.006187191,0.008365083,0.9999459],[-0.005011988,0.008365138,0.9999525],[-0.04023805,0.008358398,0.9991552],[-0.03906548,0.008358788,0.9992018],[-0.03672002,0.008359543,0.9992907],[-0.03437391,0.008360277,0.9993742],[-0.03202719,0.008360952,0.9994521],[-0.02967995,0.008361534,0.9995245],[-0.02733225,0.00836207,0.9995914],[-0.02498414,0.008362584,0.9996529],[-0.02263553,0.008363028,0.9997089],[-0.0202866,0.008363426,0.9997593],[-0.01793727,0.008363801,0.9998041],[-0.01558771,0.008364156,0.9998436],[-0.01323793,0.008364462,0.9998775],[-0.01088785,0.008364676,0.9999058],[-0.008537553,0.008364866,0.9999287],[-0.006187191,0.008365035,0.9999459],[-0.005012036,0.00836509,0.9999525],[-0.04023805,0.008358398,0.9991552],[-0.03906549,0.008358776,0.9992018],[-0.03671998,0.008359531,0.9992907],[-0.03437386,0.008360276,0.9993742],[-0.03202719,0.008360952,0.9994521],[-0.02968,0.008361535,0.9995245],[-0.02733225,0.008362071,0.9995914],[-0.02498411,0.008362608,0.9996529],[-0.02263558,0.008363076,0.9997089],[-0.02028661,0.008363474,0.9997593],[-0.01793727,0.008363849,0.9998041],[-0.01558768,0.008364179,0.9998436],[-0.01323791,0.008364487,0.9998775],[-0.01088787,0.008364747,0.9999058],[-0.008537553,0.008364914,0.9999287],[-0.006187215,0.008365011,0.9999459],[-0.005012083,0.008365043,0.9999525],[-0.04023807,0.008358421,0.9991552],[-0.0390655,0.008358811,0.9992018],[-0.03671997,0.008359543,0.9992907],[-0.03437385,0.008360217,0.9993742],[-0.03202719,0.008360857,0.9994521],[-0.02968002,0.008361463,0.9995245],[-0.02733225,0.008362022,0.9995914],[-0.02498407,0.008362561,0.9996529],[-0.02263558,0.008363076,0.9997089],[-0.02028661,0.008363522,0.9997593],[-0.01793729,0.008363873,0.9998041],[-0.01558771,0.008364156,0.9998436],[-0.01323786,0.008364439,0.9998775],[-0.01088784,0.008364723,0.9999058],[-0.008537601,0.008364914,0.9999287],[-0.006187239,0.008365035,0.9999459],[-0.005012035,0.00836509,0.9999525],[-0.04023807,0.008358421,0.9991552],[-0.0390655,0.008358811,0.9992018],[-0.03671999,0.008359543,0.9992907],[-0.03437388,0.008360217,0.9993742],[-0.03202719,0.008360857,0.9994521],[-0.02967999,0.008361487,0.9995245],[-0.02733225,0.00836207,0.9995914],[-0.02498407,0.008362561,0.9996529],[-0.02263553,0.008363029,0.9997089],[-0.0202866,0.008363473,0.9997593],[-0.01793734,0.008363825,0.9998041],[-0.01558773,0.008364132,0.9998436],[-0.01323784,0.008364415,0.9998775],[-0.01088782,0.008364651,0.9999058],[-0.008537625,0.008364842,0.9999287],[-0.006187215,0.008365011,0.9999459],[-0.00501194,0.00836509,0.9999525],[-0.04023807,0.008358421,0.9991552],[-0.03906551,0.008358799,0.9992018],[-0.03672002,0.008359519,0.9992907],[-0.03437389,0.008360205,0.9993742],[-0.03202719,0.008360857,0.9994521],[-0.02967995,0.008361487,0.9995245],[-0.02733225,0.008362071,0.9995914],[-0.02498412,0.008362561,0.9996529],[-0.02263551,0.008363005,0.9997089],[-0.0202866,0.008363426,0.9997593],[-0.01793734,0.008363825,0.9998041],[-0.01558773,0.008364179,0.9998436],[-0.01323786,0.008364439,0.9998775],[-0.01088782,0.008364651,0.9999058],[-0.008537625,0.008364842,0.9999287],[-0.006187192,0.008364987,0.9999459],[-0.005011892,0.008365043,0.9999525],[-0.04023805,0.008358398,0.9991552],[-0.03906551,0.0083588,0.9992018],[-0.03672002,0.008359566,0.9992907],[-0.03437388,0.008360289,0.9993742],[-0.03202719,0.008360952,0.9994521],[-0.02967995,0.008361534,0.9995245],[-0.02733228,0.008362046,0.9995914],[-0.02498414,0.008362536,0.9996529],[-0.02263551,0.008363005,0.9997089],[-0.0202866,0.008363426,0.9997593],[-0.01793729,0.008363824,0.9998041],[-0.01558773,0.008364179,0.9998436],[-0.01323791,0.008364439,0.9998775],[-0.01088782,0.008364651,0.9999058],[-0.008537601,0.008364866,0.9999287],[-0.006187192,0.008365035,0.9999459],[-0.00501194,0.00836509,0.9999525],[-0.04023805,0.008358398,0.9991552],[-0.0390655,0.008358787,0.9992018],[-0.03671998,0.008359555,0.9992907],[-0.03437385,0.008360288,0.9993742],[-0.03202719,0.008360952,0.9994521],[-0.02968,0.008361535,0.9995245],[-0.0273323,0.00836207,0.9995914],[-0.02498414,0.008362584,0.9996529],[-0.02263553,0.008363028,0.9997089],[-0.02028658,0.008363449,0.9997593],[-0.01793727,0.008363849,0.9998041],[-0.01558773,0.008364179,0.9998436],[-0.01323791,0.008364487,0.9998775],[-0.01088782,0.008364747,0.9999058],[-0.008537577,0.008364938,0.9999287],[-0.006187215,0.008365059,0.9999459],[-0.005012036,0.00836509,0.9999525],[-0.04023807,0.008358421,0.9991552],[-0.0390655,0.008358811,0.9992018],[-0.03671997,0.008359543,0.9992907],[-0.03437385,0.008360217,0.9993742],[-0.03202719,0.008360857,0.9994521],[-0.02968002,0.008361463,0.9995245],[-0.02733228,0.008362046,0.9995914],[-0.02498411,0.008362608,0.9996529],[-0.02263556,0.0083631,0.9997089],[-0.02028655,0.008363521,0.9997593],[-0.01793729,0.008363873,0.9998041],[-0.01558773,0.008364179,0.9998436],[-0.01323786,0.008364487,0.9998775],[-0.01088782,0.008364747,0.9999058],[-0.008537601,0.008364914,0.9999287],[-0.006187239,0.008365035,0.9999459],[-0.005012035,0.00836509,0.9999525],[-0.04023807,0.008358421,0.9991552],[-0.0390655,0.008358811,0.9992018],[-0.03671999,0.008359543,0.9992907],[-0.03437388,0.008360217,0.9993742],[-0.03202719,0.008360857,0.9994521],[-0.02967999,0.008361487,0.9995245],[-0.02733225,0.00836207,0.9995914],[-0.02498409,0.008362584,0.9996529],[-0.02263554,0.008363076,0.9997089],[-0.02028658,0.008363497,0.9997593],[-0.01793734,0.008363825,0.9998041],[-0.01558773,0.008364132,0.9998436],[-0.01323784,0.008364415,0.9998775],[-0.01088782,0.008364651,0.9999058],[-0.008537625,0.008364842,0.9999287],[-0.006187215,0.008365011,0.9999459],[-0.00501194,0.00836509,0.9999525],[-0.04023809,0.008358398,0.9991552],[-0.03906551,0.008358776,0.9992018],[-0.03671999,0.00835952,0.9992907],[-0.03437389,0.008360229,0.9993742],[-0.0320272,0.008360869,0.9994521],[-0.02967995,0.008361487,0.9995245],[-0.02733225,0.008362071,0.9995914],[-0.02498412,0.008362561,0.9996529],[-0.02263551,0.008363005,0.9997089],[-0.0202866,0.008363426,0.9997593],[-0.01793734,0.008363825,0.9998041],[-0.01558773,0.008364179,0.9998436],[-0.01323786,0.008364439,0.9998775],[-0.01088782,0.008364651,0.9999058],[-0.008537625,0.008364842,0.9999287],[-0.006187192,0.008364987,0.9999459],[-0.005011892,0.008365043,0.9999525],[-0.04023809,0.008358397,0.9991552],[-0.03906551,0.008358799,0.9992018],[-0.03671998,0.008359555,0.9992907],[-0.0343739,0.008360241,0.9993742],[-0.0320272,0.008360893,0.9994521],[-0.02967992,0.008361511,0.9995245],[-0.02733228,0.008362046,0.9995914],[-0.02498414,0.008362536,0.9996529],[-0.02263551,0.008363005,0.9997089],[-0.0202866,0.008363426,0.9997593],[-0.01793729,0.008363824,0.9998041],[-0.01558773,0.008364179,0.9998436],[-0.01323791,0.008364439,0.9998775],[-0.01088782,0.008364651,0.9999058],[-0.008537601,0.008364866,0.9999287],[-0.006187192,0.008365035,0.9999459],[-0.00501194,0.00836509,0.9999525],[-0.04023807,0.008358421,0.9991552],[-0.0390655,0.008358811,0.9992018],[-0.03671999,0.008359543,0.9992907],[-0.03437391,0.008360229,0.9993742],[-0.03202717,0.008360905,0.9994521],[-0.02967994,0.008361523,0.9995245],[-0.0273323,0.00836207,0.9995914],[-0.02498414,0.008362584,0.9996529],[-0.02263553,0.008363028,0.9997089],[-0.02028658,0.008363449,0.9997593],[-0.01793727,0.008363849,0.9998041],[-0.01558773,0.008364179,0.9998436],[-0.01323793,0.008364462,0.9998775],[-0.01088782,0.008364699,0.9999058],[-0.008537553,0.008364914,0.9999287],[-0.006187215,0.008365059,0.9999459],[-0.005012036,0.00836509,0.9999525],[-0.04023807,0.008358421,0.9991552],[-0.03906551,0.008358799,0.9992018],[-0.03672001,0.008359531,0.9992907],[-0.03437389,0.008360253,0.9993742],[-0.03202716,0.008360917,0.9994521],[-0.02967997,0.008361488,0.9995245],[-0.02733228,0.008362046,0.9995914],[-0.02498411,0.008362608,0.9996529],[-0.02263556,0.0083631,0.9997089],[-0.02028655,0.008363521,0.9997593],[-0.01793729,0.008363873,0.9998041],[-0.01558773,0.008364179,0.9998436],[-0.01323791,0.008364487,0.9998775],[-0.01088782,0.008364747,0.9999058],[-0.008537553,0.008364914,0.9999287],[-0.006187239,0.008365035,0.9999459],[-0.005012035,0.00836509,0.9999525],[-0.04023805,0.008358398,0.9991552],[-0.03906551,0.0083588,0.9992018],[-0.03672001,0.008359555,0.9992907],[-0.03437386,0.008360253,0.9993742],[-0.03202718,0.008360893,0.9994521],[-0.02968001,0.008361476,0.9995245],[-0.02733225,0.008362022,0.9995914],[-0.02498407,0.008362561,0.9996529],[-0.02263554,0.008363076,0.9997089],[-0.02028658,0.008363497,0.9997593],[-0.01793734,0.008363825,0.9998041],[-0.01558773,0.008364132,0.9998436],[-0.01323786,0.008364439,0.9998775],[-0.01088782,0.0083647,0.9999058],[-0.008537601,0.008364866,0.9999287],[-0.006187215,0.008365011,0.9999459],[-0.00501194,0.00836509,0.9999525],[-0.04023805,0.008358398,0.9991552],[-0.0390655,0.008358787,0.9992018],[-0.03671999,0.008359543,0.9992907],[-0.03437388,0.008360242,0.9993742],[-0.03202719,0.00836088,0.9994521],[-0.02968,0.008361511,0.9995245],[-0.02733226,0.008362082,0.9995914],[-0.02498407,0.008362561,0.9996529],[-0.02263551,0.008363005,0.9997089],[-0.0202866,0.008363426,0.9997593],[-0.01793734,0.008363825,0.9998041],[-0.01558773,0.008364179,0.9998436],[-0.01323786,0.008364439,0.9998775],[-0.01088782,0.008364651,0.9999058],[-0.008537625,0.008364842,0.9999287],[-0.006187192,0.008364987,0.9999459],[-0.005011892,0.008365043,0.9999525],[-0.04023807,0.008358421,0.9991552],[-0.0390655,0.008358811,0.9992018],[-0.03671998,0.008359555,0.9992907],[-0.03437389,0.008360253,0.9993742],[-0.03202719,0.008360905,0.9994521],[-0.02967996,0.008361523,0.9995245],[-0.02733229,0.008362082,0.9995914],[-0.02498412,0.008362561,0.9996529],[-0.02263551,0.008363005,0.9997089],[-0.0202866,0.008363426,0.9997593],[-0.01793729,0.008363824,0.9998041],[-0.01558773,0.008364179,0.9998436],[-0.01323791,0.008364439,0.9998775],[-0.01088782,0.008364651,0.9999058],[-0.008537601,0.008364866,0.9999287],[-0.006187192,0.008365035,0.9999459],[-0.00501194,0.00836509,0.9999525],[-0.04023807,0.008358421,0.9991552],[-0.0390655,0.008358811,0.9992018],[-0.03671999,0.008359543,0.9992907],[-0.03437389,0.008360229,0.9993742],[-0.03202719,0.00836088,0.9994521],[-0.02967996,0.008361476,0.9995245],[-0.02733228,0.008362046,0.9995914],[-0.02498414,0.008362584,0.9996529],[-0.02263553,0.008363028,0.9997089],[-0.02028658,0.008363449,0.9997593],[-0.01793727,0.008363849,0.9998041],[-0.01558773,0.008364179,0.9998436],[-0.01323793,0.008364462,0.9998775],[-0.01088785,0.008364676,0.9999058],[-0.008537553,0.008364866,0.9999287],[-0.006187191,0.008365035,0.9999459],[-0.005012036,0.00836509,0.9999525],[-0.04023809,0.008358398,0.9991552],[-0.03906551,0.008358776,0.9992018],[-0.03671999,0.00835952,0.9992907],[-0.03437388,0.008360241,0.9993742],[-0.03202719,0.008360905,0.9994521],[-0.02967998,0.008361499,0.9995245],[-0.02733224,0.008362059,0.9995914],[-0.02498411,0.008362608,0.9996529],[-0.02263556,0.0083631,0.9997089],[-0.02028655,0.008363521,0.9997593],[-0.01793729,0.008363873,0.9998041],[-0.01558773,0.008364179,0.9998436],[-0.01323791,0.008364487,0.9998775],[-0.01088787,0.008364747,0.9999058],[-0.008537553,0.008364914,0.9999287],[-0.006187215,0.008365011,0.9999459],[-0.005012083,0.008365043,0.9999525],[-0.04023812,0.008358373,0.9991552],[-0.03906552,0.008358764,0.9992018],[-0.03671998,0.00835953,0.9992906],[-0.03437386,0.008360275,0.9993742],[-0.03202719,0.008360951,0.9994521],[-0.02967999,0.008361534,0.9995245],[-0.02733223,0.008362046,0.9995915],[-0.02498409,0.008362584,0.9996529],[-0.02263556,0.008363147,0.9997088],[-0.02028656,0.008363569,0.9997593],[-0.01793732,0.008363849,0.9998041],[-0.01558773,0.008364132,0.9998436],[-0.01323789,0.008364462,0.9998775],[-0.01088787,0.008364795,0.9999058],[-0.008537577,0.008364986,0.9999286],[-0.006187239,0.008365035,0.9999459],[-0.005012083,0.008365043,0.9999525]],"colors":[[0,0,0,0.2980392]],"dim":[[17,20]],"centers":[[-32.5,-22.5,7.855604],[-27.5,-22.5,8.051086],[-22.5,-22.5,8.234818],[-17.5,-22.5,8.406796],[-12.5,-22.5,8.567019],[-7.5,-22.5,8.71549],[-2.5,-22.5,8.852207],[2.5,-22.5,8.977171],[7.5,-22.5,9.090382],[12.5,-22.5,9.191839],[17.5,-22.5,9.281544],[22.5,-22.5,9.359495],[27.5,-22.5,9.425693],[32.5,-22.5,9.480137],[37.5,-22.5,9.522827],[42.5,-22.5,9.553765],[-32.5,-17.5,7.813776],[-27.5,-17.5,8.00926],[-22.5,-17.5,8.192991],[-17.5,-17.5,8.364967],[-12.5,-17.5,8.525192],[-7.5,-17.5,8.673662],[-2.5,-17.5,8.810379],[2.5,-17.5,8.935345],[7.5,-17.5,9.048555],[12.5,-17.5,9.150013],[17.5,-17.5,9.239717],[22.5,-17.5,9.317667],[27.5,-17.5,9.383865],[32.5,-17.5,9.43831],[37.5,-17.5,9.481],[42.5,-17.5,9.511938],[-32.5,-12.5,7.771949],[-27.5,-12.5,7.967432],[-22.5,-12.5,8.151163],[-17.5,-12.5,8.32314],[-12.5,-12.5,8.483364],[-7.5,-12.5,8.631834],[-2.5,-12.5,8.768553],[2.5,-12.5,8.893517],[7.5,-12.5,9.006727],[12.5,-12.5,9.108185],[17.5,-12.5,9.197889],[22.5,-12.5,9.27584],[27.5,-12.5,9.342037],[32.5,-12.5,9.396482],[37.5,-12.5,9.439173],[42.5,-12.5,9.470111],[-32.5,-7.5,7.730122],[-27.5,-7.5,7.925605],[-22.5,-7.5,8.109335],[-17.5,-7.5,8.281313],[-12.5,-7.5,8.441536],[-7.5,-7.5,8.590008],[-2.5,-7.5,8.726726],[2.5,-7.5,8.851689],[7.5,-7.5,8.9649],[12.5,-7.5,9.066358],[17.5,-7.5,9.156062],[22.5,-7.5,9.234013],[27.5,-7.5,9.30021],[32.5,-7.5,9.354654],[37.5,-7.5,9.397346],[42.5,-7.5,9.428284],[-32.5,-2.5,7.688294],[-27.5,-2.5,7.883778],[-22.5,-2.5,8.067509],[-17.5,-2.5,8.239485],[-12.5,-2.5,8.39971],[-7.5,-2.5,8.548181],[-2.5,-2.5,8.684897],[2.5,-2.5,8.809862],[7.5,-2.5,8.923073],[12.5,-2.5,9.02453],[17.5,-2.5,9.114235],[22.5,-2.5,9.192185],[27.5,-2.5,9.258383],[32.5,-2.5,9.312827],[37.5,-2.5,9.355518],[42.5,-2.5,9.386456],[-32.5,2.5,7.646467],[-27.5,2.5,7.84195],[-22.5,2.5,8.025681],[-17.5,2.5,8.197659],[-12.5,2.5,8.357882],[-7.5,2.5,8.506353],[-2.5,2.5,8.64307],[2.5,2.5,8.768034],[7.5,2.5,8.881245],[12.5,2.5,8.982702],[17.5,2.5,9.072407],[22.5,2.5,9.150358],[27.5,2.5,9.216556],[32.5,2.5,9.271],[37.5,2.5,9.313691],[42.5,2.5,9.344629],[-32.5,7.5,7.60464],[-27.5,7.5,7.800123],[-22.5,7.5,7.983854],[-17.5,7.5,8.15583],[-12.5,7.5,8.316055],[-7.5,7.5,8.464525],[-2.5,7.5,8.601242],[2.5,7.5,8.726208],[7.5,7.5,8.839418],[12.5,7.5,8.940876],[17.5,7.5,9.03058],[22.5,7.5,9.10853],[27.5,7.5,9.174728],[32.5,7.5,9.229173],[37.5,7.5,9.271864],[42.5,7.5,9.302801],[-32.5,12.5,7.562812],[-27.5,12.5,7.758296],[-22.5,12.5,7.942026],[-17.5,12.5,8.114003],[-12.5,12.5,8.274227],[-7.5,12.5,8.422697],[-2.5,12.5,8.559416],[2.5,12.5,8.68438],[7.5,12.5,8.797591],[12.5,12.5,8.899048],[17.5,12.5,8.988752],[22.5,12.5,9.066703],[27.5,12.5,9.1329],[32.5,12.5,9.187346],[37.5,12.5,9.230036],[42.5,12.5,9.260974],[-32.5,17.5,7.520985],[-27.5,17.5,7.716468],[-22.5,17.5,7.900199],[-17.5,17.5,8.072176],[-12.5,17.5,8.232399],[-7.5,17.5,8.380871],[-2.5,17.5,8.517589],[2.5,17.5,8.642552],[7.5,17.5,8.755763],[12.5,17.5,8.857221],[17.5,17.5,8.946925],[22.5,17.5,9.024876],[27.5,17.5,9.091073],[32.5,17.5,9.145517],[37.5,17.5,9.188209],[42.5,17.5,9.219147],[-32.5,22.5,7.479157],[-27.5,22.5,7.674641],[-22.5,22.5,7.858372],[-17.5,22.5,8.030348],[-12.5,22.5,8.190573],[-7.5,22.5,8.339044],[-2.5,22.5,8.47576],[2.5,22.5,8.600725],[7.5,22.5,8.713936],[12.5,22.5,8.815393],[17.5,22.5,8.905098],[22.5,22.5,8.983048],[27.5,22.5,9.049246],[32.5,22.5,9.10369],[37.5,22.5,9.146381],[42.5,22.5,9.177319],[-32.5,27.5,7.43733],[-27.5,27.5,7.632813],[-22.5,27.5,7.816544],[-17.5,27.5,7.988522],[-12.5,27.5,8.148746],[-7.5,27.5,8.297216],[-2.5,27.5,8.433933],[2.5,27.5,8.558897],[7.5,27.5,8.672108],[12.5,27.5,8.773565],[17.5,27.5,8.86327],[22.5,27.5,8.941221],[27.5,27.5,9.007419],[32.5,27.5,9.061863],[37.5,27.5,9.104554],[42.5,27.5,9.135492],[-32.5,32.5,7.395503],[-27.5,32.5,7.590986],[-22.5,32.5,7.774717],[-17.5,32.5,7.946694],[-12.5,32.5,8.106918],[-7.5,32.5,8.255388],[-2.5,32.5,8.392105],[2.5,32.5,8.517071],[7.5,32.5,8.630281],[12.5,32.5,8.731739],[17.5,32.5,8.821443],[22.5,32.5,8.899393],[27.5,32.5,8.965591],[32.5,32.5,9.020036],[37.5,32.5,9.062727],[42.5,32.5,9.093664],[-32.5,37.5,7.353675],[-27.5,37.5,7.549159],[-22.5,37.5,7.73289],[-17.5,37.5,7.904867],[-12.5,37.5,8.06509],[-7.5,37.5,8.21356],[-2.5,37.5,8.350279],[2.5,37.5,8.475243],[7.5,37.5,8.588454],[12.5,37.5,8.689911],[17.5,37.5,8.779615],[22.5,37.5,8.857566],[27.5,37.5,8.923763],[32.5,37.5,8.978209],[37.5,37.5,9.020899],[42.5,37.5,9.051837],[-32.5,42.5,7.311848],[-27.5,42.5,7.507332],[-22.5,42.5,7.691062],[-17.5,42.5,7.863039],[-12.5,42.5,8.023264],[-7.5,42.5,8.171734],[-2.5,42.5,8.308452],[2.5,42.5,8.433415],[7.5,42.5,8.546626],[12.5,42.5,8.648084],[17.5,42.5,8.737788],[22.5,42.5,8.815739],[27.5,42.5,8.881936],[32.5,42.5,8.93638],[37.5,42.5,8.979072],[42.5,42.5,9.01001],[-32.5,47.5,7.27002],[-27.5,47.5,7.465504],[-22.5,47.5,7.649235],[-17.5,47.5,7.821212],[-12.5,47.5,7.981436],[-7.5,47.5,8.129907],[-2.5,47.5,8.266624],[2.5,47.5,8.391588],[7.5,47.5,8.504799],[12.5,47.5,8.606256],[17.5,47.5,8.695961],[22.5,47.5,8.773911],[27.5,47.5,8.840109],[32.5,47.5,8.894553],[37.5,47.5,8.937244],[42.5,47.5,8.968182],[-32.5,52.5,7.228193],[-27.5,52.5,7.423676],[-22.5,52.5,7.607407],[-17.5,52.5,7.779385],[-12.5,52.5,7.939609],[-7.5,52.5,8.088079],[-2.5,52.5,8.224796],[2.5,52.5,8.34976],[7.5,52.5,8.462971],[12.5,52.5,8.564428],[17.5,52.5,8.654133],[22.5,52.5,8.732084],[27.5,52.5,8.798282],[32.5,52.5,8.852726],[37.5,52.5,8.895417],[42.5,52.5,8.926355],[-32.5,57.5,7.186366],[-27.5,57.5,7.381849],[-22.5,57.5,7.56558],[-17.5,57.5,7.737557],[-12.5,57.5,7.897781],[-7.5,57.5,8.046251],[-2.5,57.5,8.182968],[2.5,57.5,8.307934],[7.5,57.5,8.421144],[12.5,57.5,8.522602],[17.5,57.5,8.612306],[22.5,57.5,8.690256],[27.5,57.5,8.756454],[32.5,57.5,8.810899],[37.5,57.5,8.85359],[42.5,57.5,8.884527],[-32.5,62.5,7.144538],[-27.5,62.5,7.340022],[-22.5,62.5,7.523753],[-17.5,62.5,7.69573],[-12.5,62.5,7.855954],[-7.5,62.5,8.004425],[-2.5,62.5,8.141142],[2.5,62.5,8.266106],[7.5,62.5,8.379317],[12.5,62.5,8.480774],[17.5,62.5,8.570478],[22.5,62.5,8.648429],[27.5,62.5,8.714626],[32.5,62.5,8.769072],[37.5,62.5,8.811762],[42.5,62.5,8.8427],[-32.5,67.5,7.102711],[-27.5,67.5,7.298195],[-22.5,67.5,7.481925],[-17.5,67.5,7.653902],[-12.5,67.5,7.814126],[-7.5,67.5,7.962597],[-2.5,67.5,8.099315],[2.5,67.5,8.224278],[7.5,67.5,8.337489],[12.5,67.5,8.438947],[17.5,67.5,8.528651],[22.5,67.5,8.606602],[27.5,67.5,8.672799],[32.5,67.5,8.727243],[37.5,67.5,8.769935],[42.5,67.5,8.800873]],"ignoreExtent":false,"flags":91},"5":{"id":5,"reuse":"testgldiv"},"6":{"id":6,"reuse":"testgldiv"},"50":{"id":50,"type":"bboxdeco","material":{"front":"lines","back":"lines"},"vertices":[[-20,"NA","NA"],[0,"NA","NA"],[20,"NA","NA"],[40,"NA","NA"],["NA",-20,"NA"],["NA",0,"NA"],["NA",20,"NA"],["NA",40,"NA"],["NA",60,"NA"],["NA","NA",7],["NA","NA",8],["NA","NA",9],["NA","NA",10]],"colors":[[0,0,0,1]],"draw_front":true,"newIds":[63,64,65,66,67,68,69]},"1":{"id":1,"type":"subscene","par3d":{"antialias":8,"FOV":30,"ignoreExtent":false,"listeners":1,"mouseMode":{"left":"trackball","right":"zoom","middle":"fov","wheel":"pull"},"observer":[0,0,314.501],"modelMatrix":[[0.9689531,0,0,-4.844765],[0,0.2460616,17.59741,-150.716],[0,-0.6760486,6.404935,-354.2165],[0,0,0,1]],"projMatrix":[[2.665751,0,0,0],[0,3.732051,0,0],[0,0,-3.863703,-1133.74],[0,0,-1,0]],"skipRedraw":false,"userMatrix":[[1,0,0,0],[0,0.3420201,0.9396926,0],[0,-0.9396926,0.3420201,0],[0,0,0,1]],"scale":[0.9689531,0.7194359,18.72678],"viewport":{"x":0,"y":0,"width":1,"height":1},"zoom":1,"bbox":[-35,45,-30.60419,70.15615,6.352662,10.22362],"windowRect":[0,45,672,525],"family":"sans","font":1,"cex":1,"useFreeType":true,"fontname":"/Library/Frameworks/R.framework/Versions/3.2/Resources/library/rgl/fonts/FreeSans.ttf","maxClipPlanes":6},"embeddings":{"viewport":"replace","projection":"replace","model":"replace"},"objects":[6,50,49,51,52,53,54,55,5,63,64,65,66,67,68,69],"subscenes":[],"flags":1275},"63":{"id":63,"type":"lines","material":{"lit":false,"front":"lines","back":"lines"},"vertices":[[-20,-32.11559,6.294598],[40,-32.11559,6.294598],[-20,-32.11559,6.294598],[-20,-34.71017,6.194921],[0,-32.11559,6.294598],[0,-34.71017,6.194921],[20,-32.11559,6.294598],[20,-34.71017,6.194921],[40,-32.11559,6.294598],[40,-34.71017,6.194921]],"colors":[[0,0,0,1]],"centers":[[10,-32.11559,6.294598],[-20,-33.41288,6.244759],[0,-33.41288,6.244759],[20,-33.41288,6.244759],[40,-33.41288,6.244759]],"ignoreExtent":true,"origId":50,"flags":128},"64":{"id":64,"type":"text","material":{"lit":false,"front":"lines","back":"lines"},"vertices":[[-20,-39.89933,5.995566],[0,-39.89933,5.995566],[20,-39.89933,5.995566],[40,-39.89933,5.995566]],"colors":[[0,0,0,1]],"texts":[["-20"],["0"],["20"],["40"]],"cex":[[1]],"adj":[[0.5,0.5]],"centers":[[-20,-39.89933,5.995566],[0,-39.89933,5.995566],[20,-39.89933,5.995566],[40,-39.89933,5.995566]],"family":[["sans"]],"font":[[1]],"ignoreExtent":true,"origId":50,"flags":40},"65":{"id":65,"type":"lines","material":{"lit":false,"front":"lines","back":"lines"},"vertices":[[-36.2,-20,6.294598],[-36.2,60,6.294598],[-36.2,-20,6.294598],[-38.26,-20,6.194921],[-36.2,0,6.294598],[-38.26,0,6.194921],[-36.2,20,6.294598],[-38.26,20,6.194921],[-36.2,40,6.294598],[-38.26,40,6.194921],[-36.2,60,6.294598],[-38.26,60,6.194921]],"colors":[[0,0,0,1]],"centers":[[-36.2,20,6.294598],[-37.23,-20,6.244759],[-37.23,0,6.244759],[-37.23,20,6.244759],[-37.23,40,6.244759],[-37.23,60,6.244759]],"ignoreExtent":true,"origId":50,"flags":128},"66":{"id":66,"type":"text","material":{"lit":false,"front":"lines","back":"lines"},"vertices":[[-42.38,-20,5.995566],[-42.38,0,5.995566],[-42.38,20,5.995566],[-42.38,40,5.995566],[-42.38,60,5.995566]],"colors":[[0,0,0,1]],"texts":[["-20"],["0"],["20"],["40"],["60"]],"cex":[[1]],"adj":[[0.5,0.5]],"centers":[[-42.38,-20,5.995566],[-42.38,0,5.995566],[-42.38,20,5.995566],[-42.38,40,5.995566],[-42.38,60,5.995566]],"family":[["sans"]],"font":[[1]],"ignoreExtent":true,"origId":50,"flags":40},"67":{"id":67,"type":"lines","material":{"lit":false,"front":"lines","back":"lines"},"vertices":[[-36.2,-32.11559,7],[-36.2,-32.11559,10],[-36.2,-32.11559,7],[-38.26,-34.71017,7],[-36.2,-32.11559,8],[-38.26,-34.71017,8],[-36.2,-32.11559,9],[-38.26,-34.71017,9],[-36.2,-32.11559,10],[-38.26,-34.71017,10]],"colors":[[0,0,0,1]],"centers":[[-36.2,-32.11559,8.5],[-37.23,-33.41288,7],[-37.23,-33.41288,8],[-37.23,-33.41288,9],[-37.23,-33.41288,10]],"ignoreExtent":true,"origId":50,"flags":128},"68":{"id":68,"type":"text","material":{"lit":false,"front":"lines","back":"lines"},"vertices":[[-42.38,-39.89933,7],[-42.38,-39.89933,8],[-42.38,-39.89933,9],[-42.38,-39.89933,10]],"colors":[[0,0,0,1]],"texts":[["7"],["8"],["9"],["10"]],"cex":[[1]],"adj":[[0.5,0.5]],"centers":[[-42.38,-39.89933,7],[-42.38,-39.89933,8],[-42.38,-39.89933,9],[-42.38,-39.89933,10]],"family":[["sans"]],"font":[[1]],"ignoreExtent":true,"origId":50,"flags":40},"69":{"id":69,"type":"lines","material":{"lit":false,"front":"lines","back":"lines"},"vertices":[[-36.2,-32.11559,6.294598],[-36.2,71.66756,6.294598],[-36.2,-32.11559,10.28169],[-36.2,71.66756,10.28169],[-36.2,-32.11559,6.294598],[-36.2,-32.11559,10.28169],[-36.2,71.66756,6.294598],[-36.2,71.66756,10.28169],[-36.2,-32.11559,6.294598],[46.2,-32.11559,6.294598],[-36.2,-32.11559,10.28169],[46.2,-32.11559,10.28169],[-36.2,71.66756,6.294598],[46.2,71.66756,6.294598],[-36.2,71.66756,10.28169],[46.2,71.66756,10.28169],[46.2,-32.11559,6.294598],[46.2,71.66756,6.294598],[46.2,-32.11559,10.28169],[46.2,71.66756,10.28169],[46.2,-32.11559,6.294598],[46.2,-32.11559,10.28169],[46.2,71.66756,6.294598],[46.2,71.66756,10.28169]],"colors":[[0,0,0,1]],"centers":[[-36.2,19.77598,6.294598],[-36.2,19.77598,10.28169],[-36.2,-32.11559,8.288142],[-36.2,71.66756,8.288142],[5,-32.11559,6.294598],[5,-32.11559,10.28169],[5,71.66756,6.294598],[5,71.66756,10.28169],[46.2,19.77598,6.294598],[46.2,19.77598,10.28169],[46.2,-32.11559,8.288142],[46.2,71.66756,8.288142]],"ignoreExtent":true,"origId":50,"flags":128}},"snapshot":"data:image/png;base64,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","width":673,"height":481,"sphereVerts":{"reuse":"testgldiv"}});
testgl3rgl.prefix = "testgl3";
</script>
<p id="testgl3debug">
You must enable Javascript to view this page properly.</p>
<script>testgl3rgl.start();</script>

Note now that this updated model yields a much better R-square measure of **0.7490565**, with all predictor p-values highly significant and improved F-Statistic value (**101.5**). The residuals plot also shows a randomly scattered plot indicating a relatively good fit given the transformations applied due to the non-linearity nature of the data.





In summary, we’ve seen a few different multiple linear regression models applied to the **Prestige** dataset. We tried an linear approach. We created a correlation matrix to understand how each variable was correlated. Subsequently, we transformed the variables to see the effect in the model. We’ve created three-dimensional plots to visualize the relationship of the variables and how the model was fitting the data in hand.



{% include advertisements.html %}


***

In next examples, we’ll explore some non-parametric approaches such as *K-Nearest Neighbour* and some regularization procedures that will allow a stronger fit and a potentially better interpretation. We’ll also start to dive into some Resampling methods such as *Cross-validation* and *Bootstrap* and later on we’ll approach some *Classification* problems.
