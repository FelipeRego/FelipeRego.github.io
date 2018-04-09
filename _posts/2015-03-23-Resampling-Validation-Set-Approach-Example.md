---
layout: post
title:  "Resampling with the Validation Set Approach - An Example in R"
date: 2015-03-23
excerpt: "Resampling is a technique that allows us to repeatedly draw samples from a set of observations and to refit a model on each sample in order to obtain additional information."
image: "/images/resampling1.jpg"
permalink: /blog/2015/03/23/Resampling-Validation-Set-Approach-Example
---




**Resampling** is a technique that allows us to repeatedly draw samples from a set of observations and to refit a model on each sample in order to obtain additional information. Resampling can be useful to estimate the variability of the model fit and to estimate the error rate of the model when applied in new previously unseen data. Resampling can also help in selecting a good level of model flexibility and can also be used to compare the performance (and error rates) of different predictive models.

There are many resampling techniques available. Some of the most popular ones include **Cross-Validation** techniques (**K-Fold**, **Leave-One-Out**, etc.), **Bootstrap** and **Validation Set** (training and testing set split).

In this example we’ll focus on one simple technique which will give us the basis for other cross-validation techniques in the future. We’re going to be working with the Validation Set Approach
Let’s start by loading the necessary packages. We’ll continue to use Prestige as our dataset of choice. It can be found in the car package ```library(car)```:


```r
# Load the packages.
library(car) # where our dataset Prestige resides.
library(rgl)
library(knitr)
library(scatterplot3d)
```

For illustration purposes, let’s re-create the same data structure we had before with the Prestige dataset:


