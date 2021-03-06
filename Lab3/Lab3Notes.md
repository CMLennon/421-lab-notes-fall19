---
Title: Lab 3 Homework Help
date: 20 April 2019
output:
  html_document:
    highlight: haddock
    keep_md: yes
    theme: flatly
    toc: yes
    toc_depth: 4
    toc_float: yes
---



## Lesson 0: Introduction

Hi guys! Welcome to week three. This week, we're going to cover directly applicable material: how to set up and run 
the R code for your homework. In particular, we'll cover some of the more complex coding tasks.

We'll be using a bunch of material that you guys have seen before. We'll be:
 - loading packages, 
 - running regressions (with a twist)
 - And doing some analysis/thinking on those things. Sound familiar?

## Lesson 1: Homework Essentials (Loading the data)

If you guys used your own machines, or are using lab computers, then you won't need to do this, but let's just go over it.
Our first step is to install the tidyverse, or if you have done that already, load it to our workspace.

If you haven't yet installed `pacman` then do that now by running the following line:
`install.packages("pacman", dependencies = TRUE, repos = "https://ftp.osuosl.org/pub/cran/")`

As we discussed earlier, using pacman simplifies our life, but we have to install it first.

```r
#to load pacman to our workspace, we need to use the core R library command.
library(pacman)
```

In `pacman`, we will load three packages, tidyverse, broom and magrittr. The nice thing about p_load is that this will install and load the packages to your workspace.


```r
p_load(tidyverse, broom, magrittr)
```

Some people in lab had a hard time getting pacman installed or loaded. If that happens to you, then simply `install.packages` just like we did with pacman initially, and then load them to the workspace.


```r
install.packages(c("tidyverse","magrittr"), dependencies = TRUE)
library(tidyverse,magrittr)
```

Now, we need to load the data. This has been provided to you on the canvas page, and your course's github. Once this is downloaded, Let's create a path string, like we did last lab.


```r
#let's store a character object that contains our path so we can load our data. 
#We could simply pass "/Users/connor/Dropbox/EC421_Lab/Week3/data/dataPS01.csv" but this looks cleaner IMO
my_path = "/Users/connor/Dropbox/EC421_Lab/Week3/data/dataPS01.csv"
```


Now, we are going to attach our csv to a dataframe object in R. Don't forget to give your data a name! you pick.


```r
dataPS01 <- read_csv(my_path)
```

```
## Parsed with column specification:
## cols(
##   i_callback = col_double(),
##   n_jobs = col_double(),
##   n_expr = col_double(),
##   i_military = col_double(),
##   i_computer = col_double(),
##   first_name = col_character(),
##   sex = col_character(),
##   i_female = col_logical(),
##   i_male = col_logical(),
##   race = col_character(),
##   i_black = col_logical(),
##   i_white = col_logical()
## )
```

Let's get a good look at the size of our dataset. We can do that with the dim command, which will return `row count, column count`


```r
dim(dataPS01)
```

```
## [1] 4870   12
```

If you look at the homework, you'll see we are needing to create dummy variables in order to look at between group
effects. Let's see what we need to do.

## Lesson 2: Creating Indicators

We will create a set of dummy variables for a couple of different 'classes'
- female
- black
- female and black

We spent a bit of time last lab talking about dummy variables, but let's walk through how to set this up

Let's take a look at what our dataframe looks like. Yep. Back to handy dandy head()


```r
head(dataPS01)
```

```
## # A tibble: 6 x 12
##   i_callback n_jobs n_expr i_military i_computer first_name sex   i_female
##        <dbl>  <dbl>  <dbl>      <dbl>      <dbl> <chr>      <chr> <lgl>   
## 1          0      2      6          0          1 Allison    f     TRUE    
## 2          0      3      6          1          1 Kristen    f     TRUE    
## 3          0      1      6          0          1 Lakisha    f     TRUE    
## 4          0      4      6          0          1 Latonya    f     TRUE    
## 5          0      3     22          0          1 Carrie     f     TRUE    
## 6          0      2      6          0          0 Jay        m     FALSE   
## # … with 4 more variables: i_male <lgl>, race <chr>, i_black <lgl>,
## #   i_white <lgl>
```

Huh. So our data is stored as two different characters, in two different columns. We can use the tidyverse to create
nice 0,1 variables to use.

Mutate is a tidyverse function can be passed MULTIPLE new commands to create multiple new columns. This will be handy for us. 

**Note: in your homework, i_female is identical to female as you have created it. Similarly for i_black.**


