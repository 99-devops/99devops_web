---
title: "Hands-on CI / CD on Kubernetes (PART I)."
date: 2021-05-02T14:20:12+11:00
draft: false
description: "This article is the first article in series of articles on CI / CD in kubernetes."
tags: [devops, k8s]
categories: [devops]
---

# PART I

CI / CD has been a major part of current IT operations. In this series of we will learn how to create an application and implement it through complete CI / CD operation.

We will learn how to do CI / CD of a containerised service. For this, we will learn by doing. 

# Before we begin

### What is CI (Continuous Integration) ?

### What is CD (Continuous Deployment) ?



# Learn by doing

In this first part, we will create an application and dockerise it. This would invole creating an application from scratch and containerise it using docker.

For this task, we will be working on creating a crypto pricing bot and which will check the price of crypto coins every hour and inform you. Later, we will work on running the application in kubernetes using CI / CD.

# Learning outcomes

After completing the series of articles, you will be able to do following things.

1. Creating application to do a simple crypto trade and notify about the trade to the user via chat. 
2. Send message from code to slack channel. This is integral part of ChatOps.
2. Containerise application using docker.
3. Write kubernetes manifests for application to be hosted on kubernetes.
4. Automate all these operations using Github Actions.

# Pre

Before you start working on it, make sure you are inside your application directory.

```
cd ~
mkdir -p crypto-bot
cd crypto-bot
```

### Task 1 (Get yourself slack workspace, slack channel, slack bot and slack token.)

Before we create our application, we need to do extra things some things ready i.e.

1. Slack workspace

2. Slack channel

3. Slack bot 

4. Add OAuth permission to that bot

5. Get oauth API Token.

6. Add slack bot to the particular channel.

7. Github repository

8. Dockerhub account

You can follow this video until 5 min to get your slack workspace, channel, bot setup and get the API token.

[https://www.youtube.com/watch?v=KJ5bFv-IRFM](https://www.youtube.com/watch?v=KJ5bFv-IRFM)

Once you are done, you should have a slack token which looks like this

```
xoxb-2387234-XXXXXX.....
```

For this demo i have created a slack workspace, and a slack bot named "Lekhpal" (<i>This means accountant in Nepali</i> ;) )

Note down following things:

1. The channel name

2. Slack bot username

3. Slack token somespace
# Creating a crypto bot application

Now, since we have setup out notification section, let's create a trading bot. Our crypto bot is simple. It will run API request to a provider to fetch crypto latest pricing information. I will be using python for this but you can use any language of your choice. 

### Task 2

Write a program which would give the latest crypto information and push that information to slack channel.

### Solution 2

let's first create a file and  name it "crypto-app.py"

```
touch crypto-app.py
```

Copy paste following code into the file

```bash

import os
import requests
import hashlib
import hmac
import json
from pprint import pprint
import requests
from time import time
import logging

logging.basicConfig(level=logging.DEBUG)


def get_latest_crypto_price(crypto) -> str:
    headers = {
                'Content-Type': 'application/json'
               }
    r = requests.get('https://www.coinspot.com.au/pubapi/latest', headers=headers)
    response = r.json()
    return response['prices'][crypto]['last']


def buy_order(CRYPTO_DICT):
    if CRYPTO_DICT["btc"] <= 59000:
        post_to_slack("\n\n\n :bitcoin: BTC price has dropped to defined limit cap (1.68). Buying btc Share worth AUD 10. ")
    elif CRYPTO_DICT["btc"] >= 90000:
        post_to_slack("\n\n\n :bitcoin: BTC price has gone up to defined limit cap (1.9). Selling btc Share worth AUD 20. ")

    if CRYPTO_DICT["eth"] <= 3400:
        post_to_slack("\n\n\n :eth: ETH price has dropped to defined limit cap (1.68). Buying ETH Share worth AUD 10. ")
    elif CRYPTO_DICT["eth"] >= 4000:
        post_to_slack("\n\n\n :ada: ETH price has gone up to defined limit cap (1.9). Selling ETH Share worth AUD 20. ")

    if CRYPTO_DICT["doge"] <= 0.39:
        post_to_slack("\n\n\n :doge: DOGE price has dropped to defined limit cap (1.68). Buying DOGE Share worth AUD 10. ")
    elif CRYPTO_DICT["doge"] >= 0.8:
        post_to_slack("\n\n\n :doge: DOGE price has gone up to defined limit cap (1.9). Selling DOGE Share worth AUD 20. ")

    if CRYPTO_DICT["ada"] <= 1.68:
        post_to_slack("\n\n\n :ada: ADA price has dropped to defined limit cap (1.68). Buying ADA Share worth AUD 10. ")
    elif CRYPTO_DICT["ada"] >= 2.5:
        post_to_slack("\n\n\n :ada: ADA price has gone up to defined limit cap (1.9). Selling ADA Share worth AUD 20. ")
    
def post_to_slack(text, blocks = None):
    slack_token = os.environ['SLACK_TOKEN']
    slack_user_name = 'Lekhapal'
    slack_channel = "#crypto-trading"
    text = text
    return requests.post('https://slack.com/api/chat.postMessage', {
        'token': slack_token,
        'channel': slack_channel,
        'text': text,
        'icon_url': slack_icon_url,
        'username': slack_user_name,
        'blocks': json.dumps(blocks) if blocks else None
    }).json()


if __name__=="__main__":

    CRYPTO_TO_WATCH= ["btc", "eth", "doge", "ada"]
    CRYPTO_DICT={}
    for crypto in CRYPTO_TO_WATCH:
        try:
            CRYPTO_DICT[crypto].append(float(get_latest_crypto_price(crypto)))
        except KeyError:
            CRYPTO_DICT[crypto] = float(get_latest_crypto_price(crypto))
        except AttributeError:
            CRYPTO_DICT[crypto] [ CRYPTO_DICT[crypto], float(get_latest_crypto_price(crypto))]

    post_to_slack("Latest  crypto prices \n\n:bitcoin: BTC price : "+str(CRYPTO_DICT["btc"])+"\n\n:eth: ETH price : "+str(CRYPTO_DICT["eth"])+"\n\n:ada: ADA price : "+str(CRYPTO_DICT["ada"])+"\n\n:doge: DOGE price : "+str(CRYPTO_DICT["doge"])+"\n\nThat's all for now. Will check back in an hour. :slightly_smiling_face: ")
    buy_order(CRYPTO_DICT)

```

