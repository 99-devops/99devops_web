---
title: "Teach me CI / CD with Kubernetes (PART I)."
date: 2021-05-02T14:20:12+11:00
draft: false
description: "This article is the first article in series of articles on hands on CI / CD in kubernetes. Aim of this is to educate everyone on devops practices starting with CI / CD."
tags: [devops, k8s]
categories: [devops]
---

## PART I

CI / CD has been a major part of current IT operations. In this series of articles, we will learn how to create an application and implement it through complete CI / CD operation. As there is a saying "best way to learn, is by doing.". Hence, we will be implementing our own CI / CD in kubernetes. This is the first part of series of articles on this topic as it is difficult to cover everything in one post. 


### How this article is organised ?

For each challenge there will be are tasks and solutions provided. You can do the task by yourself in your own way or you can follow the solution to see how i have done it. As long as you are understanding the things done and why it's done, we should be all good. 

## Before we begin

Let's refresh the some knowledge before we start with the workshop.

#### What is CI (Continuous Integration) ?

CI (Continuous Integration) is a process of automating code integration from multiple contributers into a single artifact or binary or package. CI is a philosophy where code from multiple developers are merged to form a single package which is later promoted to different stages in release cycle.

In this article we are doing to see an implemet of CI.

#### What is ChatOps ?

ChatOps is the streamlined use of chat applications and communication services to run development and operations functions and commands in CI / CD life cycle.

#### What is Github ?

Version control system which helps you to manage different versions of your code.

#### What is Docker hub ?

Dockerhub is a online docker image hosting registry where you can push your docker image and share it with comminuty.

### Learn by doing

In this first part, we will create an application and dockerise it. This would invole creating an application from scratch and containerise it using docker.

For this task, we will be working on creating a crypto pricing bot and which will check the price of crypto coins every hour and inform you. Later, we will work on running the application in kubernetes using CI / CD.

## Learning outcomes

After completing the series of articles, you will be able to do following things.

1. Creating application to do a simple crypto trade and notify about the trade to the user via chat. 
2. Send message from code to slack channel. This is integral part of ChatOps.
2. Containerise application using docker.
3. Write kubernetes manifests for application to be hosted on kubernetes.
4. Automate all these operations using Github Actions.

## Pre

Before you start working on it, make sure you are inside your application directory.

```
cd ~
mkdir -p crypto-bot
cd crypto-bot
```

### Task 1

Get yourself slack workspace, slack channel, slack bot and slack token. Before we create our application, we need to do extra things some things ready i.e.

1. Slack workspace

2. Slack channel

3. Slack bot 

4. Add OAuth permission to that bot

5. Get oauth API Token.

6. Add slack bot to the particular channel.

7. Github repository

8. Dockerhub account

### Solution 1

You can follow this video until 5 min to get your slack workspace, channel, bot setup and get the API token.

[https://www.youtube.com/watch?v=KJ5bFv-IRFM](https://www.youtube.com/watch?v=KJ5bFv-IRFM)

Once you are done, you should have a slack token which looks like this

```
xoxb-2387234-XXXXXX.....
```

For this demo i have created a slack workspace, and a slack bot named "Lekhpal" (<i>This means accountant in Nepali</i> ;) )

Please note down following things:

1. The channel name

2. Slack bot username

3. Slack token somespace

For github repo and docker hub account, you should be able to create them by yourself. :)
## Creating a crypto bot application

Now, since we have setup out notification section, let's create a trading bot. Our crypto bot is simple. It will run API request to a provider to fetch crypto latest pricing information. I will be using python for this but you can use any language of your choice. 

### Task 2

Write a program which would give the latest crypto information and push that information to slack channel.

### Solution 2

let's first create a folder where we will put all our source code and name it "src" and create a file named "crypto-app.py" inside it.  

```
mkdir -p src
touch crypto-app.py
```

What this application does is: 

1. It uses COINSPOT public API to get latest price of defined cryptos and creates a JSON data.

2. JSON data is then sent to a function which pushes message to slack channel which we configured in Task 1.

You can copy paste following code into the file