```r
# Subset the data to capture income, education, women and prestige.
newdata = Prestige[,c(1:4)]

# Center our predictors.
education.c = scale(newdata$education, center=TRUE, scale=FALSE)
prestige.c = scale(newdata$prestige, center=TRUE, scale=FALSE)
women.c = scale(newdata$women, center=TRUE, scale=FALSE)

# Bind these new variables into newdata and display a summary.
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

The Prestige dataset is a data frame with 102 rows and 6 columns. Each row is an observation that relate to an occupation. The columns relate to predictors such as average years of education, percentage of women in the occupation, prestige of the occupation, etc. Note also that we centered and scaled our data frame and renamed it to **newdata**.

If you recall from out previous examples, we’ve been focused on creating both simple and multiple linear regression problems with more emphasis on inference without paying too much attention to the predictive quality of the models created. We were simply assessing accuracy and quality of fit on the data used to build these models.

But for any model to have a strong predictive power, we must measure its error rate on data that was not used to build them. When we derive models from a dataset, these models estimate their coefficients by minimizing the errors found exclusively on those data points alone.

If we go on to apply these same models on sets of completely new and previously unseen data points, the model generated may end up yielding large error rates simply because the patterns found in that original dataset may not be picked up in the new dataset.

Error measures that are calculated from the dataset used to build the model (**training dataset**) are called *training dataset error*. But, as mentioned, we usually want to use the model results to predict on completely new and previously unseen data points (*testing dataset*). The error rates estimated from these *held-out* data points are called *testing dataset error*.

The Validation Set Approach is a type of method that estimates a model error rate by holding out a subset of the data from the fitting process (creating a testing dataset). The model is then built using the other set of observations (the training dataset). Then the model result is applied on the testing dataset in which we can then calculate the error (testings dataset error). In summary, this general idea allows for the model to not **overfit**.

Let’s illustrate with an example from our newdata dataframe:


```r
set.seed(7)
# Create a training and a test dataset with 50% of the data in the training set.
trainRows = sample(1:nrow(newdata),0.5*nrow(newdata))
length(trainRows)
```

```
## [1] 51
```

Note from the above code that we created a training and a testing dataset. We’ll be using the training dataset to build our predictive model and then applying it on the testing dataset to assess the error rate. Since our target variable (income from the Prestige dataset) is of a continuous nature, we’ll continue to apply linear regression models for now. Consequently, our error rate will be given by a continuous variable error measure such as the **Mean Squared Error** or the **Root Mean Squared Error**.

Before we fit a linear model in our dataset, let’s examine how our predictors are related to our target variable income:


```r
# Plot matrix of all variables.
plot(newdata[,c(1:4)], pch=16, col="blue", main="Matrix Scatterplot of Income, Education, Women and Prestige")
```

<span class="image fit"><img src="{{ "/images/ResamplingValidationSet_files/figure-html/unnamed-chunk-4-1.png" | absolute_url }}" alt="" /></span>

Remember from our previous examples, we decided to manually exclude education from our multiple regression analysis as it was overfitting the data (note from the plot above how similar education’s pattern is relative to prestige’s pattern) and was not adding a significant p-value when prestige was also present in the data. So we went on to generate a few models containing these two predictors only.

For simplicity sake, let’s fit a random multiple linear model with both prestige and women as predictors:


```r
# Fit a model in the training data with income as our response and the predictors as prestige and women.
mod1 = lm(income ~ prestige.c + women.c , data=newdata[c(trainRows), ])
summary(mod1)
```

```
## 
## Call:
## lm(formula = income ~ prestige.c + women.c, data = newdata[c(trainRows), 
##     ])
## 
## Residuals:
##     Min      1Q  Median      3Q     Max 
## -3926.3 -1001.3  -281.7   968.6 10519.8 
## 
## Coefficients:
##             Estimate Std. Error t value Pr(>|t|)    
## (Intercept)  6806.90     324.45  20.980  < 2e-16 ***
## prestige.c    175.50      18.41   9.534 1.18e-12 ***
## women.c       -48.70      11.11  -4.382 6.36e-05 ***
## ---
## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
## 
## Residual standard error: 2314 on 48 degrees of freedom
## Multiple R-squared:  0.711,  Adjusted R-squared:  0.699 
## F-statistic: 59.06 on 2 and 48 DF,  p-value: 1.148e-13
```

```r
# Plot residuals.
par(mfrow=c(2,2))
plot(mod1, pch=16)
```

<span class="image fit"><img src="{{ "/images/ResamplingValidationSet_files/figure-html/unnamed-chunk-5-1.png" | absolute_url }}" alt="" /></span>

We fit a linear model using lm function. Observe we subset our data for the model to learn only from the training set. The resulting model shows significant p-values for the predictors and the model overall. Both our F-statistic our Adjusted R-squared are not great. Our Residual Standard Error is relatively high. The residual plots also highlight the present of some outliers.

Let’s inspect the model fit on the observed data in an interactive 3D plot (you can click and drag the graph around to change the viewing angle):




```r
newdat <- expand.grid(prestige.c=seq(-35,45,by=5),women.c=seq(-25,70,by=5))
newdat$pp <- predict(mod1,newdata=newdat)
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
{"material":{"color":"#000000","alpha":1,"lit":true,"ambient":"#000000","specular":"#FFFFFF","emission":"#000000","shininess":50,"smooth":true,"front":"filled","back":"filled","size":3,"lwd":1,"fog":false,"point_antialias":false,"line_antialias":false,"texture":null,"textype":"rgb","texmipmap":false,"texminfilter":"linear","texmagfilter":"linear","texenvmap":false,"depth_mask":true,"depth_test":"less"},"rootSubscene":1,"objects":{"7":{"id":7,"type":"spheres","material":{},"vertices":[[21.96667,-17.81902,12351],[22.26667,-24.95902,25879],[16.56667,-13.27902,9271],[9.966666,-19.86902,8865],[26.66667,-17.29902,8403],[30.76667,-23.84902,11030],[25.76667,-3.32902,8258],[31.26667,-26.28902,14163],[26.26667,-27.94902,11377],[21.96667,-28.03902,11023],[15.16667,-27.06902,5902],[13.16667,-21.14902,7059],[6.966667,-13.64902,8425],[15.36667,28.33098,8049],[28.06667,19.30098,7405],[8.266666,25.79098,6336],[35.46667,-23.84902,19263],[11.26667,48.12098,6112],[11.46667,5.91098,9593],[25.96667,-24.83902,4686],[37.76667,-9.38902,12480],[12.76667,54.80098,5648],[19.26667,17.82098,8034],[40.36666,-18.41902,25308],[19.86667,-24.65902,14558],[21.56667,-22.06902,17498],[17.86667,67.14098,4614],[-11.93333,47.16098,3485],[25.26667,53.68098,5092],[22.46667,-4.26902,10432],[20.66667,47.06098,5180],[10.36667,-7.949019,6197],[10.76667,-17.82902,7562],[7.266667,-20.84902,8206],[-0.8333333,68.53098,4036],[-4.933333,66.99098,3148],[2.566667,39.26098,4348],[-4.533333,62.78098,2448],[0.8666667,46.94098,4330],[-15.93333,-17.60902,4761],[-14.13333,54.21098,3016],[-8.133333,63.88098,2901],[-10.73333,-21.35902,5511],[-9.633333,23.29098,3739],[-8.733334,67.16098,3161],[-17.43333,18.08098,4741],[4.266667,27.12098,5052],[-11.13333,10.19098,6259],[-11.23333,34.25098,4075],[-5.333333,-11.93902,7482],[-6.633333,-25.81902,8780],[-20.33333,38.84098,2594],[-32.03333,-21.97902,918],[-23.53333,-25.28902,2370],[0.4666667,-15.88902,8131],[0.2666667,-4.53902,6992],[4.266667,-5.09902,7956],[-3.333333,-28.97902,8895],[4.766667,-27.32902,8891],[-17.13333,23.02098,3116],[-26.63333,-13.46902,3930],[8.066667,-22.96902,7869],[-20.93333,67.55098,611],[-26.03333,40.33098,3000],[-29.53333,4.590981,3472],[-26.73333,1.10098,3582],[-2.733333,-25.37902,3643],[-25.33333,-1.22902,1656],[-11.53333,-28.97902,6860],[-7.933333,4.320981,4199],[-21.63333,-11.71902,5134],[-12.03333,-11.71902,5134],[-23.63333,43.26098,1890],[-13.53333,2.38098,4443],[-18.03333,10.50098,3485],[-4.333333,-27.47902,8043],[-2.633333,-24.69902,6686],[-10.93333,-26.67902,6565],[-5.033333,-23.80902,6477],[-10.93333,-15.35902,5811],[-3.133333,-23.19902,6573],[3.966667,45.56098,3942],[-9.633333,-26.05902,5449],[-18.63333,61.69098,2847],[-8.733334,-28.16902,5795],[3.466667,-28.19902,7716],[-19.53333,-28.97902,4696],[-5.933333,-27.63902,8316],[3.366667,-27.98902,7147],[4.266667,-28.32902,8880],[-7.933333,-28.41902,5299],[-10.63333,-28.45902,5959],[-16.93333,-26.51902,4549],[-3.933333,-28.36902,6928],[-20.33333,-27.88902,3910],[19.26667,-28.39902,14032],[2.066667,-28.97902,8845],[-10.93333,-19.50902,5562],[-21.73333,-25.38902,4224],[-20.73333,-28.97902,4753],[-4.633333,-15.39902,6462],[-11.63333,41.89098,3617]],"colors":[[0,0,1,1]],"radii":[[243.1443]],"centers":[[21.96667,-17.81902,12351],[22.26667,-24.95902,25879],[16.56667,-13.27902,9271],[9.966666,-19.86902,8865],[26.66667,-17.29902,8403],[30.76667,-23.84902,11030],[25.76667,-3.32902,8258],[31.26667,-26.28902,14163],[26.26667,-27.94902,11377],[21.96667,-28.03902,11023],[15.16667,-27.06902,5902],[13.16667,-21.14902,7059],[6.966667,-13.64902,8425],[15.36667,28.33098,8049],[28.06667,19.30098,7405],[8.266666,25.79098,6336],[35.46667,-23.84902,19263],[11.26667,48.12098,6112],[11.46667,5.91098,9593],[25.96667,-24.83902,4686],[37.76667,-9.38902,12480],[12.76667,54.80098,5648],[19.26667,17.82098,8034],[40.36666,-18.41902,25308],[19.86667,-24.65902,14558],[21.56667,-22.06902,17498],[17.86667,67.14098,4614],[-11.93333,47.16098,3485],[25.26667,53.68098,5092],[22.46667,-4.26902,10432],[20.66667,47.06098,5180],[10.36667,-7.949019,6197],[10.76667,-17.82902,7562],[7.266667,-20.84902,8206],[-0.8333333,68.53098,4036],[-4.933333,66.99098,3148],[2.566667,39.26098,4348],[-4.533333,62.78098,2448],[0.8666667,46.94098,4330],[-15.93333,-17.60902,4761],[-14.13333,54.21098,3016],[-8.133333,63.88098,2901],[-10.73333,-21.35902,5511],[-9.633333,23.29098,3739],[-8.733334,67.16098,3161],[-17.43333,18.08098,4741],[4.266667,27.12098,5052],[-11.13333,10.19098,6259],[-11.23333,34.25098,4075],[-5.333333,-11.93902,7482],[-6.633333,-25.81902,8780],[-20.33333,38.84098,2594],[-32.03333,-21.97902,918],[-23.53333,-25.28902,2370],[0.4666667,-15.88902,8131],[0.2666667,-4.53902,6992],[4.266667,-5.09902,7956],[-3.333333,-28.97902,8895],[4.766667,-27.32902,8891],[-17.13333,23.02098,3116],[-26.63333,-13.46902,3930],[8.066667,-22.96902,7869],[-20.93333,67.55098,611],[-26.03333,40.33098,3000],[-29.53333,4.590981,3472],[-26.73333,1.10098,3582],[-2.733333,-25.37902,3643],[-25.33333,-1.22902,1656],[-11.53333,-28.97902,6860],[-7.933333,4.320981,4199],[-21.63333,-11.71902,5134],[-12.03333,-11.71902,5134],[-23.63333,43.26098,1890],[-13.53333,2.38098,4443],[-18.03333,10.50098,3485],[-4.333333,-27.47902,8043],[-2.633333,-24.69902,6686],[-10.93333,-26.67902,6565],[-5.033333,-23.80902,6477],[-10.93333,-15.35902,5811],[-3.133333,-23.19902,6573],[3.966667,45.56098,3942],[-9.633333,-26.05902,5449],[-18.63333,61.69098,2847],[-8.733334,-28.16902,5795],[3.466667,-28.19902,7716],[-19.53333,-28.97902,4696],[-5.933333,-27.63902,8316],[3.366667,-27.98902,7147],[4.266667,-28.32902,8880],[-7.933333,-28.41902,5299],[-10.63333,-28.45902,5959],[-16.93333,-26.51902,4549],[-3.933333,-28.36902,6928],[-20.33333,-27.88902,3910],[19.26667,-28.39902,14032],[2.066667,-28.97902,8845],[-10.93333,-19.50902,5562],[-21.73333,-25.38902,4224],[-20.73333,-28.97902,4753],[-4.633333,-15.39902,6462],[-11.63333,41.89098,3617]],"ignoreExtent":false,"flags":3},"9":{"id":9,"type":"text","material":{"lit":false},"vertices":[[4.166666,87.23504,30725.82]],"colors":[[0,0,0,1]],"texts":[["3D Linear Model Fit"]],"cex":[[1]],"adj":[[0.5,0.5]],"centers":[[4.166666,87.23504,30725.82]],"family":[["sans"]],"font":[[1]],"ignoreExtent":true,"flags":40},"10":{"id":10,"type":"text","material":{"lit":false},"vertices":[[4.166666,-47.68307,-4235.823]],"colors":[[0,0,0,1]],"texts":[["prestige.c"]],"cex":[[1]],"adj":[[0.5,0.5]],"centers":[[4.166666,-47.68307,-4235.823]],"family":[["sans"]],"font":[[1]],"ignoreExtent":true,"flags":40},"11":{"id":11,"type":"text","material":{"lit":false},"vertices":[[-45.92087,19.77598,-4235.823]],"colors":[[0,0,0,1]],"texts":[["women.c"]],"cex":[[1]],"adj":[[0.5,0.5]],"centers":[[-45.92087,19.77598,-4235.823]],"family":[["sans"]],"font":[[1]],"ignoreExtent":true,"flags":40},"12":{"id":12,"type":"text","material":{"lit":false},"vertices":[[-45.92087,-47.68307,13245]],"colors":[[0,0,0,1]],"texts":[["income"]],"cex":[[1]],"adj":[[0.5,0.5]],"centers":[[-45.92087,-47.68307,13245]],"family":[["sans"]],"font":[[1]],"ignoreExtent":true,"flags":40},"13":{"id":13,"type":"surface","material":{"alpha":0.2980392,"front":"lines","back":"lines"},"vertices":[[-35,-25,1881.918],[-30,-25,2759.41],[-25,-25,3636.902],[-20,-25,4514.394],[-15,-25,5391.886],[-10,-25,6269.378],[-5,-25,7146.87],[0,-25,8024.362],[5,-25,8901.854],[10,-25,9779.347],[15,-25,10656.84],[20,-25,11534.33],[25,-25,12411.82],[30,-25,13289.32],[35,-25,14166.81],[40,-25,15044.3],[45,-25,15921.79],[-35,-20,1638.425],[-30,-20,2515.917],[-25,-20,3393.41],[-20,-20,4270.902],[-15,-20,5148.394],[-10,-20,6025.886],[-5,-20,6903.378],[0,-20,7780.87],[5,-20,8658.362],[10,-20,9535.854],[15,-20,10413.35],[20,-20,11290.84],[25,-20,12168.33],[30,-20,13045.82],[35,-20,13923.31],[40,-20,14800.81],[45,-20,15678.3],[-35,-15,1394.933],[-30,-15,2272.425],[-25,-15,3149.917],[-20,-15,4027.409],[-15,-15,4904.901],[-10,-15,5782.394],[-5,-15,6659.886],[0,-15,7537.378],[5,-15,8414.87],[10,-15,9292.362],[15,-15,10169.85],[20,-15,11047.35],[25,-15,11924.84],[30,-15,12802.33],[35,-15,13679.82],[40,-15,14557.31],[45,-15,15434.81],[-35,-10,1151.441],[-30,-10,2028.933],[-25,-10,2906.425],[-20,-10,3783.917],[-15,-10,4661.409],[-10,-10,5538.901],[-5,-10,6416.394],[0,-10,7293.886],[5,-10,8171.377],[10,-10,9048.87],[15,-10,9926.362],[20,-10,10803.85],[25,-10,11681.35],[30,-10,12558.84],[35,-10,13436.33],[40,-10,14313.82],[45,-10,15191.31],[-35,-5,907.9486],[-30,-5,1785.441],[-25,-5,2662.933],[-20,-5,3540.425],[-15,-5,4417.917],[-10,-5,5295.409],[-5,-5,6172.901],[0,-5,7050.393],[5,-5,7927.885],[10,-5,8805.378],[15,-5,9682.869],[20,-5,10560.36],[25,-5,11437.85],[30,-5,12315.35],[35,-5,13192.84],[40,-5,14070.33],[45,-5,14947.82],[-35,0,664.4564],[-30,0,1541.948],[-25,0,2419.44],[-20,0,3296.933],[-15,0,4174.425],[-10,0,5051.917],[-5,0,5929.409],[0,0,6806.901],[5,0,7684.393],[10,0,8561.885],[15,0,9439.377],[20,0,10316.87],[25,0,11194.36],[30,0,12071.85],[35,0,12949.35],[40,0,13826.84],[45,0,14704.33],[-35,5,420.9641],[-30,5,1298.456],[-25,5,2175.948],[-20,5,3053.44],[-15,5,3930.932],[-10,5,4808.425],[-5,5,5685.917],[0,5,6563.409],[5,5,7440.901],[10,5,8318.393],[15,5,9195.885],[20,5,10073.38],[25,5,10950.87],[30,5,11828.36],[35,5,12705.85],[40,5,13583.35],[45,5,14460.84],[-35,10,177.4718],[-30,10,1054.964],[-25,10,1932.456],[-20,10,2809.948],[-15,10,3687.44],[-10,10,4564.932],[-5,10,5442.424],[0,10,6319.917],[5,10,7197.409],[10,10,8074.901],[15,10,8952.393],[20,10,9829.885],[25,10,10707.38],[30,10,11584.87],[35,10,12462.36],[40,10,13339.85],[45,10,14217.35],[-35,15,-66.02045],[-30,15,811.4716],[-25,15,1688.964],[-20,15,2566.456],[-15,15,3443.948],[-10,15,4321.44],[-5,15,5198.932],[0,15,6076.424],[5,15,6953.917],[10,15,7831.409],[15,15,8708.9],[20,15,9586.393],[25,15,10463.88],[30,15,11341.38],[35,15,12218.87],[40,15,13096.36],[45,15,13973.85],[-35,20,-309.5127],[-30,20,567.9794],[-25,20,1445.471],[-20,20,2322.964],[-15,20,3200.456],[-10,20,4077.948],[-5,20,4955.44],[0,20,5832.932],[5,20,6710.424],[10,20,7587.916],[15,20,8465.408],[20,20,9342.9],[25,20,10220.39],[30,20,11097.88],[35,20,11975.38],[40,20,12852.87],[45,20,13730.36],[-35,25,-553.005],[-30,25,324.4871],[-25,25,1201.979],[-20,25,2079.471],[-15,25,2956.963],[-10,25,3834.456],[-5,25,4711.948],[0,25,5589.44],[5,25,6466.932],[10,25,7344.424],[15,25,8221.916],[20,25,9099.408],[25,25,9976.9],[30,25,10854.39],[35,25,11731.88],[40,25,12609.38],[45,25,13486.87],[-35,30,-796.4973],[-30,30,80.99486],[-25,30,958.4869],[-20,30,1835.979],[-15,30,2713.471],[-10,30,3590.963],[-5,30,4468.456],[0,30,5345.947],[5,30,6223.439],[10,30,7100.932],[15,30,7978.424],[20,30,8855.916],[25,30,9733.408],[30,30,10610.9],[35,30,11488.39],[40,30,12365.88],[45,30,13243.38],[-35,35,-1039.99],[-30,35,-162.4974],[-25,35,714.9947],[-20,35,1592.487],[-15,35,2469.979],[-10,35,3347.471],[-5,35,4224.963],[0,35,5102.455],[5,35,5979.947],[10,35,6857.439],[15,35,7734.932],[20,35,8612.424],[25,35,9489.916],[30,35,10367.41],[35,35,11244.9],[40,35,12122.39],[45,35,12999.88],[-35,40,-1283.482],[-30,40,-405.9897],[-25,40,471.5024],[-20,40,1348.995],[-15,40,2226.487],[-10,40,3103.979],[-5,40,3981.471],[0,40,4858.963],[5,40,5736.455],[10,40,6613.947],[15,40,7491.439],[20,40,8368.932],[25,40,9246.424],[30,40,10123.92],[35,40,11001.41],[40,40,11878.9],[45,40,12756.39],[-35,45,-1526.974],[-30,45,-649.4819],[-25,45,228.0102],[-20,45,1105.502],[-15,45,1982.994],[-10,45,2860.487],[-5,45,3737.979],[0,45,4615.471],[5,45,5492.963],[10,45,6370.455],[15,45,7247.947],[20,45,8125.439],[25,45,9002.932],[30,45,9880.423],[35,45,10757.92],[40,45,11635.41],[45,45,12512.9],[-35,50,-1770.466],[-30,50,-892.9742],[-25,50,-15.48208],[-20,50,862.01],[-15,50,1739.502],[-10,50,2616.994],[-5,50,3494.486],[0,50,4371.979],[5,50,5249.471],[10,50,6126.962],[15,50,7004.455],[20,50,7881.947],[25,50,8759.438],[30,50,9636.931],[35,50,10514.42],[40,50,11391.92],[45,50,12269.41],[-35,55,-2013.958],[-30,55,-1136.466],[-25,55,-258.9743],[-20,55,618.5178],[-15,55,1496.01],[-10,55,2373.502],[-5,55,3250.994],[0,55,4128.486],[5,55,5005.978],[10,55,5883.47],[15,55,6760.962],[20,55,7638.455],[25,55,8515.946],[30,55,9393.438],[35,55,10270.93],[40,55,11148.42],[45,55,12025.92],[-35,60,-2257.451],[-30,60,-1379.959],[-25,60,-502.4666],[-20,60,375.0255],[-15,60,1252.518],[-10,60,2130.01],[-5,60,3007.502],[0,60,3884.994],[5,60,4762.486],[10,60,5639.978],[15,60,6517.47],[20,60,7394.962],[25,60,8272.454],[30,60,9149.946],[35,60,10027.44],[40,60,10904.93],[45,60,11782.42],[-35,65,-2500.943],[-30,65,-1623.451],[-25,65,-745.9589],[-20,65,131.5332],[-15,65,1009.025],[-10,65,1886.517],[-5,65,2764.01],[0,65,3641.502],[5,65,4518.994],[10,65,5396.486],[15,65,6273.978],[20,65,7151.47],[25,65,8028.962],[30,65,8906.454],[35,65,9783.946],[40,65,10661.44],[45,65,11538.93],[-35,70,-2744.435],[-30,70,-1866.943],[-25,70,-989.4511],[-20,70,-111.959],[-15,70,765.5331],[-10,70,1643.025],[-5,70,2520.517],[0,70,3398.009],[5,70,4275.501],[10,70,5152.994],[15,70,6030.486],[20,70,6907.978],[25,70,7785.47],[30,70,8662.962],[35,70,9540.454],[40,70,10417.95],[45,70,11295.44]],"normals":[[-0.9635757,0.2673792,0.005490509],[-0.9635757,0.2673792,0.005490509],[-0.9635757,0.2673792,0.005490509],[-0.9635758,0.2673794,0.00549051],[-0.9635757,0.2673796,0.00549051],[-0.9635757,0.2673793,0.005490509],[-0.9635757,0.2673792,0.005490509],[-0.9635757,0.2673792,0.005490509],[-0.9635757,0.2673792,0.005490509],[-0.9635757,0.2673792,0.005490509],[-0.9635757,0.2673792,0.005490509],[-0.9635757,0.2673792,0.005490509],[-0.9635757,0.2673795,0.00549051],[-0.9635756,0.2673799,0.005490512],[-0.9635757,0.2673795,0.00549051],[-0.9635757,0.2673792,0.005490509],[-0.9635757,0.2673792,0.005490509],[-0.9635757,0.2673793,0.005490509],[-0.9635758,0.2673793,0.005490509],[-0.9635757,0.2673793,0.005490508],[-0.9635757,0.2673794,0.005490508],[-0.9635757,0.2673794,0.00549051],[-0.9635757,0.2673793,0.005490509],[-0.9635758,0.2673792,0.005490508],[-0.9635758,0.2673792,0.005490508],[-0.9635758,0.2673792,0.005490508],[-0.9635758,0.2673792,0.005490508],[-0.9635758,0.2673792,0.005490508],[-0.9635757,0.2673793,0.005490509],[-0.9635757,0.2673797,0.005490511],[-0.9635757,0.2673797,0.005490511],[-0.9635757,0.2673793,0.005490509],[-0.9635758,0.2673792,0.005490508],[-0.9635757,0.2673792,0.005490509],[-0.9635757,0.2673793,0.005490509],[-0.9635758,0.2673793,0.005490509],[-0.9635758,0.2673793,0.005490509],[-0.9635758,0.2673793,0.00549051],[-0.9635757,0.2673793,0.005490509],[-0.9635758,0.2673792,0.005490508],[-0.9635758,0.2673792,0.005490508],[-0.9635757,0.2673793,0.005490509],[-0.9635757,0.2673793,0.005490508],[-0.9635758,0.2673792,0.005490508],[-0.9635757,0.2673793,0.005490509],[-0.9635757,0.2673797,0.005490511],[-0.9635757,0.2673797,0.005490511],[-0.9635757,0.2673793,0.005490509],[-0.9635758,0.2673792,0.005490508],[-0.9635758,0.2673792,0.005490508],[-0.9635757,0.2673792,0.005490509],[-0.9635757,0.2673793,0.005490509],[-0.9635758,0.2673793,0.005490508],[-0.9635758,0.2673793,0.005490509],[-0.9635757,0.2673793,0.005490509],[-0.9635758,0.2673793,0.005490509],[-0.9635758,0.2673792,0.005490508],[-0.9635757,0.2673793,0.005490509],[-0.9635757,0.2673794,0.005490509],[-0.9635757,0.2673794,0.005490508],[-0.9635758,0.2673794,0.005490509],[-0.9635757,0.2673797,0.005490511],[-0.9635757,0.2673797,0.005490511],[-0.9635757,0.2673793,0.005490509],[-0.9635758,0.2673792,0.005490508],[-0.9635758,0.2673792,0.005490508],[-0.9635758,0.2673792,0.005490508],[-0.9635757,0.2673792,0.005490509],[-0.9635757,0.2673793,0.005490509],[-0.9635758,0.2673793,0.005490509],[-0.9635757,0.2673793,0.005490509],[-0.9635758,0.2673793,0.005490509],[-0.9635758,0.2673792,0.005490508],[-0.9635758,0.2673792,0.005490508],[-0.9635758,0.2673793,0.005490509],[-0.9635757,0.2673794,0.00549051],[-0.9635757,0.2673793,0.005490508],[-0.9635757,0.2673796,0.005490509],[-0.9635757,0.2673797,0.005490511],[-0.9635757,0.2673793,0.005490509],[-0.9635758,0.2673792,0.005490508],[-0.9635758,0.2673792,0.005490508],[-0.9635758,0.2673792,0.005490508],[-0.9635758,0.2673792,0.005490508],[-0.9635757,0.2673792,0.005490509],[-0.9635757,0.2673793,0.005490509],[-0.9635758,0.2673793,0.005490509],[-0.9635758,0.2673793,0.005490509],[-0.9635758,0.2673793,0.005490508],[-0.9635757,0.2673793,0.005490508],[-0.9635757,0.2673793,0.005490509],[-0.9635758,0.2673794,0.00549051],[-0.9635757,0.2673793,0.005490509],[-0.9635758,0.2673793,0.005490509],[-0.9635757,0.2673795,0.00549051],[-0.9635757,0.2673793,0.005490509],[-0.9635758,0.2673792,0.005490508],[-0.9635758,0.2673792,0.005490508],[-0.9635758,0.2673792,0.005490508],[-0.9635758,0.2673792,0.005490508],[-0.9635758,0.2673792,0.005490508],[-0.9635757,0.2673792,0.005490509],[-0.9635757,0.2673793,0.005490509],[-0.9635758,0.2673793,0.005490509],[-0.9635758,0.2673793,0.005490509],[-0.9635757,0.2673793,0.005490509],[-0.9635758,0.2673793,0.005490509],[-0.9635757,0.2673794,0.005490509],[-0.9635757,0.2673794,0.005490509],[-0.9635757,0.2673793,0.005490509],[-0.9635758,0.2673792,0.005490509],[-0.9635758,0.2673791,0.00549051],[-0.9635758,0.2673791,0.005490509],[-0.9635758,0.2673792,0.005490508],[-0.9635758,0.2673792,0.005490508],[-0.9635758,0.2673792,0.005490508],[-0.9635758,0.2673792,0.005490508],[-0.9635758,0.2673792,0.005490508],[-0.9635757,0.2673792,0.005490509],[-0.9635757,0.2673793,0.005490509],[-0.9635758,0.2673793,0.005490509],[-0.9635758,0.2673793,0.005490509],[-0.9635758,0.2673793,0.005490509],[-0.9635758,0.2673793,0.005490509],[-0.9635757,0.2673794,0.005490509],[-0.9635757,0.2673793,0.005490509],[-0.9635758,0.2673792,0.005490508],[-0.9635758,0.2673791,0.005490509],[-0.9635758,0.2673791,0.00549051],[-0.9635758,0.2673792,0.00549051],[-0.9635758,0.2673792,0.005490508],[-0.9635758,0.2673792,0.005490508],[-0.9635758,0.2673792,0.005490508],[-0.9635758,0.2673792,0.005490508],[-0.9635758,0.2673792,0.005490508],[-0.9635757,0.2673792,0.005490509],[-0.9635757,0.2673793,0.005490509],[-0.9635758,0.2673793,0.005490509],[-0.9635758,0.2673793,0.005490509],[-0.9635758,0.2673792,0.005490509],[-0.9635758,0.2673792,0.005490509],[-0.9635758,0.2673792,0.005490509],[-0.9635758,0.2673792,0.005490508],[-0.9635758,0.2673792,0.005490508],[-0.9635757,0.2673793,0.005490509],[-0.9635758,0.2673794,0.00549051],[-0.9635757,0.2673793,0.005490509],[-0.9635758,0.2673792,0.005490508],[-0.9635758,0.2673792,0.005490508],[-0.9635758,0.2673792,0.005490508],[-0.9635758,0.2673792,0.005490508],[-0.9635758,0.2673792,0.005490508],[-0.9635757,0.2673792,0.005490509],[-0.9635757,0.2673793,0.005490509],[-0.9635758,0.2673793,0.005490509],[-0.9635758,0.2673793,0.005490509],[-0.9635758,0.2673793,0.005490508],[-0.9635757,0.2673793,0.005490509],[-0.9635758,0.2673793,0.005490509],[-0.9635758,0.2673792,0.005490508],[-0.9635757,0.2673793,0.005490509],[-0.9635757,0.2673794,0.005490509],[-0.9635757,0.2673794,0.005490509],[-0.9635757,0.2673793,0.005490509],[-0.9635758,0.2673792,0.005490508],[-0.9635758,0.2673792,0.005490508],[-0.9635758,0.2673792,0.005490508],[-0.9635758,0.2673792,0.005490508],[-0.9635758,0.2673792,0.005490508],[-0.9635757,0.2673792,0.005490509],[-0.9635757,0.2673793,0.005490509],[-0.9635758,0.2673793,0.005490509],[-0.9635758,0.2673793,0.005490509],[-0.9635758,0.2673793,0.005490509],[-0.9635758,0.2673793,0.005490509],[-0.9635758,0.2673792,0.005490508],[-0.9635757,0.2673793,0.005490509],[-0.9635757,0.2673794,0.005490509],[-0.9635757,0.2673794,0.005490509],[-0.9635757,0.2673793,0.005490509],[-0.9635758,0.2673792,0.005490508],[-0.9635758,0.2673792,0.005490508],[-0.9635758,0.2673792,0.005490508],[-0.9635758,0.2673792,0.005490508],[-0.9635758,0.2673792,0.005490508],[-0.9635757,0.2673793,0.005490509],[-0.9635757,0.2673796,0.005490511],[-0.9635757,0.2673793,0.005490509],[-0.9635758,0.2673793,0.005490509],[-0.9635758,0.2673793,0.005490509],[-0.9635758,0.2673793,0.005490508],[-0.9635758,0.2673793,0.005490509],[-0.9635757,0.2673793,0.005490509],[-0.9635757,0.2673794,0.005490509],[-0.9635757,0.2673794,0.005490509],[-0.9635757,0.2673793,0.005490509],[-0.9635758,0.2673792,0.005490508],[-0.9635758,0.2673792,0.005490508],[-0.9635758,0.2673792,0.005490508],[-0.9635758,0.2673792,0.005490508],[-0.9635758,0.2673792,0.005490508],[-0.9635757,0.2673793,0.005490509],[-0.9635757,0.2673797,0.005490511],[-0.9635756,0.2673799,0.005490512],[-0.9635757,0.2673793,0.005490509],[-0.9635758,0.2673793,0.005490509],[-0.9635758,0.2673793,0.005490509],[-0.9635758,0.2673793,0.005490509],[-0.9635758,0.2673793,0.005490509],[-0.9635758,0.2673793,0.005490509],[-0.9635757,0.2673793,0.005490509],[-0.9635757,0.2673793,0.005490509],[-0.9635758,0.2673792,0.005490508],[-0.9635758,0.2673792,0.005490508],[-0.9635758,0.2673792,0.005490508],[-0.9635758,0.2673792,0.005490508],[-0.9635758,0.2673792,0.005490508],[-0.9635757,0.2673793,0.005490509],[-0.9635757,0.2673797,0.005490511],[-0.9635757,0.2673797,0.005490511],[-0.9635757,0.2673795,0.00549051],[-0.9635757,0.2673793,0.005490509],[-0.9635758,0.2673793,0.005490508],[-0.9635758,0.2673793,0.005490509],[-0.9635758,0.2673793,0.005490509],[-0.9635758,0.2673793,0.005490509],[-0.9635758,0.2673793,0.005490509],[-0.9635757,0.2673792,0.005490509],[-0.9635758,0.2673792,0.005490509],[-0.9635758,0.2673792,0.005490508],[-0.9635757,0.2673793,0.005490509],[-0.9635757,0.2673794,0.005490509],[-0.9635757,0.2673794,0.005490508],[-0.9635757,0.2673794,0.005490509],[-0.9635757,0.2673797,0.005490511],[-0.9635757,0.2673797,0.005490511],[-0.9635757,0.2673793,0.005490509],[-0.9635757,0.2673792,0.005490509],[-0.9635757,0.2673793,0.005490509],[-0.9635758,0.2673793,0.005490509],[-0.9635758,0.2673793,0.005490509],[-0.9635758,0.2673793,0.005490509],[-0.9635758,0.2673793,0.005490509],[-0.9635758,0.2673793,0.005490509],[-0.9635758,0.2673793,0.005490509],[-0.9635758,0.2673792,0.005490508],[-0.9635757,0.2673793,0.005490509],[-0.9635757,0.2673794,0.005490509],[-0.9635757,0.2673795,0.005490509],[-0.9635757,0.2673795,0.005490508],[-0.9635756,0.2673797,0.005490509],[-0.9635757,0.2673797,0.005490511],[-0.9635757,0.2673793,0.005490509],[-0.9635758,0.2673792,0.005490508],[-0.9635757,0.2673792,0.005490509],[-0.9635757,0.2673793,0.005490509],[-0.9635758,0.2673793,0.005490509],[-0.9635758,0.2673793,0.005490509],[-0.9635758,0.2673793,0.005490509],[-0.9635758,0.2673793,0.005490509],[-0.9635758,0.2673793,0.005490509],[-0.9635758,0.2673792,0.005490508],[-0.9635757,0.2673793,0.005490509],[-0.9635757,0.2673794,0.005490509],[-0.9635757,0.2673794,0.005490509],[-0.9635757,0.2673793,0.005490509],[-0.9635758,0.2673793,0.005490509],[-0.9635757,0.2673795,0.00549051],[-0.9635757,0.2673793,0.005490509],[-0.9635758,0.2673792,0.005490508],[-0.9635758,0.2673792,0.005490508],[-0.9635757,0.2673792,0.005490509],[-0.9635757,0.2673793,0.005490509],[-0.9635758,0.2673793,0.005490509],[-0.9635758,0.2673793,0.005490509],[-0.9635758,0.2673793,0.005490509],[-0.9635758,0.2673793,0.005490509],[-0.9635758,0.2673793,0.005490509],[-0.9635757,0.2673793,0.005490508],[-0.9635757,0.2673794,0.005490508],[-0.9635757,0.2673794,0.00549051],[-0.9635757,0.2673793,0.005490509],[-0.9635758,0.2673792,0.005490508],[-0.9635758,0.2673793,0.00549051],[-0.9635758,0.2673793,0.00549051],[-0.9635758,0.2673792,0.005490508],[-0.9635758,0.2673792,0.005490508],[-0.9635758,0.2673792,0.005490508],[-0.9635757,0.2673792,0.005490509],[-0.9635757,0.2673793,0.005490509],[-0.9635758,0.2673793,0.005490508],[-0.9635758,0.2673793,0.005490509],[-0.9635758,0.2673793,0.005490509],[-0.9635758,0.2673793,0.005490509],[-0.9635758,0.2673793,0.005490509],[-0.9635758,0.2673793,0.005490509],[-0.9635758,0.2673793,0.00549051],[-0.9635757,0.2673793,0.005490509],[-0.9635758,0.2673792,0.005490508],[-0.9635758,0.2673792,0.005490508],[-0.9635758,0.2673793,0.00549051],[-0.9635758,0.2673793,0.00549051],[-0.9635758,0.2673792,0.005490508],[-0.9635758,0.2673792,0.005490508],[-0.9635758,0.2673792,0.005490508],[-0.9635757,0.2673792,0.005490509],[-0.9635757,0.2673792,0.005490509],[-0.9635758,0.2673793,0.005490509],[-0.9635758,0.2673793,0.005490509],[-0.9635758,0.2673793,0.005490509],[-0.9635758,0.2673793,0.005490509],[-0.9635758,0.2673793,0.005490509],[-0.9635758,0.2673793,0.005490509],[-0.9635757,0.2673793,0.005490509],[-0.9635758,0.2673793,0.005490509],[-0.9635758,0.2673792,0.005490508],[-0.9635758,0.2673792,0.005490508],[-0.9635758,0.2673793,0.00549051],[-0.9635758,0.2673793,0.00549051],[-0.9635758,0.2673792,0.005490508],[-0.9635758,0.2673792,0.005490508],[-0.9635758,0.2673792,0.005490508],[-0.9635757,0.2673792,0.005490509],[-0.9635757,0.2673793,0.005490509],[-0.9635757,0.2673793,0.005490509],[-0.9635757,0.2673793,0.005490509],[-0.9635757,0.2673793,0.005490509],[-0.9635757,0.2673793,0.005490509],[-0.9635757,0.2673793,0.005490509],[-0.9635757,0.2673793,0.005490509],[-0.9635757,0.2673793,0.005490509],[-0.9635757,0.2673793,0.005490509],[-0.9635757,0.2673792,0.005490509],[-0.9635757,0.2673792,0.005490509],[-0.9635757,0.2673793,0.00549051],[-0.9635757,0.2673793,0.00549051],[-0.9635757,0.2673792,0.005490509],[-0.9635757,0.2673792,0.005490509],[-0.9635757,0.2673792,0.005490509],[-0.9635757,0.2673792,0.005490509]],"colors":[[0,0,0,0.2980392]],"dim":[[17,20]],"centers":[[-32.5,-22.5,2198.917],[-27.5,-22.5,3076.41],[-22.5,-22.5,3953.902],[-17.5,-22.5,4831.394],[-12.5,-22.5,5708.886],[-7.5,-22.5,6586.378],[-2.5,-22.5,7463.87],[2.5,-22.5,8341.362],[7.5,-22.5,9218.854],[12.5,-22.5,10096.35],[17.5,-22.5,10973.84],[22.5,-22.5,11851.33],[27.5,-22.5,12728.82],[32.5,-22.5,13606.31],[37.5,-22.5,14483.81],[42.5,-22.5,15361.3],[-32.5,-17.5,1955.425],[-27.5,-17.5,2832.917],[-22.5,-17.5,3710.409],[-17.5,-17.5,4587.901],[-12.5,-17.5,5465.394],[-7.5,-17.5,6342.886],[-2.5,-17.5,7220.378],[2.5,-17.5,8097.87],[7.5,-17.5,8975.362],[12.5,-17.5,9852.854],[17.5,-17.5,10730.35],[22.5,-17.5,11607.84],[27.5,-17.5,12485.33],[32.5,-17.5,13362.82],[37.5,-17.5,14240.31],[42.5,-17.5,15117.81],[-32.5,-12.5,1711.933],[-27.5,-12.5,2589.425],[-22.5,-12.5,3466.917],[-17.5,-12.5,4344.409],[-12.5,-12.5,5221.901],[-7.5,-12.5,6099.394],[-2.5,-12.5,6976.886],[2.5,-12.5,7854.377],[7.5,-12.5,8731.87],[12.5,-12.5,9609.362],[17.5,-12.5,10486.85],[22.5,-12.5,11364.35],[27.5,-12.5,12241.84],[32.5,-12.5,13119.33],[37.5,-12.5,13996.82],[42.5,-12.5,14874.31],[-32.5,-7.5,1468.441],[-27.5,-7.5,2345.933],[-22.5,-7.5,3223.425],[-17.5,-7.5,4100.917],[-12.5,-7.5,4978.409],[-7.5,-7.5,5855.901],[-2.5,-7.5,6733.393],[2.5,-7.5,7610.885],[7.5,-7.5,8488.378],[12.5,-7.5,9365.869],[17.5,-7.5,10243.36],[22.5,-7.5,11120.85],[27.5,-7.5,11998.35],[32.5,-7.5,12875.84],[37.5,-7.5,13753.33],[42.5,-7.5,14630.82],[-32.5,-2.5,1224.948],[-27.5,-2.5,2102.441],[-22.5,-2.5,2979.933],[-17.5,-2.5,3857.425],[-12.5,-2.5,4734.917],[-7.5,-2.5,5612.409],[-2.5,-2.5,6489.901],[2.5,-2.5,7367.393],[7.5,-2.5,8244.885],[12.5,-2.5,9122.377],[17.5,-2.5,9999.869],[22.5,-2.5,10877.36],[27.5,-2.5,11754.85],[32.5,-2.5,12632.35],[37.5,-2.5,13509.84],[42.5,-2.5,14387.33],[-32.5,2.5,981.4563],[-27.5,2.5,1858.948],[-22.5,2.5,2736.44],[-17.5,2.5,3613.933],[-12.5,2.5,4491.425],[-7.5,2.5,5368.917],[-2.5,2.5,6246.409],[2.5,2.5,7123.901],[7.5,2.5,8001.393],[12.5,2.5,8878.885],[17.5,2.5,9756.377],[22.5,2.5,10633.87],[27.5,2.5,11511.36],[32.5,2.5,12388.85],[37.5,2.5,13266.35],[42.5,2.5,14143.84],[-32.5,7.5,737.964],[-27.5,7.5,1615.456],[-22.5,7.5,2492.948],[-17.5,7.5,3370.44],[-12.5,7.5,4247.933],[-7.5,7.5,5125.424],[-2.5,7.5,6002.917],[2.5,7.5,6880.409],[7.5,7.5,7757.9],[12.5,7.5,8635.393],[17.5,7.5,9512.885],[22.5,7.5,10390.38],[27.5,7.5,11267.87],[32.5,7.5,12145.36],[37.5,7.5,13022.85],[42.5,7.5,13900.35],[-32.5,12.5,494.4717],[-27.5,12.5,1371.964],[-22.5,12.5,2249.456],[-17.5,12.5,3126.948],[-12.5,12.5,4004.44],[-7.5,12.5,4881.932],[-2.5,12.5,5759.424],[2.5,12.5,6636.917],[7.5,12.5,7514.409],[12.5,12.5,8391.9],[17.5,12.5,9269.393],[22.5,12.5,10146.88],[27.5,12.5,11024.38],[32.5,12.5,11901.87],[37.5,12.5,12779.36],[42.5,12.5,13656.85],[-32.5,17.5,250.9795],[-27.5,17.5,1128.471],[-22.5,17.5,2005.964],[-17.5,17.5,2883.456],[-12.5,17.5,3760.948],[-7.5,17.5,4638.44],[-2.5,17.5,5515.932],[2.5,17.5,6393.424],[7.5,17.5,7270.917],[12.5,17.5,8148.408],[17.5,17.5,9025.9],[22.5,17.5,9903.393],[27.5,17.5,10780.88],[32.5,17.5,11658.38],[37.5,17.5,12535.87],[42.5,17.5,13413.36],[-32.5,22.5,7.48719],[-27.5,22.5,884.9793],[-22.5,22.5,1762.471],[-17.5,22.5,2639.963],[-12.5,22.5,3517.456],[-7.5,22.5,4394.948],[-2.5,22.5,5272.44],[2.5,22.5,6149.932],[7.5,22.5,7027.424],[12.5,22.5,7904.916],[17.5,22.5,8782.408],[22.5,22.5,9659.9],[27.5,22.5,10537.39],[32.5,22.5,11414.88],[37.5,22.5,12292.38],[42.5,22.5,13169.87],[-32.5,27.5,-236.0051],[-27.5,27.5,641.4871],[-22.5,27.5,1518.979],[-17.5,27.5,2396.471],[-12.5,27.5,3273.963],[-7.5,27.5,4151.456],[-2.5,27.5,5028.948],[2.5,27.5,5906.439],[7.5,27.5,6783.932],[12.5,27.5,7661.424],[17.5,27.5,8538.916],[22.5,27.5,9416.408],[27.5,27.5,10293.9],[32.5,27.5,11171.39],[37.5,27.5,12048.88],[42.5,27.5,12926.38],[-32.5,32.5,-479.4973],[-27.5,32.5,397.9948],[-22.5,32.5,1275.487],[-17.5,32.5,2152.979],[-12.5,32.5,3030.471],[-7.5,32.5,3907.963],[-2.5,32.5,4785.455],[2.5,32.5,5662.947],[7.5,32.5,6540.439],[12.5,32.5,7417.932],[17.5,32.5,8295.424],[22.5,32.5,9172.916],[27.5,32.5,10050.41],[32.5,32.5,10927.9],[37.5,32.5,11805.39],[42.5,32.5,12682.88],[-32.5,37.5,-722.9896],[-27.5,37.5,154.5025],[-22.5,37.5,1031.995],[-17.5,37.5,1909.487],[-12.5,37.5,2786.979],[-7.5,37.5,3664.471],[-2.5,37.5,4541.963],[2.5,37.5,5419.455],[7.5,37.5,6296.947],[12.5,37.5,7174.439],[17.5,37.5,8051.932],[22.5,37.5,8929.424],[27.5,37.5,9806.916],[32.5,37.5,10684.41],[37.5,37.5,11561.9],[42.5,37.5,12439.39],[-32.5,42.5,-966.4818],[-27.5,42.5,-88.98973],[-22.5,42.5,788.5023],[-17.5,42.5,1665.994],[-12.5,42.5,2543.487],[-7.5,42.5,3420.979],[-2.5,42.5,4298.471],[2.5,42.5,5175.963],[7.5,42.5,6053.455],[12.5,42.5,6930.947],[17.5,42.5,7808.439],[22.5,42.5,8685.932],[27.5,42.5,9563.424],[32.5,42.5,10440.92],[37.5,42.5,11318.41],[42.5,42.5,12195.9],[-32.5,47.5,-1209.974],[-27.5,47.5,-332.482],[-22.5,47.5,545.0101],[-17.5,47.5,1422.502],[-12.5,47.5,2299.994],[-7.5,47.5,3177.486],[-2.5,47.5,4054.979],[2.5,47.5,4932.471],[7.5,47.5,5809.963],[12.5,47.5,6687.455],[17.5,47.5,7564.947],[22.5,47.5,8442.439],[27.5,47.5,9319.931],[32.5,47.5,10197.42],[37.5,47.5,11074.92],[42.5,47.5,11952.41],[-32.5,52.5,-1453.466],[-27.5,52.5,-575.9742],[-22.5,52.5,301.5178],[-17.5,52.5,1179.01],[-12.5,52.5,2056.502],[-7.5,52.5,2933.994],[-2.5,52.5,3811.486],[2.5,52.5,4688.979],[7.5,52.5,5566.471],[12.5,52.5,6443.962],[17.5,52.5,7321.455],[22.5,52.5,8198.946],[27.5,52.5,9076.438],[32.5,52.5,9953.931],[37.5,52.5,10831.42],[42.5,52.5,11708.92],[-32.5,57.5,-1696.959],[-27.5,57.5,-819.4665],[-22.5,57.5,58.02557],[-17.5,57.5,935.5176],[-12.5,57.5,1813.01],[-7.5,57.5,2690.502],[-2.5,57.5,3567.994],[2.5,57.5,4445.486],[7.5,57.5,5322.978],[12.5,57.5,6200.47],[17.5,57.5,7077.962],[22.5,57.5,7955.454],[27.5,57.5,8832.946],[32.5,57.5,9710.438],[37.5,57.5,10587.93],[42.5,57.5,11465.42],[-32.5,62.5,-1940.451],[-27.5,62.5,-1062.959],[-22.5,62.5,-185.4667],[-17.5,62.5,692.0254],[-12.5,62.5,1569.518],[-7.5,62.5,2447.01],[-2.5,62.5,3324.502],[2.5,62.5,4201.994],[7.5,62.5,5079.486],[12.5,62.5,5956.978],[17.5,62.5,6834.47],[22.5,62.5,7711.962],[27.5,62.5,8589.454],[32.5,62.5,9466.946],[37.5,62.5,10344.44],[42.5,62.5,11221.93],[-32.5,67.5,-2183.943],[-27.5,67.5,-1306.451],[-22.5,67.5,-428.9589],[-17.5,67.5,448.5332],[-12.5,67.5,1326.025],[-7.5,67.5,2203.517],[-2.5,67.5,3081.009],[2.5,67.5,3958.501],[7.5,67.5,4835.994],[12.5,67.5,5713.486],[17.5,67.5,6590.978],[22.5,67.5,7468.47],[27.5,67.5,8345.962],[32.5,67.5,9223.454],[37.5,67.5,10100.95],[42.5,67.5,10978.44]],"ignoreExtent":false,"flags":91},"5":{"id":5,"type":"light","vertices":[[0,0,1]],"colors":[[1,1,1,1],[1,1,1,1],[1,1,1,1]],"viewpoint":true,"finite":false},"4":{"id":4,"type":"background","material":{"fog":true},"colors":[[0.2980392,0.2980392,0.2980392,1]],"centers":[[0,0,0]],"sphere":false,"fogtype":"none","flags":0},"6":{"id":6,"type":"background","material":{"lit":false,"back":"lines"},"colors":[[1,1,1,1]],"centers":[[0,0,0]],"sphere":false,"fogtype":"none","flags":0},"8":{"id":8,"type":"bboxdeco","material":{"front":"lines","back":"lines"},"vertices":[[-20,"NA","NA"],[0,"NA","NA"],[20,"NA","NA"],[40,"NA","NA"],["NA",-20,"NA"],["NA",0,"NA"],["NA",20,"NA"],["NA",40,"NA"],["NA",60,"NA"],["NA","NA",0],["NA","NA",5000],["NA","NA",10000],["NA","NA",15000],["NA","NA",20000],["NA","NA",25000]],"colors":[[0,0,0,1]],"draw_front":true,"newIds":[21,22,23,24,25,26,27]},"1":{"id":1,"type":"subscene","par3d":{"antialias":8,"FOV":30,"ignoreExtent":false,"listeners":1,"mouseMode":{"left":"trackball","right":"zoom","middle":"fov","wheel":"pull"},"observer":[0,0,67827.12],"modelMatrix":[[201.4993,0,0,-1007.497],[0,51.17001,0.542538,-7401.868],[0,-140.5884,0.1974677,-67372.59],[0,0,0,1]],"projMatrix":[[2.665751,0,0,0],[0,3.732051,0,0],[0,0,-3.863704,-244508.9],[0,0,-1,0]],"skipRedraw":false,"userMatrix":[[1,0,0,0],[0,0.3420201,0.9396926,0],[0,-0.9396926,0.3420201,0],[0,0,0,1]],"scale":[201.4993,149.6111,0.5773569],"viewport":{"x":0,"y":0,"width":1,"height":1},"zoom":1,"bbox":[-35,45,-30.60419,70.15616,-2744.435,26300.13],"windowRect":[0,45,672,525],"family":"sans","font":1,"cex":1,"useFreeType":true,"fontname":"/Library/Frameworks/R.framework/Versions/3.2/Resources/library/rgl/fonts/FreeSans.ttf","maxClipPlanes":6},"embeddings":{"viewport":"replace","projection":"replace","model":"replace"},"objects":[6,8,7,9,10,11,12,13,5,21,22,23,24,25,26,27],"subscenes":[],"flags":1275},"21":{"id":21,"type":"lines","material":{"lit":false,"front":"lines","back":"lines"},"vertices":[[-20,-32.1156,-3180.104],[40,-32.1156,-3180.104],[-20,-32.1156,-3180.104],[-20,-34.71018,-3928.001],[0,-32.1156,-3180.104],[0,-34.71018,-3928.001],[20,-32.1156,-3180.104],[20,-34.71018,-3928.001],[40,-32.1156,-3180.104],[40,-34.71018,-3928.001]],"colors":[[0,0,0,1]],"centers":[[10,-32.1156,-3180.104],[-20,-33.41289,-3554.053],[0,-33.41289,-3554.053],[20,-33.41289,-3554.053],[40,-33.41289,-3554.053]],"ignoreExtent":true,"origId":8,"flags":128},"22":{"id":22,"type":"text","material":{"lit":false,"front":"lines","back":"lines"},"vertices":[[-20,-39.89934,-5423.797],[0,-39.89934,-5423.797],[20,-39.89934,-5423.797],[40,-39.89934,-5423.797]],"colors":[[0,0,0,1]],"texts":[["-20"],["0"],["20"],["40"]],"cex":[[1]],"adj":[[0.5,0.5]],"centers":[[-20,-39.89934,-5423.797],[0,-39.89934,-5423.797],[20,-39.89934,-5423.797],[40,-39.89934,-5423.797]],"family":[["sans"]],"font":[[1]],"ignoreExtent":true,"origId":8,"flags":40},"23":{"id":23,"type":"lines","material":{"lit":false,"front":"lines","back":"lines"},"vertices":[[-36.2,-20,-3180.104],[-36.2,60,-3180.104],[-36.2,-20,-3180.104],[-38.26,-20,-3928.001],[-36.2,0,-3180.104],[-38.26,0,-3928.001],[-36.2,20,-3180.104],[-38.26,20,-3928.001],[-36.2,40,-3180.104],[-38.26,40,-3928.001],[-36.2,60,-3180.104],[-38.26,60,-3928.001]],"colors":[[0,0,0,1]],"centers":[[-36.2,20,-3180.104],[-37.23,-20,-3554.053],[-37.23,0,-3554.053],[-37.23,20,-3554.053],[-37.23,40,-3554.053],[-37.23,60,-3554.053]],"ignoreExtent":true,"origId":8,"flags":128},"24":{"id":24,"type":"text","material":{"lit":false,"front":"lines","back":"lines"},"vertices":[[-42.38,-20,-5423.797],[-42.38,0,-5423.797],[-42.38,20,-5423.797],[-42.38,40,-5423.797],[-42.38,60,-5423.797]],"colors":[[0,0,0,1]],"texts":[["-20"],["0"],["20"],["40"],["60"]],"cex":[[1]],"adj":[[0.5,0.5]],"centers":[[-42.38,-20,-5423.797],[-42.38,0,-5423.797],[-42.38,20,-5423.797],[-42.38,40,-5423.797],[-42.38,60,-5423.797]],"family":[["sans"]],"font":[[1]],"ignoreExtent":true,"origId":8,"flags":40},"25":{"id":25,"type":"lines","material":{"lit":false,"front":"lines","back":"lines"},"vertices":[[-36.2,-32.1156,0],[-36.2,-32.1156,25000],[-36.2,-32.1156,0],[-38.26,-34.71018,0],[-36.2,-32.1156,5000],[-38.26,-34.71018,5000],[-36.2,-32.1156,10000],[-38.26,-34.71018,10000],[-36.2,-32.1156,15000],[-38.26,-34.71018,15000],[-36.2,-32.1156,20000],[-38.26,-34.71018,20000],[-36.2,-32.1156,25000],[-38.26,-34.71018,25000]],"colors":[[0,0,0,1]],"centers":[[-36.2,-32.1156,12500],[-37.23,-33.41289,0],[-37.23,-33.41289,5000],[-37.23,-33.41289,10000],[-37.23,-33.41289,15000],[-37.23,-33.41289,20000],[-37.23,-33.41289,25000]],"ignoreExtent":true,"origId":8,"flags":128},"26":{"id":26,"type":"text","material":{"lit":false,"front":"lines","back":"lines"},"vertices":[[-42.38,-39.89934,0],[-42.38,-39.89934,5000],[-42.38,-39.89934,10000],[-42.38,-39.89934,15000],[-42.38,-39.89934,20000],[-42.38,-39.89934,25000]],"colors":[[0,0,0,1]],"texts":[["0"],["5000"],["10000"],["15000"],["20000"],["25000"]],"cex":[[1]],"adj":[[0.5,0.5]],"centers":[[-42.38,-39.89934,0],[-42.38,-39.89934,5000],[-42.38,-39.89934,10000],[-42.38,-39.89934,15000],[-42.38,-39.89934,20000],[-42.38,-39.89934,25000]],"family":[["sans"]],"font":[[1]],"ignoreExtent":true,"origId":8,"flags":40},"27":{"id":27,"type":"lines","material":{"lit":false,"front":"lines","back":"lines"},"vertices":[[-36.2,-32.1156,-3180.104],[-36.2,71.66756,-3180.104],[-36.2,-32.1156,26735.8],[-36.2,71.66756,26735.8],[-36.2,-32.1156,-3180.104],[-36.2,-32.1156,26735.8],[-36.2,71.66756,-3180.104],[-36.2,71.66756,26735.8],[-36.2,-32.1156,-3180.104],[46.2,-32.1156,-3180.104],[-36.2,-32.1156,26735.8],[46.2,-32.1156,26735.8],[-36.2,71.66756,-3180.104],[46.2,71.66756,-3180.104],[-36.2,71.66756,26735.8],[46.2,71.66756,26735.8],[46.2,-32.1156,-3180.104],[46.2,71.66756,-3180.104],[46.2,-32.1156,26735.8],[46.2,71.66756,26735.8],[46.2,-32.1156,-3180.104],[46.2,-32.1156,26735.8],[46.2,71.66756,-3180.104],[46.2,71.66756,26735.8]],"colors":[[0,0,0,1]],"centers":[[-36.2,19.77598,-3180.104],[-36.2,19.77598,26735.8],[-36.2,-32.1156,11777.85],[-36.2,71.66756,11777.85],[5,-32.1156,-3180.104],[5,-32.1156,26735.8],[5,71.66756,-3180.104],[5,71.66756,26735.8],[46.2,19.77598,-3180.104],[46.2,19.77598,26735.8],[46.2,-32.1156,11777.85],[46.2,71.66756,11777.85]],"ignoreExtent":true,"origId":8,"flags":128}},"snapshot":"data:image/png;base64,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","width":673,"height":481,"sphereVerts":{"vb":[[0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0.07465783,0.1464466,0.2126075,0.2705981,0.3181896,0.3535534,0.3753303,0.3826834,0.3753303,0.3535534,0.3181896,0.2705981,0.2126075,0.1464466,0.07465783,0,0,0.1379497,0.2705981,0.3928475,0.5,0.5879378,0.6532815,0.6935199,0.7071068,0.6935199,0.6532815,0.5879378,0.5,0.3928475,0.2705981,0.1379497,0,0,0.18024,0.3535534,0.51328,0.6532815,0.7681778,0.8535534,0.9061274,0.9238795,0.9061274,0.8535534,0.7681778,0.6532815,0.51328,0.3535534,0.18024,0,0,0.1950903,0.3826834,0.5555702,0.7071068,0.8314696,0.9238795,0.9807853,1,0.9807853,0.9238795,0.8314696,0.7071068,0.5555702,0.3826834,0.1950903,0,0,0.18024,0.3535534,0.51328,0.6532815,0.7681778,0.8535534,0.9061274,0.9238795,0.9061274,0.8535534,0.7681778,0.6532815,0.51328,0.3535534,0.18024,0,0,0.1379497,0.2705981,0.3928475,0.5,0.5879378,0.6532815,0.6935199,0.7071068,0.6935199,0.6532815,0.5879378,0.5,0.3928475,0.2705981,0.1379497,0,0,0.07465783,0.1464466,0.2126075,0.2705981,0.3181896,0.3535534,0.3753303,0.3826834,0.3753303,0.3535534,0.3181896,0.2705981,0.2126075,0.1464466,0.07465783,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,-0,-0.07465783,-0.1464466,-0.2126075,-0.2705981,-0.3181896,-0.3535534,-0.3753303,-0.3826834,-0.3753303,-0.3535534,-0.3181896,-0.2705981,-0.2126075,-0.1464466,-0.07465783,-0,-0,-0.1379497,-0.2705981,-0.3928475,-0.5,-0.5879378,-0.6532815,-0.6935199,-0.7071068,-0.6935199,-0.6532815,-0.5879378,-0.5,-0.3928475,-0.2705981,-0.1379497,-0,-0,-0.18024,-0.3535534,-0.51328,-0.6532815,-0.7681778,-0.8535534,-0.9061274,-0.9238795,-0.9061274,-0.8535534,-0.7681778,-0.6532815,-0.51328,-0.3535534,-0.18024,-0,-0,-0.1950903,-0.3826834,-0.5555702,-0.7071068,-0.8314696,-0.9238795,-0.9807853,-1,-0.9807853,-0.9238795,-0.8314696,-0.7071068,-0.5555702,-0.3826834,-0.1950903,-0,-0,-0.18024,-0.3535534,-0.51328,-0.6532815,-0.7681778,-0.8535534,-0.9061274,-0.9238795,-0.9061274,-0.8535534,-0.7681778,-0.6532815,-0.51328,-0.3535534,-0.18024,-0,-0,-0.1379497,-0.2705981,-0.3928475,-0.5,-0.5879378,-0.6532815,-0.6935199,-0.7071068,-0.6935199,-0.6532815,-0.5879378,-0.5,-0.3928475,-0.2705981,-0.1379497,-0,-0,-0.07465783,-0.1464466,-0.2126075,-0.2705981,-0.3181896,-0.3535534,-0.3753303,-0.3826834,-0.3753303,-0.3535534,-0.3181896,-0.2705981,-0.2126075,-0.1464466,-0.07465783,-0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0],[-1,-0.9807853,-0.9238795,-0.8314696,-0.7071068,-0.5555702,-0.3826834,-0.1950903,0,0.1950903,0.3826834,0.5555702,0.7071068,0.8314696,0.9238795,0.9807853,1,-1,-0.9807853,-0.9238795,-0.8314696,-0.7071068,-0.5555702,-0.3826834,-0.1950903,0,0.1950903,0.3826834,0.5555702,0.7071068,0.8314696,0.9238795,0.9807853,1,-1,-0.9807853,-0.9238795,-0.8314696,-0.7071068,-0.5555702,-0.3826834,-0.1950903,0,0.1950903,0.3826834,0.5555702,0.7071068,0.8314696,0.9238795,0.9807853,1,-1,-0.9807853,-0.9238795,-0.8314696,-0.7071068,-0.5555702,-0.3826834,-0.1950903,0,0.1950903,0.3826834,0.5555702,0.7071068,0.8314696,0.9238795,0.9807853,1,-1,-0.9807853,-0.9238795,-0.8314696,-0.7071068,-0.5555702,-0.3826834,-0.1950903,0,0.1950903,0.3826834,0.5555702,0.7071068,0.8314696,0.9238795,0.9807853,1,-1,-0.9807853,-0.9238795,-0.8314696,-0.7071068,-0.5555702,-0.3826834,-0.1950903,0,0.1950903,0.3826834,0.5555702,0.7071068,0.8314696,0.9238795,0.9807853,1,-1,-0.9807853,-0.9238795,-0.8314696,-0.7071068,-0.5555702,-0.3826834,-0.1950903,0,0.1950903,0.3826834,0.5555702,0.7071068,0.8314696,0.9238795,0.9807853,1,-1,-0.9807853,-0.9238795,-0.8314696,-0.7071068,-0.5555702,-0.3826834,-0.1950903,0,0.1950903,0.3826834,0.5555702,0.7071068,0.8314696,0.9238795,0.9807853,1,-1,-0.9807853,-0.9238795,-0.8314696,-0.7071068,-0.5555702,-0.3826834,-0.1950903,0,0.1950903,0.3826834,0.5555702,0.7071068,0.8314696,0.9238795,0.9807853,1,-1,-0.9807853,-0.9238795,-0.8314696,-0.7071068,-0.5555702,-0.3826834,-0.1950903,0,0.1950903,0.3826834,0.5555702,0.7071068,0.8314696,0.9238795,0.9807853,1,-1,-0.9807853,-0.9238795,-0.8314696,-0.7071068,-0.5555702,-0.3826834,-0.1950903,0,0.1950903,0.3826834,0.5555702,0.7071068,0.8314696,0.9238795,0.9807853,1,-1,-0.9807853,-0.9238795,-0.8314696,-0.7071068,-0.5555702,-0.3826834,-0.1950903,0,0.1950903,0.3826834,0.5555702,0.7071068,0.8314696,0.9238795,0.9807853,1,-1,-0.9807853,-0.9238795,-0.8314696,-0.7071068,-0.5555702,-0.3826834,-0.1950903,0,0.1950903,0.3826834,0.5555702,0.7071068,0.8314696,0.9238795,0.9807853,1,-1,-0.9807853,-0.9238795,-0.8314696,-0.7071068,-0.5555702,-0.3826834,-0.1950903,0,0.1950903,0.3826834,0.5555702,0.7071068,0.8314696,0.9238795,0.9807853,1,-1,-0.9807853,-0.9238795,-0.8314696,-0.7071068,-0.5555702,-0.3826834,-0.1950903,0,0.1950903,0.3826834,0.5555702,0.7071068,0.8314696,0.9238795,0.9807853,1,-1,-0.9807853,-0.9238795,-0.8314696,-0.7071068,-0.5555702,-0.3826834,-0.1950903,0,0.1950903,0.3826834,0.5555702,0.7071068,0.8314696,0.9238795,0.9807853,1,-1,-0.9807853,-0.9238795,-0.8314696,-0.7071068,-0.5555702,-0.3826834,-0.1950903,0,0.1950903,0.3826834,0.5555702,0.7071068,0.8314696,0.9238795,0.9807853,1],[0,0.1950903,0.3826834,0.5555702,0.7071068,0.8314696,0.9238795,0.9807853,1,0.9807853,0.9238795,0.8314696,0.7071068,0.5555702,0.3826834,0.1950903,0,0,0.18024,0.3535534,0.51328,0.6532815,0.7681778,0.8535534,0.9061274,0.9238795,0.9061274,0.8535534,0.7681778,0.6532815,0.51328,0.3535534,0.18024,0,0,0.1379497,0.2705981,0.3928475,0.5,0.5879378,0.6532815,0.6935199,0.7071068,0.6935199,0.6532815,0.5879378,0.5,0.3928475,0.2705981,0.1379497,0,0,0.07465783,0.1464466,0.2126075,0.2705981,0.3181896,0.3535534,0.3753303,0.3826834,0.3753303,0.3535534,0.3181896,0.2705981,0.2126075,0.1464466,0.07465783,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,-0,-0.07465783,-0.1464466,-0.2126075,-0.2705981,-0.3181896,-0.3535534,-0.3753303,-0.3826834,-0.3753303,-0.3535534,-0.3181896,-0.2705981,-0.2126075,-0.1464466,-0.07465783,-0,-0,-0.1379497,-0.2705981,-0.3928475,-0.5,-0.5879378,-0.6532815,-0.6935199,-0.7071068,-0.6935199,-0.6532815,-0.5879378,-0.5,-0.3928475,-0.2705981,-0.1379497,-0,-0,-0.18024,-0.3535534,-0.51328,-0.6532815,-0.7681778,-0.8535534,-0.9061274,-0.9238795,-0.9061274,-0.8535534,-0.7681778,-0.6532815,-0.51328,-0.3535534,-0.18024,-0,-0,-0.1950903,-0.3826834,-0.5555702,-0.7071068,-0.8314696,-0.9238795,-0.9807853,-1,-0.9807853,-0.9238795,-0.8314696,-0.7071068,-0.5555702,-0.3826834,-0.1950903,-0,-0,-0.18024,-0.3535534,-0.51328,-0.6532815,-0.7681778,-0.8535534,-0.9061274,-0.9238795,-0.9061274,-0.8535534,-0.7681778,-0.6532815,-0.51328,-0.3535534,-0.18024,-0,-0,-0.1379497,-0.2705981,-0.3928475,-0.5,-0.5879378,-0.6532815,-0.6935199,-0.7071068,-0.6935199,-0.6532815,-0.5879378,-0.5,-0.3928475,-0.2705981,-0.1379497,-0,-0,-0.07465783,-0.1464466,-0.2126075,-0.2705981,-0.3181896,-0.3535534,-0.3753303,-0.3826834,-0.3753303,-0.3535534,-0.3181896,-0.2705981,-0.2126075,-0.1464466,-0.07465783,-0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0.07465783,0.1464466,0.2126075,0.2705981,0.3181896,0.3535534,0.3753303,0.3826834,0.3753303,0.3535534,0.3181896,0.2705981,0.2126075,0.1464466,0.07465783,0,0,0.1379497,0.2705981,0.3928475,0.5,0.5879378,0.6532815,0.6935199,0.7071068,0.6935199,0.6532815,0.5879378,0.5,0.3928475,0.2705981,0.1379497,0,0,0.18024,0.3535534,0.51328,0.6532815,0.7681778,0.8535534,0.9061274,0.9238795,0.9061274,0.8535534,0.7681778,0.6532815,0.51328,0.3535534,0.18024,0,0,0.1950903,0.3826834,0.5555702,0.7071068,0.8314696,0.9238795,0.9807853,1,0.9807853,0.9238795,0.8314696,0.7071068,0.5555702,0.3826834,0.1950903,0]],"it":[[0,1,2,3,4,5,6,7,8,9,10,11,12,13,14,15,17,18,19,20,21,22,23,24,25,26,27,28,29,30,31,32,34,35,36,37,38,39,40,41,42,43,44,45,46,47,48,49,51,52,53,54,55,56,57,58,59,60,61,62,63,64,65,66,68,69,70,71,72,73,74,75,76,77,78,79,80,81,82,83,85,86,87,88,89,90,91,92,93,94,95,96,97,98,99,100,102,103,104,105,106,107,108,109,110,111,112,113,114,115,116,117,119,120,121,122,123,124,125,126,127,128,129,130,131,132,133,134,136,137,138,139,140,141,142,143,144,145,146,147,148,149,150,151,153,154,155,156,157,158,159,160,161,162,163,164,165,166,167,168,170,171,172,173,174,175,176,177,178,179,180,181,182,183,184,185,187,188,189,190,191,192,193,194,195,196,197,198,199,200,201,202,204,205,206,207,208,209,210,211,212,213,214,215,216,217,218,219,221,222,223,224,225,226,227,228,229,230,231,232,233,234,235,236,238,239,240,241,242,243,244,245,246,247,248,249,250,251,252,253,255,256,257,258,259,260,261,262,263,264,265,266,267,268,269,270,0,1,2,3,4,5,6,7,8,9,10,11,12,13,14,15,17,18,19,20,21,22,23,24,25,26,27,28,29,30,31,32,34,35,36,37,38,39,40,41,42,43,44,45,46,47,48,49,51,52,53,54,55,56,57,58,59,60,61,62,63,64,65,66,68,69,70,71,72,73,74,75,76,77,78,79,80,81,82,83,85,86,87,88,89,90,91,92,93,94,95,96,97,98,99,100,102,103,104,105,106,107,108,109,110,111,112,113,114,115,116,117,119,120,121,122,123,124,125,126,127,128,129,130,131,132,133,134,136,137,138,139,140,141,142,143,144,145,146,147,148,149,150,151,153,154,155,156,157,158,159,160,161,162,163,164,165,166,167,168,170,171,172,173,174,175,176,177,178,179,180,181,182,183,184,185,187,188,189,190,191,192,193,194,195,196,197,198,199,200,201,202,204,205,206,207,208,209,210,211,212,213,214,215,216,217,218,219,221,222,223,224,225,226,227,228,229,230,231,232,233,234,235,236,238,239,240,241,242,243,244,245,246,247,248,249,250,251,252,253,255,256,257,258,259,260,261,262,263,264,265,266,267,268,269,270],[17,18,19,20,21,22,23,24,25,26,27,28,29,30,31,32,34,35,36,37,38,39,40,41,42,43,44,45,46,47,48,49,51,52,53,54,55,56,57,58,59,60,61,62,63,64,65,66,68,69,70,71,72,73,74,75,76,77,78,79,80,81,82,83,85,86,87,88,89,90,91,92,93,94,95,96,97,98,99,100,102,103,104,105,106,107,108,109,110,111,112,113,114,115,116,117,119,120,121,122,123,124,125,126,127,128,129,130,131,132,133,134,136,137,138,139,140,141,142,143,144,145,146,147,148,149,150,151,153,154,155,156,157,158,159,160,161,162,163,164,165,166,167,168,170,171,172,173,174,175,176,177,178,179,180,181,182,183,184,185,187,188,189,190,191,192,193,194,195,196,197,198,199,200,201,202,204,205,206,207,208,209,210,211,212,213,214,215,216,217,218,219,221,222,223,224,225,226,227,228,229,230,231,232,233,234,235,236,238,239,240,241,242,243,244,245,246,247,248,249,250,251,252,253,255,256,257,258,259,260,261,262,263,264,265,266,267,268,269,270,272,273,274,275,276,277,278,279,280,281,282,283,284,285,286,287,18,19,20,21,22,23,24,25,26,27,28,29,30,31,32,33,35,36,37,38,39,40,41,42,43,44,45,46,47,48,49,50,52,53,54,55,56,57,58,59,60,61,62,63,64,65,66,67,69,70,71,72,73,74,75,76,77,78,79,80,81,82,83,84,86,87,88,89,90,91,92,93,94,95,96,97,98,99,100,101,103,104,105,106,107,108,109,110,111,112,113,114,115,116,117,118,120,121,122,123,124,125,126,127,128,129,130,131,132,133,134,135,137,138,139,140,141,142,143,144,145,146,147,148,149,150,151,152,154,155,156,157,158,159,160,161,162,163,164,165,166,167,168,169,171,172,173,174,175,176,177,178,179,180,181,182,183,184,185,186,188,189,190,191,192,193,194,195,196,197,198,199,200,201,202,203,205,206,207,208,209,210,211,212,213,214,215,216,217,218,219,220,222,223,224,225,226,227,228,229,230,231,232,233,234,235,236,237,239,240,241,242,243,244,245,246,247,248,249,250,251,252,253,254,256,257,258,259,260,261,262,263,264,265,266,267,268,269,270,271,273,274,275,276,277,278,279,280,281,282,283,284,285,286,287,288],[18,19,20,21,22,23,24,25,26,27,28,29,30,31,32,33,35,36,37,38,39,40,41,42,43,44,45,46,47,48,49,50,52,53,54,55,56,57,58,59,60,61,62,63,64,65,66,67,69,70,71,72,73,74,75,76,77,78,79,80,81,82,83,84,86,87,88,89,90,91,92,93,94,95,96,97,98,99,100,101,103,104,105,106,107,108,109,110,111,112,113,114,115,116,117,118,120,121,122,123,124,125,126,127,128,129,130,131,132,133,134,135,137,138,139,140,141,142,143,144,145,146,147,148,149,150,151,152,154,155,156,157,158,159,160,161,162,163,164,165,166,167,168,169,171,172,173,174,175,176,177,178,179,180,181,182,183,184,185,186,188,189,190,191,192,193,194,195,196,197,198,199,200,201,202,203,205,206,207,208,209,210,211,212,213,214,215,216,217,218,219,220,222,223,224,225,226,227,228,229,230,231,232,233,234,235,236,237,239,240,241,242,243,244,245,246,247,248,249,250,251,252,253,254,256,257,258,259,260,261,262,263,264,265,266,267,268,269,270,271,273,274,275,276,277,278,279,280,281,282,283,284,285,286,287,288,1,2,3,4,5,6,7,8,9,10,11,12,13,14,15,16,18,19,20,21,22,23,24,25,26,27,28,29,30,31,32,33,35,36,37,38,39,40,41,42,43,44,45,46,47,48,49,50,52,53,54,55,56,57,58,59,60,61,62,63,64,65,66,67,69,70,71,72,73,74,75,76,77,78,79,80,81,82,83,84,86,87,88,89,90,91,92,93,94,95,96,97,98,99,100,101,103,104,105,106,107,108,109,110,111,112,113,114,115,116,117,118,120,121,122,123,124,125,126,127,128,129,130,131,132,133,134,135,137,138,139,140,141,142,143,144,145,146,147,148,149,150,151,152,154,155,156,157,158,159,160,161,162,163,164,165,166,167,168,169,171,172,173,174,175,176,177,178,179,180,181,182,183,184,185,186,188,189,190,191,192,193,194,195,196,197,198,199,200,201,202,203,205,206,207,208,209,210,211,212,213,214,215,216,217,218,219,220,222,223,224,225,226,227,228,229,230,231,232,233,234,235,236,237,239,240,241,242,243,244,245,246,247,248,249,250,251,252,253,254,256,257,258,259,260,261,262,263,264,265,266,267,268,269,270,271]],"primitivetype":"triangle","material":null,"normals":null,"texcoords":[[0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0.0625,0.0625,0.0625,0.0625,0.0625,0.0625,0.0625,0.0625,0.0625,0.0625,0.0625,0.0625,0.0625,0.0625,0.0625,0.0625,0.0625,0.125,0.125,0.125,0.125,0.125,0.125,0.125,0.125,0.125,0.125,0.125,0.125,0.125,0.125,0.125,0.125,0.125,0.1875,0.1875,0.1875,0.1875,0.1875,0.1875,0.1875,0.1875,0.1875,0.1875,0.1875,0.1875,0.1875,0.1875,0.1875,0.1875,0.1875,0.25,0.25,0.25,0.25,0.25,0.25,0.25,0.25,0.25,0.25,0.25,0.25,0.25,0.25,0.25,0.25,0.25,0.3125,0.3125,0.3125,0.3125,0.3125,0.3125,0.3125,0.3125,0.3125,0.3125,0.3125,0.3125,0.3125,0.3125,0.3125,0.3125,0.3125,0.375,0.375,0.375,0.375,0.375,0.375,0.375,0.375,0.375,0.375,0.375,0.375,0.375,0.375,0.375,0.375,0.375,0.4375,0.4375,0.4375,0.4375,0.4375,0.4375,0.4375,0.4375,0.4375,0.4375,0.4375,0.4375,0.4375,0.4375,0.4375,0.4375,0.4375,0.5,0.5,0.5,0.5,0.5,0.5,0.5,0.5,0.5,0.5,0.5,0.5,0.5,0.5,0.5,0.5,0.5,0.5625,0.5625,0.5625,0.5625,0.5625,0.5625,0.5625,0.5625,0.5625,0.5625,0.5625,0.5625,0.5625,0.5625,0.5625,0.5625,0.5625,0.625,0.625,0.625,0.625,0.625,0.625,0.625,0.625,0.625,0.625,0.625,0.625,0.625,0.625,0.625,0.625,0.625,0.6875,0.6875,0.6875,0.6875,0.6875,0.6875,0.6875,0.6875,0.6875,0.6875,0.6875,0.6875,0.6875,0.6875,0.6875,0.6875,0.6875,0.75,0.75,0.75,0.75,0.75,0.75,0.75,0.75,0.75,0.75,0.75,0.75,0.75,0.75,0.75,0.75,0.75,0.8125,0.8125,0.8125,0.8125,0.8125,0.8125,0.8125,0.8125,0.8125,0.8125,0.8125,0.8125,0.8125,0.8125,0.8125,0.8125,0.8125,0.875,0.875,0.875,0.875,0.875,0.875,0.875,0.875,0.875,0.875,0.875,0.875,0.875,0.875,0.875,0.875,0.875,0.9375,0.9375,0.9375,0.9375,0.9375,0.9375,0.9375,0.9375,0.9375,0.9375,0.9375,0.9375,0.9375,0.9375,0.9375,0.9375,0.9375,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1],[0,0.0625,0.125,0.1875,0.25,0.3125,0.375,0.4375,0.5,0.5625,0.625,0.6875,0.75,0.8125,0.875,0.9375,1,0,0.0625,0.125,0.1875,0.25,0.3125,0.375,0.4375,0.5,0.5625,0.625,0.6875,0.75,0.8125,0.875,0.9375,1,0,0.0625,0.125,0.1875,0.25,0.3125,0.375,0.4375,0.5,0.5625,0.625,0.6875,0.75,0.8125,0.875,0.9375,1,0,0.0625,0.125,0.1875,0.25,0.3125,0.375,0.4375,0.5,0.5625,0.625,0.6875,0.75,0.8125,0.875,0.9375,1,0,0.0625,0.125,0.1875,0.25,0.3125,0.375,0.4375,0.5,0.5625,0.625,0.6875,0.75,0.8125,0.875,0.9375,1,0,0.0625,0.125,0.1875,0.25,0.3125,0.375,0.4375,0.5,0.5625,0.625,0.6875,0.75,0.8125,0.875,0.9375,1,0,0.0625,0.125,0.1875,0.25,0.3125,0.375,0.4375,0.5,0.5625,0.625,0.6875,0.75,0.8125,0.875,0.9375,1,0,0.0625,0.125,0.1875,0.25,0.3125,0.375,0.4375,0.5,0.5625,0.625,0.6875,0.75,0.8125,0.875,0.9375,1,0,0.0625,0.125,0.1875,0.25,0.3125,0.375,0.4375,0.5,0.5625,0.625,0.6875,0.75,0.8125,0.875,0.9375,1,0,0.0625,0.125,0.1875,0.25,0.3125,0.375,0.4375,0.5,0.5625,0.625,0.6875,0.75,0.8125,0.875,0.9375,1,0,0.0625,0.125,0.1875,0.25,0.3125,0.375,0.4375,0.5,0.5625,0.625,0.6875,0.75,0.8125,0.875,0.9375,1,0,0.0625,0.125,0.1875,0.25,0.3125,0.375,0.4375,0.5,0.5625,0.625,0.6875,0.75,0.8125,0.875,0.9375,1,0,0.0625,0.125,0.1875,0.25,0.3125,0.375,0.4375,0.5,0.5625,0.625,0.6875,0.75,0.8125,0.875,0.9375,1,0,0.0625,0.125,0.1875,0.25,0.3125,0.375,0.4375,0.5,0.5625,0.625,0.6875,0.75,0.8125,0.875,0.9375,1,0,0.0625,0.125,0.1875,0.25,0.3125,0.375,0.4375,0.5,0.5625,0.625,0.6875,0.75,0.8125,0.875,0.9375,1,0,0.0625,0.125,0.1875,0.25,0.3125,0.375,0.4375,0.5,0.5625,0.625,0.6875,0.75,0.8125,0.875,0.9375,1,0,0.0625,0.125,0.1875,0.25,0.3125,0.375,0.4375,0.5,0.5625,0.625,0.6875,0.75,0.8125,0.875,0.9375,1]]}});
testglrgl.prefix = "testgl";
</script>
<p id="testgldebug">
You must enable Javascript to view this page properly.</p>
<script>testglrgl.start();</script>

