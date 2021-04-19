# **Predicting Github Commit Disengagement**

## 1.	Introduction

### 1.1 Background
The Github open source community provides a unique opportunity to both businesses and the independent developers. To the businesses it provides a means to open the software and other products to a large base of inspectors who can quickly detect issues and provide feedback. To the independent developers, the open source platform provides a means to improve one’s technical skillset, work with more experienced practitioner and grow their professional network. 

The success of this platform depends on the employees of the organizations to maintain the Github repositories. These employees play a variety of roles, such as attend to code issues, perform code reviews, manage pull requests and perform commits. Maintaining a good work environment for these employees is important to secure the employee’s engagement in a sustainable way.

### 1.2 Project Aim
This analysis looks to study the potential predictive nature of variables i) workload related factors (i.e. number of commits by employee in their initial months) and ii) the stability of the work environment (i.e. work staff changes in the employee’s initial months) on employee engagement/disengagement. We consider an employee is “possibly_disengaged”, if we see a drastic reduction in their number of commits in the future months compared to their participation in the initial months to an organization. If we see no such drastic reduction, we label those employees as “no_signs”. We take the first six months as the beginning months of a developer as the baseline period. Our analysis will focus on the developers committing to the Apache Github organization.

### 1.3 Hypothesis
In terms of workload related factors, it is hypothesized that the magnitude of workload in the beginning months predicts future disengagement. For example, over-worked employees (i.e. performing a very high number of commits) in the beginning months would tend to disengage in the future months. 
In terms of work environment stability related factors, it is hypothesized that frequent changes in the committer team personnel to manage a repo contributes to an unstable work environment and predicts future employee disengagement.

### 1.4 Adaptation From Selected Paper
This analysis attempts to reproduce one of the research questions of the selected paper [1]. One of the research questions addressed in the selected paper is “Given a developer’s first six monthly report data, can we effectively predict whether the developer will leave the company or not (i.e. not-retained vs. retained) after he/she enters the company for one year?”. This analysis poses the research question “Given a developer’s first six months of commit participation to a Github organization, can we effectively predict whether a developer will disengage in the future from the organization?” Hence,  instead of developer contribution to projects, we look at developer contribution to repositories for the organization. And instead of hours worked, we look into number of commits performed.

The features utilized in this project is similar in nature to the features used in the selected paper. The selected paper looks into the monthly activity summary self-reports of the developers and considers features such as working hours (e.g. as a measure of magnitude of workload), the overall project contribution statistics (e.g. number of projects) and the project team member changes (i.e. as a measure of the project team stability).  The paper tests various classification models such as Random Forest, SVM etc. to predict if the employee will leave the company in one year.

The remainder of the paper is structured as follows. Section 2 describes the data preparation and overview of the dataset.  Section III describes the experiment setup and exploratory analysis. Section IV presents the results of the research question. Section V concludes the paper and discusses future directions.

## 2.	Data Preparation
The dataset used was the commits table within the github_repos dataset available on Google BigQuery. We focus our analysis on the commit activity in repos within the Apache GitHub organization. Fig 2.1 provides an overview of the SQL code used to extract data from Google BigQuery. Fig 2.2 shows a snapshot of the pre-processed dataset. Table 2.1 provides a high level overview of the dataset.
Fig 2.1 Google BigQuery Query
 





Fig 2.2 Pre-Processed Dataset from Google BigQuery
 
 

Table 2.1 Data Set High Level Overview
Number of Records	1,048,476
Time Period	June 2006 till October 2016
Number of Repos	409
Number of Committers	7165

Please refer to the Jupyter Notebook for a detailed discussion on the data preparation steps.

## 3.	Case Study Setup

### 3.1 Research Question Setup
We formulate the prediction task of developer engagement as a binary classification problem based on the developers’ participation towards the organization’s repos based on their commit activity. The selected paper defines the binary target variable as employees who left the company vs. employees who stayed with the company. However, for our analysis, we define the binary target variable whether the employee is disengaged or engaged. If the number of commits in the last six months of the analysis (i.e. between months 30 to 36), are less than 20% of the commits made in the first six months, then we deem the person to be “possibly_disengaged”, else the person is labelled as “no_signs” since they do not show signs of disengagement. This stringent criterion (i.e. 20%) is used to account for variations in number of commits by employee due to vacation, sick leave etc. This strict criterion should enable us to focus on the developers with drastic reductions in their commit activity.

