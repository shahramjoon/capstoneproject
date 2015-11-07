---
title       : Mexican Restaurant Reviews in Yelp 
subtitle    : Do words used in Reviews matter in Review's Ratings ?
author      : Data Scientist
job         : Coursera Capstone Project 
framework   : io2012        # {io2012, html5slides, shower, dzslides, ...}
highlighter : highlight.js  # {highlight.js, prettify, highlight}
hitheme     : tomorrow      # 
widgets     : []            # {mathjax, quiz, bootstrap}
mode        : selfcontained # {standalone, draft}
knit        : slidify::knit2slides
output      : ioslides_presentation




--- &twocol

### _Mexican Restaurant businesses in Yelp Data set_


There are 95692 reviews for 2208 Mexican Restaurants in Data Set.

Majority of businesses are rated 3.5 and 4.

Restaruants in Arizona and Nevada had the most reviews in Data Set.



*** =left



<div style='text-align: center;'>
    <img height='450' src='assets/fig/img1-1.png' />
</div>

*** =right



<div style='text-align: center;'>
    <img height='450' src='assets/fig/img2-1.png' />
</div>

--- &twocol

###  Mexican Restaurants Reviews

There are 54138 distinct Yelp Users that wrote 95692 reivews for Mexican Restaturants. 

*** =left

Reviewers that gave star rating of 4 for their reviews on average, wrote 80 reviews on average, 33 reviews for 5 star reviewers versus reviewers that gave ratings of 0,1,2 and 3 wrote 23 reviews on average combined. 

Are there words used in 4 and 5 ratings that are not used in 0,1,2, 3 ratings? 


*** =right



<div style='text-align: center;'>
    <img height='450' src='assets/fig/img3-1.png' />
</div>


--- &twocol

###  WordCloud of Mexican Restaurants Reviews

*** =left

50 most Frequent words used in mexican restaurants rated 4 or 5

Words used frequent: taco, great, place, love, like


<div style='text-align: center;'>
    <img height='560' src='assets/fig/corpus4-1.png' />
</div>

*** =right


50 most frequent words used in mexican restaurants rated 1 or 2 or 3

Words used most: Order, Time, good, like 



<div style='text-align: center;'>
    <img height='560' src='assets/fig/corpus5-1.png' />
</div>


--- 

###  Building a model 

We build a model to  show the frequency of words used across good reviews ( 4/5 Ratings) and bad reviews(0/1/2/3 Ratings)

Created a Corpus of all words in all reviews

```r
data <- filtered
corpus <- Corpus (VectorSource(data$text))
corpus <- tm_map(corpus, stripWhitespace)
corpus <- tm_map(corpus, removePunctuation)
corpus <- tm_map(corpus, content_transformer(tolower)) 
corpus <- tm_map(corpus, removeWords, stopwords("english"))
corpus <- tm_map(corpus, stemDocument)
```
Created a Matrix of all words used at least in %3 of all reivews 

```r
dtm = DocumentTermMatrix(corpus)
sparse = removeSparseTerms(dtm, 0.97) 
words = as.data.frame(as.matrix(sparse))
dim(words)
```

```
## [1] 95692   332
```
Seperated our data set to two sets: Training and testing

```r
words$review_stars_bin <- data$review_stars_bin
levels(words$review_stars_bin) <- c ("bad", "bad", "bad", "bad", "good", "good")
set.seed ( 32343)
inTrain <- createDataPartition ( y=words$review_stars_bin, p=.75, list =FALSE)
training <- words[inTrain,]
testing <- words[-inTrain,]
```

---

### Model

We run our model. It had 82.3% accuracy


```r
modelFit <- train(review_stars_bin~., data=training, method="glm")
predictions <- predict(modelFit, newdata=testing)
library(gmodels)
CrossTable(predictions, testing$review_stars_bin, 
           dnn=c("pridicted" , "Actual") , 
           prop.t=FALSE, prop.c=FALSE, prop.r=FALSE , prop.chisq = FALSE)
```

```
## 
##  
##    Cell Contents
## |-------------------------|
## |                       N |
## |-------------------------|
## 
##  
## Total Observations in Table:  23922 
## 
##  
##              | Actual 
##    pridicted |       bad |      good | Row Total | 
## -------------|-----------|-----------|-----------|
##          bad |      6183 |      1537 |      7720 | 
## -------------|-----------|-----------|-----------|
##         good |      2682 |     13520 |     16202 | 
## -------------|-----------|-----------|-----------|
## Column Total |      8865 |     15057 |     23922 | 
## -------------|-----------|-----------|-----------|
## 
## 
```


--- 

### _Result_


Our model identified  some words like _love, favorite, flavor , well, margaritta_ appears in good rating reviews. The word 'good' was used _two times_ more in _good_ reviews compared to _bad_ reviews.


```
## png 
##   2
```



<div style='text-align: center;'>
    <img  src='assets/fig/result-1.png' />
</div>

