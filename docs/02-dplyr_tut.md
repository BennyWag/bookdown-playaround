# `dplyr` - A brief introduction to tidy data manipulation



<img src="data/dplyr-doc/dplyr.png" width="250" style="display: block; margin: auto;" />


## Introducing the koala dataset

Today we will have a look at some koala data. You can download the dataset for this tutorial [here](https://www.dropbox.com/s/1zhru8ui3btmg0n/koala.csv?dl=1). Futhermore you need to install the `tidyverse` package, which contains `dplyr`. 


```r
install.packages('tidyverse')
```

First we need to load the dataset we're working with.


```r
koala<-read.csv('data/koala.csv')
```

It should contain the following colums:


```r
names(koala)
```

```
##  [1] "species" "X"       "Y"       "state"   "region"  "sex"     "weight" 
##  [8] "size"    "fur"     "tail"    "age"     "color"   "joey"    "behav"  
## [15] "obs"
```

Lets look at a structure and summary of this dataset:


```r
str(koala)
```

```
## 'data.frame':	242 obs. of  15 variables:
##  $ species: Factor w/ 1 level "Phascolarctos cinereus": 1 1 1 1 1 1 1 1 1 1 ...
##  $ X      : num  153 148 153 153 153 ...
##  $ Y      : num  -27.5 -22.5 -27.5 -27.5 -27.5 ...
##  $ state  : Factor w/ 4 levels "New South Wales",..: 2 2 2 2 2 2 2 2 2 2 ...
##  $ region : Factor w/ 2 levels "northern","southern": 1 1 1 1 1 1 1 1 1 1 ...
##  $ sex    : Factor w/ 2 levels "female","male": 2 1 2 2 1 2 2 2 1 1 ...
##  $ weight : num  7.12 5.45 6.63 6.47 5.62 ...
##  $ size   : num  70.8 70.4 68.7 73 65.2 ...
##  $ fur    : num  1.86 1.85 2.48 1.92 1.95 ...
##  $ tail   : num  1.17 1.56 1.06 1.8 1.63 ...
##  $ age    : int  8 10 1 1 10 12 9 1 1 1 ...
##  $ color  : Factor w/ 6 levels "chocolate brown",..: 3 4 6 3 4 4 6 4 3 3 ...
##  $ joey   : Factor w/ 2 levels "No","Yes": 1 2 1 1 1 1 1 1 1 1 ...
##  $ behav  : Factor w/ 3 levels "Feeding","Just Chillin",..: 3 3 2 3 3 1 2 3 1 3 ...
##  $ obs    : Factor w/ 3 levels "Opportunistic",..: 2 1 2 3 3 1 3 2 2 2 ...
```

```r
summary(koala)
```

```
##                    species          X               Y         
##  Phascolarctos cinereus:242   Min.   :138.6   Min.   :-39.00  
##                               1st Qu.:150.0   1st Qu.:-34.49  
##                               Median :152.0   Median :-32.67  
##                               Mean   :150.3   Mean   :-32.36  
##                               3rd Qu.:152.9   3rd Qu.:-30.31  
##                               Max.   :153.6   Max.   :-21.39  
##              state          region        sex          weight      
##  New South Wales:181   northern:165   female:127   Min.   : 5.406  
##  Queensland     : 16   southern: 77   male  :115   1st Qu.: 6.574  
##  South Australia: 14                               Median : 7.277  
##  Victoria       : 31                               Mean   : 7.923  
##                                                    3rd Qu.: 8.765  
##                                                    Max.   :17.889  
##       size            fur             tail            age       
##  Min.   :64.81   Min.   :1.110   Min.   :1.004   Min.   : 1.00  
##  1st Qu.:68.43   1st Qu.:2.410   1st Qu.:1.272   1st Qu.: 3.00  
##  Median :70.27   Median :2.797   Median :1.534   Median : 7.00  
##  Mean   :70.94   Mean   :2.896   Mean   :1.507   Mean   : 6.43  
##  3rd Qu.:72.33   3rd Qu.:3.217   3rd Qu.:1.750   3rd Qu.: 9.00  
##  Max.   :81.91   Max.   :5.876   Max.   :1.981   Max.   :12.00  
##              color     joey              behav                obs    
##  chocolate brown:21   No :185   Feeding     : 48   Opportunistic:65  
##  dark grey      :36   Yes: 57   Just Chillin: 67   Spotlighting :94  
##  grey           :69             Sleeping    :127   Stagwatching :83  
##  grey-brown     :53                                                  
##  light brown    :20                                                  
##  light grey     :43
```

We can see that our dataset contains the positions of each koala in Latitude and Longitude (`X` and `Y`) as well as variables describing their physiology, behavior and how they were recorded. This is typical presence-only wildlife data, combining observations with some data describing each individual, which could e.g. be used for distribution modeling or to test influences of other variables such as climate on behavior and physiology of this particular speces. Often in these types of studies, we are not interested in all the recorded variables and thus first need to 'clean' our data to make it easier to work with it. `dplyr` is a package desinged to make data 'cleaning' and manipulation of large datasets easier by introducing specific syntax. Let's see how it works and compares to base `R` functionality!

## Working with `dplyr`

Base `R` subsetting can be very tedious. Imagine we want to got the mean age for our koalas, but split it by sex. Getting one mean is easy:


```r
mean(koala[koala$sex == 'male', "age"],na.rm = TRUE)
```

```
## [1] 6.626087
```

Summarizing both sexes and savinf it in a table takes a few lines of code:


```r
female_mean<-mean(koala[koala$sex == 'female', "age"],na.rm = TRUE)

male_mean<-mean(koala[koala$sex == 'male', "age"],na.rm = TRUE)

means<-rbind(c(female_mean, male_mean))

means<-as.data.frame(means)

names(means)<-c('female', 'male')
```

Lets have a look at the results.


```r
means
```

```
##     female     male
## 1 6.251969 6.626087
```

That's all good, but with that many lines of code quite error prone ... `dplyr` makes data manipulation simpler. For this example, we would only require one line of code!


```r
library(dplyr)

mean_age_koala<-koala%>%group_by(sex)%>%summarise(mean_age = mean(age))
```


```
## # A tibble: 2 x 2
##   sex    mean_age
##   <fct>     <dbl>
## 1 female     6.25
## 2 male       6.63
```

So simple, and looks even better than our base `R` table too! The main functions we will explore here are `dplyr`'s pipe `%>%`, `select()`, `filter()`, `group_by()`, `summarise()` and `mutate()`.

### select() and dplyr's pipe

If, for example, we wanted to move forward with only a few of the variables in our dataframe we could use the `select()` function. This will keep only the variables you select.


```r
koala_select<-select(koala, species, sex, age)
```


```
##                  species    sex age
## 1 Phascolarctos cinereus   male   8
## 2 Phascolarctos cinereus female  10
## 3 Phascolarctos cinereus   male   1
## 4 Phascolarctos cinereus   male   1
## 5 Phascolarctos cinereus female  10
## 6 Phascolarctos cinereus   male  12
```

If we open up `koala_select` we'll see that it only contains the species, sex and age columns. Above we used 'normal' `R` grammar, but the strengths of dplyr lie in combining several functions using pipes. Since the pipes grammar is unlike anything we've seen in R before, let's repeat what we've done above using pipes.


```r
koala_select_pipe<-koala%>%select(species, sex, age)
```

To help you understand why we wrote that in that way, let's walk through it step by step. First we summon the koala dataframe and pass it on, using the pipe syntax `%>%`, to the next step, which is the `select()` function. In this case we don't specify which data object we use in the `select()` function since in gets that from the previous pipe.

### filter()

`filter()` is one of the most useful dplyr functions for data manipulation. Say you're conducting a study of only male koalas. You won't need any data on female koalas. So lets get rid of it!


```r
koala_filter<-koala%>%filter(sex == 'male')
```

Did it work?


```r
summary(koala_filter$sex)
```

```
## female   male 
##      0    115
```

No more females in the data! Let's test our knowledge with a challenge.

### Challenge 1

**Write a single command (which can span multiple lines and includes pipes) that will produce a dataframe that has the values for age, size and color for females only. How many rows and columns does your dataframe have and why?**

*Extra challenge: out of this new dataset, filter only koalas >70cm in size. How many are there?*



This should be your data structure:


```r
nrow(challenge1)
```

```
## [1] 127
```

```r
ncol(challenge1)
```

```
## [1] 3
```

We removed all the males, so our row number reduces from 242 to 127. Then we filter our desired columns and are now at 3 instead of 15.




```r
nrow(challenge1.2)
```

```
## [1] 46
```

You can find the solutions to all challenges posed here at the end of the document. Don't peek!

### group_by() and summarise()

Now, we were supposed to be reducing the error prone repetitiveness of what can be done with base R, but up to now  we haven't done that since we would have to repeat the above for each sex. Instead of `filter()`, which will only pass observations that meet your criteria (in the above: `sex=="female"`), we can use `group_by()`, which will essentially use every unique criteria that you could have used in `filter()`. Let's see what happens with our data structure when using dplyr's `group_by()`.


```r
koala_group<-koala%>%group_by(sex)

str(koala_group)
```

```
## tibble [242 x 15] (S3: grouped_df/tbl_df/tbl/data.frame)
##  $ species: Factor w/ 1 level "Phascolarctos cinereus": 1 1 1 1 1 1 1 1 1 1 ...
##  $ X      : num [1:242] 153 148 153 153 153 ...
##  $ Y      : num [1:242] -27.5 -22.5 -27.5 -27.5 -27.5 ...
##  $ state  : Factor w/ 4 levels "New South Wales",..: 2 2 2 2 2 2 2 2 2 2 ...
##  $ region : Factor w/ 2 levels "northern","southern": 1 1 1 1 1 1 1 1 1 1 ...
##  $ sex    : Factor w/ 2 levels "female","male": 2 1 2 2 1 2 2 2 1 1 ...
##  $ weight : num [1:242] 7.12 5.45 6.63 6.47 5.62 ...
##  $ size   : num [1:242] 70.8 70.4 68.7 73 65.2 ...
##  $ fur    : num [1:242] 1.86 1.85 2.48 1.92 1.95 ...
##  $ tail   : num [1:242] 1.17 1.56 1.06 1.8 1.63 ...
##  $ age    : int [1:242] 8 10 1 1 10 12 9 1 1 1 ...
##  $ color  : Factor w/ 6 levels "chocolate brown",..: 3 4 6 3 4 4 6 4 3 3 ...
##  $ joey   : Factor w/ 2 levels "No","Yes": 1 2 1 1 1 1 1 1 1 1 ...
##  $ behav  : Factor w/ 3 levels "Feeding","Just Chillin",..: 3 3 2 3 3 1 2 3 1 3 ...
##  $ obs    : Factor w/ 3 levels "Opportunistic",..: 2 1 2 3 3 1 3 2 2 2 ...
##  - attr(*, "groups")= tibble [2 x 2] (S3: tbl_df/tbl/data.frame)
##   ..$ sex  : Factor w/ 2 levels "female","male": 1 2
##   ..$ .rows:List of 2
##   .. ..$ : int [1:127] 2 5 9 10 12 13 15 17 20 22 ...
##   .. ..$ : int [1:115] 1 3 4 6 7 8 11 14 16 18 ...
##   ..- attr(*, ".drop")= logi TRUE
```


You will notice that the structure of the dataframe where we used `group_by()` (`koala_group`) is not the same as the original `koala` dataset. A grouped dataset can be thought of as a list where each item in the list is a data.frame which contains only the rows that correspond to the a particular value 'Sex' (at least in the example above).

The above was a bit on the uneventful side because `group_by()` is only really useful in conjunction with `summarise()`. This will allow you to create new variable(s) by using functions that repeat for each of the  sex-specific data frames. That is to say, using the `group_by()` function, we split our original dataframe into multiple pieces, then we can run functions such as `mean()` or `sd()` within `summarise()`:


```r
koala_group_sum<-koala%>%group_by(sex)%>%
  summarise(mean_age=mean(age))
```


```
## # A tibble: 2 x 2
##   sex    mean_age
##   <fct>     <dbl>
## 1 female     6.25
## 2 male       6.63
```

And there we go. We got what we wanted and summarised the mean age of our koalas for both sexes separately. And we did that using only one simple line of code! I think it is time for another challenge to test our skills!

### Challenge 2

**Calculate the average weight value per state and Sex. Which combination of state and sex has the heaviest and which combination had the lightest koalas?**




```
## # A tibble: 8 x 3
## # Groups:   state [4]
##   state           sex    mean_weight
##   <fct>           <fct>        <dbl>
## 1 New South Wales female        6.54
## 2 New South Wales male          9.07
## 3 Queensland      female        5.68
## 4 Queensland      male          6.82
## 5 South Australia female        7.59
## 6 South Australia male         16.8 
## 7 Victoria        female        7.58
## 8 Victoria        male          7.48
```

That is already quite powerful, but it gets even better! You're not limited to defining only one new variable in `summarise()`:


```r
challenge2_ext<-koala%>%group_by(state, sex)%>%
  summarise(mean_weight = mean(weight),
            sd_weight = sd(weight),
            sample_no = n())
```

We can create a new dataframe with as many new variables as we want. Very useful for our inital data exploration! Let's get our hands another very useful function: `mutate()`.

### mutate()

We can  create an entirely new variables in our initial dataset prior to (or even after) summarizing information using `mutate()`. Let's say we're interested in the weight:size ratio of our Koalas. Also we want to give each individual a numeric identifier to be able to better work with our data later on.


```r
koala_mutate<-koala%>%mutate(weight_size_ratio = size/weight, ID = row_number())
```


```
##  [1] "species"           "X"                 "Y"                
##  [4] "state"             "region"            "sex"              
##  [7] "weight"            "size"              "fur"              
## [10] "tail"              "age"               "color"            
## [13] "joey"              "behav"             "obs"              
## [16] "weight_size_ratio" "ID"
```

Our dataset now has two extra columns containing the variables we were interested in. If you do not want to manipulate your raw data, you can use mutate before grouping and summarising to create the summary table straight away:


```r
koala_mutate_weight_size<-koala%>%mutate(weight_size_ratio = size/weight)%>%
  group_by(sex)%>%
  summarise(mean_weight = mean(weight),
            sd_weight = sd(weight),
            mean_weight_size = mean (weight_size_ratio),
            max_weight_size = max(weight_size_ratio))
```


```
## # A tibble: 2 x 5
##   sex    mean_weight sd_weight mean_weight_size max_weight_size
##   <fct>        <dbl>     <dbl>            <dbl>           <dbl>
## 1 female        6.67     0.533            10.4             12.9
## 2 male          9.31     2.38              8.21            11.9
```

Great! Let's end the lesson with another challenge, combining all the functions we have looked at today.

### Challenge 3

***Calculate the average tail length and fur thickness for a group of 20 randomly selected males and females from New South Wales. Then arrange the mean tail length in descending order.*** 

*Hint: Use the dplyr functions* `arrange()` *and* `sample_n()`*, they have similar  syntax to other dplyr functions. Look at the help by calling '?function', e.g.* `?arrange`*.* 




```
## # A tibble: 2 x 3
##   sex    mean_tail mean_fur
##   <fct>      <dbl>    <dbl>
## 1 male        1.47     2.94
## 2 female      1.41     2.41
```

Since we are sampling randomly, these will look different for each of you.

### Solution to all challenges

#### Challenge 1


```r
challenge1<-koala%>%filter(sex == 'female')%>%
  select(age, size, color)
```


```r
challenge1.2<-challenge1%>%filter(size>70)
```

#### Challenge 2


```r
challenge2<-koala%>%group_by(state, sex)%>%
  summarise(mean_weight = mean(weight))
```

#### Challenge 3


```r
challenge3<-koala%>%filter(state == 'New South Wales')%>%
  group_by(sex)%>%
  sample_n(20)%>%
  summarise(mean_tail = mean(tail), mean_fur = mean(fur))%>%
  arrange(desc(mean_tail))
```

