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

## RM-ANOVA VS MLM Equations
To better understand the differences between those two methods, Let's look at the equations for RM-ANOVA versus MLM using the Netflix example I discussed earlier:

**Repeated Measures ANOVA Equation**: Y<sub>ij</sub> = μ + α<sub>i</sub> + β<sub>j</sub> +(αβ)<sub>ij</sub> +ϵ<sub>ij</sub>
 * Y<sub>ij</sub> ​is the satisfaction score for subject i at time j.
 *  μ is the overall mean satisfaction score.
 *   α<sub>i</sub> ​and β<sub>j</sub>represent the individual and time effects, respectively.
 *   (αβ)<sub>ij</sub> is the interaction effect.
 *    ϵ<sub>ij</sub>​is the error term, assumed to be equally correlated across all observations.

**Multilevel Modeling Equation**:Y<sub>ij</sub>= γ<sub>00</sub>+γ<sub>10</sub>×Time<sub>ij</sub>+u<sub>0i</sub>+u<sub>1i</sub>×Time<sub>ij</sub>+ϵ<sub>ij</sub>
  * Y<sub>ij</sub>​ is the satisfaction score for subject i at time j.
  * γ<sub>00</sub> and γ<sub>10</sub>are the fixed effects for the intercept and time, respectively.
  *  u<sub>0i</sub> and u<sub>1i</sub> ​are the random intercept and slope, respectively, capturing individual differences.
  *  ϵ<sub>ij</sub> ​is the unique error term for each observation.

The error terms in these models are different. MLM accounts for individual differences in satisfaction scores and the relationship between time and satisfaction across individuals. RM-ANOVA, on the other hand, has a single error term.

# MLM Terminologies

Now that we've explored how MLM offers a more nuanced approach than RM-ANOVA, particularly in capturing individual differences over time, let's delve into some key terminologies often encountered in multilevel modeling.

## 1. Fixed Effects
* Fixed effects are assumed constant across all units in a dataset. They include fixed intercepts and fixed slopes.
* In the Netflix example, the fixed intercept represents the average consumer satisfaction score at the start point (e.g., month 1).
* The fixed slope is the effects of time, indicating how each unit increase in month of measurement uniformly affects satisfaction across all users.

## 2. Random Effects
* Random effects account for variations across units, allowing individual uniqueness. They are divided into random slopes and random intercepts.
* A random intercept in the Netflix scenario acknowledges that different users might have varying baseline satisfaction levels.
* A random slope indicates that the impact of month of measurement on satisfaction differs among users. For some, a month goes by might significantly boost satisfaction, while for others, the effect could be minimal.
* Considering these concepts of fixed versus random effects, think about the potential research questions in your projects. Using the Netflix example, you could explore:
  * Time-varying factors (like age or survey wave) influencing customer satisfaction.
  * The impact of personal attributes (gender, race, etc.) on satisfaction.
  * How these personal attributes affect initial satisfaction (intercept) and its evolution over time (slope).

> Remember, the heart of your analysis lies in framing the right questions and constructing an accurate model. Running the model is comparatively straightforward.

## 3. Levels in MLM
* MLM always involves multiple levels. In longitudinal data, Level 1 often represents time or time-varying variables, while Level 2 captures individual differences with time-invariant factors like race, gender, or other demographic characteristics that do not change across time.

## 4. Residuals vs. Random Effects
* We often refer to Level 2 error terms as 'random effects' and Level 1 error terms as 'residuals.'

## 5. Balanced vs. Imbalanced Designs
* In longitudinal MLM, when every subject has the same interval between measurements, for example, every six months,the design is considered balanced as everyone is measured on the same schedule.
* In cases where every subject is measured on the same set schedule but the intervals are not equal, for example, everyone is measured at month 1, month 4, and month 12, the design remains balanced. Even though the interval is not consistent (e.g., 3 months from wave 1 to wave 2 and 8 months from wave 2 to wave 3), it is still considered a balanced design because the measurement schedule is consistent across subjects.
* However, if people are measured on different schedules, then we have an imbalanced design. This scenario requires a different type of MLM approach, which is beyond the scope of this post.










