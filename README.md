# amplify-add-cognito-trigger
Add cognito trigger to an Amplify app

## Why do I need that
Amplify make it very easy to integrate Cognito to your app within minutes. No need to reinvent the wheel, and you endup with a pretty secure website, at least way more than if you do it yourself is you are not an expert!
But Cognito is made to grant access to your user, and that's all. Its puprpose is not to store your users personal information and even less to let them access other users data.
However, in many case, that is exactly what you want to do. For example, you want your user to store a public description/avatar and to link you user to publications/comments etc. That is exactly the database purpose, so we need to create a link between the Cognito and DynamoDb (that I ll use here because it is well integrated with Amplify). This link is a trigger, we are going to ask a Lambda to add a entry in our User table in DynamoDb when a new user is created.


## Prerequisit
For this tutorial you will need:
- An React project with Amplify initialized (https://aws-amplify.github.io/docs/js/react)
- An DynamoDB table *user* created useing amplify add api
- Having Cognito already initialized in your project (amplify add auth)

## Create the trigger function
First update your cognito to create the trigger.
```
amplify update cognito
```

Answer as followed:
```
You have configured resources that might depend on this Cognito resource.  Updating this Cognito resource could have unintended side effects.

Using service: Cognito, provided by: awscloudformation
 What do you want to do? Walkthrough all the auth configurations
 Select the authentication/authorization services that you want to use: User Sign-Up, Sign-In, connected with AWS IAM controls (Enables per-user Storage features for
 images or other content, Analytics, and more)
 Allow unauthenticated logins? (Provides scoped down permissions that you can control via AWS IAM) No
 Do you want to enable 3rd party authentication providers in your identity pool? Yes
 Select the third party identity providers you want to configure for your identity pool: (Press <space> to select, <a> to toggle all, <i> to invert selection)
 Do you want to add User Pool Groups? No
 Do you want to add an admin queries API? No
 Multifactor authentication (MFA) user login options: OFF
 Email based user registration/forgot password: Enabled (Requires per-user email entry at registration)
 Please specify an email verification subject: Your verification code
 Please specify an email verification message: Your verification code is {####}
 Do you want to override the default password policy for this User Pool? No
 Specify the app's refresh token expiration period (in days): 30
 Do you want to specify the user attributes this app can read and write? No
 Do you want to enable any of the following capabilities? (Press <space> to select, <a> to toggle all, <i> to invert selection)
 Do you want to use an OAuth flow? No
? Do you want to configure Lambda Triggers for Cognito? Yes
? Which triggers do you want to enable for Cognito Post Confirmation
? What functionality do you want to use for Post Confirmation Create your own module
Succesfully added the Lambda function locally
? Do you want to edit your custom function now? No
Successfully updated resource recognimagec0ef587c locally
```

## Update your Lambda permissions
```
"lambdaexecutionpolicy": {
            "DependsOn": ["LambdaExecutionRole"],
            "Type": "AWS::IAM::Policy",
            "Properties": {
                "PolicyName": "lambda-execution-policy",
                "Roles": [{ "Ref": "LambdaExecutionRole" }],
                "PolicyDocument": {
                    "Version": "2012-10-17",
                    "Statement": [
                        {
                            "Effect": "Allow",
                            "Action":["logs:CreateLogGroup",
                            "logs:CreateLogStream",
                            "logs:PutLogEvents"],
                            "Resource": { "Fn::Sub" : [ "arn:aws:logs:${region}:${account}:log-group:/aws/lambda/${lambda}:log-stream:*", { "region": {"Ref": "AWS::Region"},  "account": {"Ref": "AWS::AccountId"}, "lambda": {"Ref": "LambdaFunction"}} ]}
                        },
			{
				"Effect": "Allow",
				"Action": [
					"dynamodb:PutItem",
					"dynamodb:DeleteItem",
					"dynamodb:GetItem",
					"dynamodb:Query",
					"dynamodb:UpdateItem"
				],
				"Resource": "arn:aws:dynamodb:*:*:table/*"
			},
			{
				"Effect": "Allow",
				"Action": "dynamodb:Query",
				"Resource": "arn:aws:dynamodb:*:*:table/*/index/*"
			}
                    ]
                }
            }
        }
```

```
const AWS = require('aws-sdk')
const ddb = new AWS.DynamoDB();
```

## Add your code
### Toogle between your envs
```
// Table parameters
const userTable = "User-***-dev";
if (process.env.ENV === 'prod') {
  // Set prod env
  console.log("Prod env")
  userTable = "User-***-prod";
}
```

### Create a new entry in your User table
```
exports.handler = function(event, context, callback) {
    var d = new Date();
    const userSub = event.request.userAttributes.sub;
    var paramsUser = {
        TableName: userTable,
        Item: {
            "id": { "S": userSub },
            "userName": { "S": event.userName },
            "email": { "S": event.request.userAttributes.email },
            "create": { "S": d.toISOString()},
            "lastUpdate": { "S": d.toISOString()},
        }
    };
    ddb.putItem(paramsUser, function(err, data) {
      if (err) {
        console.log("Error", err);
      } else {
        console.log("Exit", data);
        event.request.userAttributes.sub = userSub;
        callback(null, event);
      }
    });  
};
```

## Push the result
You are done, now when a user sign in and is validate, a new entry will be add to your DynamoDb, easy to manipulate, display and update!

