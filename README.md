This is the project writeup file.

# ANLY502 - Massive Data Analytics Big Data Project - Shiqi Ning
## Predict the Reddit Comment Score

## Introduction
The Reddit Comments archive contains various information about the Reddit posts, the data collected includes the files/comments information for Oct/Nov/Dec 2018 and Jan 2019. For this mini project, I will mainly focus on predicting how different factors can affect score on public posts.


The features contained in each file are identical, and each file contains 39 features which include: 
* `archived`: whether the post is archived
* `author`: the account name of the poster. null if this is a promotional line
* `author_cakeday`: whether the day of the post is the yearly anniversary of when the author signed up on Reddit
* `author_flair_background_color`: the background color of author’s flair
* ` author_flair_text_color`: the text color of author’s flair
* ` author_patreon_flair`: whether the author is a patreon flair
* ` controversiality`: whether difference between the upvoters and downvoters are hugh, with 1 being huge, 0 being the opposite
* `body`: the content of the post
* `distringuished`: whether the posts have been distinguished by moderators/admins. `null` = not distinguished. `moderator` = the green [M]. `admin` = the red [A]. `special` = various other special distinguishes
* `edited` =  `false` if not edited, edit date in UTC epoch-seconds otherwise. 
* `gilded`: the number of times this comment received reddit gold
* `id`: this item’s identifier
* `no_follow`: whether there is follow up post after the original post
* `score`: the net-score of the comment
* `stickied`: true if the post is set as the sticky in its subreddit
* `subreddit`: `null` if not a comment, genre of the posts
* `subreddit_type`: the subreddit's type - one of "public", "private", "restricted", or in very special cases "gold_restricted" or "archived"
* etc.
* Note: most the above description of the features are collected from the following repository: https://github.com/reddit-archive/reddit/wiki/JSON, others are searched from the Reddit discussion forum.


## Code
For the code file go to [Big Data Project.ipynb](https://github.com/gu-anly502/spring2019-miniproject-lemonning0713/blob/master/Big%20Data%20Project.ipynb), which contains different segments of the analysis: 
* Accessing files from Spark
* Exploratory Analysis
	* Data Cleaning
	* Feature Generation and Sampling
* Machine Learning
	* Logistic Regression
	* ROC plot



## Methods section:

### Exploratory Analysis
For the exploratory analysis, I checked the features by counting the number of unique values/level they have. I also used `groupBy` command to calculate the corresponding highest appearing subreddit groups when their `score` are positive and negative. And I selected the values in the `score` column where the values are not null using `spark.sql` queries and used `rdd.map` to map the `score` column into an array. Then, I imported the `matplotlib.pyplot` package and plotted a boxplot for it to check its distribution. By summarizing `score`, and closely examining the plot, I found that `score` ranges from negative numbers to over 90000. I further modified the `score` column in the next section for the purpose of getting a more accurate prediction result

### Data Cleaning:
Since I’m only interested in certain types of Reddit posts that are public and valid, I filtered the each file by selecting `author` = `not null`, `can_gild` = `true`, `can_mod_post` = `true`, `subreddit_type` = “`public`”, `subreddit` = `not null`, and `score` = `not null` and reserved only a few of the features that are more relevant in affecting the score, as well as removed irrelevant features like author personal information, id, and content about the posts, and etc..

The final features that I reserved for this projects are: `author_cakeday`: boolean, `author_flair_background_color`: string, `author_flair_css_class`: string, `author_flair_text_color`: sting, `author_flair_type`: string, `author_patreon_flair`: boolean, `controversiality`: long, `distinguished`: string, `gilded`: long, `is_submitter`: boolean, `no_follow`: boolean, `send_replies`: boolean, `stickied`: boolean, `subreddit`: string, and `score`: long, as `score` is treated as the prediction. 

### Feature Generation and Sampling
To be specific, I wanted to investigate what kind of factors can lead to a negative/positive score of the post. So, I grouped `score` into two groups: 1: positive scores and 0: negative scores by first change the `score` column from long type to double type, and then used a `Binarizer` transformers to get the class of 0 and 1. And a servere problem emerged, as the data becomes highly imbalanced. With the size of the negative score group is about only one tenth of the size of the positive score group. So, I sampled out the same size of the negative score group from the positive score group. 

From the exploratory analysis from the previous section, null values exist in most of the columns, and need to be fixed. In this project, after closely examing the null values in each feature, I identified all the null values in boolean-typed features as false, and assigned all the null values of string-typed features to a new class within each column as `unknown`, so that I can also see how these null values can affect the final result.

And I used one of the sample files (1m-line-sample.json.lzo) provided as my intermediate dataset, and used it to test my codes. And for the final results I concatenated the four data files together and ran the machine learning procedure.


## Machine Learning
### Logistic Regression
Since I have a binary class as the response, I chose to use the Logistic Regression to make classifications, since this algorithm works well with binary classes. I imported  packages from  pyspark.ml.classification,  pyspark.ml to perform machine learning. 

I first used `randomSplit` to separate the data into training and testing set with the proportion of 0.8:0.2. Then, I defined the estimator (logistic regression) and then built the pipeline consisting of the `Stringindexer`, `binarizer`, `vectorAssembler` and `estimator`. Then, I used `StringIndexer` to change the features with string type to numeric. And assembled the different vectors by VectorAssembler. Next, I trained the model with the training set. 

## Results
I imported the pyspark.ml.evaluation package, evaluated the model and checked the model accuracy using `BinaryClassificationEvaluator` on the testing set. I found a prediction accuracy of about 0.75, which is quite good. And I did a ROC curve to demonstrate my prediction result. This shows that all the features that I’ve chosen has an effect on the score.

## Future Work
1. I can separate the `score` into multiple class, instead of only two classes, so that I can get a more detailed prediction for different levels of `score`.
2. I can apply different models like Random Forest, SVM or Naive Bayes to redo the machine learning to see how different machine learning algorithms can make a difference on the final prediction accuracy rate.
3. I’d also like to include `body`, `text` columns and counted the most commonly appeared words in the posts, and check the potential words can appeared the most for some high-scored posts.
4. I found that the subreddits with the highest three occurrances for positive and negative scores respectively are both "AskReddit", "nfl" and "politics", this may indicate interesting trend which can be further investigated.