In the paper, since every new developer in the company has a six-month probationary period, the authors use the first six monthly reports after the developers enter into the company to predict the future developer turnover. For our analysis, we use a data-driven approach to select this time period.  We want to make sure that we select a time period that is relatively stable, such that it provides a good baseline measure for each developer to compare to future participation. From Fig 3.1, we notice that the average number of commits per developer remains fairly steady over the first six months. Hence, we choose the level of participation in the first six months as a reasonable baseline, that we can use to compare with future level of participation to assess engagement/disengagement for each developer.  
Figure 3.1 Average Number of Commits by Developer
 

In terms of the time period of the analysis, the selected paper aims to predict the employee leaving the company in one year after joining. However, employees can leave an organization for other reasons such as better salary or increased responsibility etc. The longer the time period of the analysis, the greater the effect of this confounding variable. Hence, to limit the effect of this confounding variable in our dataset, (that spans 10 years of commit data), by limiting our analysis to only the first two years, since a developer’s first commit.  A data-driven approach was taken to select this. We define period after baseline (PAB) as each successive 6 months durations after the baseline period. For example, PAB=1 represents the time period between month 6 and month 12 for the developer since the time of the developer’s first commit. PAB=2 is time period between month 12 to month 18 and so on. We calculate the number of developers that contribute less than 20% of their baseline for each PAB. Then, we plot the rate of change of developers from one PAB to the next using formula [y{PAB+1} – y{PAB} ] / y{PAB} as visualized in Fig. 3.2. Here, we see that the periods with the greatest increase in the number of “possibly disengaged” developers are the first 4 PABs. Hence, the time period for our analysis is selected to be 2 years. 
Fig 3.2 Rate of Change of Disengagements
 
Hence, the research question of this analysis is ““Given a developer’s first six months of commit participation to a Github organization, can we effectively predict whether a developer will disengage in their first 3 years since their first commit to the organization’s Github open source platform?”

### 3.2 Factors Potentially Affecting Developers’ Departure
Factors potentially affecting developer departure (talk about various potential factors like in paper, univariate and bivariate exploration)
In this study, we consider a developer’s first six months of commit participation to the organization and extract 13 set of features along two dimensions, that might be correlated with developers’ engagement. Please refer to Jupyter Notebook for a detailed describtion on the meaning of each factor.

Stability of Work Environment refers to factors that influence the working environment in the team managing the repository. Working environment might have a very important effect on a developer’s working experience. For example, the good collaboration with other members in the repository can improve a developer’s work efficiency and experience. For each month, we calculate the following measures of the repository which the developer is working for: number of new developers and number of absent developers for the repo. The number of changed developers within a repo might indicate the stability of the project. The developers often prefer stay at a stable project. We also count the number of developer changed in the project which a developer works for (total_repo_comm_absent total_repo_comm_new agg_absent_months new_ppl_months no_change_months ), since the stability of the project might have impact on the working experience of a developer. 

Developer Workload refers to factors related to the workload of the developer. The workload of the developer is correlated to the number of commits performed by the developer. Developers are often asked to take heavy workload and have tight deadlines. Heavy workload might be a factor which affects a developer’s engagement. For each month, we calculate the following measures of the repository which the developer is working for: the number of project members (P(n)_repo_committers), the sum of commits of project members (P(n)_numb_commits_repo), and check whether a developer takes part in more than one project (P(n)_multi_ratio), The number of repo team members is an indicator of project size. Small project size usually means more workload to each individual in the project. The number of commits by project members could reflect the overall workload in the project.

### 3.3 Data Exploration
We find that there are no missing values in the data. Moreover, we also see that indeed, all repositories start with the word ‘apache’ signifying they are under the Apache organization. Not knowing exactly how the data is collected by Google BigQuery from Github, we recognize the possibility of erroneous entries in the data, which constitutes to random noise in our prediction models. We proceed to perform univariate and bivariate exploration of our dataset. Given the time-consuming nature of running queries in JuputerNotebook, we extract the spark dataframe into csv files, and use Tableau and Python Seaborn library only for data exploration purposes.

