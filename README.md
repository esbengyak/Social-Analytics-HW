

```python
import numpy as np
import pandas as pd
import matplotlib.pyplot as plt
import json
import tweepy
import time
import seaborn as sns

from vaderSentiment.vaderSentiment import SentimentIntensityAnalyzer
analyzer = SentimentIntensityAnalyzer()

consumer_key = "Ed4RNulN1lp7AbOooHa9STCoU"
consumer_secret = "P7cUJlmJZq0VaCY0Jg7COliwQqzK0qYEyUF9Y0idx4ujb3ZlW5"
access_token = "839621358724198402-dzdOsx2WWHrSuBwyNUiqSEnTivHozAZ"
access_token_secret = "dCZ80uNRbFDjxdU2EckmNiSckdoATach6Q8zb7YYYE5ER"
```


```python
def TweetAnalyzer():
    auth = tweepy.OAuthHandler(consumer_key, consumer_secret)
    auth.set_access_token(access_token, access_token_secret)
    api = tweepy.API(auth, parser=tweepy.parsers.JSONParser())

    mentions = api.search(q="@it_plot Analyze:")
    
    words = []

    target_account = ""

    command = mentions["statuses"][0]["text"]
    requesting_user = mentions["statuses"][0]["user"]["screen_name"]

    try:
        words = command.split("Analyze:")
        target_account = words[1].strip()

        print("Target Account: " + target_account)
        print("Requesting User: " + requesting_user)

    except Exception:
        raise

    tweets = api.user_timeline()

    repeat = False

    for tweet in tweets:
        if target_account in tweet["text"]:
            repeat = True
            print("Nah I'm good")

        else:
            continue

    if not (repeat):

        tweet_data = {
            "tweet_requester": [],
            "tweet_text": [],
            "tweet_date": [],
            "tweet_vader_score": [],
            "tweet_negative_score": [],
            "tweet_positive_score": [],
            "tweet_neutral_score": []
        }

        for x in range(25):

            tweets = api.user_timeline(target_account, page=x)

            for tweet in tweets:

                tweet_data["tweet_requester"].append(tweet["user"]["name"])
                tweet_data["tweet_text"].append(tweet["text"])
                tweet_data["tweet_date"].append(tweet["created_at"])

                tweet_data["tweet_vader_score"].append(
                    analyzer.polarity_scores(tweet["text"])["compound"])
                tweet_data["tweet_positive_score"].append(analyzer.polarity_scores(tweet["text"])["pos"])
                tweet_data["tweet_negative_score"].append(analyzer.polarity_scores(tweet["text"])["neg"])
                tweet_data["tweet_neutral_score"].append(analyzer.polarity_scores(tweet["text"])["neu"])
                

    if not (repeat):

        tweet_data = {
            "tweet_source": [],
            "tweet_text": [],
            "tweet_date": [],
            "tweet_vader_score": [],
            "tweet_negative_score": [],
            "tweet_positive_score": [],
            "tweet_neutral_score": []
        }

        for x in range(25):

            tweets = api.user_timeline(target_account, page=x)

            for tweet in tweets:

                tweet_data["tweet_source"].append(tweet["user"]["name"])
                tweet_data["tweet_text"].append(tweet["text"])
                tweet_data["tweet_date"].append(tweet["created_at"])

                tweet_data["tweet_vader_score"].append(analyzer.polarity_scores(tweet["text"])["compound"])
                tweet_data["tweet_negative_score"].append(analyzer.polarity_scores(tweet["text"])["neg"])                
                tweet_data["tweet_positive_score"].append(analyzer.polarity_scores(tweet["text"])["pos"])
                tweet_data["tweet_neutral_score"].append(analyzer.polarity_scores(tweet["text"])["neu"])

        tweet_df = pd.DataFrame(tweet_data, columns=["tweet_source",
                                                     "tweet_text",
                                                     "tweet_date",
                                                     "tweet_vader_score",
                                                     "tweet_negative_score",
                                                     "tweet_positive_score",
                                                     "tweet_neutral_score"])

        tweet_df.head()

    if not (repeat):

        tweet_df["tweet_date"] = pd.to_datetime(tweet_df["tweet_date"])

        tweet_df.sort_values("tweet_date", inplace=True)
        tweet_df.reset_index(drop=True, inplace=True)

        tweet_df.head()

    if not (repeat):

        plt.clf()

        plt.plot(np.arange(-len(tweet_df["tweet_vader_score"]), 0, 1),
                 tweet_df["tweet_vader_score"], marker="o", linewidth=0.5,
                 alpha=0.8, label="%s" % target_account)

        plt.title("Sentiment Analysis of Tweets (%s)" % time.strftime("%x"))
        plt.ylabel("Tweet Polarity")
        plt.xlabel("Tweets Ago")
        plt.xlim([-len(tweet_df["tweet_vader_score"]) - 7, 7])
        plt.ylim([-1.05, 1.05])
        plt.grid(True)

        lgnd = plt.legend(fontsize="medium", mode="Expanded",
                          numpoints=1, scatterpoints=1,
                          loc="best", bbox_to_anchor=(1, 1),
                          title="Tweets", labelspacing=0.5)

        file_path = "analysis/" + target_account + ".png"
        plt.savefig(file_path, bbox_extra_artists=(lgnd, ),
                    bbox_inches='tight')

    if not (repeat):

        api.update_with_media(file_path,
                              "New Tweet Analysis: %s (You're Welcome @%s!!)" %
                              (target_account, requesting_user))

while(True):
    AnalyzeTweets()
    time.sleep(300)

```


    ---------------------------------------------------------------------------

    IndexError                                Traceback (most recent call last)

    <ipython-input-44-768fd133d65f> in <module>()
        141 
        142 while(True):
    --> 143     AnalyzeTweets()
        144     time.sleep(300)
    

    <ipython-input-40-c1030482cb66> in AnalyzeTweets()
         18 
         19     # Grab the most recent command tweet
    ---> 20     command = mentions["statuses"][0]["text"]
         21     requesting_user = mentions["statuses"][0]["user"]["screen_name"]
         22 
    

    IndexError: list index out of range

