---
published: true
title: "A more detailed solution to GDMC 2017"
categories:
  - video games
  - behavioral data
  - machine learning
---  
## Part 1: Prediction engineering: How to Set Up Your Machine Learning Problem 


While data science has become essential to almost all industries,  in video games - reducing player churn is crucial to increase player engagement and game
monetization; besides, data science and predictive analytics show promises of transforming the video game industry as personalized user experience.
Game data sets are very had to find publicly available, especially coming from  massively multiplayer online role-playing game, not to mention any data science
solution related to a specific business problem. For this, the [Game Data Mining Competition (GDMC) 2017](https://cilab.sejong.ac.kr/gdmc2017/) organized by NCSoft provided a large data set from Blade and Soul
game log data. It consisted of one training dataset and two test datasets. Each of the datasets covered different periods of time and
had information about different users. The aim of this competition  was to predict churn and life expectancy during a certain prediction period.

![](/images/futureOfGaming1.png)
*https://qz.com/is/what-happens-next-2/1438720/future-of-gaming/


This is the first in a three-part series on how to approach machine learning.

This article covers the concepts and full implementation as applied to predicting customer churn to the data set from GDMC 2017. 
The project [Jupyter Notebooks](https://github.com/DanielaLaura/dataproc_unittest/blob/master/Data/Data_processed/working%20on%20data%20formatting%20games.ipynb) and [Python files](https://github.com/DanielaLaura/dataproc_unittest) (also, organized as for a data production pipeline) for this  project are all available on GitHub.

When working with real-world data on a machine learning task, we define the problem, which means we have to develop our own labels - historical 
examples of what we want to predict, that is to train a supervised model. The idea of making our own labels may  seem foreign to data scientists  
who got started on Kaggle competitions or textbook datasets where the labels are already included, and this is the case of GDMC 2017. 
The concept behind prediction engineering-making labels to train a supervised machine learning model-is not a standardized process  and is done 
by data scientists on an as-needed basis. 

The inputs to prediction engineering are the parameters which define the prediction problem for the business requirement , and the historical dataset for finding 
examples of what we want to predict.
The output of prediction engineering is a label times table: a set of labels with negative and positive examples made from past data along with an associated 
cutoff time indicating when we have to stop using data to make features for that label. The labels are not complete without the cutoff time which represents when we have 
to stop using data to make features for a label. All the features for each label must use data from before this time to prevent the problem of data leakage. Cutoff times 
are a crucial part of building successful solutions to time-series problems that many companies do not account for. I have seen research paper describing the winning 
solution for this competition, accounting for the time after cuttoff as censored time. To be mentioned that using invalid data to make features leads to models that do 
well in development but fail in deployment.

### Dataset for Players Churn

Log data were provided in CSV format. Each player’s action log was stored in an individual 
file, with the user ID as the file name. In log files, where each row represents an observation and each column represents a variable
such as the time stamp, action type, and detailed information. Table 2 shows a sample log file.There were 82 action types, which represented 
various in-game player behaviors, such as logging in,leveling up, joining a guild, or buying an item.   For more details about the log data, the reader
is referred to the [competition website](https://cilab.sejong.ac.kr/gdmc2017/) .

To be able to do an extensive data analysis,a unique data frame which contains all users and their simple statistics for action types is created  as follows:


<script src="https://gist.github.com/DanielaLaura/db1f80b4c22c3278975d5f6c130c5e4e.js" charset="utf-8"></script>
<center>Data processing code.</center>

### Finding Historical Labels

The key to making prediction engineering adaptable to different problems is to follow a repeatable process for extracting training labels from a dataset. 
At a high-level this is outlined as follows:

1. Define positive and negative labels in terms of key business parameters
2. Search through past data for positive and negative examples
3. Make a table of cutoff times and associate each cutoff time with a label

For customer churn, the parameters are: 
- the prediction date (cutoff time): the point at which we make a prediction and when we stop using data to make features for the label
- number of days without playing before a user is considered a churn
- lead time: the number of days or months in the future we want to predict
- prediction window: the period of time we want to make predictions for
The following diagram shows each of these concepts while filling in the details with the problem definition we’ll work through.

![](/images/Diagram.png)
*Parameters defining the customer churn prediction problem.

Unlike the telecommunication services or other online subscriptions, for which a user churning can be easily defined and identified 
from the user unsubscribing, such isn’t the case for online game services. Online game players seldom delete their accounts
or unsubscribe although they have no intention of resuming playing the game. In this case, a user who does not play a game for
more than five weeks is defined as a churner.

### Labeling Implementation

To make labels, we develop 2 functions: 


<script src="https://gist.github.com/DanielaLaura/c10c462195f2ff812bb8ff99dd31fac4.js" charset="utf-8"></script>
<center>Label-customer function.</center>


This table has a set of prediction times—the cutoff times—and the label during the prediction window for each cutoff time corresponding to a single customer.


<script src="https://gist.github.com/DanielaLaura/e507285ab1a3c9670e0c2d8afd1b4ebf.js" charset="utf-8"></script>
<center>Make_labels function.</center>


The make_labels function then takes in the transactions for all customers along with the parameters and returns a table with the cutoff times and the label 
for every customer.


### Conclusion
The process of prediction engineering is captured in three steps:

1. Identify a business need that can be solved with available data
2. Translate the business need into a supervised machine learning problem
3. Create label times from historical data

Getting prediction engineering right is crucial and requires input from both the business and data science sides of a business. 
By writing the code for prediction engineering to accept different parameters, we can rapidly change the prediction problem if the needs of our company change.