```bash

# Importing packages
import os
import requests
import hashlib
import hmac
import json
from pprint import pprint
import requests
from time import time
import logging

# Setting log level
logging.basicConfig(level=logging.DEBUG)


def get_latest_crypto_price(crypto) -> str:
""" Gets latest crypto prices"""
    headers = {
                'Content-Type': 'application/json'
               }
    r = requests.get('https://www.coinspot.com.au/pubapi/latest', headers=headers)
    response = r.json()
    return response['prices'][crypto]['last']


def buy_order(CRYPTO_DICT):
""" If crypto price if over or under certain value, it BUYs or SELLs items."""
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
""" Notify in Slack """
    slack_token = 'xoxb-XXXX' # Slack TOKEN here
    slack_user_name = 'Lekhapal' # Slack bot username
    slack_channel = "#crypto-trading"  # Slack channel name
    text = text
    return requests.post('https://slack.com/api/chat.postMessage', {
        'token': slack_token,
        'channel': slack_channel,
        'text': text,
        'username': slack_user_name,
        'blocks': json.dumps(blocks) if blocks else None
    }).json()


if __name__=="__main__":
    # list of crypto to watch
    CRYPTO_TO_WATCH= ["btc", "eth", "doge", "ada"]
    CRYPTO_DICT={}
    for crypto in CRYPTO_TO_WATCH:
        try:
            CRYPTO_DICT[crypto].append(float(get_latest_crypto_price(crypto)))
        except KeyError:
            CRYPTO_DICT[crypto] = float(get_latest_crypto_price(crypto))
        except AttributeError:
            CRYPTO_DICT[crypto] [ CRYPTO_DICT[crypto], float(get_latest_crypto_price(crypto))]

    # Send prepared json data to slack containing prices
    post_to_slack("Latest  crypto prices \n\n:bitcoin: BTC price : "+str(CRYPTO_DICT["btc"])+"\n\n:eth: ETH price : "+str(CRYPTO_DICT["eth"])+"\n\n:ada: ADA price : "+str(CRYPTO_DICT["ada"])+"\n\n:doge: DOGE price : "+str(CRYPTO_DICT["doge"])+"\n\nThat's all for now. Will check back in an hour. :slightly_smiling_face: ")
    buy_order(CRYPTO_DICT)

```

Now, run you application to see if your application send the messages to slack. In  console you should see something like this.

```bash
python3 src/crypto-app.py     

DEBUG:urllib3.connectionpool:Starting new HTTPS connection (1): www.coinspot.com.au:443
...
..
..
```

In slack, you should now see your bot sending message

![slack_notification](/img/slack_notification.png)

Wooo..hooo.... :)

Congratulations. You have created a bot which checks BTC price and does fake buys of BTC and notifies in slack about your transaction.

## Containerise your application

Now, since we have our application ready, we are ready to containerise it. I will be using docker for it, but you can use any container runtime to create a container.

### Task 3

Write a Dockerfile for your application

### Solution 3

Create a dockerfile into the main working directory i.e crypto-bot directory.

```
touch Dockerfile
```

Copy and paste the following contents of into that Dockerfile. 

What happens here is:

1. It pulls the base python:3.9 image from dockerhub

2. Adds label to the docker image when we create it.

3. Installsrequired packages

4. Copies the src folder into /app directory inside container

5. Installs all the extra packages required to run the applications from requirements.txt file

