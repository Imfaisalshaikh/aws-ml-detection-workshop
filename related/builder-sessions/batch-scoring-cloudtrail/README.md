# Building a State Machine to scan CloudTrail logs in batches - and score the appropriate USER+API calls
This step function will read CloudTrail logs, submit them to the Sagemaker model and assign an insight score for the specific activities for each identity ARN discovered.  
This is done by chaining 3 Lambda functions - each with its own code and set of permissions.

# Contents of folder
* ProactiveExploration-StateMachine.json: code snippet of State Machine
* ProactiveExploration-policy.json: JSON template to create IAM policy to allow State machine to run lambda functions
* ProactiveExploration-event.json: JSON to use to start State Machine
* ProactiveExploration-ReadCloudTrail.py: lambda python code that will read CloudTrail logs
* ProactiveExploration-SendToModel.py: lambda python code that will send CloudTrail logs to Sagemaker model
* ProactiveExploration-PushAlertsToSNS.py: lambda python code that will create SNS notifications
* lambda-policy-s3.json: JSON template to create IAM policy to allow lambda to read objects, list buckets and put objects in s3 (used to read and write to the state-machine-specific s3 bucket you will create)
* lambda-cloudtrail-policy-s3.json: JSON template to create IAM policy to allow read objects and list buckets in s3 (used for reading CloudTrail logs)
* lambda-policy-sns.json: JSON template to create IAM policy to allow lambda to publish to SNS topic

# Requirements
* Basic experience with:
    * Creating, reading and writting to S3 buckets
    * Creating and publishing to SNS topics
    * Creating and executing Lambda funtions
    * Creating and executing Step Functions
    * Interpreting CloudWatch logs
    * Interpreting CloudTrail logs
    * Creating and using jupyter notenbooks with Sagemaker
    * Basic python experience

# How to install
* Create S3 bucket:
    * Note the name and arn of the S3 bucket - you will use it later
* Create SNS Topic:
    * Note the name and arn of the SNS topic - you will use it later
    * subscribe to the topic using the e-mail address you would like to receive notifications
* Create lambda functions using AWS console:
    * Function name: ProactiveExploration-ReadCloudTrail
        * Use ProactiveExploration-ReadCloudTrail.py as the code for the lambda function
        * Note location in code to configure the CloudTrail logs key structure for your environment
        * Add to the role created automatically for the function the following policy template:
            * lambda-cloudtrail-policy-s3.json - Modify according to your environment
        * Set lambda function environment variables:
            * OUTPUT_BUCKET = use the name of the bucket created
            * OUTPUT_KEY = learning/learning.csv
    * Function name: ProactiveExploration-SendToModel
        * Use ProactiveExploration-SendToModel.py as the code for the lambda function
        * Add to the role created automatically for the function the following policy template:
            * lambda-policy-s3.json - modify according to your environment
        * Set lambda function environment variables:
            * OUTPUT_BUCKET = use the name of the S3 bucket created
            * OUTPUT_KEY = insights/insights.csv
    * Function name: ProactiveExploration-PushAlertsToSNS
        * Use ProactiveExploration-PushAlertsToSNS.py as the code for the lambda function
        * Add to the role created automatically for the function the following policy template:
            * lambda-policy-s3.json - modify according to your environment
            * lambda-policy-sns.json - modify according to your environment
        * Set lambda function environment variables:
            * INSIGHT_THRESHOLD = 0.2
            * SNS_TOPIC_ARN = <the arn of the SNS topic created>
* Create State machine:
    * Name as ProactiveExploration-StateMachine
    * Use ProactiveExploration-StateMachine.json as code for the State machine - modify according to your environment
    * Use ProactiveExploration-policy.json as template for policy to be attached to role that will execute the step function - modify according to your environment

# How to run
* Customize the values of the keys according to your environment using the template provided
    * ProactiveExploration-event.json

Syntactically correct ficticious example:
{
  "cloudtrail_bucket": "awsexamplebucket",
  "aws_account": "111122223333",
  "aws_region": "us-west-1",
  "day_of_interest": "2019/07/04"
}
_In the above example, the cloudtrail__bucket, aws__account, aws__region, and day__of__interest refer to the CloudTrail logs you wish to scan._

* Start the execution of the State machine
* When prompted, copy and paste the JSON from the customized template
* If any CloudTrail events surpass the insight threshold, a SNS notification will be received 