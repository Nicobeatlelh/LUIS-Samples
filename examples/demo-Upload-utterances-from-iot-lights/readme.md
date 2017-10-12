# Upload utterances from IOT Lights

A sample Node.Js application to use queries from a hypothetical IOT application. This application parses the IOT queries into the acceptable LUIS query format, and [uploads as a batch](https://westus.dev.cognitive.microsoft.com/docs/services/5890b47c39e2bb17b84a55ff/operations/5890b47c39e2bb052c5b9c09).

The application main file is the [index.js]('./index.js). This file contains the configuration settings and calls into the three files:

- [_parse.js](./_parse.js) : convert CSV from query logs into JSON for upload
- [_upload.js](./_upload) : upload JSON to batch label API

The application assumes the IOT queries are in a JSON file format. An example is found at:

- [./example-files/iot-lights-utterances.json](./example-files/iot-lights-utterances.json) : convert JSON from IOT logs into JSON for upload

The application will create files associated with each step:

- [utterances.json](./example-files/utterances.json) : batch labels
- [utterances.upload.json](./example-files/utterances.upload.json) : final response body from upload API

If one or all of these files is missing, their was an error with the application. 

### Prerequisites
The minimum prerequisites to run this sample are:
* Latest Node.js with NPM. Download it from [here](https://nodejs.org/en/download/).
* A [trained](https://docs.microsoft.com/en-us/azure/cognitive-services/LUIS/train-test) and [published](https://docs.microsoft.com/en-us/azure/cognitive-services/LUIS/publishapp) LUIS Application. 
* **[Recommended]** Visual Studio Code for IntelliSense and debugging, download it from [here](https://code.visualstudio.com/) for free.

### Install
Install the Node.js dependencies from NPM in the terminal/command line.

````
> npm install
````

### Change Configuration Settings
In order to use this application, you need to change the values in the index.js file to your own subscription key, app ID, and version ID.

Open the index.js file, and change these values at the top of the file.

````JavaScript
// TBD: CHANGE THESE VALUES
const LUIS_subscriptionKey = "YOUR_SUBSCRIPTION_KEY";
const LUIS_appId = "YOUR_APP_ID";
const LUIS_versionId = "0.1";
````
### Run the application
Run the application from a terminal/command line with Node.js.

````
> node index.js
````
### Application progress
While the application is running, the terminal/command line will show progress.

````
> node index.js
download done
parse done
upload done
process done
````

### LUIS APIs used in this sample
This sample uses the [batch add labels](https://westus.dev.cognitive.microsoft.com/docs/services/5890b47c39e2bb17b84a55ff/operations/5890b47c39e2bb052c5b9c09) API.

### Format of the JSON for the batch upload
The format of the JSON for the batch upload is noted in the [batch add labels](https://westus.dev.cognitive.microsoft.com/docs/services/5890b47c39e2bb17b84a55ff/operations/5890b47c39e2bb052c5b9c09) API.

### Use your own private apps
If you incorrectly use an app ID that you do not have permission to upload to, such as any public apps, you will recieve an error.

### Intent And Entities are not created if NOT found
Any intent or entity uploaded that is not found in your LUIS app will cause an error. It is important that all intents and entities used in the batch already exist in the app.

### Errors in output file of the application
The final response body from upload API is in the 'utterances.upload.json' file. This file will be an array of responses, one response for each item in the batch.

Each item in the batch can succeed or fail independent of any other item, so it is important to check the response.

#### Examples of correctly formatted items:

````JavaScript
// successfully formated item
    {
        "row": 1,
        "text": "go to paris",
        "intentName": "BookFlight",
        "entityLabels": [
            {
                "entityName": "Location::LocationTo",
                "startCharIndex": 6,
                "endCharIndex": 10
            }
        ]
    }
````

#### An example of a successful item upload response:

````JavaScript
// successfully uploaded item
{
    "value": {
        "UtteranceText": "go to paris",
        "ExampleId": -175128
    },
    "hasError": false
}
````

#### Examples of successful request with failed items :

````JavaScript
// failed uploaded item - don't upload built-ins
{
    "value": null,
    "hasError": true,
    "error": {
        "code": "FAILED",
        "message": "ticket to seattle tomorrowtimezoneOffset=0. Error: The entity extractor builtin.number doesn't exist in the selected application"
    }
}
````

````JavaScript
// failed uploaded item - missing intent
{
    "value": null,
    "hasError": true,
    "error": {
        "code":"FAILED","message":"turn on the left light. Error: The intent classifier TurnOn does not exist in the selected application"
    }
}
````
 
#### Examples of failed requests because of malformed items

Batch upload items (or the whole batch) can result in parsing errors in the LUIS API. These errors are generally returned as HTTP 400 status errors instead of returning a successful response with an array of items, some of which failed.

````JavaScript
// malformed item - entityLabels first array item is present but empty
// fix - should remove {}
{
    "row": 2,
    "text": "ticket to paris",
    "intentName": "BookFlight",
    "entityLabels": [
        {
            
        }
    ]
}


// Http Error 400 - 

```` 

````JavaScript
// malformed item - malformed JSON - no comma
// fix - add comma after every key:value pair
[
    {
        "text": "Hello"
        "intent": "Greetings"
    },
    {
        "text": "I want bread"
        "intent": "Request"
    }
]
```` 

````JavaScript
// malformed item - malformed JSON - extra comman at end of key:value pair
// fix - remove extra comma
[
    {
        "text": "Hello",
        "intent": "Greetings",
    },
    {
        "text": "I want bread",
        "intent": "Request"
    }
]
````