In today's post, we're going to pivot from our usual focus on machine learning and AI to delve into the world of longitudinal multilevel modeling. This statistical approach is a cornerstone underpinning several ML algorithms, including Generalized Linear Mixed-Model (GLMM) trees. [My previous article](https://towardsdatascience.com/hierarchical-linear-modeling-a-step-by-step-guide-424b486ac6a3) on multilevel modeling, published during my PhD and featured by [Towards Data Science](https://towardsdatascience.com), reached an audience of over 30,000 readers. With the benefit of further experience and knowledge, I'm excited to revisit this topic and explore MLM for longitudinal data in greater depth.

**The most important aspect of multilevel modeling is understanding how to construct an equation and interpret the model. You don’t have to worry too much about coding; it’s quite straightforward whether you are an R or Python user.** 

# Repeated Measures ANOVA (RM-ANOVA)Versus Longitudinal Multilevel Modeling (MLM) 

Before we get started, let’s discuss the differences between RM-ANOVA and Longitudinal MLM (also known as Hierarchical Linear Modeling, Mixed Modeling).

## Repeated Measures ANOVA

* Simpler than MLM, this method is used to detect changes across time points and conditions (e.g., control vs. treatment). The analysis emphasizes whether a main effect of time or a time*treatment interaction exists.
* However, the model does not necessarily consider differences across individuals—such as whether different people have different baselines or if their responses to a predictor vary over time.
* Appropriate when all cases have the same number of data waves, meaning each case is measured the same number of times, and the intervals between measurements are equal.
* Predominantly used for discrete/categorical predictors.
* Time-varying variables, like the wave of measurement, are treated categorically rather than continuously. For continuous predictors, RM-ANCOVA would be utilized.
* Although simpler, it's a general linear model that must meet certain assumptions, including linearity, normal distribution of error, independence of errors, and homogeneity of errors—some of which can be challenging to satisfy with longitudinal data.
* Consider, for instance, if you work for Netflix and you conducted surveys once a month for three months to assess audience satisfaction with a new feature that suggests shows in different languages. Using RM-ANOVA, you would assume that the variance, or differences in how customers rate their satisfaction, should be equal across the three time points. To elaborate further, you're assuming that the variability in satisfaction ratings—the extent to which people's opinions differ from one another—is consistent across all time points. Additionally, you expect that the relationship between consecutive satisfaction ratings remains stable over time.
* This means you anticipate that if there is a broad range of opinions in the first month, a similar range will be observed in the second and third months. Similarly, if someone highly appreciated the feature in the first month, it's expected that they will continue to like it in subsequent months, indicating a consistent relationship over time.
* However, this assumption often does not align with reality, as people's opinions can evolve based on various factors, and individuals may have distinct average ratings over time. It is, therefore, somewhat unrealistic to presume that everyone would begin with the same opinion and alter their attitudes or behaviors at an identical pace as time progresses.
* RM-ANOVA also struggles with missing data, as it employs ordinary least squares. If a user misses a survey, their data is excluded from the analysis.
* Best used for designs with few time points, such as a pre-post design, since only two data points may not adequately capture individual changes over time.
* To learn more about how F statistics for multilevel modeling are calculated, I found [this article](https://statistics.laerd.com/statistical-guides/repeated-measures-anova-statistical-guide.php) very helpful.

## MLM

* More complex than RM-ANOV -- this approach captures random effects, including differences in average outcomes (random intercepts) and variations in how predictors relate to outcomes (random slopes) across subjects.
* More flexible than RM-ANOVA, as cases do not need to have the same number of time points or intervals. Note that more data waves are advantageous for capturing changes over time in MLM. At least three time points per subject is recommended.
* Can accommodate both categorical and continuous predictors.
* Can be categorized into general linear mixed models, assuming a normal distribution of errors, or generalized linear mixed models, which do not assume this and can handle categorical outcomes. These models allow for various forms of change over timem including non-linear relationships.
* Handles missing data more effectively than RM-ANOVA by using residual (or restricted) maximum likelihood (REML) for continuous outcomes or maximum likelihood for categorical outcomes, instead of ordinary least squares, which is utilized by RM-ANOVA. Both REML and ML account for non-independence and potential heteroscedasticity and autocorrelation of residuals.
* MLM relies on t-test statistics (or z-test if we standardize the scores of variables). Keep in mind that the degrees of freedom depend on the level of the predictor.
* For Level 1 (Within-Subject Level), the degrees of freedom are typically based on the number of observations within each subject minus the number of parameters estimated at this level. For example, in the Netflix example, if you have three repeated measurements from the same subject and are estimating a slope and an intercept for each subject, the df for each subject would be three minus two.
* For Level 2 (Between-Subject Level), the degrees of freedom are generally determined by the number of groups or subjects minus the number of parameters estimated at this level. For example, if you have 50 subjects and are estimating a random intercept for each, your df at Level 2 would be 50 minus the number of parameters (such as random effects) you are estimating for the groups/subjects.

## RM-ANOVA VS MLM Equations
To better understand the differences between those two methods, Let's look at the equations for RM-ANOVA versus MLM using the Netflix example I discussed earlier:

**RM-ANOVA Equation**:Y<sub>ij</sub> = μ + α<sub>i</sub> + β<sub>j</sub> + (αβ)<sub>ij</sub> +  ϵ<sub>ij</sub>
 * Y<sub>ij</sub> ​is the satisfaction score for subject i at time j.
 *  μ is the overall mean satisfaction score.
 *   α<sub>i</sub> ​and β<sub>j</sub>represent the individual and time effects, respectively.
 *   (αβ)<sub>ij</sub> is the interaction effect.
 *    ϵ<sub>ij</sub>​is the error term, assumed to be equally correlated across all observations.

**MLM Equation**:Y<sub>ij</sub>= γ<sub>00</sub>+γ<sub>10</sub>×Time<sub>ij</sub>+u<sub>0i</sub>+u<sub>1i</sub>×Time<sub>ij</sub>+ϵ<sub>ij</sub>
  * Y<sub>ij</sub>​ is the satisfaction score for subject i at time j.
  * γ<sub>00</sub> and γ<sub>10</sub>are the fixed effects for the intercept and time, respectively.
  *  u<sub>0i</sub> and u<sub>1i</sub> ​are the random intercept and slope, respectively, capturing individual differences.
  *  ϵ<sub>ij</sub> ​is the unique error term for each observation.

Note that the error terms in these models differ significantly. MLM accommodates individual variances in satisfaction scores and the interplay between time and satisfaction across individuals. In contrast, RM-ANOVA employs a singular error term (ϵ<sub>ij</sub>), which captures the discrepancy between predicted and observed values for individual i at time j. Neglecting hierarchical structures in data can lead to underestimating the standard errors of regression coefficients, potentially exaggerating the statistical significance of results, known as Type I error rates. This issue frequently arises in single-level regression applied to clustered data. While RM-ANOVA addresses this to some degree by incorporating αβ<sub>ij</sub>, it has limitations such as reduced flexibility in managing missing data and not accounting for variations in intercepts and slopes across groups. Consequently, MLM is generally regarded as superior in minimizing error terms.

# MLM Terminologies

Now that we've explored how MLM offers a more nuanced approach than RM-ANOVA, particularly in capturing individual differences over time, let's delve into some key terminologies often encountered in multilevel modeling.

## 1. Fixed Effects
* Fixed effects are assumed constant across all units in a dataset. They include fixed intercepts and fixed slopes.
* In the Netflix example, the fixed intercept represents the average consumer satisfaction score at the start point (e.g., month 1).
* The fixed slope represents how each unit increase in time uniformly affects satisfaction across all users. We can incorporate more than one predictor into the model, and thus, have more than one fixed slope. For instance, in addition to the time of measurement, we may examine whether and to what extent a one-unit increase in user engagement with the new feature (e.g., clicking on the feature, time spent viewing the new feature) can improve their satisfaction

## 2. Random Effects
* Random effects account for variations across units, allowing individual uniqueness. They are divided into random slopes and random intercepts.
* A random intercept in the Netflix scenario acknowledges that different users might have varying baseline satisfaction levels.
* A random slope indicates that the impact of month of measurement on satisfaction differs among users. For some, a month goes by might significantly boost satisfaction, while for others, the effect could be minimal.
* Considering these concepts of fixed versus random effects, think about the potential research questions in your projects. Using the Netflix example, you could explore:
  * Time-varying factors (like age or survey wave) influencing customer satisfaction.
  * The impact of personal attributes (gender, race, etc.) on satisfaction.
  * How these personal attributes affect initial satisfaction (intercept) and its evolution over time (slope).

> [!IMPORTANT]
> Remember, the heart of your analysis lies in framing the right questions and constructing an accurate model. Running the model is comparatively straightforward.

## 3. Levels in MLM
* MLM always involves multiple levels. In longitudinal data, Level 1 often represents time or time-varying variables, while Level 2 captures individual differences with time-invariant factors like race, gender, or other demographic characteristics that do not change across time.

## 4. Residuals vs. Random Effects
* We often refer to Level 2 error terms as 'random effects' and Level 1 error terms as 'residuals.'

## 5. Balanced vs. Imbalanced Designs
* In longitudinal MLM, when every subject has the same interval between measurements, for example, every six months,the design is considered balanced as everyone is measured on the same schedule.
* In cases where every subject is measured on the same set schedule but the intervals are not equal, for example, everyone is measured at month 1, month 4, and month 12, the design remains balanced. Even though the interval is not consistent (e.g., 3 months from wave 1 to wave 2 and 8 months from wave 2 to wave 3), it is still considered a balanced design because the measurement schedule is consistent across subjects.
* However, if people are measured on different schedules, then we have an imbalanced design. This scenario requires a different type of MLM approach, which is beyond the scope of this post.

## 6. Variance 
* In the realm of MLM, variance is categorized into two types: Within-Group Variance (σ²) and Between-Group Variance (τ<sub>00</sub>). Within-Group Variance (σ²) refers to the variance of the residuals (e<sub>ij</sub>) at the individual level. It is calculated as the average of the squared differences between each observation and its group's predicted value.
*  On the other hand, Between-Group Variance (τ<sub>00</sub>) pertains to the variance of the random intercepts (u<sub>ij</sub>) at the group level. This is assessed by examining the variability in the group-level intercepts to determine how much the average outcomes vary across different groups.
*  The total variance of the outcome variable is the sum of these two components. Additionally, if your model includes random slopes, these variances will also contribute to the variance of the outcome variable.
  
## 7. Intraclass Correlation Coefficient (ICC)
* Central to the discussion of variance in MLM is the concept of the Intraclass Correlation Coefficient (ICC), a pivotal metric for quantifying the proportion of variance attributable to each level. The ICC is calculated as follows:

$$\text{ICC} = \frac{\tau_{00}}{\tau_{00} + \sigma^2}\$$

* ICC values range from 0 to 1. Higher values, approaching 1, indicate that a significant portion of the total variance is due to differences between groups. For instance, an ICC of 1 suggests that the data aligns closely with longitudinal modeling, implying no observable change over time within subjects; instead, the variance is predominantly driven by differences between subjects. This scenario signifies that the data is not representative of clustered longitudinal patterns.
* Conversely, an ICC close to 0, or exactly 0, indicates that time effects are highly individualized, with each person demonstrating distinct responses over time. This scenario suggests a minimal influence of group-level factors.
* In most cases, the ICC will fall somewhere between 0 and 1. It's important to recognize ICC as a form of effect size, offering valuable insight into the relative impact of within-group versus between-group variability in your data.
## 8. Covariance 
* Covariance refers to the measure of how much two random variables change together. In the context of MLM, you often deal with random effects at different levels (like random intercepts and slopes), and the covariance between these effects can be informative.
* In our example, time is nested within Netflix users. You might be interested in the covariance between the random intercept  and the random slope of time acorss users. This would tell you how much the average satisfaction scores (intercept) and the effect of time (slope) co-vary across users.
* A positive covariance would suggest that users with higher intercepts (e.g., folks with overall higher satisfaction scores) also have higher slopes (e.g., a stronger effect of time change on satisfaction scores). A negative covariance would indicate an inverse relationship.
## 9. Correlation
* Correlation is related to covariance but provides a standardized measure of the relationship between variables.
* In other words, while covariance gives you the raw magnitude of the co-variation, correlation scales this relationship to a value between -1 and 1, making it easier to interpret.
  
# General Notation in Multilevel Modeling

Multilevel modeling utilizes specific subscript notation to denote levels and parameters within the model. This notation typically follows a pattern:
* **i**: Indicates the lower-level unit (e.g., time point in longitudinal studies, students in cross-sectional educational research, etc.).
*  **j**: Represents the higher-level unit in models with more than two levels (e.g., classrooms, schools). In longitudinal multilevel modeling, 'i' often denotes individuals, while 'j' denotes time points. Therefore, a subscript 'ij' refers to individual 'i' at time 'j'.
*   **0, 1, 2, …, n**: These numbers as subscripts represent the indices of predictors in the model. '0' refers to the intercept; '1' to the first variable; '2' to the second variable, and so on. In cases where there is more than one number in a subscript, the first number typically denotes the order of predictors at Level 1, and the second refers to the order of predictors at Level 2. For instance:
    * γ<sub>01</sub> represents the effect of the first Level 2 predictor on the Level 1 intercept.
    * γ<sub>11</sub> indicates the effect of the first Level 2 predictor on the slope of the first Level 1 predictor.
    * γ<sub>00</sub> denotes the expected value of the outcome variable when all predictors are set to the reference level. This parameter is interpreted as the average outcome across all individuals and time points, prior to considering specific effects or variations.

# Model Building in Longitudinal MLM

Understanding the notations in Multilevel Modeling (MLM) for longitudinal data can be challenging. To demystify this process, let's explore building an MLM model step by step. The fundamental approach to constructing a longitudinal MLM model involves starting with a basic, or null, model devoid of any predictors. This initial step helps us assess whether the average outcomes for each individual, observed across multiple time points, significantly deviate from zero.

<img width="774" alt="Screen Shot 2024-01-19 at 11 00 39 AM" src="https://github.com/KayChansiri/Longtitudinal-Multilevel-Modeling/assets/157029107/b0ac34ca-ce15-4216-96ee-7ef1fbf5f698">

The subsequent stage involves incorporating fixed effects, or time-varying variables, at Level 1. The objective here is to determine whether the addition of these variables offers a significant enhancement over the null model. Should this be the case, we proceed to the introduction of a random intercept term at Level 2. This addition aims to explore the variability in average outcomes across different individuals.

When the inclusion of the random intercept term significantly augments the model's fit compared to the fixed-effects-only model, it signals the appropriateness of integrating a random slope term. This term enables us to examine the diversity in the relationships between predictors and outcomes across different subjects.

In scenarios involving multiple time-varying predictors at Level 1, each is added sequentially to evaluate their individual contributions to improving the model's fit. The final advancement in model development involves adding time-invariant variables at Level 2, seeking to further refine the random slope model.

**Throughout this modeling process, one aspect is varied at a time, facilitating a comparative analysis with the preceding model iteration to confirm improvements in model fit. Key statistical measures like the Likelihood Ratio Test (LRT), Akaike Information Criterion (AIC), and Bayesian Information Criterion (BIC) serve as valuable tools in this assessment.**

## 1. Null Model
To provide a practical perspective on model building and evaluation, let’s delve into this methodology using a Netflix case study as an illustrative example. To set the stage for our exploration, I'll begin by creating a simulated dataset and start with the null or fixed intercept only model.
```ruby
set.seed(123)  # Set seed for reproducibility

# Set the number of subjects to be 100
num_subjects <- 100

# Create a data frame for subjects with constant variables: gender and income
subjects_df <- data.frame(
  subject_id = 1:num_subjects,
  gender = sample(c("male", "female"), num_subjects, replace = TRUE),
  income = sample(c("<19999", "20000-39999", "40000-59999", "60000-79999", "80000-99999", ">100000"), num_subjects, replace = TRUE)
)

# Function to generate repeated measurements for each subject
generate_measurements <- function(subject_id, gender, income) {
  data.frame(
    subject_id = subject_id,
    time = rep(c("month1", "month2", "month3"), each = 1),
    satisfaction = sample(1:5, 3, replace = TRUE),
    engagement = sample(1:100, 3, replace = TRUE),
    gender = rep(gender, 3),
    income = rep(income, 3)
  )
}

# Apply the function to each subject and combine the results
longitudinal_data <- do.call(rbind, lapply(1:num_subjects, function(i) {
  generate_measurements(subjects_df$subject_id[i], subjects_df$gender[i], subjects_df$income[i])
}))

# Print the first 10 rows of the dataset
head(longitudinal_data, 10)
```

According to the dataset, we have 100 Netflix users, each of whom was measured for their satisfaction with a new feature three times (months 1, 2, and 3). This setup represents a balanced MLM (Multilevel Modeling) design. The satisfaction score of the Netflix user, which ranges from 1 to 5, is randomly generated for each time point. Additionally, a measure of the user's engagement, randomly chosen between 1 and 100, is recorded at each time point. Both gender and income serve as fixed effects, meaning their values remain constant across all time points for each subject.

The equation of the  null model is Y<sub>ij</sub> = γ<sub>0i</sub> +ϵ<sub>ij</sub>. To fil the model in R according to the equation, we write the following code: 
​
```ruby
# Load the necessary library
library(lme4)

# Fit the null model
null_model <- lm(satisfaction ~ 1, data = longitudinal_data)

# View the summary of the model
summary(null_model)
```
Here is the output:

<img width="484" alt="Screen Shot 2024-01-19 at 4 45 38 PM" src="https://github.com/KayChansiri/Longtitudinal-Multilevel-Modeling/assets/157029107/63e5f772-c293-4391-8f76-d6572875457f">

According to the output, the average satisfaction score across all subjects and time points is 3, with the standard deviation of the sampling distribution of the intercept at 0.08027. The p-value is significant, indicating that the mean satisfaction score significantly differs from zero. It's important to note that the degrees of freedom are 299, calculated based on the number of observations at this level (i.e., 300 satisfaction measurements across 100 individuals over three time points) minus one parameter in the model, which is the intercept.

## 2. Fixed Slope Model

Let's build a bit more complicated model by adding the time of measuredment, which reflects the months when each user's satisfaction was assessed, as another fixed effect in the model.

satisfaction<sub>ij</sub>​= π<sub>0i</sub>​ + π<sub>1i</sub>​(time)+ ϵ<sub>ij</sub>​ 

* satisfaction<sub>ij</sub>: This denotes the satisfaction score of user i at time j.
* π<sub>0i</sub>: The intercept for individual i. This value predicts the expected satisfaction score for user i at the baseline time point, which, in our case, is month 1.
* π<sub>1i</sub>​(time): The slope of time for user i. This coefficient indicates the extent of change in satisfaction scores for user i with each incremental month.
Time: This is a key variable indicating the time point of measurement — month 1, 2, or 3. In different studies, time-varying variables might vary and could include factors like age, years, or other temporal measures relevant to your specific research project.
ϵ<sub>ij</sub>​: The residual error for individual i at time j. It represents the portion of satisfaction variation that remains unexplained even after considering the time effect.

Let's fit the model in R.

```ruby
​# Load the necessary package
library(lm4)
# Convert time to a numeric variable such that the model understands the progression of time
longitudinal_data$time_numeric <- as.numeric(gsub("month", "", longitudinal_data$time))

# Fit the linear model without random effects
level1_fixed_model <- lm(satisfaction ~ time_numeric, data = longitudinal_data)

# Display the summary of the model
summary(level1_fixed_model)
```
Here is the output: 

<img width="500" alt="Screen Shot 2024-01-19 at 5 22 54 PM" src="https://github.com/KayChansiri/Longtitudinal-Multilevel-Modeling/assets/157029107/9baaecc4-294d-48a1-90f3-7634ea03cf8e">

Notice that the effects of time is not significant. In this case, I do not need to compare this model with the null model, as I know from the non-significant p-value that adding time does not substantially improve the model's fit. However, let's pretend we have a significant p-value for time effects and proceed to building the model in the next step.

## 3. Random Intercept model
Let's construct a more complex model to investigate whether the average satisfaction score among Netflix users varies.
### Level-1 Equation (Within-Subject Model):satisfaction<sub>ij</sub> = β<sub>0j</sub> + β<sub>1</sub>×time_numeric<sub>ij</sub>+e<sub>ij</sub>

* satisfaction<sub>ij</sub> is the satisfaction score for the j measurement of the i subject
* β<sub>0j</sub> is the subject-specific intercept for the i user
*  β<sub>1</sub>  s the fixed effect of time (time_numeric), representing the average change in satisfaction score for a unit increase in time.
*  e<sub>ij</sub> is the residual error for the j-th measurement of the i-th user.
### Level-2 Equation (Between-Subject Model): β<sub>0j</sub> =  β<sub>00</sub> + u<sub>0j</sub>
* β<sub>00</sub> is the overall average intercept across all subjects.
* u<sub>0j</sub> is the random effect for the i user, representing the deviation of the user’s intercept from the overall average intercept.

Let's build the model in R:
```ruby
random_intercept_model <- lmer(satisfaction ~ time_numeric + (1 | subject_id), data = longitudinal_data)

# View the summary of the model
summary(random_intercept_model)
```
Here is the output: 


<img width="443" alt="Screen Shot 2024-01-22 at 1 04 49 PM" src="https://github.com/KayChansiri/Longtitudinal-Multilevel-Modeling/assets/157029107/88ffcfd0-e6e7-4cd8-a024-d9a4a63ea0ec">


* According to the output,the variance in the intercept across different subjects (`subject_id`) is 0.0174, with a standard deviation of 0.1319. This suggests some variability in the baseline satisfaction scores among different Netflix users.The residual variance is 1.9141 with a standard deviation of 1.3835, indicating the variability in satisfaction scores within subjects that is not explained by the time variable.
* Regarding the fixed Effects, the intercept (3.22000) represents the average satisfaction score at the reference level of the `time_numeric` variable (i.e., month 1). The coefficient for `time_numeric` is -0.11000. This suggests that, on average, satisfaction scores decrease slightly as the `time_numeric` variable increases. The negative sign indicates an inverse relationship. However, the t-value of -1.124, which  is quit small,  suggest that this effect is not statistically significant. Tis is consistent with our previous model indicating the non-significance effects of time.
* The correlation of -0.924 between the intercept and `time_numeric` reflects a strong inverse relationship, given that correlation scores range from -1 to 1. This implies that an increase in the `time_numeric` variable is generally associated with a decrease in the average satisfaction score, and vice versa. However, this zero-order correlation doesn't consider other variances/residuals that might influence the relationship between time and satisfaction scores. This explains the high correlation coefficient, despite the non-significant effect of time observed in our regression models.
* The REML criterion at convergence (1052.8) is a measure of the model fit. Lower values generally indicate a better fit, but this metric is more useful for comparing different models on the same data. Let's compare this random intercept model with the fixed model we previously ran, using BIC and AIC.

```ruby
AIC(level1_fixed_model)
BIC(level1_fixed_model)
```


```ruby
AIC(random_intercept_model)
BIC(random_intercept_model)
```
Here are the outputs: 


<img width="279" alt="Screen Shot 2024-01-22 at 1 47 23 PM" src="https://github.com/KayChansiri/Longtitudinal-Multilevel-Modeling/assets/157029107/25921bd5-b337-4473-a402-285dd8fb7a78">

Lower AIC and BIC values generally indicate a preferable model, suggesting that the level1_fixed model provides a better fit than the random_intercept_model. This could imply that although there are variations in user satisfaction scores, the differences across users might not be significant enough to warrant the additional complexity of the random intercept model. Let's further explore those individual differences using the ranef() function.

```ruby
ranef(random_intercept_model)
```
<img width="179" alt="Screen Shot 2024-01-22 at 1 53 29 PM" src="https://github.com/KayChansiri/Longtitudinal-Multilevel-Modeling/assets/157029107/cd825c3a-2260-44d7-b44a-dc22dc7f1593">

Examining the intercepts for the first five users, we observe that each person's score slightly deviates from the average user satisfaction score. Specifically, Subject 1 has an intercept of 0.01769544, Subject 2 is at -0.008847719, Subject 3 again at 0.01769544, Subject 4 at -0.02654316, and Subject 5 at 0.03539088. These variations, while present, do not appear to be drastically varied. You may also wonder why the summary function of the lmer function does not provide any p-values of the fixed or random effects in the model. To get the p-values of the fixed effects in your model, you may use the lmerTest instead of the lme4 package that we have been using in this tutorial. Both packages are sutiable for liner mixed models and provide the lmer function for model fitting. However, the lmerTest does provide p- values of the fixed effects in the output: 

```ruby
random_intercept_model <- lmerTest::lmer(satisfaction ~ time_numeric + (1 | subject_id), data = longitudinal_data)

# View the summary of the model
summary(random_intercept_model)
```

<img width="633" alt="Screen Shot 2024-01-22 at 2 02 44 PM" src="https://github.com/KayChansiri/Longtitudinal-Multilevel-Modeling/assets/157029107/381da6d1-48aa-4f68-8277-760e89d6d186">


As of now, neither the `lme4` nor the `lmerTest` packages provide p-values for the random intercepts in multilevel models (MLM). This is because the significance of random effects is primarily assessed based on the variance components associated with these effects. Essentially, understanding the significance of random effects involves estimating the extent of variability in the data in terms of the slopes or intercepts of the variables of interest. Therefore, to determine if a model is improved by including a random effect, common approaches include using model fit measures such as likelihood ratio tests, AIC, or BIC.

In addition to identifying the random effects for each case in the dataset using the ranef() function, you can employ fixef() to obtain the intercept and coefficients of the fixed effects in your model. Furthermore, the coef() function allows you to retrieve both the intercepts and fixed effects for each individual case in the dataset, as illustrated below:

<img width="566" alt="Screen Shot 2024-01-23 at 7 38 17 AM" src="https://github.com/KayChansiri/Longtitudinal-Multilevel-Modeling/assets/157029107/8747e422-42a8-48b5-8f02-117333d9faec">

<img width="266" alt="Screen Shot 2024-01-23 at 7 39 58 AM" src="https://github.com/KayChansiri/Longtitudinal-Multilevel-Modeling/assets/157029107/f720e931-2dc8-4078-8de4-3d1d2752d219">

## 4. Random Slope Model 
Although the previous model suggests that including a random intercept term may not significantly improve the model, let's assume it does for the sake of continuing our model testing process. In this step, I'll include a random slope for time to explore if different users exhibit varying satisfaction levels as time progresses. To address the research question, we can formulate the following equations:
### ### Level-1 Equation (Within-Subject Model):satisfaction<sub>ij</sub> = β<sub>0j</sub> + β<sub>1</sub>×time_numeric<sub>ij</sub>+e<sub>ij</sub>
* satisfaction<sub>ij</sub> is the satisfaction score for the i measurement of the j user
* β<sub>0j</sub> is the subject-specific intercept for the j user
*  β<sub>1</sub>  s the fixed effect of time (time_numeric), representing the average change in satisfaction score for a unit increase in time.
*  e<sub>ij</sub> is the residual error for the j-th measurement of the i-th user.
### Level-2 Equation (Between-Subject Model/Random Intercept): β<sub>0j</sub> =  γ<sub>00</sub> + u<sub>0j</sub>
* β<sub>00</sub> is the overall average intercept across all subjects.
* u<sub>0j</sub> is the random effect for the i user, representing the deviation of the user’s intercept from the overall average intercept.
### Level-2 Equation (Between-Subject Model/Random Slope): β<sub>1</sub>×time_numeric<sub>ij</sub> =  γ<sub>01</sub> + u<sub>1j</sub>
* γ<sub>01</sub> is the overall average slope (effect of time) across all users.
* u<sub>1j</sub> is are the random effects capturing the deviation of the j<sup>th</sup> user's intercept and slope from the overall averages, respectively. Let's program this in R.

```ruby
random_slope_model <- lmer(satisfaction ~ time_numeric + (time_numeric | subject_id), data = longitudinal_data)

# View the summary of the model
summary(random_slope_model)
```

Here is the output:
<img width="678" alt="Screen Shot 2024-01-23 at 2 25 07 PM" src="https://github.com/KayChansiri/Longtitudinal-Multilevel-Modeling/assets/157029107/363dd9a6-c13b-4f73-abc5-4e9103cbf95e">

The output indicates the warning "boundary (singular) fit", which suggests that the model might be overfitting or that there isn't enough data to support the complexity of the model, especially with regard to the random effects structure. This is not surprising as our generated data may not have enough variation in satisfaction scores across users to estimate the random slopes. In other words, users are not significantly different in their satisfaction change across time, which is consistent with the non-significant effects of time that we found in the previous model. If this assumption is true and we compare the model fit of  our random slope against the random intercpet models, the fit would not change much:
