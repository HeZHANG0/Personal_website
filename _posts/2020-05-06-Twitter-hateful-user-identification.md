---
title: "Twitter hateful user identification using deep learning graph models"
date: 2020-05-06
published: true
tags: [GraphSAGE, Twitter, hateful, graph model, GloVe, neural network]
excerpt: "Identifying hateful Twitter users using graph node features and graph network."
toc: true
toc_sticky: true
---

# Overview
Detecting hateful users protect the online social environment from contamination. This project builds models from the Twitter user and network dataset to predict whether a user is hateful or not. The data consists of user information, user's tweet information, user's network information and the complete network graph. I first explored the data, and found out that hateful users are usually more active, more powerful in the social network, and tweets more negative words. Then I built a logistic regression model using manually engineered features of user, tweet, and nerwork information. The second attempt was to directly learn a predictive model from the user's tweet embedding (GloVe embedding in this data). Lastly, I built Graph Convolutional Network models using both user features and the network graph, and compared GraphSAGE, GCN, and GAT models. It turns out that the logistic regression model and GraphSAGE have the most satisfying performance. The advantage of GraphSAGE is that it can effectively be trained on a much smaller portion of the data to achieve satisfying performance. The graph embedding is also shown to differentiate hateful users from normal users via a TSNE analysis.

# Data
This project uses the data publicly available on Kaggel: https://www.kaggle.com/manoelribeiro/hateful-users-on-twitter#users_neighborhood_anon.csv

# Methods
I combined different methods and data to build predictive models: 1) Logistic regression on user, tweet, and network information; 2) simple neural network on tweet embedding; 3) Graph Convolutional Networks on user and tweet information together with the complete network graph. 

# Exploratory Data Analysis: What Are Hateful Users Like?
Below I compared the user, tweet, and network features across different categories of users: hateful vs. normal users; neighbor of hateful users vs. non-neighbor of hateful users; suspended users vs. non-suspended users.

On average, the hateful users are more active:
![active]({{ site.url }}{{ site.baseurl }}/assets/twitter/active.png)

More powerful and central in the network:
![power]({{ site.url }}{{ site.baseurl }}/assets/twitter/power.png)
![central]({{ site.url }}{{ site.baseurl }}/assets/twitter/central.png)

Focuses more on tweeting negative contents:
![sentiment]({{ site.url }}{{ site.baseurl }}/assets/twitter/sentiment.png)

One might also conjecture different tweet empath features between hateful and normal users, but this is harder to tell immediately given the high dimension of empath. In further analysis, an empath-focused dimension reduction analysis could be helpful.
![empath]({{ site.url }}{{ site.baseurl }}/assets/twitter/empath.png)

# Predictive Modeling
## Data preprocessing
In the dataset, about 5% of the users are annotated as 'hateful' or 'normal'. I trained my models on these users (n=4971).

I dropped the columns that are mostly null values. Since the numerical variables have long tails, I appled power transformation to map those to Gaussian distribution. For example: 

![pre-transform]({{ site.url }}{{ site.baseurl }}/assets/twitter/pre-transform.png)
![post-transform]({{ site.url }}{{ site.baseurl }}/assets/twitter/post-transform.png)

We have an unimbalabced dataset here. Among the 4971 users, 544 are 'hateful' and 4427 are 'normal'. Therefore, I calculated the class weights to feed into model training. It turns out that each 'hateful' label has a weight of 4.57 and each 'normal' has a weight of 0.56.

For the train/test data split, I used 0.7/0.3 for the first two models. For the GCN models, given its properties and referenced best practice, I used 0.15/0.85 split. Therefore, the GCN models will require much fewer training samples than the other methods.

## Build a model on user, tweet, and graph features
I first built a model on manually engineered features that cover information of the users, their tweets, and their characteristics in the network. It achieves a pretty high testing accuracy of 0.928, and a high AUC value of 0.965. The precision is 0.715 and the recall is 0.571, yielding a F1 score of 0.635. The ROC curve of the model is shown below:



## Build a model directly on tweet embedding


## Build models on user & tweet features, and the network graph

## Model Comparison

# Reference
Referenced Github resources:

https://github.com/manoelhortaribeiro/HatefulUsersTwitter/tree/master/analysis

https://github.com/manoelhortaribeiro/GraphSageHatefulUsers

https://github.com/stellargraph/stellargraph/blob/develop/demos/use-cases/hateful-twitters.ipynb

Other referenced resources: http://snap.stanford.edu/graphsage/




