![A green square contains a cycle of money from a piggy bank flowing to a man in a mask to a hacker sitting behind a keyboard and then finally to a credit card. The text reads "Alexa Skill - Scam Sense. Daily fraud and scam alerts.](assets/ScamSense.png)
# ScamSense

Scam Sense is an Alexa skill designed to empower users with daily summaries of news articles focused on financial fraud and scams. By leveraging Llama (via SambaNova Cloud) and robust AWS infrastructure, Scam Sense delivers concise and insightful updates to help users stay informed about potential risks.

The skill utilizes the following technologies and features:

- **News Aggregation**: Integrates with The News API to fetch the latest articles related to financial and cyber fraud.
- **AI Summarization**: Employs SambaNova Cloud for generating concise, human-readable summaries of the fetched articles using Llama.
- **Daily Updates**: Automatically updates news summaries at 6:00 AM Central Time using an AWS EventBridge trigger.
- **Alexa Integration**: Stores summaries in DynamoDB, making them accessible via Alexa-enabled devices when invoked.

This guide provides step-by-step instructions for deploying Scam Sense using AWS CloudFormation, ensuring a seamless setup process. Whether you're a developer or a curious enthusiast, Scam Sense is a valuable tool for staying ahead of the latest financial and digital scams.


# Scam Sense CloudFormation Deployment Guide

## Overview
This guide explains how to deploy the Scam Sense Alexa Skill using the provided AWS CloudFormation template. The template sets up the necessary AWS infrastructure, including Lambda functions, DynamoDB, Secrets Manager, EventBridge, and IAM roles. The skill fetches news articles, generates summaries using SambaNova Cloud, and stores the results in DynamoDB for Alexa to retrieve.

---

## Prerequisites
Before deploying the CloudFormation template, ensure you have the following:

1. **AWS CLI**: Installed and configured with appropriate permissions to deploy resources.
2. **AWS Account**: Active account with necessary access to create Lambda functions, DynamoDB tables, and other resources.
3. **CloudFormation Template**: Ensure the provided YAML file is saved locally (e.g., `scam_sense_cf_template.yaml`).
4. **API Keys**:
   - **SambaNova Cloud API Key** ([Get one here](https://sambanova.ai/))
   - **The News API Token** ([Get one here](https://www.thenewsapi.com))
5. **Lambda Layer ZIP**: Ensure the `lambda-layer.zip` file (containing the OpenAI SDK) is uploaded to an S3 bucket named `scamsense` in the same region you plan on deploying this CloudFormationdtemlate.
6. **Permissions**: IAM permissions to create and manage the following AWS resources:
   - Lambda functions
   - DynamoDB tables
   - EventBridge rules
   - IAM roles
   - Secrets Manager

---

## Deployment Steps

### Step 1: Validate the CloudFormation Template
Before deploying, validate the template to ensure there are no syntax errors.
```bash
aws cloudformation validate-template --template-file cloudformation.yml
```
If the validation is successful, proceed to the next step.

### Step 2: Deploy the CloudFormation Stack
Run the following command to create the stack:
```bash
aws cloudformation create-stack \
  --stack-name ScamSenseStack \
  --template-file cloudformation.yml \
  --parameters ParameterKey=SambaNovaCloudKey,ParameterValue=<Your_SambaNova_API_Key> \
               ParameterKey=NewsApiToken,ParameterValue=<Your_News_API_Token> \
  --capabilities CAPABILITY_NAMED_IAM
```
Replace `<Your_SambaNova_API_Key>` and `<Your_News_API_Token>` with your actual API keys.

### Step 3: Monitor the Deployment
Monitor the progress of the stack creation:
```bash
aws cloudformation describe-stacks --stack-name ScamSenseStack
```
Once the status changes to `CREATE_COMPLETE`, the stack is successfully deployed.

---

## Post-Deployment Configuration

### Retrieve Outputs
After the stack is created, retrieve the output values to configure your Alexa skill:
```bash
aws cloudformation describe-stacks --stack-name ScamSenseStack --query "Stacks[0].Outputs"
```
- **LambdaFunctionArn**: The ARN of the Alexa Skill Lambda function.
- **DatabaseName**: The name of the DynamoDB table.

### Alexa Skill Integration
1. Log in to the Alexa Developer Console.
2. Create a new skill or update an existing one.
3. Access the Alexa Skill Lambda function from the AWS developer console. It can be found in the Resources tab of the CloudFormation deployment as well.
4. On the Alexa Developer Console, access the Endpoint section and copy the Skill ID.
5. On the Lambda function page, add the Alexa trigger by adding a new trigger at the top of the page and selecting it from the dropdown. Paste teh Skill ID when prompted.
5. In the Endpoint section, set the ARN of the Lambda function from the `LambdaFunctionArn` output.

### Testing
Ensure the skill functions correctly by:
- Invoking it on an Alexa-enabled device.
- Verifying summaries are retrieved from the DynamoDB table.

---

## Maintenance and Updates

### Update the Stack
To update the stack with a new version of the template, use the following command:
```bash
aws cloudformation update-stack \
  --stack-name ScamSenseStack \
  --template-file cloudformation.yml \
  --parameters ParameterKey=SambaNovaCloudKey,ParameterValue=<Your_SambaNova_API_Key> \
               ParameterKey=NewsApiToken,ParameterValue=<Your_News_API_Token> \
  --capabilities CAPABILITY_NAMED_IAM
```

### Delete the Stack
To remove all resources created by the template, delete the stack:
```bash
aws cloudformation delete-stack --stack-name ScamSenseStack
```

---

## Troubleshooting

### Common Issues
1. **Permissions Errors**:
   - Ensure the IAM user or role executing the deployment has the necessary permissions.

2. **Lambda Function Fails**:
   - Check the CloudWatch logs for debugging.
   - Verify API keys in Secrets Manager.

3. **DynamoDB Table is Empty**:
   - Ensure the EventBridge rule triggers the Lambda function daily.
   - Verify news articles are fetched successfully.

---

## Resources Created
- **Lambda Functions**:
  - `ScamSenseDailySummaryFunction`: Fetches news articles and stores summaries.
  - `ScamSenseAlexaSkillFunction`: Responds to Alexa skill requests.
- **DynamoDB Table**: `ScamSenseSummaries`
- **Secrets Manager Secret**: `ScamSenseSecrets`
- **EventBridge Rule**: Triggers the daily summary Lambda function.
- **IAM Role**: Provides permissions for Lambda functions.

