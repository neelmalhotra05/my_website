---
categories:
- ""
- ""
date: "2017-10-31T22:26:09-05:00"
description: Lorem Etiam Nullam
draft: false
image: pic09.jpg
keywords: ""
slug: magna
title: Omega Group pay discrimination analysis
---

# Omega Group plc- Pay Discrimination

At the laast board meeting of Omega Group Plc., the headquarters of a large multinational company, the issue was raised that women were being discriminated in the company, in the sense that the salaries were not the same for male and female executives. A quick analysis of a sample of 50 employees (of which 24 men and 26 women) revealed that the average salary for men was about 8,700 higher than for women. This seemed like a considerable difference, so it was decided that a further analysis of the company salaries was warranted. 

You are asked to carry out the analysis. The objective is to find out whether there is indeed a significant difference between the salaries of men and women, and whether the difference is due to discrimination or whether it is based on another, possibly valid, determining factor. 

## Loading the data


```{r load_omega_data}
omega <- read_csv(here::here("data", "omega.csv"))
glimpse(omega) # examine the data frame
```

## Relationship Salary - Gender ?

The data frame `omega`  contains the salaries for the sample of 50 executives in the company. Can you conclude that there is a significant difference between the salaries of the male and female executives?

Note that you can perform different types of analyses, and check whether they all lead to the same conclusion 

.	Confidence intervals
.	Hypothesis testing
.	Correlation analysis
.	Regression


Calculate summary statistics on salary by gender. Also, create and print a dataframe where, for each gender, you show the mean, SD, sample size, the t-critical, the SE, the margin of error, and the low/high endpoints of a 95% condifence interval

```{r, confint_single_valiables}
# Summary Statistics of salary by gender
mosaic::favstats (salary ~ gender, data=omega)


omega %>% 
  # calculate needed statistics in each group
  group_by(gender) %>% 
  
  summarise(
    #calculate number of observations in each group
    n=n(), 
    
    # calculate mean and standar deviation of salary in in each group
    mean=mean(salary), 
    sd=sd(salary),
    
    # calculate t-critical value in our case
    t_critical=qt(0.975, n-1),
    
    #calculate SE and margin of error in each group
    se=sd/sqrt(n), 
    margin_error=se*t_critical, 
    
    # calculate confidence interval in each group
    lower_ci=mean-margin_error, 
    upper_ci=mean+margin_error)
```

> The lower bound of the average salary for male is higher than the upper bound of the average salary for female, so there is a significant difference between the salaries of the male and female executives.

You can also run a hypothesis testing, assuming as a null hypothesis that the mean difference in salaries is zero, or that, on average, men and women make the same amount of money. You should tun your hypothesis testing using `t.test()` and with the simulation method from the `infer` package.

```{r, hypothesis_testing}
# hypothesis testing using t.test() 
t.test(salary~gender, data=omega)


# hypothesis testing using infer package
observed_statistic <- omega %>%
  
  # specify variables
  specify(salary ~ gender) %>%
  
  # calculate statistic of difference, namely "diff in means"
  calculate(stat = "diff in means", order = c("female", "male"))


null_dist_2_sample <- omega %>%
  
  # specify variables
  specify(salary ~ gender) %>%
  
  # assume null hypothesis to be independent
  hypothesize(null = "independence") %>%
  
  # generate 1000 reps, of type "permute"
  generate(reps = 1000, type = "permute") %>%
  
  # calculate statistic of difference, namely "diff in means"
  calculate(stat = "diff in means", order = c("female", "male"))


null_dist_2_sample %>%
  # visualize the result in a ggplot
  visualize() + 
  shade_p_value(observed_statistic,
                direction = "two-sided") +
  
  # add title and subtitle
  labs(x = "Difference in Salary\n(For Male and Female)",
       y = "Count",
       subtitle = "Red line shows observed difference in mean salaray")

# get p_value from the result
p_value_2_sample <- null_dist_2_sample %>%
  get_p_value(obs_stat = observed_statistic,
              direction = "two-sided")

# check p_value for the hypothesis testing
p_value_2_sample
```