```r
#first step: assign our command to the dataframe
dataPS01<- dataPS01 %>%mutate(
  #first female
  female = ifelse(sex=="f",1,0),
  #now black
  black = ifelse(race =="b",1,0),
  #we can use 2 logicals to get female_black
  female_black = ifelse(race =="b" & i_female=="f",1,0 )
)
```

Now let's look at the total fraction of females in our sample:


```r
mean(dataPS01$female)
```

```
## [1] 0.7691992
```

another way of doing this is by passing the dataframe directly.


```r
#If we filter our dataframe on sex = 'f' then we can simply count how many times 'f' shows up!
mean(dataPS01$sex == 'f')
```

```
## [1] 0.7691992
```

As you can see, these are the same. There's many ways to do the same thing in R. Here is another, if we wanted a snapshot:


```r
#using 'head()' to limit the number of rows that show up
head(dataPS01 %>% filter(sex == 'f'))
```

```
## # A tibble: 6 x 15
##   i_callback n_jobs n_expr i_military i_computer first_name sex   i_female
##        <dbl>  <dbl>  <dbl>      <dbl>      <dbl> <chr>      <chr> <lgl>   
## 1          0      2      6          0          1 Allison    f     TRUE    
## 2          0      3      6          1          1 Kristen    f     TRUE    
## 3          0      1      6          0          1 Lakisha    f     TRUE    
## 4          0      4      6          0          1 Latonya    f     TRUE    
## 5          0      3     22          0          1 Carrie     f     TRUE    
## 6          0      2      5          0          1 Jill       f     TRUE    
## # … with 7 more variables: i_male <lgl>, race <chr>, i_black <lgl>,
## #   i_white <lgl>, female <dbl>, black <dbl>, female_black <dbl>
```

notice:the female column has only ones. That means our mutate command above worked!

##Lesson 3: Calculating group statistics

Often, we want stats on subsets of the data. Maybe one group is seeing odd callback counts. Let's dive in! 

How do we figure out what is the mean callback rate for females?

R has a TON of ways to do this. Let's start with the massive tidyverse approach!



```r
#remember: R doesn't care about vertical space. So we can be greedy if it makes stuff look better.

#starting a new line after an =, or a comma, or a tidyverse pipe %>% won't change how R interprets your code. Let's create a new dataframe with some summary statistics in it
female_stats = 
  dataPS01 %>% 
  #group_by() takes a variable from your dataframe and then will perform summary statistics 
  #at that 'level' see the output to see what I mean. Group_by can take any number of variables.
  group_by(sex) %>%
  #summarise sort of works like mutate, but gathers data based on the group_by label above and returns summary statistics
  summarise(
    total_callback= sum(i_callback),
    callback_mean= mean(i_callback),
    callback_sd = sd(i_callback)
  )

#let's output that dataframe we just made

female_stats
```

```
## # A tibble: 2 x 4
##   sex   total_callback callback_mean callback_sd
##   <chr>          <dbl>         <dbl>       <dbl>
## 1 f                309        0.0825       0.275
## 2 m                 83        0.0738       0.262
```

Now, we have a NEW tibble, with some information on our two groups. Handy right? 

What does the 'mean' column mean in this context? Think about what we've been doing so far.

 
Another way to do the same thing is to subset based on sex, using the `subset()` command.


```r
#this will create two dataframes, one where there are only males, and one with only females
m=subset(dataPS01, sex=="m")
f= subset(dataPS01, sex=="f")
```

MID LAB EXCERCISE BREAKDOWN!!!!

How would you verify that these methods are the same? Try to do this on your own. I'll put a solution below on the lab notes.


```r
#no peaking! If you do peak, try to guess what each of these lines is actually doing
mean(f$i_callback)
```

```
## [1] 0.08248799
```

```r
sd(f$i_callback)
```

```
## [1] 0.2751435
```

```r
nrow(f[f$i_callback == 1,])
```

```
## [1] 309
```

## Difference in two groups. No regression
Now let's conduct a statistical test for the difference in the two groups' average callback rates. What are our hypotheses?

H0: The difference is zero. H1: The difference is not zero
One way to do this (by hand) is to calculate all of the means and SD and then plug them into a t-test. First, we need the standard deviations and the number of each group:


```r
#counting the number of black/white observations
num.b <- nrow(filter(dataPS01, race == "b"))
num.w <- nrow(filter(dataPS01, race == "w"))
#calculating the standard deviation for black names/white names only
sd.b <- sd(filter(dataPS01, race == "b")$i_callback)
sd.w <- sd(filter(dataPS01, race == "w")$i_callback)
```

Remember your stats classes? We will need to plug these into a formula that looks like:

(x1-x2)/sqrt([sd(1)^2/n1] + [sd(2)^2/n2]) 

