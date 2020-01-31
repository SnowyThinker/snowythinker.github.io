
~~~python
import os
import time
from threading import Thread

from properties.p import Property
from tweetfeels import TweetFeels


def principle():
    prop = Property()
    prop_loader = prop.load_property_files(os.path.expanduser('~/config/env.properties'))

    consumer_key = prop_loader.get('oauth.consumerKey')
    consumer_secret = prop_loader.get('oauth.consumerSecret')
    access_token = prop_loader.get('oauth.accessToken')
    access_token_secret = prop_loader.get('oauth.accessTokenSecret')

    login = [consumer_key, consumer_secret, access_token, access_token_secret]
    return login


def print_feels(seconds=10):
     while go_on:
         time.sleep(seconds)
         print(f'[{time.ctime()}] Sentiment Score: {trump_feels.sentiment.value}')


if __name__ == '__main__':
    go_on = True
    t = Thread(target=print_feels)

    trump_feels = TweetFeels(principle(), tracking=['trump'])
    trump_feels.start()

    t.start()

~~~