6. Starts the application

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
docker build -t crypto-app:latest .
```

run docker container to see if it works.

```bash
docker run --rm --name crypto crypto-app:latest 
```

You should see the same message notification in slack.

![slack_notification](/img/slack_notification.png)

## Pushing image to registry

Kubernetes pull image from container registry. We will be using DockerHub as our container registry. You can use any other container registry like Jfrog etc.

Before this, you would need docker hub username and password. You can sign up for docker hub if you do not already have an account.

### Task 4

Create docker hub account and save username and access token somewhere.
### Solution

Sign up for docker hub from here [https://hub.docker.com/signup](https://hub.docker.com/signup)


Once you have the docker hub account, you need to create a repository for your docker image that we are going to push. I have named the repo name "crypto-app" as that is the image name that we are going to use.


![create-repo](/img/create-repo.png)


![docker-hub-create-repo](/img/docker-hub-create-repo.png)


Once you have the repository created it will look like this

![dockerhub-repo](/img/docker-hub-repo.png)

### Task 5 (Add dockerhub creds to Github secrets)

Add docker hub username and access token to github secrets.

### Solution 5

Now, since we are able to push image, we will try to do automatically using Github Actions. This will push build our container and push our image to dockerhub automatically. First of all, you need access token from docker hub to push image. To get access token [https://hub.docker.com/settings/security](https://hub.docker.com/settings/security) go here and create a token named GITHUB_ACTIONS, save that token somewhere. we will need it later.

It will look like this once you created the access token

![dockerhub-access-token](/img/dockerhub-access-token.png)

Now, Create github secret for your dockerhub username and access token which github will use to build and push image to dockerhub.

In order to create secret, go to repo Settings > Secrets.

Create two secrets named <code>DOCKERHUB_USERNAME</code> and <code>DOCKERHUB_TOKEN</code> once you add it, it will look like this

![github-secrets](/img/github-secrets.png)

### Task 6 

Setup CI workflows in github actions to push image to your dockerhub repository.

### Solution 6 

Create .github folder and create another folder inside .github called workflows and again create a file named 'CI.yml' inside workflows folder.

```
mkdir -p .github/workflows
touch .github/workflows/CI.yml
```

Inside that CI.yml file add following code which will automatically build and push your image to dockerhub. 


#### **NOTE: I have have used the tag name as ${{ secrets.DOCKERHUB_USERNAME }}/crypto-app:latest where crypto-app is the name of our dockerhub repository.**



```bash
# This is a basic workflow to help you get started with Actions

name: CI

# Controls when the action will run. 
on:
  # Triggers the workflow on push or pull request events but only for the main branch
  push:
    branches: [ main ]
  pull_request:
    branches: [ main ]

  # Allows you to run this workflow manually from the Actions tab
  workflow_dispatch:

# A workflow run is made up of one or more jobs that can run sequentially or in parallel
jobs:
  # This workflow contains a single job called "build"
  build:
    # The type of runner that the job will run on
    runs-on: ubuntu-latest

    # Steps represent a sequence of tasks that will be executed as part of the job
    steps:
      # Checks-out your repository under $GITHUB_WORKSPACE, so your job can access it
      - uses: actions/checkout@v2

      - name: Set up QEMU
        uses: docker/setup-qemu-action@v1
      
      - name: Set up Docker Buildx
        uses: docker/setup-buildx-action@v1

      - name: Login to DockerHub
        uses: docker/login-action@v1 
        with:
          username: ${{ secrets.DOCKERHUB_USERNAME }}
          password: ${{ secrets.DOCKERHUB_TOKEN }}

      - name: Build and push
        id: docker_build
        uses: docker/build-push-action@v2
        with:
          push: true
          tags: ${{ secrets.DOCKERHUB_USERNAME }}/crypto-app:latest
```

Once you add these, you are ready to commit to github. Your main directory folder structure should look like this

![directory-structure](/img/directory-structure.png)

Now commit your change

```
git add .
git commit -m "Adding Continuous integration"
git push origin master
```

When you commit the change into github, you can go to actions in tabs in your repository, you should see the CI build running and hopefully passing ;). This builds docker image from the Dockerfile that we created and tags the image appropriately so that it is pushed in the docker hub repo that we created.

![github-ci-pass](/img/github-ci-pass.png)


and pushing image to dockerhub which you can verify by going to dockerhub.

![dockerhub-image-push](/img/dockerhub-image-push.png)


## What did we learned ?

1. Use cURL to send post requests
2. Parse Json response using python
3. Send message from application to slack channel
4. Containerise python application into a docker image
5. Run docker image
6. Push image to docker hub using github action
7. Hands-on continuous integration
## Conclusion

This concludes the first part of our workshop. We did a lot of things here and learned a lot of stuffs. We have succesfully completed continuous integration part of our CI / CD workshop. In next article, i will be showing you how you can do the continuous delivery and deployment (CD) side of things on kubernetes. 

Please share it with your friends if you liked what you learned today. We aim of this is to educate everyone on devops practices starting with CI / CD.

### Congratulations on the hard work, here are some cookie for you.... :)

![cookie](/img/cookie.png)


## **While this project that we created is a base line for your future iterations and optimisation, please do not use this in production or use it to run your crypto transactions. We will be implementing DevSecOps practices later in upcoming articles.**