#### 3.3.1 Univariate Analysis
We see that we have an imbalanced classification problem from Fig 3.3. Moreover, from Fig 3.4., we see that there is no apparent difference in the distribution of “possibly_disengaged” and “no_signs” for any of the features. From Fig 3.5. we see that there is a general increase in the number commits, number of unique committers and number of repositories over the years.
Fig 3.3. Target Variable Distribution
 

Fig 3.4. Univariate Distributions
 	 
 	 
 	 

Fig 3.5. Number of Commits, Number of Unique Committers and Number of Repositories w.r.t. Year
 	 	 

#### 3.3.2 Bivariate Analysis
From Fig 3.6, we notice an interesting effect of total_repo_comm_new on the target variable distribution on the specific subset. This suggests that frequent changes in the repository committer team, specifically frequent addition of new members, may be related to future developer disengagement. Moreover, in Fig 3.7, we notice an interesting strong correlation value of 0.90 between total_repo_comm_new and total_repo_comm_absent, suggesting that work staff changes in repositories involves a replacement of personnel as opposed to a net decrease or net increase in the committer team personnel. We also see the expected correlation between agg_absent_periods, new_ppl_periods and no_change_periods since new_ppl_periods and no_change_periods are used to calculate agg_absent_periods. 


Fig 3.6 Effect of total_repo_comm_new
 

Fig 3.7. Pearson Correlation Plot
 


## 4.	Results
For our commit activity data, we use our proposed factors to train a classifier to predict whether a developer will be disengaged after he/she performs the first commit in two years. The selected paper studies different classifiers, including Naive Bayes, Support Vector Machine (SVM), Decision Tree, K-Nearest Neighbor (kNN), and Random Forest. As the model metric, the selected paper interprets the model using the F1-score. For our analysis, we use the logistic regression model  and the AUC-ROC score as a basis to interpret the model results in the context of our research question. Note that due to technical limitations, only a subset of the features were used for the model, these are: 'no_change_periods',  'tot_repo_comm_absent',  'total_repo_comm_new',  'agg_absent_periods',  'new_ppl_periods' and  'avg_multi_ratio'.
Table 4.1 below provides a summary of the results obtained. We observe that our models does not classify the dataset particularly well. The F1-score is 0.6. As a result, there is no evidence to ascertain the hypothesis of this analysis that that the magnitude of workload and the work environment stawhich  remains unconfirmed. 
Table 4.1 – Model Evaluation
Criteria	Value
Train Set Size 	733,710
Test Set Size	314,826
Logistic Regression (AUC-ROCScore)	0.53

## 5.	Conclusion
In this paper, based on commit activity of Apache repositories extracted from Google BigQuery, we use data mining technique to investigate whether a developer will become disengaged within the first 2 years after he/she performs the first commit to the organization. The commit activity dataset we used contains about 1 million commits, performed by 7,000 developers in 400 Apache repositories in about 10 year period. Our study reveals the most effective classifier (i.e., random forest) for the prediction of developers’ departure was unable to identify relationships between the two dimensions of factors and developers’ disengagement. As future work, instead of just relying on changes in the number of commits to ascertain possible disengagement, the feature set of the model can be expanded to also include other forms of developer participation e.g. number of pull requests, issue resolutions and code reviews. These would help build a more comprehensive view of employee participation towards the organization. As a future work, it is also suggested to studying a greater number of Github organizations to further increase the number of observations in the dataset and hence, further strengthen the results of the analysis. 

## 6.	References
This section cites the selected paper on which this analysis is based.
BAO, Lingfeng; XING, Zhenchang; XIA, Xin; LO, David; and LI, Shanping. Who will leave the company?: A large-scale industry study of developer turnover by mining monthly work report. (2017). 14th IEEE/ACM International Conference on Mining Software Repositories: MSR 201, Buenos Aires, Argentina, 2017 May 20-21. Research Collection School Of Information Systems. Available at: https://ink.library.smu.edu.sg/sis_research/3696
