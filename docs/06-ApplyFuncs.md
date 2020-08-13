# Functions, loops, and `apply`
## Introduction

Most data analysis in R requires executing a particular task on multiple data observations, groups of observations, or separate data sets. Many novice users begin with the strategy of assigning a new variable for each input:




```r
x1 <- dataset1
x2 <- dataset2

meanx1 <- mean(x1)
meanx2 <- mean(x2)
```

While this certainly works, it can be very time consuming and introduces substantial risk of user error. There are several alternative strategies that can be used to repeat mathematical operations or sequences of R functions for multiple variables or data sets that avoid this risk.

The foundation for any of these strategies is a function to be repeated. Once you have a function that you would like to repeat on multiple inputs, there are a few approaches for iterating through each repetition.

In this tutorial, we will focus on the `apply` functions, which are a set of base R functions that can be used to execute a particular chunk of code on multiple inputs and generate appropriate outputs. The `apply` functions are simple to use and create very neat code, but one limitation is that they can only be used in operations for which the order of iteration is not important.

We will also introduce `for` and `while` loops, which can be used when the order of operations *does* matter.  
  
    
## Writing basic functions

Functions in R take an input value, apply a process, and generate an output. R packages include numerous functions that can do nearly anything you desire, but research often requires specific processes or sequences of commands designed specifically to work with your data structure. You can write your own functions in R to perform these tasks.

Here is a very simple function:


```r
fun1 <- function (x, y){
  x + y
}
```

Here, `x` and `y` are identified as the required input variables. The code written within the brackets describes the mathematical operation to be applied to the input variables.


```r
fun1(x = 7, y = 12)
```

```
## [1] 19
```
  
  
A function can be applied to a vector of inputs. In this scenario, the function will be applied to each pair of values in the vector sequence:


```r
vals_x <- c(4, 7, 1)
vals_y <- c(2, 1, 3)

fun1(x = vals_x, y = vals_y)
```

```
## [1] 6 8 4
```
#### Nesting functions

Functions can also include other functions, which is particularly useful in the context of repeating complex data analyses. Here is a basic example:


```r
fun2 <- function (x, y){
  mean(x) + y
}


fun2(x = vals_x, y = vals_y)
```

```
## [1] 6 5 7
```

It is important to note that in this situation, the internal function (`mean()`) takes a vector of input values. This function will calculate the mean of the `vals_x` vector and then add this to each separate value of `vals_y`.

This can be very useful, but it's critical that you know exactly how your function processes input values or you may run into major problems!

#### Assigning variables within functions and `return()`

In some more complex analyses, the nested functions may generate data that needs to be stored as variables so they can be used as input for the next process. Here is a function that calculates the mean of our x values, adds y, and then generates 10 numbers from a normal distribution with a mean of the value calculated in the first step.


```r
fun3 <- function (x, y){
  val <- mean(x) + y
  dist <- rnorm(1:10, mean = val, sd = 1)
}


output <- fun3(x = vals_x, y = 3)
output
```

```
##  [1] 5.424814 6.691232 8.002993 7.730695 7.117511 7.268080 5.686973 8.485264
##  [9] 6.356877 6.405854
```

If your function assigns multiple variables, you can use `return()` to determine the data that is output from the function. This can be useful if your function includes writing a data set, or includes another process that is not intended to return anything to R as well as some process intended to return data to the R environment.


```r
dir.create('outputs')
```


```r
#Let's see what happens without using the return() function
fun4 <- function (x, y){
  val <- mean(x) + y
  dist <- rnorm(1:10, mean = x, sd = 1)
  write.csv(dist, "outputs/filename.csv")
}


output <- fun4(x = vals_x, y = 3)
output
```

```
## NULL
```

Our output here is NULL because the default output for a function will come from the last command within the function. Here, `write.csv()` does not produce data to be returned to R. If we add `return()`, our function will write the .csv and also return the data we want into the environment:


```r
#Now let's add return()
fun4 <- function (x, y){
  val <- mean(x) + y
  dist <- rnorm(1:10, mean = x, sd = 1)
  write.csv(dist, "outputs/filename.csv")
  return(dist)
}


output <- fun4(x = vals_x, y = 3)
output
```

```
##  [1] 4.3102801 7.1364540 0.4559926 4.1286090 7.9435964 2.2381301 3.1237081
##  [8] 7.0596780 2.1325763 4.1591631
```

Much better!

## Using `apply` functions