We need means of callback by group next:


```r
mean_all <- mean(dataPS01$i_callback)
#remember, we are looking for percent callback rates based on race. So we need to filter, and then call the variable of
#interest on our resulting dataframe. Let's do that:
perc.callback.b <- mean(filter(dataPS01, race == "b")$i_callback)
perc.callback.w <- mean(filter(dataPS01, race == "w")$i_callback)
```

Plugging in our declared variables:


```r
t <- (perc.callback.b - perc.callback.w) / sqrt(sd.b ^ 2 / num.b + sd.w ^ 2 / num.w)
t
```

```
## [1] -4.114705
```

This will return a t-score. If we wanted significance, we'd have to look up a table, and check it. R has some functions to check significance too. However, we have another approach:

We can do this with a z-test too!

The formula for this one is:

```r
z.stat = (perc.callback.b - perc.callback.w)/sqrt(mean_all*(1-mean_all)*(1/num.b + 1/num.w))
2*pnorm(abs(z.stat), lower.tail = F)
```

```
## [1] 3.983887e-05
```

This gives us a p-value for our test. Remember, the e-05 is important to consider.

##Lesson 4: Practice regressions, part two
  
Let's do something suspiciously close to your problem 3.

First let's regress i_callback on i_female

To be clear, our estimating equation is: $$callback_i = B0 + B1*female_i + e_i $$

This type of model is called a linear probability model. Why would we call it that? What are our coefficients measuring if callback_i is just a 0-1 indicator for getting a callback?

Let's estimate the equation! Maybe that will help.

```r
call_f_model = lm(i_callback~female,data=dataPS01)
summary(call_f_model)
```

```
## 
## Call:
## lm(formula = i_callback ~ female, data = dataPS01)
## 
## Residuals:
##      Min       1Q   Median       3Q      Max 
## -0.08249 -0.08249 -0.08249 -0.07384  0.92616 
## 
## Coefficients:
##             Estimate Std. Error t value Pr(>|t|)    
## (Intercept) 0.073843   0.008116   9.099   <2e-16 ***
## female      0.008645   0.009253   0.934     0.35    
## ---
## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
## 
## Residual standard error: 0.2721 on 4868 degrees of freedom
## Multiple R-squared:  0.0001792,	Adjusted R-squared:  -2.614e-05 
## F-statistic: 0.8727 on 1 and 4868 DF,  p-value: 0.3502
```

Nice! Ok. So what do we have here? What do we do now. How would we interpret these coefficients.

##Interpretation.

Any thoughts on the last regression?

Specifically: how do we interpret $B1$? $B0$?

**Hint: think about the "reference group" What are we really measuring here? What group will be affected by ONLY the intercept**
 
What do you (can you) conclude from the coefficient on female not being significant?
  
##Lesson 6: Regression with Interaction terms
  
Now our goal is to estimate the following equation:
  
callback_i = a0+a1*female_i+a2*black_i+a3*female_i*black_i+e_i
  
What are these variables doing there?? What are our new coefficients doing to our interpretations?

How would we interpret a0? What about a3?

Just like with typical dummy variables, these coefficients always come down to thinking about who your *reference* group is.
In this case, a0 = white guys. literally. That's who we left out. 

So coefficients on the variables are the *difference* in callback rates between each group and white guys as our base comparison!

##Lesson 7: Regression with Interaction terms

Throwing an interaction term into the regression code is easy: just put it in there. We have a nice `:` operator to do
this for us.


```r
# Note where the : shows up between female and black at the end
interaction_reg= lm(i_callback ~ female+black + female:black, data=dataPS01)
summary(interaction_reg)
```

```
## 
## Call:
## lm(formula = i_callback ~ female + black + female:black, data = dataPS01)
## 
## Residuals:
##      Min       1Q   Median       3Q      Max 
## -0.09892 -0.09892 -0.06628 -0.06628  0.94171 
## 
## Coefficients:
##               Estimate Std. Error t value Pr(>|t|)    
## (Intercept)   0.088696   0.011329   7.829    6e-15 ***
## female        0.010229   0.012963   0.789   0.4301    
## black        -0.030408   0.016211  -1.876   0.0607 .  
## female:black -0.002239   0.018482  -0.121   0.9036    
## ---
## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
## 
## Residual standard error: 0.2717 on 4866 degrees of freedom
## Multiple R-squared:  0.003669,	Adjusted R-squared:  0.003054 
## F-statistic: 5.973 on 3 and 4866 DF,  p-value: 0.0004642
```

Interpret your coefficients! Good luck on the HW, and as always, feel free to email me with questions over the weekend and I'll help, no matter how small they seem.
