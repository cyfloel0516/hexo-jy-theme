---
title: NBA prediction based on player performance 
date: 2016-12-10 09:00:00
tags: 
    - Data Mining
    - Python
    - Classifier
---

A few months ago, I decided to implement a NBA game predictor for the term project of my Data Mining course. After weeks of work, I finnaly made some progress and decided to write something about it.

## Motivation
There have been so many predictors for NBA games in the world so why do we need another one? The reason is most of the models today predict the game based on the game history between teams but there are some flaws:
1. It is difficult to predict a match in the beginning of a season because the performance of
a team may vary due to the coach changing, player retirement or some other reason.
2. The prediction result becomes unreliable when big trades happen. A trade may involve
up to three teams and can definitely influence the team’s performance in the following
games.
3. The traditional way of prediction is different with human’s logic. Fans who want to do a
prediction will often check the rosters of their teams as well as the opponent’s. Then
they will compare each position and find the advantages. Moreover, if a key player got
injured and cannot play in the next game, then the team will definitely have lower
chance to win the game.

## The idea of the new predictor
As mentioned before, there are some flaws for a tradional prediction model. One easy way to solve these problems comes into my head that I can use the player performance comparison data instead of using the game history to predict the game. 

The basic idea is to generate the comparison data between pair of two players and uses these comparison data to perform the prediction.

Take a game for example. If we want to predict a result of an incoming game between teamA and teamB, and we have player_a_1...player_a_5 in teamA, player_b_1...player_b_5 in teamB. In my model, the prediction of the game is based on the comparison data between player_a_1-player_b_1, player_a_1-player_b_2 and so on. 

It is clear that this way of prediction is a more natural way to predict a game. Unlike most of current prediction model, it is going to measure the player performance.

Now the question is how to compare two players? There are many ways to do this, for example, you can compare their win rate or also you can compare their head to head stats. In my project, I use the head to head career stats as the comparison data. The stats includes: Minutes Played, Points, Offense Rebounds, Defense Rebounds, Assists, Steals, Blocks, Turnover, 3-Point attempt, 3-Point made, Field-Goal attempt, Field-Goal made. So one comparison data for a pair of two players looks like this:
``PTS_1, PTS_2, MIN_1, MIN_2, OREB_1, OREB_2 ...``

## Classify the comparison data
Once we have the  comparison data for pairs of players we can predict the game based on the comparison data. The first step is to know the class of the comparison data. What I mean by class can be simply understood as that we need to know who in a pair will be the winner between two players. In another word, we need to find out the winner between two players.

In order to get this, I compare the win rate between two players. The idea is very easy: if player 1 wins more games against player 2 when they play as opponents then I say player 1 will be the winner and the class of the comparison data will be 1. Otherwise the class is 0 which means player 2 is the winner.

By doing this, we will have many comparison data and also the class for each of them. When we have a new comparison data of two players we just use our existing comparison data to classify the new data.

## SVM and Naive Bayes Classifier
SVM and Naive Bayes can also be used to classify the comparison data. However, the Naive Bayes is not a appropriate classifier here because the it assumes fields are independent but actually they are not. For example the Points field has relation to the Minutes. Therefore, I mainly focus on SVM classifier in this project but I still do a comparison between these two classifiers.

