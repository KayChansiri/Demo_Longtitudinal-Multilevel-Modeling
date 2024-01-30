In today's post, we're going to pivot from our usual focus on machine learning and AI to delve into the world of longitudinal multilevel modeling. This statistical approach is a cornerstone underpinning several ML algorithms, including Generalized Linear Mixed-Model (GLMM) trees. [My previous article](https://towardsdatascience.com/hierarchical-linear-modeling-a-step-by-step-guide-424b486ac6a3) on multilevel modeling, published during my PhD and featured by [Towards Data Science](https://towardsdatascience.com), reached an audience of over 30,000 readers. With the benefit of further experience and knowledge, I'm excited to revisit this topic and explore MLM for longitudinal data in greater depth!

**The most important aspect of multilevel modeling is understanding how to construct an equation and interpret the model. You don’t have to worry too much about coding; it’s quite straightforward whether you are an R or Python user.** 

# Repeated Measures ANOVA (RM-ANOVA) Versus Longitudinal Multilevel Modeling (MLM) 

Before we get started, let’s discuss the differences between RM-ANOVA and Longitudinal MLM (also known as Hierarchical Linear Modeling, Mixed Modeling).

## RM-ANOVA

* RM-ANOVA is simpler than MLM. The analysis simply explores whether a main effect of time or a time*treatment interaction exists. With its simplicity, RM-ANOVA is best used for designs with few time points, such as a pre-post design.
* However, the model does not necessarily consider differences across individuals—such as whether different people have different baselines or if their responses to a predictor vary over time.
* RM-ANOVA is appropriate when all cases have the same number of data waves, meaning each case is measured the same number of times, and the intervals between measurements are equal.
* The analysis technique is predominantly used for discrete/categorical predictors. Time-varying variables, such as the wave of measurement, are treated categorically rather. Thus, if you wish to measure growth as a continuum over time, RM-ANOVA might not be a good option. If you have any continuous predictors in your model or would like to measure growth, Growth Curve Models, a specific type of MLM, should be utilized.
* RM-ANOVA must meet certain general linear model assumptions, including normal distribution of errors, independence of errors, and homogeneity of errors—some of which can be challenging to satisfy with longitudinal data.
* For instance, if you work for Netflix and you conducted surveys once a month for three months to assess user satisfaction with a new feature. Using RM-ANOVA, you expect that the relationship between consecutive satisfaction ratings remains stable over time. This means you anticipate that if there is a broad range of opinions in the first month, a similar range will be observed in the second and third months. Similarly, if someone highly disliked the feature in the first month, it's expected that they will continue to dislike it in subsequent months.
  
![Unknown](https://github.com/KayChansiri/Longtitudinal-Multilevel-Modeling/assets/157029107/508e1b72-fee1-4a88-8d01-1e49358111ba)

* According to the plot above, all users start with similar satisfaction scores and follow a consistent pattern over time, which is an assumption that needs to be met for RM-ANOVA. However, this assumption often does not align with reality, since people's opinions are likely to vary at the baseline and may evolve differently over time. The figure below likely better reflects real-world data.

<img width="653" alt="Screen Shot 2024-01-25 at 3 21 50 PM" src="https://github.com/KayChansiri/Longtitudinal-Multilevel-Modeling/assets/157029107/aef632d2-ab38-4678-ac06-ba6c680e66a0">

* RM-ANOVA also struggles with missing data mangagement, as it employs ordinary least squares. If a user misses one out of the three wave survey, their data is excluded from the analysis as the algorithm cannot calculate the distance between observed versus predicted values of that user during a specific timepoint.


## MLM

* MLM is more complex than RM-ANOV. The approach captures random effects, including differences in average outcomes (random intercepts) and variations in how predictors relate to outcomes (random slopes) across subjects.
* MLM is more flexible than RM-ANOVA, as cases do not need to have the same number of time points or intervals. However, note that more data waves are advantageous for capturing changes over time in MLM. At least three time points per subject is recommended.
* MLM can accommodate both categorical and continuous predictors. Thus, you can use the analysis technique to assess growth.
* With a continuous outcome, MLM is considered a general linear mixed model that assumes a normal distribution of errors. With a categorical or binary outcome, MLM is considered a generalized linear mixed model, which does not assume a normal distribution of errors and allows for various forms of change over time, including non-linear relationships
* MLM handles missing data more effectively than RM-ANOVA by using residual (or restricted) maximum likelihood (REML) for continuous outcomes or maximum likelihood for categorical outcomes, instead of ordinary least squares. Both REML and ML account for non-independence and potential heteroscedasticity and autocorrelation of residuals.
* MLM relies on t-test statistics (or z-test if we standardize the scores of variables). 
* The degrees of freedom for MLM are calculated differently from those for RM-ANOVA, considering that the model addresses both within-subjects (often referred to as level 1 analysis) and between-subjects (often referred to as level 2 analysis) effects. For within-subject analysis, the degrees of freedom are based on the number of observations within each subject minus the number of parameters estimated at this level. For example, in the Netflix example, if you have three repeated measurements from the same user and are estimating a slope and an intercept for each user, the degrees of freedom for each user would be 3 - 2 (slope and intercept) = 1. For Level 2 (between-subject effects), the degrees of freedom are generally determined by the number of groups or subjects minus the number of parameters estimated at this level. For instance, if you have 50 Netflix users and are estimating a random intercept for each, your degrees of freedom at Level 2 would be 50 - 1 (i.e., the random intercept) = 49. The calculation of degrees of freedom for RM-ANOVA is beyond the scope of the current topic, but you can learn more about it [here](https://statistics.laerd.com/statistical-guides/repeated-measures-anova-statistical-guide.php).

## RM-ANOVA VS MLM Equations
To better understand the differences between those two methods, Let's look at the equations for RM-ANOVA versus MLM using the Netflix example I discussed earlier:

**RM-ANOVA Equation**:Y<sub>ij</sub> = μ + α<sub>i</sub> + β<sub>j</sub> + (αβ)<sub>ij</sub> +  ϵ<sub>ij</sub>
 * Y<sub>ij</sub> ​is the satisfaction score for time i of user j.
 *  μ is the overall mean satisfaction score.
 *   α<sub>i</sub> ​and β<sub>j</sub>represent the time and user (i.e., individual) effects, respectively.
 *   (αβ)<sub>ij</sub> is the interaction effect of time and users.
 *    ϵ<sub>ij</sub> ​is the error term, assumed to be equally correlated across all observations.

**MLM Equation**:Y<sub>ij</sub> = γ<sub>00</sub> + γ<sub>10</sub>×Time<sub>ij</sub> + u<sub>0i</sub> + u<sub>1i</sub>×Time<sub>ij</sub> + ϵ<sub>ij</sub>
  * Y<sub>ij</sub>​ is the satisfaction score at time i of subject j.
  * γ<sub>00</sub> and γ<sub>10</sub> are the fixed effects for the intercept and time, respectively.
  *  u<sub>0i</sub> and u<sub>1i</sub> ​are the random intercept and slope, respectively, capturing individual differences.
  *  ϵ<sub>ij</sub> ​is the error term for each observation.

Note that the error terms in the RM-ANOVA versus MLM models differ significantly. MLM accommodates individual variances in satisfaction scores (i.e., random intercept) and the interplay between time and satisfaction across individuals (i.e., random slope). In contrast, RM-ANOVA employs a singular error term (ϵ<sub>ij</sub>), which captures the discrepancy between predicted and observed values for individual j at time i. Neglecting random effect in data can lead to underestimating the standard errors of regression coefficients, potentially exaggerating the statistical significance of results, known as Type I error rates. Thus, MLM is generally regarded as superior in minimizing error terms when it comes to longitudinal data.

# MLM Terminologies

Now that we've explored how MLM offers a more nuanced approach than RM-ANOVA, particularly in capturing individual differences over time, let's delve into some key terminologies often encountered in multilevel modeling.

## 1. Fixed Effects
* Fixed effects are assumed constant across all units in a dataset. They include fixed intercepts and fixed slopes.
* In the Netflix example, the fixed intercept represents the average consumer satisfaction score at the start point (e.g., month 1).
* The fixed slope represents how each unit increase in time uniformly affects satisfaction across all users. We can incorporate more than one predictor into the model, and thus, have more than one fixed slope. For instance, in addition to the time of measurement, we may examine whether and to what extent a one-unit increase in user engagement with the new feature (e.g., clicking on the feature, time spent viewing the new feature) can improve their satisfaction

## 2. Random Effects
* Random effects account for variations across units, allowing individual uniqueness. They are divided into random slopes and random intercepts.
* The random intercept in the Netflix scenario acknowledges that different users might have varying baseline satisfaction levels.
* The random slope indicates that the impact of month of measurement on satisfaction differs among users. For some, a month goes by might significantly boost satisfaction, while for others, the effect could be minimal or even occur in the opposite direction.
* Considering these concepts of fixed versus random effects, think about the potential research questions in your projects. Using the Netflix example, you could explore:
  * Time-varying factors (like age or survey wave) influencing customer satisfaction.
  * The impact of personal attributes (gender, race, etc.) on satisfaction.
  * How these personal attributes affect initial satisfaction (intercept) and its evolution over time (slope).

> [!IMPORTANT]
> Remember, the heart of your analysis lies in framing the right questions and constructing an accurate model. Running the model is comparatively straightforward.

## 3. Levels in MLM
* MLM always involves multiple levels. In longitudinal data, Level 1 often represents time or time-varying variables (e.g., age or anything that could change over time), while Level 2 captures individual differences with time-invariant factors like race, gender, or other demographic characteristics that do not change across time points.

## 4. Residuals vs. Random Effects
* We often refer to Level 2 error terms as 'random effects' and Level 1 error terms as 'residuals.' Remember that these two are different. Random effects assess the variation across subjects/groups or the Level 2 categories. Residuals, on the other hand, represent any other variance not explained by the predictors in the model.

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

* ICC values range from 0 to 1. Higher values, approaching 1, indicate that a significant portion of the total variance is due to differences between groups. For instance, an ICC of 1 suggests that the data has NO observable change over time within subjects; instead, the variance is predominantly driven by differences between subjects. This scenario signifies that the data is not representative of clustered longitudinal patterns.
* Conversely, an ICC close to 0, or exactly 0, indicates that time effects are highly individualized, with each person demonstrating distinct responses over time. This scenario suggests a minimal influence of group-level factors.
* In most cases, the ICC will fall somewhere between 0 and 1. It's important to recognize ICC as a form of effect size, offering valuable insight into the relative impact of within-group versus between-group variability in your data.
## 8. Covariance 
* Covariance refers to the measure of how much two random variables change together. In the context of MLM, you often deal with different types of random effects (like random intercepts and slopes), and the covariance between these effects can be informative.
* In our example, time is nested within Netflix users. You might be interested in the covariance between the random intercept  and the random slope of time acorss users. This would tell you how much the average satisfaction scores (intercept) and the effect of time (slope) co-vary across users.
* A positive covariance would suggest that users with higher intercepts (e.g., folks with overall higher satisfaction scores) also have higher slopes (e.g., a stronger effect of time change on satisfaction scores). A negative covariance would indicate an inverse relationship.
## 9. Correlation
* Correlation is related to covariance but provides a standardized measure of the relationship between variables.
* In other words, while covariance gives you the raw magnitude of the co-variation, correlation scales this relationship to a value between -1 and 1, making it easier to interpret.
  
# General Notation in Multilevel Modeling

Multilevel modeling utilizes specific subscript notation to denote levels and parameters within the model. This notation typically follows a pattern:
* **i**: Indicates the lower-level unit (e.g., time point in longitudinal studies, students in cross-sectional educational research, etc.).
*  **j**: Represents the higher-level unit in models with more than one levels (e.g., classrooms, schools). In longitudinal multilevel modeling, 'i' often denotes time points, while 'j' denotes individuals. Therefore, a subscript 'ij' refers to individual 'j' at time 'i.' However, some scholars may prefer to write it in reverse, as in a cross-sectional MLM, 'i' often refers to individuals. Consequently, some longitudinal MLM studies refer to 'ij' as time 'j' of individual 'i'.
*   **0, 1, 2, …, n**: These numbers as subscripts represent the indices of predictors in the model. '0' refers to the intercept; '1' to the first variable; '2' to the second variable, and so on. In cases where there is more than one number in a subscript, the first number typically denotes the order of predictors at Level 1, and the second refers to the order of predictors at Level 2. For instance:
    * γ<sub>01</sub> represents the effect of the first Level 2 predictor on the Level 1 intercept.
    * γ<sub>11</sub> indicates the effect of the first Level 2 predictor on the slope of the first Level 1 predictor.
    * γ<sub>00</sub> denotes the expected value of the outcome variable when all predictors are set to the reference level. This parameter is interpreted as the average outcome across all individuals and time points, prior to considering specific effects or variations.

# Model Building in Longitudinal MLM

Understanding the notations in Multilevel Modeling (MLM) for longitudinal data can be challenging. To demystify this process, let's explore building an MLM model step by step. The fundamental approach to constructing a longitudinal MLM model involves starting with a basic, or null, model devoid of any predictors. This initial step helps us assess whether the average outcomes for each individual, observed across multiple time points, significantly deviate from zero.

<img width="774" alt="Screen Shot 2024-01-19 at 11 00 39 AM" src="https://github.com/KayChansiri/Longtitudinal-Multilevel-Modeling/assets/157029107/b0ac34ca-ce15-4216-96ee-7ef1fbf5f698">

The subsequent stage involves incorporating fixed effects, or time-varying variables, at Level 1. The objective here is to determine whether the addition of these variables offers a significant enhancement over the null model (i.e., the model with only the intercept term without any other predictors). Should this be the case, we proceed to the introduction of a random intercept term at Level 2. This addition aims to explore the variability in average outcomes across different individuals.

When the inclusion of the random intercept term significantly augments the model's fit compared to the fixed-effects-only model, it signals the appropriateness of integrating a random slope term. This term enables us to examine the diversity in the relationships between predictors and outcomes across different subjects.

In scenarios involving multiple time-varying predictors at Level 1, each is added sequentially to evaluate their individual contributions to improving the model's fit. The final advancement in model development involves adding time-invariant variables at Level 2, seeking to further refine the random slope model.

**Throughout this modeling process, one aspect is varied at a time, facilitating a comparative analysis with the preceding model iteration to confirm improvements in model fit. Key statistical measures like the Likelihood Ratio Test (LRT), Akaike Information Criterion (AIC), and Bayesian Information Criterion (BIC) serve as valuable tools in this assessment.**

## 1. Null Model
To provide a practical perspective on model building and evaluation, let’s delve into this methodology using the Netflix fake study above as an illustrative example. To set the stage for our exploration, I'll begin by creating a simulated dataset and start with the null or fixed intercept only model.
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

The equation of the  null model is Y<sub>ij</sub> = β<sub>0i</sub> +ϵ<sub>ij</sub>. To fil the model in R according to the equation, we write the following code: 
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

Let's build a bit more complicated model by adding the time of measurement, which reflects the months when each user's satisfaction was assessed, as a fixed effect in the model.

satisfaction<sub>ij</sub>​= β<sub>0j</sub>​ + β<sub>1j</sub>​(time)+ ϵ<sub>ij</sub>​ 

* satisfaction<sub>ij</sub>: This denotes the satisfaction score of user j at time i.
* β<sub>0j</sub>: This is the random intercept for individual j, representing the expected satisfaction score for individual j when time is at its reference point (i.e., month 1).
* β<sub>1j</sub>​(time): This denotes the slope of the time predictor for individual j, indicating how much their satisfaction is expected to change with each time increment (usually assuming time is measured continuously).
* Time: This is a key variable indicating the time point of measurement — month 1, 2, or 3. In different studies, time-varying variables might vary and could include factors like age, years, or other temporal measures relevant to your specific research project.
* ϵ<sub>ij</sub>​: The residual error for individual j at time i. It represents the portion of satisfaction variation that remains unexplained even after considering the time effect.

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
### Level-1 Equation (Within-Subject Model): Satisfaction<sub>ij</sub> = β<sub>0j</sub> + β<sub>1</sub>time_numeric<sub>ij</sub>+ϵ<sub>ij</sub>

* satisfaction<sub>ij</sub> is the satisfaction score for the i measurement of the j subject
* β<sub>0j</sub> is the subject-specific intercept for the j user. It represents the expected satisfaction score for subject j at month 1.
*  β<sub>1</sub>  is the fixed effect of time (time_numeric), representing the average change in satisfaction score for a unit increase in time.
*  e<sub>ij</sub> is the residual error for the i measurement of the j user.
### Level-2 Equation (Between-Subject Model): β<sub>0j</sub> =  γ<sub>00</sub> + u<sub>0j</sub>
* γ<sub>00</sub> is the overall average intercept across all subjects.
* u<sub>0j</sub> is the random effect for the j user, representing the deviation of the user’s intercept from the overall average intercept.

Let's build the model in R:
```ruby
random_intercept_model <- lmer(satisfaction ~ time_numeric + (1 | subject_id), data = longitudinal_data)

# View the summary of the model
summary(random_intercept_model)
```
Here is the output: 


<img width="443" alt="Screen Shot 2024-01-22 at 1 04 49 PM" src="https://github.com/KayChansiri/Longtitudinal-Multilevel-Modeling/assets/157029107/88ffcfd0-e6e7-4cd8-a024-d9a4a63ea0ec">


* According to the output,the variance in the intercept across different users (`subject_id`) is 0.0174, with a standard deviation of 0.1319. This suggests some variability in the baseline satisfaction scores among different Netflix users.The residual variance is 1.9141 with a standard deviation of 1.3835, indicating the variability in satisfaction scores that is not explained by the time variable.
* Regarding the fixed Effects, the intercept (3.22000) represents the average satisfaction score at the reference level of the `time_numeric` variable (i.e., month 1). The coefficient for `time_numeric` is -0.11000. This suggests that, on average, satisfaction scores decrease slightly as the `time_numeric` variable increases. The negative sign indicates an inverse relationship. However, the t-value of -1.124, which  is quit small,  suggest that this effect is not statistically significant. Tis is consistent with our previous model indicating the non-significance effects of time.
* The correlation of -0.924 between the intercept and `time_numeric` reflects a strong inverse relationship, given that correlation scores range from -1 to 1. This implies that an increase in the `time_numeric` variable is generally associated with a decrease in the average satisfaction score, and vice versa. However, this zero-order correlation doesn't consider other variances/residuals that might influence the relationship between time and satisfaction scores. This explains the high correlation coefficient, despite the non-significant effect of time observed in our regression models.
* Notice the output is a linear mixed model fitted using REML (Restricted/Residual Maximum Likelihood), which is The REML criterion at convergence (1052.8) is a measure of the model fit. Lower values generally indicate a better fit, but this metric is more useful for comparing different models on the same data. Let's compare this random intercept model with the fixed model we previously ran, using BIC and AIC.

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
### Level-1 Equation (Within-Subject Model): Satisfaction<sub>ij</sub> = β<sub>0j</sub> + β<sub>1j</sub>time_numeric<sub>ij</sub>+e<sub>ij</sub>
* satisfaction<sub>ij</sub> is the satisfaction score for the i measurement of the j user.
* β<sub>0j</sub> is the subject-specific intercept for the j user.
*  β<sub>1j</sub> is the effect of time (time_numeric), representing the average change in satisfaction score for a unit increase in time.
*  e<sub>ij</sub> is the residual error for the j measurement of the i user.
### Level-2 Equation (Between-Subject Model/Random Intercept): β<sub>0j</sub> =  γ<sub>00</sub> + u<sub>0j</sub>
* β<sub>00</sub> is the overall average intercept across all subjects.
* u<sub>0j</sub> is the random effect for the j user, representing the deviation of the user’s intercept from the overall average intercept.
### Level-2 Equation (Between-Subject Model/Random Slope): β<sub>1j</sub>=  γ<sub>01</sub> + u<sub>1j</sub>
* γ<sub>01</sub> is the overall average slope (effect of time) across all users.
* u<sub>1j</sub> is are the random effects capturing the deviation of the j<sup>th</sup> user's intercept and slope from the overall averages, respectively. Let's program this in R.

```ruby
random_slope_model <- lmer(satisfaction ~ time_numeric + (time_numeric | subject_id), data = longitudinal_data)

# View the summary of the model
summary(random_slope_model)
```

Here is the output:


<img width="678" alt="Screen Shot 2024-01-23 at 2 25 07 PM" src="https://github.com/KayChansiri/Longtitudinal-Multilevel-Modeling/assets/157029107/363dd9a6-c13b-4f73-abc5-4e9103cbf95e">

The output displays a warning of a "boundary (singular) fit," indicating potential overfitting or insufficient data to support the complexity of the model, especially in terms of its random effects structure. This finding is not surprising, as our generated data might lack sufficient variation in satisfaction scores across users to estimate random slopes accurately. Essentially, this suggests that there is no significant difference in how users' satisfaction changes over time, consistent with the non-significant time effects observed in the previous model.

To test this assumption, let's generate a new dataset that introduces greater variability in baseline satisfaction scores across users and ensures that users' satisfaction consistently increases or decreases over the months. This is to see whether we still get the boundary (singular) fit warning message.


```ruby
set.seed(123)  # Set seed for reproducibility

# Number of subjects
num_subjects <- 100

# Create a data frame for subjects with constant variables: gender and income
subjects_df <- data.frame(
  subject_id = 1:num_subjects,
  gender = sample(c("male", "female"), num_subjects, replace = TRUE),
  income = sample(c("<19999", "20000-39999", "40000-59999", "60000-79999", "80000-99999", ">100000"), num_subjects, replace = TRUE)
)

# Adding a random intercept for each subject to introduce variability
set.seed(123)  # Ensure reproducibility
subjects_df$random_intercept <- rnorm(num_subjects, mean = 0, sd = 2)  # Adjust mean and sd as needed

# Function to generate repeated measurements for each subject
generate_measurements <- function(subject_id, gender, income, random_intercept) {
  # Base satisfaction level for each time point
  satisfaction_base <- c(1, 2, 3)  # Fixed effect of time (increasing satisfaction) to simulate a scenario where satisfaction increases by one unit with each subsequent month.
  data.frame(
    subject_id = subject_id,
    time = rep(c("month1", "month2", "month3"), each = 1),
    time_numeric = rep(1:3, each = 1),
    satisfaction = satisfaction_base + random_intercept + sample(-1:1, 3, replace = TRUE),  # Add variability
    engagement = sample(1:100, 3, replace = TRUE),
    gender = rep(gender, 3),
    income = rep(income, 3)
  )
}

# Apply the function to each subject and combine the results
longitudinal_data <- do.call(rbind, lapply(1:num_subjects, function(i) {
  generate_measurements(subjects_df$subject_id[i], subjects_df$gender[i], subjects_df$income[i], subjects_df$random_intercept[i])
}))

# Print the first 10 rows of the dataset
head(longitudinal_data, 10)

```
Now, let's redo the whole model building process using the new dataset, starting from the null model. 

## Model Building with the New Dataset 
### Null Model

```ruby
library(lme4)

# Fit the null model
null_model <- lm(satisfaction ~ 1, data = longitudinal_data)

# View the summary of the model
summary(null_model)
```
Here is the output:

<img width="528" alt="Screen Shot 2024-01-23 at 5 02 50 PM" src="https://github.com/KayChansiri/Longtitudinal-Multilevel-Modeling/assets/157029107/8be93782-46fc-401c-8f91-934eed194b4a">

### Fixed Slope Model

```ruby
longitudinal_data$time_numeric <- as.numeric(gsub("month", "", longitudinal_data$time))

# Fit the linear model without random effects
level1_fixed_model <- lm(satisfaction ~ time_numeric, data = longitudinal_data)

# Display the summary of the model
summary(level1_fixed_model)
```
Here is the output:

<img width="555" alt="Screen Shot 2024-01-23 at 5 05 35 PM" src="https://github.com/KayChansiri/Longtitudinal-Multilevel-Modeling/assets/157029107/7cab70ff-a79d-451e-be29-ae66df6ea67d">

According to the output, you can see that the fixed effect of time is significant, indicating that a one-unit increase in time (i.e., month) significantly predicts a 0.89 change in satisfaction scores.

### Random Intercept Model
```ruby
random_intercept_model <- lmerTest::lmer(satisfaction ~ time_numeric + (1 | subject_id), data = longitudinal_data)

# View the summary of the model
summary(random_intercept_model)
```

Here is the output:

<img width="458" alt="Screen Shot 2024-01-25 at 12 39 23 PM" src="https://github.com/KayChansiri/Longtitudinal-Multilevel-Modeling/assets/157029107/d0498ca7-d6e1-4f4c-ae66-36bdd9282b1b">

* Regarding random effects, the random intercept for each user, with a variance of 3.148 and a standard deviation of 1.774, implies individual differences among users' baseline satisfaction levels. As for the residual variance (error variance), which is 0.678 with a standard deviation of 0.8234, these numbers indicate the variation in satisfaction scores not explained by the model.
* For fixed effects, the coefficient for 'time_numeric' is 0.89000 with a significant p-value, indicating that for each unit increase in time, there's an expected increase of 0.89 in the satisfaction score.
* Regarding the correlation of fixed effects, the correlation between the intercept and 'time_numeric' is -0.535. This suggests a moderate inverse relationship between these parameters within the model. In other words, if the intercept were higher (i.e., if the average satisfaction score at month 1 were higher), the effect of time on satisfaction (the slope) would tend to be lower, and vice versa. This means that users with initially high satisfaction scores might show less change over time compared to subjects with initially low scores.

### Random Slope Model 

```ruby
random_slope_model <- lmerTest::lmer(satisfaction ~ time_numeric + (time_numeric | subject_id), data = longitudinal_data)

# View the summary of the model
summary(random_slope_model)
```
Here is the output:

<img width="638" alt="Screen Shot 2024-01-25 at 1 07 00 PM" src="https://github.com/KayChansiri/Longtitudinal-Multilevel-Modeling/assets/157029107/9911fdc0-d1c8-42d9-8c7f-3d04be986e41">


* Notice that we no longer receive the 'boundary (singular) fit' warning after working with the new dataset, which we purposefully modified to include both random and fixed effects. However, in the real world, you may still encounter this warning depending on the nature of the dataset you are working with.
* According to the output, the REML Criterion at Convergence (1001.7) suggests that the random slope model might provide a better fit than our previous model, as lower values generally indicate a better fit. We will reconfirm this assumption by checking the AIC and BIC values across models.
* The distribution of residuals ranges from -1.79912 to 1.94029, suggesting that there are no extreme outliers in the model's predictions, as the deviation between predicted and observed values is not very high.
* Regarding random effects, the random intercept variance is 2.35461 with a standard deviation of 1.5345, indicating variation in baseline satisfaction levels across users. The variance of the random slope for time, at 0.06494 with a standard deviation of 0.2548, indicates variability in how satisfaction changes over time across users, although this variation is likely small given the variance is only 0.06.
* The correlation between the random intercept and random slope is 0.35, suggesting a moderate positive relationship. This implies that users with higher baseline satisfaction tend to have a different rate of change in satisfaction over time compared to those with lower baseline satisfaction.
* The residual variance is 0.61335 with a standard deviation of 0.7832, indicating the variation in satisfaction scores not explained by the model.
* Regarding fixed effects, the effect of time on satisfaction is estimated at 0.89000 with a significant p-value, suggesting that satisfaction changes significantly over time.
* The correlation of -0.400 between the intercept and time_numeric suggests a moderate inverse relationship between these two estimates in the model, as mentioned in the previous model.
* Now, let's compare the model fit. This time, I will use the anova() function for comparison. However, keep in mind that this function is applicable only if the two models are built using the same function. For example, in the current context, both the random_intercept_model and the random_slope_model are built using lmerTest::lmer and therefore can be compared using anova(). However, comparing the level1_fixed_model, which was built with the lm() function, to the random_intercept_model, which was built with the lmer() function, would not be appropriate for anova comparison.
  
## Model Fit Comparison

```ruby
anova(random_intercept_model, random_slope_model)
```
Here is the output:

<img width="598" alt="Screen Shot 2024-01-25 at 2 02 08 PM" src="https://github.com/KayChansiri/Longtitudinal-Multilevel-Modeling/assets/157029107/6231d724-42aa-4303-b9b2-b1ee9ba75932">

* The interpretation of the output is a bit more nuanced. For both BIC and AIC, lower values indicate a better fit. The random_intercept_model has a lower BIC (1025.9) compared to the random_slope_model (1030.6), suggesting that the simpler random_intercept_model might be more appropriate. However, the random_slope_model has a lower AIC than the random_intercept_model, suggesting it might provide a better fit when considering model complexity.
* When AIC and BIC point in different directions, it's important to consider other evidence such as log-likelihood or previous empirical studies to see whether user satisfaction of a streaming service is likely to vary across individuals over time. This information can inform whether we should include the random slope parameter. Deciding between the models also depends on the specific goals of your analysis and the nature of each project you are working with.
* LogLik (Log-Likelihood) is another model fit parameter where higher values indicate a better fit. According to the output, the random_slope_model has a higher log-likelihood (-498.17) than the random_intercept_model (-501.55). These results are consistent with the AIC values.
* Regarding deviance, lower values indicate a better fit. Thus, a lower deviance for the random_slope_model (996.34) compared to the random_intercept_model (1003.09) indicates a better fit.
* The chi-squared statistic (6.7546) indicates a significant p-value (0.03414), suggesting that the model with more parameters (random_slope_model) provides a significantly better fit to the data.
* In summary, the random_slope_model provides a statistically significantly better fit to the data than the random_intercept_model, as indicated by the lower AIC, higher log-likelihood values, and the significant result in the chi-squared test. This suggests that allowing the effect of time on satisfaction to vary across users (random slope) is important in modeling these data. Let's check the ICC to see if our assumption about adding random effects to improve the model fit is likely true.

## ICC
* Earlier in the post, I mentioned that the Intraclass Correlation Coefficient (ICC) is used to quantify the proportion of the total variability in the data attributable to the grouping structure in mixed-effects models (such as between subjects in a longitudinal study). In other words, the ICC explains how much of the variability in the outcome variable can be explained by the random effects (i.e., random slopes and random intercepts) as opposed to the residual error (the variance not explained by the model). 
* An ICC close to 1 suggests that most of the variability in the outcome is due to differences between groups (subjects). In longitudinal data, this implies that the effects of time are not as strong as interpersonal or intergroup variability.
* An ICC close to 0 suggests that most of the variability is due to individual differences within groups or measurement error. In longitudinal data, an ICC at 0 indicates that group or individual differences do not really explain the variance in the outcome. Let's see what it's like in our case:

```ruby
performance::icc(random_slope_model)
```
Here is the output:

<img width="419" alt="Screen Shot 2024-01-25 at 2 25 27 PM" src="https://github.com/KayChansiri/Longtitudinal-Multilevel-Modeling/assets/157029107/2567d957-8502-430b-9834-5729e5e5ba0b">

* According to the output, we have two types of ICC: the Adjusted ICC and the Conditional ICC.
* The Adjusted ICC is an estimate of the proportion of the total variance in the outcome that is attributable to the between-group (subject-level) variability when controlling for the fixed effects in the model. The value 0.840 indicates that 84% of the total variability in satisfaction scores is due to differences between users, after accounting for the fixed effects in our model, which, in this case, we do not have any except the intercept term of within-subject effects.
* This high value suggests that our data is clustered and therefore multilevel modeling is a good analysis strategy to understand individual differences among users and the influence of the variation in satisfaction scores.
* The Conditional ICC, on the other hand, represents the proportion of the total variance that is attributable to the the between-group (subject-level) variability when considering both the fixed and random effects. This is unlike Adjusted ICC that considers only the fixed effects.
* The Conditional ICC of 0.738 in the output means that when considering the entire model, including both fixed and random effects, about 73.8% of the variability in satisfaction scores can be attributed to differences between users. 
* In summary, both the Adjusted and Conditional ICC values are quite high, indicating that the differences between subjects are a major source of variability in satisfaction scores. This underscores the importance of considering the subject-level random effects in the model. The difference between the Adjusted and Conditional ICC values shows the additional variance explained by the random effects in the model.



