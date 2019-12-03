# amplify-add-cognito-trigger
Add cognito trigger to an Amplify app

```
amplify update cognito
```

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
const uuid = require('uuid/v4')
const ddb = new AWS.DynamoDB();
```

```
// Table parameters
const userTable = "User-***-dev";
if (process.env.ENV === 'prod') {
  // Set prod env
  console.log("Prod env")
  userTable = "User-***-prod";
}
```

