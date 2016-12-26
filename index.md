---
title       : Next Word Prediction
subtitle    : Implementation of the Modified Kneser Ney Algorithm
author      : Gareth Cork
job         : Data Science Student
framework   : io2012        # {io2012, html5slides, shower, dzslides, ...}
highlighter : highlight.js  # {highlight.js, prettify, highlight}
hitheme     : tomorrow      # 
widgets     : [mathjax]            # {mathjax, quiz, bootstrap}
mode        : selfcontained # {standalone, draft}
knit        : slidify::knit2slides
url:
  assets: ./assets

---



## Context


Many of us use next word prediction each day - most commonly on our mobile phones. These tools can help improve speed of typing and improve spelling accuracy. 



<div style="width: 35%; float: left; padding-right:5%">

<p>The aim of this project is to produce an application that predicts the next word, given the context of the words keyed by the user. The project utilises best practice algorithms to maximise predictive performance.</p>

<p>The application is hosted within the Shiny framework and can be found <a href="http://www.google.com">here</a>.</p>

<p><strong>How does the app work?</strong> The user enters text into the input area, the most probably next words are then shown to the user in descending order.</p>

</div>

<div style="width: 55%; float: left; padding-right:5%">
<p><strong>Image 1: Application</strong></p><p>
<img src="assets/screen.jpg", style="width:100%; height:auto" /></p>
</div>

---

## Data and Processing


|Data    |Size   |Documents |Words |Vocabulary |% Valid English |% Dictionary coverage |
|:-------|:------|:---------|:-----|:----------|:---------------|:---------------------|
|Twitter |159 MB |2.4m      |29.5m |397k       |82.6%           |74.3%                 |
|News    |196 MB |1.0m      |33.5m |337k       |86.8%           |79.5%                 |
|Blogs   |200 MB |0.9m      |36.9m |387k       |88.9%           |88.5%                 |



The data consists of 1.0m news articles, 0.9m blog articles and 2.4m, tweets and covers the majority of common words in the English Language.

- URLs, email addresses, punctuation have not been included as they do not contribute to model quality.
- Stop words have not been removed as these add value to the prediction model
- Explicit words have been removed to prevent explicit predictions
- All words have been converted to lower case for consistency and to allow a smaller corpus for the same predictive ability
- Unknown words will not be an output of the prediction model however these will be included for performance evaluation
- 9% of the data is used to build the model (a term frequency matrix) and 1% is used for testing

---

## Algorithm


In the task of wanting to predict the next word given some history, we can formulate this problem as the probability of a word $w$ given its history $h$ as $P(w|h)$. The probability is calculated as the relative frequency of the word or permutation of words. $P(w|h) = \frac{P(w, h)}{P(h)}$.

The predictive model is based on an n-gram model. N-grams are one of the most common tools for building language models An n-gram is a sequence of $n$ words for example "Thank you" is a 2-gram or bi-gram, "How are you" is a 3-gram or a tri-gram, and so on, these n-grams are extracted from the training data set.

A smoothing algorithm  is used, so that permutations of words not observed in training are still given  probability mass:

<strong>Modified Kneser Ney (Chen and Goodman)</strong>


$$
p_{KN}(w_i|w_{i-n+1}^{i-1}) = \frac{max(c_{KN}(w_{i-n+1}^{i}) - D(w_{i-n+1}^{i-1}), 0)}{c_{KN}(w_{i-n+1}^{i-1})}+\lambda(w_{i-n+1}^{i-1})p_{KN}(w_i|w_{i-n+2}^{i-1})
$$

All count values are stored in seperate `data.table`'s, resulting in values being pulled in real time. Discount values are pre-calculated for each n-gram table and cached. Pre-calculation of variables results in a lengthy build process but fast response times for the user.

---

## Performance

$$
\text{Perplexity}(w_1^n) = \exp \left( -\frac{1}{n} \sum_{i=1}^n \ln  P \left( w_i \middle| w^{i-1}_{i-n+1} \right) \right)
$$

Perplexity was used as the main metric for evaluation, it represents the average branching factor - the average number of words that can follow a given word. Perplexity is calculated on a held out test set. 

Performance of Maximum Likelihood Estimation (MLE) without any smoothing results in a very good perplexity (exc. OOV) score, however the number of Out of Vocabulary (OOV) words is extremely high indicating that smoothing is required. With the Modified Kneser Ney Interpolated model, the OOV percentage is reduced to that of the 1-gram model and the perplexity including OOV words is much smaller, indicating superior performance.



|Model                       | Perplexity - Exc. OOV|OOV | Perplexity - Inc. OOV|
|:---------------------------|---------------------:|:---|---------------------:|
|MLE - 3gram                 |                    39|52% |                 24322|
|MLE - 2gram                 |                   140|19% |                  1112|
|MLE - 1gram                 |                  1228|1%  |                  1371|
|Modified Kneser Ney - 4gram |                   347|1%  |                   387|

