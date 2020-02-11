# A Question and Answer Bot Using Amazon Lex and Amazon Alexa

> Build a chat bot to answer questions. 

Read this in other languages: [日本語](README.jp.md)

## Overview
This repository contains code for the QnABot, described in the AWS AI blog post [“Creating a Question and Answer Bot with Amazon Lex and Amazon Alexa”](https://aws.amazon.com/blogs/ai/creating-a-question-and-answer-bot-with-amazon-lex-and-amazon-alexa/).

See the "Getting Started" to launch your own QnABot

**New features in 2.6.0** [Kendra Fallback and MultiLanguage Support](#new-features)

## Prerequisites

- Run Linux. (tested on Amazon Linux)
- Install npm >6.13.1 and node >10.16.3. ([instructions](https://nodejs.org/en/download/))
- Clone this repo.
- Set up an AWS account. ([instructions](https://AWS.amazon.com/free/?sc_channel=PS&sc_campaign=acquisition_US&sc_publisher=google&sc_medium=cloud_computing_b&sc_content=AWS_account_bmm_control_q32016&sc_detail=%2BAWS%20%2Baccount&sc_category=cloud_computing&sc_segment=102882724242&sc_matchtype=b&sc_country=US&s_kwcid=AL!4422!3!102882724242!b!!g!!%2BAWS%20%2Baccount&ef_id=WS3s1AAAAJur-Oj2:20170825145941:s))
- Configure AWS CLI and a local credentials file. ([instructions](http://docs.AWS.amazon.com/cli/latest/userguide/cli-chap-welcome.html))  


## Getting Started
First, install all prerequisites:
```shell
npm install 
```

Next, set up your configuration file:

```shell
npm run config
```

now edit config.json with you information.

| param | description | 
|-------|-------------|
|region | the AWS region to launch stacks in |
|profile| the AWS credential profile to use |
|namespace| a logical name space to run your templates in such as dev, test and/or prod |
|devEmail(required) | the email to use when creating admin users in automated stack launches |

Next, use the following command to launch a CloudFormation template to create the S3 bucket to be used for lambda code and CloudFormation templates. Wait for this template to complete (you can watch progress from the command line or [AWS CloudFormation console](https://console.AWS.amazon.com/cloudformation/home))  
```shell
npm run bootstrap
```

Finally, use the following command to launch template to deploy the QnA bot in your AWS account. When the stack has completed you will be able to log into the Designer UI (The URL is an output of the template). A temporary password to the email in your config.json:
```shell
npm run up
```

If you have an existing stack you can run the following to update your stack:
```shell 
npm run update
```

## Components
### CloudFormation Templates
The CloudFormation test templates are in the templates/test folder. The current templates are:

1. Master: the template contains all the resources for QnABot.
2. Public: this is a version of the Master template with less parameters, less outputs, and the bootstrap bucket hardcoded to the publicBucket in config.json
3. various templates in /templates/dev: needed for local testing of the lambda functions. 

Run a template test with:
```shell
npm run stack test/{template-name}
```

For example, if you want to test the domain template run:
```shell
npm run stack test/domain
```

To understand the command more run: 
```shell 
npm run stack -h
```

You also can check a template's syntax with:
```shell
npm run check {template-name}
```
ex. 
```shell
npm run check domain
```

To understand the command more run: 
```shell 
npm check stack -h
```

### Lambda Functions
Lambda functions are found in the /lambda directory. Refer to the README.md file in each directory for instructions on setting up a dev environment and testing. 
[Fulfillment](lambda/fulfillment/README.md)
[CFN](lambda/handler/README.md)
[Lex-Build](lambda/lex-build/README.md)
[Import](lambda/import/README.md)

### Web Interface
The Designer UI and client UI code is in the /website directory. 

To Test the web ui, Launch a development master stack:
```shell
npm run stack dev/master up
```
when that stack has finished run:
```shell
cd ./website ; make dev
```
this will launch a running webpack process that will watch for changes to files and upload the changes to your running dev/master stack. 

#### Designer UI Compatibility 
Currently the only browsers supported are:  
- Chrome  
- FireFox  
We are currently working on adding Microsoft Edge support.  

## Built With

* [Vue](https://vuejs.org/) 
* [Webpack](https://webpack.github.io/)

## License
See the [LICENSE.md](LICENSE.md) file for details

## New features

### Kendra Fallback Support
QnABot version 2.6.0 optionally supports integration with Amazon Kendra as a fallback mechanism if a question/answer can not
be found in QnABot.
 
**Important note. Use of Kendra as a fallback mechanism will incur additional charges for your AWS Account. Please 
review the Kendra pricing structure. The fallback mechanism for QnABot can be useful when deploying Kendra as an
Enterprise search solution.**
 
To enable this support for your Kendra indexes add the Custom Property 'ALT_SEARCH_KENDRA_INDEXES' 
to QnABot's set of custom properties in Systems Manager Parameter Store. 

This custom property will contain a string value that specifies an array of 
Kendra indexes to search. At least one index must be specified. Until this custom property 
is set in QnABot, use of Kendra as a fallback mechanism will return an error. 

To find your custom property name used in Parameter store, open the QnABot CF stack outputs. You'll set key in QnABot CF
Stack Outputs called 'CustomSettingsSSMParameterName'. If will have a value similar to 

```
CFN-CustomQnABotSettings-EOVHQJcYx9Ms
```

Once you know the name of your parameter, use the SSM Parameter Store UI to edit the parameter. 
Add your Kendra IndexId to a key/value pair in this json object. If you already
have set a new custom property for QnABot you will need to 
add to the object a new key/value pair rather than replacing the entire string. Once completed your property
will look similar to the following.  

```
{"ALT_SEARCH_KENDRA_INDEXES":"[\"857710ab-9637-4a46-910f-9a1456d02596\"]"}
```

**Note the Escaped Quote marks around the array of Kendra index ids are required**

**Don't forget to use your Kendra Index ID rather than the one in the sample**

The last step you need to perform to enable this feature is to use the QnABot Designer UI to import a Sample/Extension 
named KendraFallback.

This loads a new question with a qid of "KendraFallback". Edit this question in the Designer UI and change its question from 
"no_hits_alternative" to "no_hits" and save the changes. 

If you have previously loaded the QnAUtility.json from Examples/Extensions you need to either remove 
the question with the ID "CustomNoMatches" or change the question for this ID from "no_hits" to "no_hits_original"

Once the new question, "KendraFallback" is configured as the response for "no_hits", the Kendra index will be
searched for an answer whenever a curated answer can not be found. Once setup, Kendra provides a fallback 
mechanism prior to telling the user an answer could not be found. 

A [workshop](https://github.com/aws-samples/aws-ai-qna-bot/tree/master/workshops/reinvent2019/readme.md) is available in github 
that will walk you through setting up this feature. 

**Important note. Use of Kendra as a fallback mechanism will incur additional charges for your AWS Account. Please 
review the Kendra pricing structure. The fallback mechanism for QnABot can be useful when deploying Kendra as an
Enterprise search solution.**

### MultiLanguage Support

QnABot version 2.6.0 supports use of multiple languages. Once enabled, if the user enters a question in a language other 
than english, QnABot will attempt to return an answer in the other language. Its does this by using AWS Comprehend to 
identify the language spoken or typed. If Comprehend can identify the language based on a configured minimum confidence, 
QnABot will serve up content based on that locale.

It converts the question posed by the user to English and performs a lookup of the answer in Elastic Search 
just as it normally does. Once it finds the question, QnABot will serve up the configured answer. The answers specifying 
blocks to use for different locales.

Users can also set a preferred language whereby QnABot will always attempt to respond with content in the chosen
locale.  If the user sets the preferred language to be Spanish, QnABot will always try and serve up content using
Spanish when possible. 

By default this feature is disabled. Use the following three steps to enable and configure this feature. Step 1 enables 
the feature. Step 2 loads in two questions from this extension that allow the user to select a preferred language. The 
defaults supplied in this question are English, Spanish, French, German, and Italian. You can extend this list to
support other languages. 

Step 1) Custom Property

a) QnABot uses a property named ENABLE_MULTI_LANGUAGE_SUPPORT. It is a boolean and has a default value of false. 
You can override this setting using SSM Parameter Store to add a new key/value pair as a custom property. 
Find your custom property name from the QnABot CF stack outputs. The key in the QnABot CF stack outputs is 
'CustomSettingsSSMParameterName'. If will have a value similar to 

```
CFN-CustomQnABotSettings-EOVHQJcYx9Ms
```

b) Identify your CustomSettingsSSMParameterName property name using CloudFormation and then open the AWS Systems 
Manager console and navigate to ParameterStore. Filter the list using your custom property name name. Open and Edit the parameter. 

Set the value to be the following and save the changes. Be careful if you already have existing key/value pairs in
this property. If you do have existing key/value apirs, be sure to add the key and value as a new attribute of the object. 

```
{"ENABLE_MULTI_LANGUAGE_SUPPORT":true}
```

Step 2) Use the Designer UI to import the Sample/Extension named Language / Multiple Language Support. 

This will add two questions to the system: Language.000 and Language.001. 

Language.000 provides a question that allows the user to set the current sessions preferred output saying a simple word 
such as French, German, or Spanish, or Italian. 

Language.001 resets the preferred language. This can be performed by saying or typing 'reset language' or 'detect language'. 
You can also input using your language of choice assuming AWS Translate can translate the input back to English. 

Once you've imported this extension question try typing the question 'Spanish'. You should see a Spanish response. 

Next enter 'English' and you will have switched your preference back to English. 

Next enter 'reset language' and your preference will be reset and language auto detection will occur again.

The answer for Language.000 uses the following handlebar syntax

```
{{#setLang 'fr' false}} D'accord. J'ai défini votre langue préférée sur l'anglais. {{/setLang}}
{{#setLang 'es' false}} Okay. He configurado tu idioma preferido al inglés.  {{/setLang}}
{{#setLang 'de' false}} In Ordnung. Ich habe Ihre bevorzugte Sprache auf Englisch eingestellt. {{/setLang}}
{{#setLang 'it' false}} Ok. Ho impostato la tua lingua preferita sull'inglese.{{/setLang}}
{{#setLang 'en' true}} Ok. I've set your preferred language to English. {{/setLang}}
```

The helper function setLang performs the necessary processing based on the language/locale detected by Comprehend. To
add support for other languages just extend the answer in Language.000 with additional locales. 

Step 3) In order to serve up content that is locale specific modify answers in questions to respond in multiple languages. 
Lets modify the question sun.1. The following would be an example where the handlebar function ifLang is used to 
specify a response for spanish. 

Use the handlebar template defaultLang to specify the response QnABot should provide when the language is unknown. By
default this is typically in English but could be in any language as needed. 

{{#defaultLant}}{{/defaultLang}} must be the last element in the answer block. 

```
{{#ifLang 'es'}}
Nuestro sol tiene 4.600 millones de años. Se considera una enana amarilla con un diámetro de 1,392,684 kilómetros y una circunferencia de 4,370,005 kilómetros. Tiene una masa que es igual a 333,060 tierras y una temperatura superficial de 5,500 grados centígrados. ¡Muy caliente!
{{/ifLang}}
{{#defaultLang}}
Our sun is 4.6 billion years old. Its considered a yellow dwarf with a diameter of 1,392,684 kilometers and a circumference of 4,370,005 kilometers. It has a mass that is equal to 333,060 earths and a surface temperature of 5,500 degrees celsius. Really Hot!
{{/defaultLang}}
```

The handlebar function ifLang takes locale as a quoted parameter. This tells QnABot which locale to associate with the subsequent
text. 


A [workshop](https://github.com/aws-samples/aws-ai-qna-bot/tree/master/workshops/reinvent2019/readme.md) is available in github 
that will walk you through setting up this feature. 