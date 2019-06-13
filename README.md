# Build An Alexa Hello World Skill With Express-With-Lambda

### This is a simple tutorial to introduce a simple Alexa skill with Express-With-Lambda.

## Step 1 : 
    Clone this [hello world skill](https://github.com/alexa/skill-sample-nodejs-hello-world.git)

### Add express-server.js file

1. Create a express-server.js inside lambda/custom/
2. Add following code in it.

`'use strict'
const express = require('express'),
      bodyParser = require('body-parser'),
      app = express();

app.use(bodyParser.json());
app.use(bodyParser.urlencoded({ extended: true }));

app.post("/", async(req, res) => {

    res.send("Hello World!");
});

module.exports = app;`

## Step 2 :
    We're using claudia.js to automate the process of creating AWS lambda function and API gateway.

### Install Claudia and configure AWS credential
    `npm install -g claudia`

1. Create an AWS profile with IAM full access, Lambda full access and API Gateway Administrator privileges.
2. Add the keys to your .aws/credentials file

`[claudia]
aws_access_key_id = YOUR_ACCESS_KEY
aws_secret_access_key = YOUR_ACCESS_SECRET`

Note : If we configured aws credentails before then we can skip above point 2.

### Generate AWS Lambda wrapper for Express app

To make your app work correctly with AWS Lambda, you need to generate AWS Lambda wrapper for your Express app. With Claudia, you can do so by running the following command in your terminal.

`claudia generate-serverless-express-proxy --express-module express-server --source lambda/custom/ --proxy-module-name express-lambda --profile <aws-profile-name>`

where express-server is a name of an entry file of your Express app, just without the .js extension.

This step generated a file named express-lambda.js

### Deploy Express App

Note : We should remove index.js file inside lambda/custom/

Now you only need to deploy your Express app (with express-lambda.js file) to AWS Lambda and API Gateway using the claudia create command.

`claudia create --source lambda/custom/ --name express-lambda-service  --handler express-lambda.handler --deploy-proxy-api --region us-east-1 --profile <aws-profile-name>`

The command printed the following response:

`{  "lambda": {    "role": "awesome-serverless-expressjs-app-executor",    "name": "awesome-serverless-expressjs-app",    "region": "eu-central-1"  },  "api": {    "id": "iltfb5bke3",    "url": "https://iltfb5bke3.execute-api.eu-central-1.amazonaws.com/latest"  }}
`

Now test our express-lambda service is working or not by copying the printed url in your terminal, test it with postman, the API should return 'Hello World!'

## Step 3 :

 Replace `express-server.js` code with following:

`'use strict'
const express = require('express'),
      bodyParser = require('body-parser'),
      Alexa = require('ask-sdk-core'),
      awsServerlessExpressMiddleware = require('aws-serverless-express/middleware'),
      app = express();

app.use(bodyParser.json())
app.use(bodyParser.urlencoded({ extended: true }))
app.use(awsServerlessExpressMiddleware.eventContext())
      
app.post("/", async(req, res) => {

    console.log(req);

    const LaunchRequestHandler = {
    canHandle(handlerInput) {
        return handlerInput.requestEnvelope.request.type === 'LaunchRequest';
    },
    handle(handlerInput) {
        const speechText = 'Welcome to the Alexa Skills Kit, you can say hello!';

        return handlerInput.responseBuilder
        .speak(speechText)
        .reprompt(speechText)
        .withSimpleCard('Hello World', speechText)
        .getResponse();
    },
    };

    const HelloWorldIntentHandler = {
    canHandle(handlerInput) {
        return handlerInput.requestEnvelope.request.type === 'IntentRequest'
        && handlerInput.requestEnvelope.request.intent.name === 'HelloWorldIntent';
    },
    handle(handlerInput) {
        const speechText = 'Hello World!';

        return handlerInput.responseBuilder
        .speak(speechText)
        .withSimpleCard('Hello World', speechText)
        .getResponse();
    },
    };

    const HelpIntentHandler = {
    canHandle(handlerInput) {
        return handlerInput.requestEnvelope.request.type === 'IntentRequest'
        && handlerInput.requestEnvelope.request.intent.name === 'AMAZON.HelpIntent';
    },
    handle(handlerInput) {
        const speechText = 'You can say hello to me!';

        return handlerInput.responseBuilder
        .speak(speechText)
        .reprompt(speechText)
        .withSimpleCard('Hello World', speechText)
        .getResponse();
    },
    };

    const CancelAndStopIntentHandler = {
    canHandle(handlerInput) {
        return handlerInput.requestEnvelope.request.type === 'IntentRequest'
        && (handlerInput.requestEnvelope.request.intent.name === 'AMAZON.CancelIntent'
            || handlerInput.requestEnvelope.request.intent.name === 'AMAZON.StopIntent');
    },
    handle(handlerInput) {
        const speechText = 'Goodbye!';

        return handlerInput.responseBuilder
        .speak(speechText)
        .withSimpleCard('Hello World', speechText)
        .getResponse();
    },
    };

    const SessionEndedRequestHandler = {
    canHandle(handlerInput) {
        return handlerInput.requestEnvelope.request.type === 'SessionEndedRequest';
    },
    handle(handlerInput) {
        console.log(`Session ended with reason: ${handlerInput.requestEnvelope.request.reason}`);

        return handlerInput.responseBuilder.getResponse();
    },
    };

    const ErrorHandler = {
    canHandle() {
        return true;
    },
    handle(handlerInput, error) {
        console.log(`Error handled: ${error.message}`);

        return handlerInput.responseBuilder
        .speak('Sorry, I can\'t understand the command. Please say again.')
        .reprompt('Sorry, I can\'t understand the command. Please say again.')
        .getResponse();
    },
    };

    const skillBuilder = Alexa.SkillBuilders.custom()
    .addRequestHandlers(
        LaunchRequestHandler,
        HelloWorldIntentHandler,
        HelpIntentHandler,
        CancelAndStopIntentHandler,
        SessionEndedRequestHandler
    )
    .addErrorHandlers(ErrorHandler)
    .create();

    const response = await skillBuilder.invoke(req.body, req.apiGateway.context);

    res.send(response);
})
module.exports = app;`

> Run update command
    `claudia update --source lambda/custom/ --profile <aws-profile-name>`

## Step 4 :

    Deploy skill and model using `ASK CLI`
    `ask deploy -t skill`
    `ask deploy -t model`

## Step 5 :
> Add https endpoint generated in Step 2 into the Skill Developer Portal Endpoint->HTTPS and SSL certificate type as 'My development endpoint is a sub-domain of a domain that has a wildcard certificate from a certificate authority'.
> Save Endpoints
> Save Model
> Build Model

To check everything has deployed successfully, open the Developer Portal and check the Hello World Skill.