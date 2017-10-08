---
layout: post
title: Quick Guide to Retrieve Data from Twitter
idescription: A post introducing a way to easily retrieve data from Twitter
tags: api twitter 
---


Due to my work, I started using and creating APIs, thus in my spare time, I had some fun with exploring, analysing and visualising collected data from Twitter which has an awesome REST API available for developers. In this post, I will introduce how to use Python to easily retrieve data from Twitter.

In order to access to Twitter’s API, you need to create an APP to interact with Twitter’s API. If you don’t have it, go to https://apps.twitter.com , click the button (create new app) on the up right corner from which you will need fill in some details about this APP, so you have the APP now! 

![app_creation](https://github.com/charlesdong1991/charlesdong1991.github.io/blob/master/images/app_creation.png)

Once it’s done, you can see your consumer key and consumer secret which should be always kept private. One more thing to be aware is, after creating the new APP, you need to acquire the access token and corresponding token secret, the function is to provide application access to Twitter’s API on your created account’s behalf. The way I usually do to keep such thing is using an independent cfg file calling it: “ authorization.cfg ”, like so:

```yaml
[Keys]
consumer_key: YOUR_CONSUMER_KEY
consumer_secret: YOUR_CONSUMER_SECRET
access_token: YOUR_ACCESS_TOKEN
access_token_secret: YOUR_ACCESS_TOKEN_SECRET
```

And I can get these private information in my working Jupyter Notebook script without letting people see them. And you store all of these information in a dictionary that can be used later on.

{% highlight python %}
import configparser
config = configparser.RawConfigParser()
config.read('authorization.cfg')
keys = {
      "consumer_key": config.get('Keys', 'consumer_key'),
      "consumer_secret": config.get('Keys', 'consumer_secret'),
      "access_token": config.get('Keys', 'access_token'),
      "access_token_secret": config.get('Keys', 'access_token_secret')
  }
{% endhighlight %}

Well, you are on the half way getting your first bunch of data from Twitter! Now, time to use some cool Python library to make our life easier. The library I use is tweepy, you can find the docs here: http://docs.tweepy.org/en/v3.5.0/getting_started.html, you need to import OAuthHandler to handle the access issue.

{% highlight python %}
import tweepy
from tweepy import OAuthHandler
auth = OAuthHandler(keys['consumer_key'],keys['consumer_secret'])
auth.set_access_token(keys['access_token'],keys['access_token_secret'])

api = tweepy.API(auth)
{% endhighlight %}
We can take a look at the data we collect from Twitter now by using Cursor and search methods, let’s see a simple example to get a feel of how it works:

{% highlight python %}
for tweet in tweepy.Cursor(api.search,q='#schiphol OR #Amsterdam Airport',lang='en').items():
    print(tweet.text)
{% endhighlight %}
By this request, you can get tweets which use either schiphol or Amsterdam Airport as hashtag in their tweets and the language of the tweets is specified to English. Of course, there are much more things you can do with tweepy, in the future posts, I will show some interesting result got from Twitter! 
