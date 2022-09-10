[![Gitter](https://img.shields.io/badge/Available%20on-Intersystems%20Open%20Exchange-00b2a9.svg)](https://openexchange.intersystems.com/package/interoperability-soap)
 [![Quality Gate Status](https://community.objectscriptquality.com/api/project_badges/measure?project=intersystems_iris_community%2Finteroperability-soap&metric=alert_status)](https://community.objectscriptquality.com/dashboard?id=intersystems_iris_community%2Finteroperability-soap)
 [![Reliability Rating](https://community.objectscriptquality.com/api/project_badges/measure?project=intersystems_iris_community%2Finteroperability-soap&metric=reliability_rating)](https://community.objectscriptquality.com/dashboard?id=intersystems_iris_community%2Finteroperability-soap)
# interoperability-test
I started from iris-interoperability-template. It contained a simple interoperablity solution which reads data from Reddit, filters it, and directs outputs into files or sends via email.

## Error encountered
```
the --mount option requires BuildKit. Refer to https://docs.docker.com/go/buildkit/ to learn how to build images with BuildKit enabled
ERROR: Service 'iris' failed to build : Build failed
```
## Workaround
My docker-compose installation did not support BuildKit. So I built the image separately and used the image in docker-compose.yaml.
```
DOCKER_BUILDKIT=1 sudo docker build --no-cache --progress=plain --tag testint .
```
## Why interoperability-test?

My team works on IRIS interoperability solution running on Red Hat OpenShift Kubernetes Container Platform. We are trying to implement CI/CD, but we are missing testing tool. We are making changes to our POC interfaces. I wanted to see what can be done with Unit Testing tools provided by InterSystems.

We are working on POC interface. Below is a screenshot of a visual trace in Health Connect:
![screenshot](https://github.com/oliverwilms/bilder/blob/main/TracePOCinHC.PNG)

IncomingPOCFile Service reads a file and sends it to BPL process. Next step is sending an Authorization request to an external system (DMLSS). After Auth request is approved, a response file can be sent to POCResponseFile operation.

We are not connected to DMLSS during build time. I decided to simulate the Authorization process. The Authorization request is sent to this Production due to System Default Settings which are shown on the screenshot below. It is received by Generic REST Service which sends a request to POC Auth BPL. POC Auth BPL simulates the response from DMLSS.

![screenshot](https://github.com/oliverwilms/bilder/blob/main/testint.PNG)
This sample has an interoperability [production](https://github.com/intersystems-community/iris-interoperability-template/blob/master/src/dc/Demo/Production.cls) with an inbound [Reddit Adapter](https://github.com/intersystems-community/iris-interoperability-template/blob/master/src/dc/Reddit/InboundAdapter.cls) which is used by a [Business Service](https://github.com/intersystems-community/iris-interoperability-template/blob/master/src/dc/Demo/RedditService.cls) to read data from Reddit.com.
It reads from reddit.com/new/.json every 15 sec.
You can alter both the URL and frequency in the service's settings.
<img width="1411" alt="Screenshot 2020-10-29 at 19 33 14" src="https://user-images.githubusercontent.com/2781759/97603605-a6d0af00-1a1d-11eb-99cc-481efadb0ec6.png">

The production has a business process with a rule, which filters on news that mentions cats and dogs. The business process then sends this data to a business operation which either saves data to a source folder /output/Dog.txt or /output/Cat.txt.
<img width="864" alt="Screenshot 2020-10-29 at 19 38 58" src="https://user-images.githubusercontent.com/2781759/97606568-fcf32180-1a20-11eb-90de-4257dd2cf552.png"> 

## Prerequisites
Make sure you have [git](https://git-scm.com/book/en/v2/Getting-Started-Installing-Git) and [Docker desktop](https://www.docker.com/products/docker-desktop) installed.

## Installation: ZPM

Open IRIS Namespace with Interoperability Enabled.
Open Terminal and call:
USER>zpm "install interoperability-test"

## Installation: Docker
Clone/git pull the repo into any local directory

```
$ git clone https://github.com/intersystems-community/interoperability-test.git
```

Open the terminal in this directory and run:

```
DOCKER_BUILDKIT=1 sudo docker build --no-cache --progress=plain --tag testint .
$ docker-compose up -d
```

## How to Run UnitTest

```
$ docker-compose up -d
sudo docker exec -it interoperability-test_iris_1 iris session iris
zpm "test interoperability-test -only -v"
```

## How to Run the Sample

Open the [production](http://localhost:52795/csp/user/EnsPortal.ProductionConfig.zen?PRODUCTION=dc.Demo.Production) and start it.
It will start gathering news from reddit.com/new/ and filter it on cats and dogs into /output/Dog.txt or /output/Cat.txt files.

You can alter the [business rule](http://localhost:52795/csp/user/EnsPortal.RuleEditor.zen?RULE=dc.Demo.FilterPostsRoutingRule) to filter for different words, or to use an email operation to send posts via email.
<img width="1123" alt="Screenshot 2020-10-29 at 20 05 34" src="https://user-images.githubusercontent.com/2781759/97607761-77707100-1a22-11eb-9ce8-0d14d6f6e315.png">

## How to alter the template 
Use the green    "Use this template" button on Github to copy files into a new repository and build a new IRIS interoperability solution using this one as an example.

This repository is ready to code in VSCode with the ObjectScript plugin.
Install [VSCode](https://code.visualstudio.com/), [Docker](https://marketplace.visualstudio.com/items?itemName=ms-azuretools.vscode-docker) and [ObjectScript](https://marketplace.visualstudio.com/items?itemName=daimor.vscode-objectscript) plugin and open the folder in VSCode.

Use the handy VSCode menu to access the production and business rule editor and run a terminal:
<img width="656" alt="Screenshot 2020-10-29 at 20 15 56" src="https://user-images.githubusercontent.com/2781759/97608650-aa673480-1a23-11eb-999e-61e889304e59.png">