Now, run you application to see if your application send the messages to slack. In  console you should see something like this.

```bash
python3 src/crypto-app.py     

DEBUG:urllib3.connectionpool:Starting new HTTPS connection (1): www.coinspot.com.au:443
DEBUG:urllib3.connectionpool:https://www.coinspot.com.au:443 "GET /pubapi/latest HTTP/1.1" 200 None
DEBUG:urllib3.connectionpool:Starting new HTTPS connection (1): www.coinspot.com.au:443
DEBUG:urllib3.connectionpool:https://www.coinspot.com.au:443 "GET /pubapi/latest HTTP/1.1" 200 None
DEBUG:urllib3.connectionpool:Starting new HTTPS connection (1): www.coinspot.com.au:443
DEBUG:urllib3.connectionpool:https://www.coinspot.com.au:443 "GET /pubapi/latest HTTP/1.1" 200 None
DEBUG:urllib3.connectionpool:Starting new HTTPS connection (1): www.coinspot.com.au:443
DEBUG:urllib3.connectionpool:https://www.coinspot.com.au:443 "GET /pubapi/latest HTTP/1.1" 200 None
DEBUG:urllib3.connectionpool:Starting new HTTPS connection (1): slack.com:443
DEBUG:urllib3.connectionpool:https://slack.com:443 "POST /api/chat.postMessage HTTP/1.1" 200 454
```

In slack, you should now see your bot sending message

![notification(/crypto-bot/asset.sac.png)

Wooo..hooo....

Congratulations. You have created a bot which checks BTC price and does fake buys of BTC and notifies in slack about your transaction.

# Containerise your application

Now, since we have our application ready, we are ready to containerise it. I will be using docker for it, but you can use any container runtime to create a container.

### Task 3

Write a Dockerfile for your application

### Solution 3

Here is the Dockerfile of my application. 

```bash
# Base Image
FROM python:3.9

# Label
LABEL MAINTAINER="Prabesh.Thapa@audinate.com"

# Installing package
RUN apt-get update && apt-get install -y python-dev libldap2-dev libsasl2-dev libssl-dev --no-install-recommends && apt-get clean && rm -rf /var/lib/apt/lists/*


# Moving app to /app directory in container
COPY src /app
WORKDIR /app

# Application specific package
RUN pip install -r requirements.txt


ENTRYPOINT ["python","./crypto-app.py"]
```

Let's try to build the image and run the container.

Build docker image

```bash
docker build -t crypto:latest .
```

run docker container to see if it works.

```bash
docker run --rm --name crypto crypto:latest 
```

You should see the same message notification in slack.

![notification(/crypto-bot/asset.sac.png)

# Pushing image to registry

Kubernetes pull image from container registry. We will be using DockerHub as our container registry. You can use any other container registry like Jfrog etc.

Before this, you would need docker hub username and password. You can sign up for docker hub if you do not already have an account.

### Task 4 (Create dockerhub and push image to docker hub automatically using Github Actions)

Create docker hub account and save username and password somewhere and push image to dockerhub.
### Solution

Sign up for docker hub from here [https://hub.docker.com/signup](https://hub.docker.com/signup)


Once you have the docker hub account, you need to create a repository for your docker image that we are going to push.

Image


Once you have the repository created it will look like this

image

You can push image using cli using following command
```
docker login
docker tag <LOCAL_IMAGE_NAME_CREATED>:latest <DOCKERHUB_USERNAME>/<REPO_NAME>:latest
docker push <DOCKERHUB_USERNAME>/<REPO_NAME>:latest
```
Check if the image is there in dockerhub or not.

### Task 5 (Add dockerhub creds to Github secrets)

Add docker hub username and access token to github secrets.

### Solution 5

Now, since we are able to push image, we will try to do automatically using Github Actions. This will push build our container and push our image to dockerhub automatically. First of all, create github secret for your dockerhub username and access token which github will use to build and push image to dockerhub.

TO get access token [https://hub.docker.com/settings/security](https://hub.docker.com/settings/security) go here and create a token named GITHUB_ACTIONS, save that token somewhere. we will need it later.

It will look like this once you created the access token

Image

In order to create secret, go to repo Settings > Secrets.

Create two secrets named <code>DOCKERHUB_USERNAME</code> and <code>DOCKERHUB_TOKEN</code> once you add it, it will look like this

image



Create .github folder and create another folder inside .github called workflows and again create a file named 'CI.yml' inside workflows folder.

```
mkdir -p .github/workflows
touch .github/workflows/CI.yml
```

Inside that CI.yml file add following code which will automatically build and push your image to dockerhub.