Once you have written a function, you will likely need to execute that function on multiple data sets or variables. This is where the `apply` functions come in handy.

The `apply` functions are a set of functions that can be used to, well, apply functions to lists, dataframes, and matrices. `apply` functions are similar to loops, but have a much simpler syntax.

There are several different functions that can be used depending upon the inputs and desired outputs. We will work through the following `apply` functions:

1. `apply` - used with arrays, including matrices, and returns a vector, array, or list.
2. `sapply` - (simplified apply) applies the function to each element of a vector, list, or matrix, and returns the outputs as a vector or matrix.
3. `lapply` - (list apply) is similar to `sapply`, but returns a list of objects, rather than a vector.
4. `mapply` - (multivariate apply) applies a function to the first elements of two or more vectors or lists, and then the second pair of elements, etc..

If this doesn't make much sense, don't stress! We will walk though an example of each function and situations for which they might be useful.

### `apply`

Let's generate a theoretical transition matrix for a "before" and "after" data set that shows the number of transitions from one type to another.


```r
before <- rpois(1000, lambda = 1)
after <- rpois(1000, lambda = 1)
tab <- table(before, after)
tab
```

```
##       after
## before   0   1   2   3   4   5
##      0 132 130  92  16   8   1
##      1 141 139  69  27   5   1
##      2  65  66  33   9   2   0
##      3  17  15   9   2   0   0
##      4   4  10   4   0   0   0
##      5   0   0   0   1   1   0
##      6   0   1   0   0   0   0
```

We would like to convert this table to the relative probability for each transition- that is, what is the probability that an object of type "0" in the "before" dataset transitions to a 1? To a 2? 3?

To calculate the probability, we will need to get the sum of observations in each row, and then divide each transition in that particular row by the row sum. We can do this using the `apply` function. 

`apply` has three arguments: `X` is the array to which the function will be applied. `MARGIN` is used to identify whether the function should be applied to the rows, columns, or both. '1' is for rows, '2' is for columns, and c(1,2) can be used for both. `FUN` is the function to be applied.

In this scenario, we set `MARGIN = 1` so it will perform the function row wise. Up to this point, we have defined our functions outside of other commands, but in this scenario, it is a very simple function and so we can define the function within the command itself.

*Note: The `t()` function surrounding the command transposes the data set so it returns the matrix in the original orientation (before as rows and after as columns)


```r
rel_prob <- t(apply(tab, MARGIN = 1, FUN = function (i) {i/sum(i)} ))
rel_prob
```

```
##       after
## before         0         1         2          3          4           5
##      0 0.3482850 0.3430079 0.2427441 0.04221636 0.02110818 0.002638522
##      1 0.3691099 0.3638743 0.1806283 0.07068063 0.01308901 0.002617801
##      2 0.3714286 0.3771429 0.1885714 0.05142857 0.01142857 0.000000000
##      3 0.3953488 0.3488372 0.2093023 0.04651163 0.00000000 0.000000000
##      4 0.2222222 0.5555556 0.2222222 0.00000000 0.00000000 0.000000000
##      5 0.0000000 0.0000000 0.0000000 0.50000000 0.50000000 0.000000000
##      6 0.0000000 1.0000000 0.0000000 0.00000000 0.00000000 0.000000000
```

Hurray! Please not that there are other ways to perform the same task, but this one is a conceptually a simple example :)

### `sapply`

Let's check out the Loblolly dataset. It contains the height, age, and seed source of a sample of Loblolly pines.


```r
library(datasets)

data(Loblolly)
head(Loblolly)
```

```
##    height age Seed
## 1    4.51   3  301
## 15  10.89   5  301
## 29  28.72  10  301
## 43  41.74  15  301
## 57  52.70  20  301
## 71  60.92  25  301
```

The data set includes a range of ages. Let's see if we can write a function that calculates the mean height for each age group.


```r
#Create a list with each unique age group
ages <- unique(Loblolly$age)
ages
```

```
## [1]  3  5 10 15 20 25
```


```r
#Create a function that calculates the mean height for each age group
fun5 <- function (x){
  agegroup <- subset(Loblolly, age == x)
  mean_height <- mean(agegroup$height)
  return(mean_height)
}

fun5(ages)
```

```
## [1] 32.3644
```

Hmmm, this returns one value rather than a separate mean for each group. Why is that?
In this function, x is a vector, and so the subset function selects the cases where age is equal to any of the values included in the `ages` vector. Because we have selected every value of age, we will get the mean height for the entire dataset.

