# Deploying to AWS

## Demo

![demo](assets/2020-07-16%2013-13-13.gif)

URL: [https://un64uvk2oi.execute-api.ap-south-1.amazonaws.com/dev/](https://un64uvk2oi.execute-api.ap-south-1.amazonaws.com/dev/)

## CloudWatch Logs

![logs](assets/logs.png)

## Instructions and Notes

Install Node and NPM

```bash
curl -sL https://deb.nodesource.com/setup_14.x | sudo -E bash -
sudo apt-get install -y nodejs
```

Install serverless

```bash
sudo npm install -g serverless
```

Setup serverless

```bash
sudo chown -R $USER:$(id -gn $USER) /home/shadowleaf/.config
sls config credentials --provider aws --key AKIBWDBSFP1ACHJPAAGQ --secret PZWjxph8vFUkvLinL0frtBk1NijOKjI18DjFMEqm --overwrite
```

Use the template to create a new project

```bash
sls create --template aws-python3 --name sls-flask-ml-test
```

```txt
Serverless: Generating boilerplate...
 _______                             __
|   _   .-----.----.--.--.-----.----|  .-----.-----.-----.
|   |___|  -__|   _|  |  |  -__|   _|  |  -__|__ --|__ --|
|____   |_____|__|  \___/|_____|__| |__|_____|_____|_____|
|   |   |             The Serverless Application Framework
|       |                           serverless.com, v1.74.1
 -------'

Serverless: Successfully generated boilerplate for template: "aws-python3"
```

Install anaconda from https://docs.anaconda.com/anaconda/install/linux/

Create a new environment in Anaconda

```bash
conda create --name sls-flask
conda activate sls-flask
```

```bash
sls plugin install -n serverless-python-requirements
sls plugin install -n serverless-wsgi
```

Convert model to JIT

```python
>>> import torch
>>> model = torch.hub.load('pytorch/vision:v0.6.0', 'mobilenet_v2', pretrained=True)
>>> traced_model = torch.jit.trace(model, torch.randn(1, 3, 224, 224))
>>> traced_model.save('mobilenetv2.pt')
```

> IMPORTANT

Make sure to add Binary Media Types in Amazon API Gateway Settings

```txt
multipart/form-data
*/*
```

Deploy to SLS using

```bash
sls deploy
```

> NOTE: If you get any kind of error which says .serverless/requirements not found, just rerun `sls deploy` 2-3 times, it'll fix itself

## Some Gotchas

- Python Logging does not work the way you think, there's already a lambda logger running, so you'll have to override that logger
- i faced a lot of issues with PIL, and the image wasn't opening, then in realised that i hadn't enabled the binary media type in the gateway settings, my bad
- If you want to test image upload, please use the bare flask app, having the pytorch depency along with it makes the size 142 MB and everytime uploading that much size hurts me, so just use the bare flask app, test it, and then finally integrate the model
- LOG EVERYTHING ! LOGS ARE YOUR BEST FRIEND
- Look into this https://pytorch.org/tutorials/intermediate/flask_rest_api_tutorial.html and https://github.com/ShannonAI/service-streamer/wiki/Vision-Recognition-Service-with-Flask-and-service-streamer