To do the SVM classification is very easy in Python with the [SKlearn library](http://scikit-learn.org/stable/modules/generated/sklearn.svm.SVC.html) but we still need a training set.

### SVM training set
I generate the training set with the following steps:
1. Go through all games from 2010-11 to 2016-17 seasons and generate player pairs for each game.
2. Reduce the player pairs generate the comparison data for each unique pair.
3. For each comparison data, assign a class to it by compare the win rates between the players in pair.

It is not difficult to generate the training set but there are some restrictions when generate player pairs:
1. Only the player who playes more than 5 minutes average through the career will be taken into account.
2. Only two players in the same position will be taken into account.

These two restrictions can help to reduce the size of training set and can also help to eliminate noise comparison data.

## Predict the game based on comparison data
It is time to predict the game. The basic the idea is to generate many comparison data just like what we do for training set and then use SVM model to classify each of the comparison data. But there is a problem: how do we predict the game based on many comparison data? The simplest way is to take the average of the class and compare the average to a threshold. For example, we have five comparison data and after applying SVM classifier on them we got the classes [1, 1, 1, 0, 0] which meas for first three comparison data we predict the player in team 1 will win the game and for the last 2 comparison data we prediction the player in team 2 will win the game. Then we calculate the average which is 3 / 5 = 0.6. If we assume the threshold is 0.5 then because 0.6 > 0.5 then we predict team 1 will win the game.

In my project, in order to get a better accuracy, I make some modification on this method. Instead of taking the average of the classes, I take the weight average of the classes. The idea is if two players play a lot of time in a game then they should contribute more to the final prediction result. 

To do this, I calculate the total minutes played of two players in a comparison and compare the number to a weight array. For example, if the total minutes is greater than 70 minutes, then I assign 1.5 weight to the comparison data class, if the total minutes is greater than 50 then I assign 1.2 to it... By doing this, the formula to calculate the average of classes becomes to class1*weight1 + class2*weight2...and we still compare the weighted average to some threshold to determine the final prediction result.

## Some results
I collect all games from 2010-11 to 2016-17 seasons and generate the comparison data as the training set and then run an automated test to find the best weight and threshold. The parameter of the testcase can be defined like this:
```js
{
    "seasons": ["2015-16", "2014-15", "2016-17"],
    "sample_size": 100,
    "time_weight": [
        [
            [70, 1.3],
            [50, 0.65],
            [30, 0.4],
            [0, 0.2]
        ],
        [
            [70, 1.1],
            [50, 0.7],
            [30, 0.5],
            [0, 0.15]
        ],
        [
            [70, 1],
            [50, 0.85],
            [30, 0.5],
            [0, 0.15]
        ],
        [
            [0, 1]
        ]
    ],
    "threshold": [0.5 , 1]
}
```
When my application run the testcase based on this parameter, it will sample 100 games from 2015-16, 2014-15 and 2016-17 season respectively and for each game, the application will use the different time weight and the thredhols from 0.5 to 1(with 0.05 step length) to make a prediction. Eventually the application will give us the accuracy for each combination of time weight and threshold. 

### Result for SVM classifier:
The best accuracy for SVM classifier is 0.66 with the paramter { weight: [[0, 1]], threshold: 0.8 } which means just take the average of the classes and compare the average to 0.8.

### Result for Naive Bayes classifier:
The best accuracy for Naive Bayes classifier is 0.56 with the parameter { weight: [[0, 1]], threshold: 0.8 } which is same as the best parameter of SVM classifier. 

### Comparison between SVM and Naive Bayes
No doubt, the prediction based on SVM obtains a better accuracy. This result fits our previous analysis about SVM and Naive Bayes. As a result, SVM is much more suitable for our prediction model.

## Problems
While I was doing project, I found some problem with my prediction model and I cannot solve them within a short time. For these problems I will leave them alone for now and maybe sometime in future I will try to figure out some ways to address them.

The first problem is that when we use SVM classifier, the minutes and points fileds will dominate the model because the value of minutes and points are often from 20 to 40 but the value of blocks, assists or others are usually below 10. That makes other fields meaningless in the training and classification process but in fact they also play a very important role. I was trying to solve this by scale the comparison data, but another problem comes out after scaling, all fields are as important as each other and the accuray of the prediction drops significantly. 

The second problem is becuase the classes of comparison data is uneven and this makes it difficult to find the best threshold mathematically. What I mean by uneven is that I cannot tell how many comparison data will have class 1 and how many will have class 0. So currently the only way to find the threshold is by running the test on each possible threshold and find the best one.

The last problem is about the performance of the application. The data collection process takes too much time so I want to use map-reduce to make the whole process fast.

## Future
I was thinking about to build a prediction website with this model but as I mentioned before, there are still some problems with this prediction model so I can only postpone this idea. But I plan to continue working on this because I love basketball and this is really an interesting topic for me.

All code can be find in this Github page: [NBA Prediction Project](https://github.com/cyfloel0516/NBA_Predictor)

Also, thanks to a great library so that I can easily collect necessary NBA data: [nba_py](https://github.com/seemethere/nba_py)
 