If we would like a vector of the means for each height, we will can use `sapply`.


```r
#Create a function that calculates the mean height for each age group
mean_heights <- sapply(X = ages, FUN = fun5)
mean_heights
```

```
## [1]  4.237857 10.205000 27.442143 40.543571 51.468571 60.289286
```

Excellent! We now have a vector with the mean height for each age group!

### `lapply`

You can use the `lapply` function to get the same information, but in a list format.


```r
heights <- lapply(X = ages, FUN = fun5)
heights
```

```
## [[1]]
## [1] 4.237857
## 
## [[2]]
## [1] 10.205
## 
## [[3]]
## [1] 27.44214
## 
## [[4]]
## [1] 40.54357
## 
## [[5]]
## [1] 51.46857
## 
## [[6]]
## [1] 60.28929
```

In this scenario, `sapply` might be a better fit. `lapply` is generally more useful if your output is a an object like a `data.frame` or produces objects of different types.

Let's write a function that creates a subsetted `data.frame` for each value of tree age.


```r
ages <- unique(Loblolly$age)

func6 <- function (x){
  subset(Loblolly, age == x)
}

sub_df <- lapply(ages, func6)

#Let's look at items 1 and 2 in the list
sub_df[1:2]
```

```
## [[1]]
##    height age Seed
## 1    4.51   3  301
## 2    4.55   3  303
## 3    4.79   3  305
## 4    3.91   3  307
## 5    4.81   3  309
## 6    3.88   3  311
## 7    4.32   3  315
## 8    4.57   3  319
## 9    3.77   3  321
## 10   4.33   3  323
## 11   4.38   3  325
## 12   4.12   3  327
## 13   3.93   3  329
## 14   3.46   3  331
## 
## [[2]]
##    height age Seed
## 15  10.89   5  301
## 16  10.92   5  303
## 17  11.37   5  305
## 18   9.48   5  307
## 19  11.20   5  309
## 20   9.40   5  311
## 21  10.43   5  315
## 22  10.57   5  319
## 23   9.03   5  321
## 24  10.79   5  323
## 25  10.48   5  325
## 26   9.92   5  327
## 27   9.34   5  329
## 28   9.05   5  331
```

### `mapply`

`mapply` is an extremely useful function that is a great way to avoid using `for` loops! With `mapply`, you can use multiple input variables and it will iterate through each pair (or trio or more) of variables as inputs for the function.

Let's say we would like to generate a theoretical normal distribution of tree height for each age group based on the mean and standard deviation of height. We will need a vector of means and a vector of heights to use as input values. We have already genereated a vector of means in our `sapply` example, so we will use a similar equation to generate a vector of sd.


```r
fun6 <- function (x){
  agegroup <- subset(Loblolly, age == x)
  sd(agegroup$height)
}

sd_height <- sapply(ages, fun6)
sd_height
```

```
## [1] 0.4036026 0.8155767 1.5378866 1.9508444 2.2118278 2.2688339
```

Now, we can write a function that takes the two variables and applied it to each pair of input values to generate 6 six distributions.


```r
fun7 <- function (x, y){
  rnorm(100, mean = x, sd = y)
}

height_dists <- mapply(FUN = fun7, x = mean_heights, y = sd_height)
colnames(height_dists) <- ages #This names the columns by the tree age
summary(height_dists)
```

```
##        3               5                10              15       
##  Min.   :3.296   Min.   : 7.242   Min.   :24.45   Min.   :35.67  
##  1st Qu.:4.032   1st Qu.: 9.521   1st Qu.:26.48   1st Qu.:38.90  
##  Median :4.240   Median :10.225   Median :27.57   Median :40.37  
##  Mean   :4.266   Mean   :10.211   Mean   :27.69   Mean   :40.51  
##  3rd Qu.:4.493   3rd Qu.:10.906   3rd Qu.:28.73   3rd Qu.:42.23  
##  Max.   :5.225   Max.   :12.192   Max.   :32.02   Max.   :44.45  
##        20              25       
##  Min.   :46.44   Min.   :54.12  
##  1st Qu.:49.92   1st Qu.:58.51  
##  Median :51.32   Median :59.89  
##  Mean   :51.33   Mean   :60.12  
##  3rd Qu.:52.88   3rd Qu.:61.92  
##  Max.   :55.47   Max.   :64.76
```

## Loops

I haven't done this part yet!