Note how the model plane fits relatively well around the lower section of the income scale but given a handful of outliers, we end up having a relatively weak linear relationship. Let’s now - to keep in line with the objective of this example - simply apply the model onto the testing dataset (those observations we’ve previously reserved to test how well the model would fit on unused data):



```r
# Predict on the testing dataset and calculate Root Mean Squared Error. 
sqrt(mean((newdata$income-predict(mod1, newdata))[-trainRows]^2))
```

```
## [1] 2804.979
```

Note from the above code that we use the predict function to apply the model onto the testing set. The results calculate the Root Mean Squared Error (RMSE). The Root Mean Squared Error is simply the square root of the average squared errors found between the actual data point and the model fitted values. Taking the square root allows the error to have the same units as the quantity being estimated in the Y axis which yields a more easily interpretable result. The Root Mean Squared Error measure tends to be influenced by variances due to outliers (which is our case here since we got a RMSE of **$2,805**). RMSE is most useful when large errors are not desired. We could have used another measure such as **Mean Absolute Error** which measures the average value of the errors between the prediction and the actual data giving similar weight to all error values.

So up to now, we’ve seen that the Validation Set approach provides a simple estimate of the test error rate (applying the model on previously unseen data). However, if we run the training and testing split with a different set of data for each sample, we will obtain somewhat different errors on the testing set. Let’s see how this plays out in the dataset. We’ll create a *for loop* to generate 100 sample sets for train and test with the model and see how the Root Mean Squared Error varies:


