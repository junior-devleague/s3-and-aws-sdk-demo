# AWS-SDK and S3 Demo
[Advanced] - Using AWS-SDK to Create and List S3 Bucket

## What is Amazon s3?
- Stands for simple storage service
- One of the first services in AWS
- S3 is a safe place to store any type of files (text files, images, mp3)
- You can think of s3 as Dropbox or GoogleDrive
- Is Object based(Key & Value pair)
- FIles are stored in buckets(similar to folder)
- Bucket Names must be unique

## What is aws-sdk?
- Software Development Kit
- A collection of tools for developers creating web applications provided by aws
- Provides APIs, class libraries, and code samples
- We will be using aws-sdk to communicate with s3

## S3 and aws-sdk demo
---
1. After creating your serverless boilerplate/template run
```
npm init --yes
```
  - This will create a package.json file for you
  - package.json will allow us to install aws-sdk npm pakaage to use
2. To install aws-sdk run
```
npm install aws-sdk
```
  - You should now see a `node_modules` directory in your current directory

3. Require aws-sdk and create an s3 client in our handler.js file
```js
  const AWS = require('aws-sdk');
  const s3 = new AWS.S3();
```
## Create an s3 bucket and pass in data **locally**

[Create Bucket - AWS SDK for JavaScript](https://docs.aws.amazon.com/AWSJavaScriptSDK/latest/AWS/S3.html#createBucket-property)
```js
s3.createBucket({ Bucket: "YOUR_BUCKET_NAME"}, function(err, data) {
   if (err) console.log(err, err.stack); // an error occurred
   else     console.log(data); // successful response
 });
```
1. Hard code YOUR_BUCKET_NAME for now(We will add dynamically through Postman)
2. Remove `async` keyword from lambda function
3. Add `callback` parameter in lambda function
4. Store return vaule in a variable named `response`
5. Pass `response` into callback function `callback(null, response)`
6. Run lambda function locally
7. Pass in bucket name data dynamically
8. Create a `dummyData.json` file. Add the following:
```json
{
    "bucketName": "testingjay123"
}
```
13. run with --path to pass in `dummyData.json`
```
serverless invoke local --path ./data.json
```
## Post to a deployed lambda
1. Before deploying lambda function, lets add a few code snippets to our `serverless.yml` file
2. Add `iamRoleStatements:` to the end of `provider:`:
```yml
  provider:
  name: aws
  runtime: nodejs8.10
  stage: dev
  region: us-west-2
  iamRoleStatements:
    - Effect: Allow
      Action:
        - s3:*
      Resource: "*"
```
- This allows our deployed lambda function to access the s3 service
3. Add `integration: lambda` under `cors: true`
```yml
functions:
  create:
    handler: handler.create
    events:
      - http: 
          path: create
          method: post
          cors: true
          integration: lambda
          request:
            passThrough: WHEN_NO_TEMPLATES
            template:
              application/x-www-form-urlencoded: '{ "body" : "$input.body" }'

```
- This allows us to recieve data passed in through Postman
4. Deploy serverless
5. Copy and paste the returned POST endpoint url and paste into Postman  Post method
6. In Postman, select `Body` -> `raw` -> select `JSON(application/json)` from the dropdown that says `Text`
7. In the below input field, type in JSON data format exactly like the `dummyData.json` file

## How to Create Multiple Lambda Functions
1. Create new directory `lambdas`
2. Rename `handler.js` to lambda name `create.js`
3. Move existing lambda function `create.js` to 'lambdas' directory
4. Change `module.exports.create` to `module.exports.main` in your `create.js` file
5. Change `handler` below `functions` to `create`
6. change value of `handler` property to lambdas/create.main
```yml
functions:
  create:
    handler: lambdas/create.main
```

## Class Excercise
- Make a `GET` request to get All Buckets data
- Look through [AWS SDK for JavaScript](https://docs.aws.amazon.com/AWSJavaScriptSDK/latest/AWS/S3.html#listBuckets-property) documentation to find the correct method
