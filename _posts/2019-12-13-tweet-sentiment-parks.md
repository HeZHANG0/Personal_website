---
title: "Parks & open spaces and user sentiment: A case study of San Francisco using Twitter data"
date: 2019-12-13
published: true
tags: [dataviz, geopandas, hvplot, holoviews, tweet, sentiment analysis, park, San Francisco]
excerpt: "Parks & open spaces and user sentiment: A case study of San Francisco using Twitter data"
folium-loader:
  folium-chart-1: ["charts/local_tweet_heat_map.html", "400"]

custom-css-list:
  - "assets/css/leaflet.timedimension.control.min.css"
toc: true
toc_sticky: true
---

Parks and open spaces in cities are believed to have benefits on well-being of city residents. The psychological benefits of urban green infrastructure have been acknowledged by various studies. Thanks to social media, there is a source of disaggregated data to examine individual experience with regard to parks and open spaces (referred to as “parks” below): geo-tagged tweets. This exploratory project uses real-time streaming twitter data to correlate quantified sentiment inside/outside parks in San Francisco. It tries to answer the question: are people more positive when they are in parks compared to when they are not? Specifically, are people expressing more positively through Twitter when they are visiting parks? The result of this preliminary study answers “Yes” to this question. Further study using more extensive data and theoretical review of the use of tweets as indicator of spatiotemporal public expressed sentiment are required to dive deeper into this question.

As a trailer of this analysis, below the GIF shows in-park tweet sentiment by time of day on a weekend in San Francisco:

![tweet_sentiment]({{ site.url }}{{ site.baseurl }}/charts/tweet_sentiment.gif)

## Data Collection
### Tweets
I collected geo-located tweets by real-time streaming, and added a filtering condition of a coordinate bounding box as a proxy to San Francisco boundary. I streamed tweets from 9am to 6pm Pacific Time on Dec. 14th (Sat.) and Dec. 15th (Sun.) and collected in total 30886 tweets with geographical location. For every tweet, I extracted information of user id, user location, text (extended tweet if available), latitude, longitude, place type, posting time. 

![bounding_box]({{ site.url }}{{ site.baseurl }}/charts/bounding_box.png)

To better obtain an understanding of the relation between parks and local resident well-being, I focused on local residents instead of tourists who could tend to express more positive sentiment on their vacation. After filtering by a tighter bound for the city, a strict 9am-6pm time window, and local user location (San Francisco or CA), I had 6833 tweets for analysis. Then I cleaned up the texts by removing URLs, @users, and hashtags. Finally, I tokenized the tweets and removed punctuations and stop words for later analysis.

### Parks
I obtained the park geometry, park score, and city boundary data from the open data portal DataSF. I aggregated the park type to 4 categories: neighborhood park, regional park, civic or library, mini park. 

![sf_parks]({{ site.url }}{{ site.baseurl }}/charts/sf_parks.png)

## Exploratory analysis

### Classify in-park and outside-park tweets
Where do people tweet:
<div id="folium-chart-1"></div>
I classified “in-park” tweets by spatially joining park information to the tweets . In this way, 2093 tweets were classified as “in-park”.

### Twitter text EDA



## Sentiment analysis

## Summary and discussions




## Reference
Hu, Minqing, and Bing Liu. "Mining and summarizing customer reviews." Proceedings of the tenth ACM SIGKDD international conference on Knowledge discovery and data mining. ACM, 2004.
Schwartz, Aaron J., et al. "Visitors to urban greenspace have higher sentiment and lower negativity on Twitter." People and Nature (2019).
Plunz, Richard A., et al. "Twitter sentiment in New York City parks as measure of well-being." Landscape and Urban Planning 189 (2019): 235-246.
Lim, Kwan Hui, et al. "The grass is greener on the other side: Understanding the effects of green spaces on Twitter user sentiments." Companion Proceedings of the The Web Conference 2018. International World Wide Web Conferences Steering Committee, 2018.