```r
set.seed(5)

rmse=c()

for(i in 1:100){
  train = sample(1:nrow(newdata),0.5*nrow(newdata))
  test = setdiff(1:nrow(newdata),train)
    mods <- lm(income ~ prestige.c + women.c, data=newdata[train, ])
    preds = predict(mods,newx=newdata[test,])
    rmse[i] = sqrt(mean((newdata$income-predict(mods, newdata))[test]^2))
    # cat(rmse[length(rmse)],'\n') # you can uncomment this line if you want to see all values printed out.
}

mean(rmse)
```

```
## [1] 2597.934
```

```r
# Plot histogram of scores.
hist(rmse, density=35, main = "Test RMSE over 100 Samples", xlab = "Value of Obtained RMSE", col="blue", border="black")
abline(v=mean(rmse), lwd=3, col="red")
```

<span class="image fit"><img src="{{ "/images/ResamplingValidationSet_files/figure-html/unnamed-chunk-8-1.png" | absolute_url }}" alt="" /></span>

The histogram above highlights (the vertical red line) the average RMSE across 100 different samples as well as the spread in which the RMSE can reach. The RMSE of the income variable we got from this resampling example ranges from **$1,606** to **$3,695**. These are moderately high RMSE values.

These results ultimately show us how the linear model fit is not appropriate in this case. It also shows that despite the fact the validation set approach is simple to implement, it can be highly variable depending on which observations are part of either the training or the testing datasets.

From here we could try to include a quadratic term on the predictors and a transformation for the target variable income - as we did in previous examples - to see how better the model would fit the data. But for brevity sake, we’ll leave it as is for now.

***

In future examples, we’ll see other techniques such as **cross-validation** and **bootstrap** which attempt to address the issues of variability in the training / testing sample due to the existence of fewer observations in which the model is trained on. We’ll also illustrate an example of a **classification** problem.
