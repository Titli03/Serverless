# Serverless

This project will demonstrate the creation of CRUD microservice using Serverless services, Lambda function, API gateway and DynamoDB.

You can create a Lambda function (LambdaFunctionOverHttps) using the AWS Lambda console. Next, you create a DynamoDB (lambda-apigateway) table using the DynamoDB console. Next, you create an REST API (DynamoDBOperations) using  the API Gateway console and define one method (POST) on it. Then you test your API.

Design Overview:
![image](https://github.com/Titli03/Serverless/assets/89897324/efaf1692-7f3c-422b-a2c3-c9146a0cd633)

When you invoke your REST API, API Gateway routes the request to your Lambda function. The method (POST) for REST API (DynamoDBManager) is backed by a Lambda function (LambdaFunctionOverHttps). That is, when you call the API through an HTTPS endpoint, Amazon API Gateway invokes the Lambda function. The Lambda function will run the logic (as per the code updated in the function) interacts with DynamoDB and returns a response to API Gateway. API Gateway then returns a response to you.

The POST method on the DynamoDBManager resource supports the following DynamoDB operations:

Create, update, and delete an item.

Read an item.

Scan an item.

Other operations (echo, ping), not related to DynamoDB, that you can use for testing.

The request payload you send in the POST request identifies the DynamoDB operation and provides necessary data. For example:

The following is a sample request payload for a DynamoDB create item operation: 

Maximize image
Edit image
Delete image


The following is a sample request payload for a DynamoDB read item operation:

Maximize image
Edit image
Delete image


Setup
Create Lambda IAM Role
Create the execution role that gives your function permission to access AWS resources.

To create an execution role

Open the roles page in the IAM console.

Choose Create role.

Create a role with the following properties. Trusted entity – Lambda.Role name – lambda-apigateway-role. Permissions – Custom policy with permission to DynamoDB and CloudWatch Logs. This custom policy has the permissions that the function needs to write data to DynamoDB and upload logs.

Minimize image
Edit image
Delete image


Maximize image
Edit image
Delete image




Create Lambda Function
To create the function

1. Click "Create function" in AWS Lambda Console

2. Select "Author from scratch". Use name LambdaFunctionOverHttps , select Python 3.12 as Runtime. Under Permissions, select "Use an existing role", and select lambda-apigateway-role that we created, from the drop down

3. Click "Create function"

Minimize image
Edit image
Delete image


4. Replace the boilerplate coding with the following code snippet and click "Save"

Maximize image
Edit image
Delete image


Minimize image
Edit image
Delete image




Test Lambda Function
Let's test our newly created function. We haven't created DynamoDB and the API yet, so we'll do a sample echo operation. The function should output whatever input we pass.



1. Click the arrow on "Select a test event" and click "Configure test events"

Minimize image
Edit image
Delete image


2. Paste the following JSON into the event. The field "operation" dictates what the lambda function will perform. In this case, it'd simply return the payload from input event as output. Click "Create" to save

Maximize image
Edit image
Delete image


Minimize image
Edit image
Delete image


3. Click "Test", and it will execute the test event. You should see the output in the console. 

Minimize image
Edit image
Delete image


We're all set to create DynamoDB table and an API using our lambda as backend!

Create DynamoDB Table
Create the DynamoDB table that the Lambda function uses.

To create a DynamoDB table

1. Open the DynamoDB console.

2. Choose Create table.

3. Create a table with the following settings.

   Table name – lambda-apigateway

   Primary key – id (string)

4. Choose Create.

Minimize image
Edit image
Delete image


Create API
To create the API

1. Go to API Gateway console

2. Click Create API

Minimize image
Edit image
Delete image


3. Scroll down and select "Build" for REST API

Minimize image
Edit image
Delete image


4. Give the API name as "DynamoDBOperations", keep everything as is, click "Create API"

Minimize image
Edit image
Delete image


5. Each API is collection of resources and methods that are integrated with backend HTTP endpoints, Lambda functions, or other AWS services. Typically, API resources are organized in a resource tree according to the application logic. At this time you only have the root resource, but let's add a resource next

Click "Actions", then click "Create Resource"

Minimize image
Edit image
Delete image


6. Input "DynamoDBManager" in the Resource Name, Resource Path will get populated. Click "Create Resource"

Minimize image
Edit image
Delete image


7. Let's create a POST Method for our API. With the "/dynamodbmanager" resource selected, Click "Actions" again and click "Create Method".

Minimize image
Edit image
Delete image


8. Select "POST" from drop down , then click checkmark

Minimize image
Edit image
Delete image


9. The integration will come up automatically with "Lambda Function" option selected. Select "LambdaFunctionOverHttps" function that we created earlier. As you start typing the name, your function name will show up, Select and click "Save". A popup window will come up to add resource policy to the lambda to be invoked by this API. Click "Ok"

Minimize image
Edit image
Delete image


Our API-Lambda integration is done!

Deploy the API
In this step, you deploy the API that you created to a stage called prod.

1. Select "Deploy API"

2. Now it is going to ask you about a stage. Select "[New Stage]" for "Deployment stage". Give "Prod" as "Stage name". Click "Deploy"

Maximize image
Edit image
Delete image


3. We're all set to run our solution! To invoke our API endpoint, we need the endpoint URL. In the "Stages" screen, expand the stage "Prod", select "POST" method, and copy the "Invoke URL" from screen

Minimize image
Edit image
Delete image


Running our solution
1. The Lambda function supports using the create operation to create an item in your DynamoDB table. To request this operation, use the following JSON



Maximize image
Edit image
Delete image


2. To execute our API from local machine, we are going to use Postman and Curl command. You can choose either method based on your convenience and familiarity.

To run this from Postman, select "POST" , paste the API invoke url. Then under "Body" select "raw" and paste the above JSON. Click "Send". API should execute and return "HTTPStatusCode" 200.

Minimize image
Edit image
Delete image


To run this from terminal using Curl, run the below

$ curl -X POST -d "{\"operation\":\"create\",\"tableName\":\"lambda-apigateway\",\"payload\":{\"Item\":{\"id\":\"1\",\"name\":\"Bob\"}}}" https://$API.execute-api.$REGION.amazonaws.com/prod/DynamoDBManager
3. To validate that the item is indeed inserted into DynamoDB table, go to Dynamo console, select "lambda-apigateway" table, select "Explore item" tab, and the newly inserted item should be displayed.

Minimize image
Edit image
Delete image


4. To get all the inserted items from the table, we can use the "list" operation of Lambda using the same API. Pass the following JSON to the API, and it will return all the items from the Dynamo table

Minimize image
Edit image
Delete image


Performance test using Postman
1. Install native desktop version postman tool and sign in to an account.

2. Create new collection and post method.

3. Run collection in performance test mode

Minimize image
Edit image
Delete image


4. Below is the report for the average response time and error rate where the configuration for Lambda function was 128 MB Memory and 10 sec time out.

Minimize image
Edit image
Delete image


5. Then I updated the Lambda configuration by increasing 1024 MB Memory and 10 sec time out and now if I run the performance test you can see the average response time is 283 ms (decreased) as per below performance report.

Minimize image
Edit image
Delete image


Cleaning up DynamoDB
1. To delete the table, from DynamoDB console, select the table "lambda-apigateway", and click "Delete table" 

Minimize image
Edit image
Delete image


2. To delete the Lambda, from the Lambda console, select lambda "LambdaFunctionOverHttps", click "Actions", then click Delete

Minimize image
Edit image
Delete image


3. To delete the API we created, in API gateway console, under APIs, select "DynamoDBOperations" API, click "Actions", then "Delete"

Minimize image
Edit image
Delete image