> What can you conclude from your analysis? A couple of sentences would be enough

The p-value of the t-test is less than 0.05, we have sufficient evidence to reject the $H_0$, we can conclude that under the confidence level of 95%, there is a significant difference between the salaries of the male and female executives.


## Relationship Experience - Gender?

At the board meeting, someone raised the issue that there was indeed a substantial difference between male and female salaries, but that this was attributable to other reasons such as differences in experience. A questionnaire send out to the 50 executives in the sample reveals that the average experience of the men is approximately 21 years, whereas the women only have about 7 years experience on average (see table below).

```{r, experience_stats}
# Summary Statistics of salary by gender
favstats (experience ~ gender, data=omega)
```

Based on this evidence, can you conclude that there is a significant difference between the experience of the male and female executives? Perform similar analyses as in the previous section. Does your conclusion validate or endanger your conclusion about the difference in male and female salaries?  

```{r}
# hypothesis testing using t.test() 
t.test(experience~gender, omega)

# hypothesis testing using infer package
observed_statistic_exp <- omega %>%
  
  # specify variables
  specify(experience ~ gender) %>%
  
  # calculate statistic of difference, namely "diff in means"
  calculate(stat = "diff in means", order = c("female", "male"))


null_dist_2_sample_exp <-  omega %>%
  
  # specify variables
  specify(experience ~ gender) %>%
  
  # assume null hypothesis to be independent
  hypothesize(null = "independence") %>%
  
  # generate 1000 reps, of type "permute"
  generate(reps = 1000, type = "permute") %>%
  
  # calculate statistic of difference, namely "diff in means"
  calculate(stat = "diff in means", order = c("female", "male"))


# visualize the result in a ggplot
null_dist_2_sample_exp %>%
  visualize() + 
  shade_p_value(observed_statistic_exp,
                direction = "two-sided") +
  labs(x = "Difference in Experience\n(For Male and Female)",
       y = "Count",
       subtitle = "Red line shows observed difference in mean experience")


# get p_value from the result
p_value_2_sample_exp <- null_dist_2_sample_exp %>%
  get_p_value(obs_stat = observed_statistic_exp,
              direction = "two-sided")
p_value_2_sample_exp
```

The result suggests that the p-value is less than 0.05, so there is a significant difference between the experience of the male and female executives. So this conclusion endangers our conclusion about the difference in male and female salaries we get before.



## Relationship Salary - Experience ?

Someone at the meeting argues that clearly, a more thorough analysis of the relationship between salary and experience is required before any conclusion can be drawn about whether there is any gender-based salary discrimination in the company.

Analyse the relationship between salary and experience. Draw a scatterplot to visually inspect the data


```{r, salary_exp_scatter}
# draw a scatter plot between salary against experience
ggplot(data=omega, aes(x=experience, y=salary)) + 
  geom_point() + 
  geom_smooth(method="lm")
```


## Check correlations between the data
You can use `GGally:ggpairs()` to create a scatterplot and correlation matrix. Essentially, we change the order our variables will appear in and have the dependent variable (Y), salary, as last in our list. We then pipe the dataframe to `ggpairs()` with `aes` arguments to colour by `gender` and make ths plots somewhat transparent (`alpha  = 0.3`).

```{r, ggpairs}
omega %>% 
  select(gender, experience, salary) %>% #order variables they will appear in ggpairs()
  ggpairs(aes(colour=gender, alpha = 0.3))+
  theme_bw()
```

> Look at the salary vs experience scatterplot. What can you infer from this plot? Explain in a couple of sentences

Approximately, there is a positive linear relationship between salary and experience, which means that the more experience they have, the higher their salary will be. Therefore, we may not be able to conclude that the average higher salary in male is due to their gender, since male also seems to have longer average experience than female.
