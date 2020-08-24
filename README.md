# Quick recipe for deploying hello-world app with SAM on Node.js 

- Install latest NodeJS, AWS and SAM CLI version.

- On AWS console create an ```user```, and a ```group```. Attach ```administratives rights``` to that group (those permisions are needed for SAM deployment).
- Create a root folder and from the CLI init a sam project (in our case we will use nodejs10x).

```
sam init --runtime nodejs10.x
```

- Choose: ``` 1 - AWS Quick Start Templates ```.

- Project name [sam-app]: ``` sam-app ``` (Choose the name of your preference, for this tutorial, let's keep it ``` sam-app ```).

- Wait for the cloning template process.

- Once downloaded, open the ``` template.yaml``` file and add a new ``` Resource``` (Keep the indentation to avoid any issues). In our case let's create the ``` ClockFunction ```. This will create a simple API Gateway
with a ``` GET``` method retrieving the current hour.

```
  ClockFunction:
    Type: AWS::Serverless::Function # More info about Function Resource: https://github.com/awslabs/serverless-application-model/blob/master/versions/2016-10-31.md#awsserverlessfunction
    Properties:
      CodeUri: clock/
      Handler: handler.clock
      Runtime: nodejs10.x
      Events:
        ClockApi:
          Type: Api
          Properties:
            Path: /clock
            Method: get
```

- On ``` sam-app ``` folder create the ``` clock ``` folder. Inside ``` clock ``` create a file called ``` handler.js``` and paste the next code:

```js
const moment = require('moment');
exports.clock = async(event) => {
    console.log('Clock function run!');

    const message = moment().format();

    const response = {
        statusCode: 200,
        body: JSON.stringify(message)
    };

    return response;
}
```
- Open a terminal and type: ```npm install --save moment ```. This will save the dependencies needed on ```handler.js```.

- cd the the root folder ``` sam-app ```.

- Start the building by typing ``` sam build ```.

- Build a ```S3``` bucket in order to save all the files from this tutorial, typing:

```aws s3 --region us-west-2 mb s3://bucket-name-of-your-preference```.

- Package all the files (I decided using ``` package.yml ``` name for convention, use whichever you want to ):

```sam package --template-file template.yaml --output-template-file package.yml --s3-bucket bucket-name-of-your-preference```.

- Deploy (check your location, ours is ``` us-west-2```, and for the ```stack-name``` we used just ```sam-app```, chose whichever you want to):

``` sam deploy --region us-west-2 --capabilities CAPABILITY_IAM --template-file package.yml --stack-name sam-app```

# What to do when we make an update (for example adding a new handler in this case)

- Let's add the ```ConvertTimeFunction``` on the ```template.yaml```
```
  ConvertTimeFunction:
    Type: AWS::Serverless::Function # More info about Function Resource: https://github.com/awslabs/serverless-application-model/blob/master/versions/2016-10-31.md#awsserverlessfunction
    Properties:
      CodeUri: convert-time/
      Handler: handler.ConvertTime
      Runtime: nodejs10.x
      Events:
        ConvertTimeAPI:
          Type: Api
          Properties:
            Path: /convert-time
            Method: get
```
- On ``` sam-app ``` folder create a new folder called ```convert-time```.

- On ```convert-time``` create a file called ``` handler.js ```.

- Paste the next code on ```handler.js```:

```js
const moment = require('moment-timezone');

exports.ConvertTime = async(event) => {
    console.log('Convert Time was executed');

    let tz = event.queryStringParameters && event.queryStringParameters.tz;

    let formattedDate = moment().format();

    if (tz !== null) {
        formattedDate = moment().tz(tz).format();
    } else {
        tz = "GMT 0";
    }

    const message = "The time in " + tz + " is " + formattedDate;

    const response = {
        statusCode: 200,
        body: JSON.stringify(message)
    };

    return response;
}
```
- Open a terminal on ```convert-time``` and save the dependencies needed for the handler

```
npm install --save convert-time
```

Go back to ``` sam-app ``` folder , rebuild and repackage the whole project and deploy:

``` sam build ```

```sam package --template-file template.yaml --output-template-file package.yml --s3-bucket bucket-name-of-your-preference```.


```sam deploy --region us-west-2 --capabilities CAPABILITY_IAM --template-file package.yml --stack-name sam-app```.

If all went go, you should see the updates on the lambdas and API gateways as well \0/ \0/ :) .

# How to delete all the application from the SAM CLI (Be careful when using this command!, AWS deletes all quite quick!!!).

``` aws cloudformation delete-stack --stack-name sam-app ```