# Study-AB-Testing


## Intro: SO...Not Enough Samples ? 
#### Estimation techniques for finding "good statistics"
 - Maximum Likelihood Estimation
 - Method of Moments Estimation
 - Bayesian Estimation
 - *Aggregate approach (CI, Hypothesis_test)
 - Individual approach (machine learning, but does need more samples, doesn't care finding "good statistics")
> Confidence intervals and Hypothesis tests: It takes an `aggregate approach` towards the conclusions made based on data, as these tests are aimed at understanding population parameters (which are aggregate population values).

> Machine learning techniques: It takes an `individual approach` towards making conclusions, as they attempt to predict an outcome for each specific data point.

#### Hypothesis testing
 - Testing a population mean (One sample t-test).
 - Testing the difference in means (Two sample t-test)
 - Testing the difference before and after some treatment on an the same individual (Paired t-test)
 - Testing a population proportion (One sample z-test)
 - Testing the difference between population proportions (Two sample z-test)
 - ETC. instead of memorizing how to perform all of these tests, we can find the statistics that best estimates the parameter(s) we want to estimate, we can bootstrap to simulate the sampling distribution. Then we can use our sampling distribution to assist in choosing the appropriate hypothesis.
 
#### Once we set up 'H0', we need to use our data to figure out which hypothesis is true. There are two ways to choose. 
 - __Using C.I:__ where we simulate sampling distribution of our statistics, then we could see if our hypothesis is consistent with what we observe in the sampling distribution.  
 - __Simulating what we believe to be a possible under the H0,__ then seeing if our data is consistent with that.  

# 1) Using C.I & Bootstrapping
__*When estimating population parameters, we build the confidence intervals.__
 - Basically;
   - Increasing your sample size will decrease the width of your confidence interval (by the law of large numbers).
   - Increasing your confidence level (say 95% to 99%) will increase the width of your confidence interval. 
 - C.I. that Capturing pop-mean/proportion
<img src="https://user-images.githubusercontent.com/31917400/34266269-6c003960-e670-11e7-857f-3a755839c4b9.jpg" width="200" height="150" />  

 - C.I. that Capturing the difference in pop-mean/proportion
<img src="https://user-images.githubusercontent.com/31917400/34266277-6e55239c-e670-11e7-924e-212e7c9f7876.jpg" width="290" height="150" /> 

 - C.I. with t-test as the Traditional mean comparison method with sampling - ex>"height"
```
df_samp = df.sample(200)

import statsmodels.stats.api as sm
X1 = df_samp[df_samp[condition] == True]['height'] 
X2 = df_samp[df_samp[condition] == False]['height']

cm = sm.CompareMeans(sm.DescrStatsW(X1), sm.DescrStatsW(X2))
cm.tconfint_diff(usevar='unequal')
```
All of these formula have underlying "assumptions" (Central Limit Theorem - regarding the sampling distribution ie.the distribution of statistics) that may or maynot be true. But **Bootstrapping** does not need the assumptions of these intervals. Bootstrapping only assumes the sample is representitive of the popluation. With large enough sample size, Bootstrapping and the traditional methods would provide the same Confidence Interval.    

__*Bootstrapping and C.I.__
 - We just use a bootstrapping with 10,000 iterations to build a confidence interval !
```
df_samp = df.sample(200)

A_means, B_means, diffs = [], [], []
for i in range(10000):
    bootsamp = df_samp.sample(200, replace = True)
    A_mean = bootsamp[bootsamp[condition] == True]['height'].mean()
    B_mean = bootsamp[bootsamp['condition'] == False]['height'].mean()
    A_means.append(A_mean)
    B_means.append(B_mean)
    diffs.append(A_mean - B_mean)   
```
 - Compute the C.I.
``` 
A_lower, A_upper = np.percentile(A_means, 2.5), np.percentile(A_means, 97.5)
B_lower, B_upper = np.percentile(B_means, 2.5), np.percentile(B_means, 97.5)
diff_lower, diff_upper = np.percentile(diffs, 2.5), np.percentile(diffs, 97.5)
```
 - See the distribution of parameters and see if the H0 can be rejected
``` 
plt.hist(A_means);
plt.axvline(x=A_lower, color='r', linewidth=2)
plt.axvline(x=A_upper, color='r', linewidth=2);

plt.hist(B_means);
plt.axvline(x=B_lower, color='r', linewidth=2)
plt.axvline(x=B_upper, color='r', linewidth=2);

plt.hist(diffs);
plt.axvline(x=diff_lower, color='r', linewidth=2)
plt.axvline(x=diff_upper, color='r', linewidth=2);
```

# 2) Simulating From the Null Hypothesis
__* 
 
 
 
 
 

## What if our sample is large?
One of the most important aspects of interpreting any statistical results (and one that is frequently overlooked) is assuring that your sample is **truly representative** of your population of interest. Particularly in the way that data is collected today in the age of computers, `response bias` is so important to keep in mind. In the 2016 U.S election, polls conducted by many news media suggested a staggering difference from the reality of poll results. 
> Two things to consider
 - a) Is my sample representative of the population?
 - b) What is the impact of large sample size on my result? (with large sizes, everything will be statistically significant..then we'd always choose H1---[Type-I. Error]) 

#### Multi-testing Correction
When performing more than one hypothesis test, your type-I error compounds. In order to correct for this, a common technique is called the `Bonferroni correction`. This correction is very conservative, but says that your new type-I error rate should be the error rate you actually want divided by the number of tests you are performing. Therefore, if you would like to hold an **allowable type-I error rate of 1%** (99% confidence means alpha=0.01) for each of 20 hypothesis tests, the Bonferroni corrected rate would be 0.01/20 = 0.0005. This would be the new rate you should use as your comparison to the p-value for each of the 20 tests to make your decision.

## A/B testing 
 - When a company wants to test new versions of a webpage? 
 - A/B tests are used to test changes on a web page by running an experiment where a control group sees the old version, while the experiment group sees the new version. A **metric** is then chosen to measure the level of engagement from users in each group. These results are then used to judge whether one version is more effective than the other. A/B testing is very much like hypothesis testing.
   - Null Hypothesis: The new version is no better, or even worse, than the old version (H0: 'new' =< 'old')
   - Alternative Hypothesis: The new version is better than the old version (H1: 'new' > 'old')
 - If we reject the null hypothesis, the results would suggest launching the change. These tests can be used for a wide variety of changes to see what change maximizes your metric the most.
 - A/B testing also has its drawbacks: 
   - Type I. (Says False, but H0 actually True) Change Aversion or `false positives`: Existing users may give an unfair advantage to the old version, simply because they are unhappy with change, even if it’s ultimately for the better.
   - Type II. (Says True, but H0 actually False) Novelty Effect or `false negatives`: Existing users may give an unfair advantage to the new version, because they’re excited or drawn to the change, even if it isn’t any better in the long run.



