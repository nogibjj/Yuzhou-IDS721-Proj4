<!-- [![CI](https://github.com/nogibjj/python-template/actions/workflows/cicd.yml/badge.svg)](https://github.com/nogibjj/python-template/actions/workflows/cicd.yml) -->

# Project4
## Serverless Data Engineering Pipeline

### Goals
 - [x] Reproduce the architecture of the example serverless data engineering project or perform something similar using only serverless technologies

### Week 9: Calling Amazon Comprehend service inside AWS Cloud9
![success_ipython_sentiment](assets/success_ipython_sentiment.png)
![lambda_function](assets/lambda_function.png)
![lambda_function_result](assets/lambda_function_result.png)

### Week 10: Create serverless pipeline
![architecture](assets/architecture.png)

0. Preparation: 
    i. Create table of company names inside DynamoDB
![DynamoDB_table](assets/DynamoDB_table.png)

    ii. Create SQS `producer`
![sqs](assets/sqs.png)

    iii. Create AdministratorAccess role
![lambda_function_role](assets/lambda_function_role.png)

1. Initiate, Build, and Deploy SAM application. 
```
sam init
sam build --use-container
sam deploy --guided
```
Before build and deploy, change app.py with the code provided in 
* [tutorial/populate_sqs.py](https://github.com/noahgift/awslambda/blob/master/example_src/populate_sqs.py)

Also remember to update requirements.txt to have the packages configured

And create CloudWatch Events Trigger
![lambda_function_sam](assets/lambda_function_sam.png)

2. Modify the Execution Role at this newly created lambda function: Configuration -> Edit Execution role -> choose the role with AdministratorAccess you created before

3. Check SQS to see if have messages received
![sqs_messages](assets/sqs_messages.png)
![sqs_messages_detail](assets/sqs_messages_detail.png)

4. Follow same process, create a consumer SAM application
```
sam init
```
Before build and deploy, change app.py with the code provided in 
* [tutorial/serverless_sentiment_lambda.py](https://github.com/noahgift/awslambda/blob/master/example_src/serverless_sentiment_lambda.py)

Since the consumer application depends on several third party packages, we need to install the packages at the directory above `app.py before we build and deploy it.
```
cd consumer-sam-app-p4-april5/hello_world
python -m venv ../venv
source ../venv/bin/activate
pip install -r requirements.txt --target ../
```

Then deploy and build:
```
sam build --use-container
sam deploy --guided
```

And create SQS Trigger
![lambda_function_sam_consumer](assets/lambda_function_sam_consumer.png)

And you can observe the drop of the number of messages received in the SQS monitor.
![sqs_n_msg_received](assets/sqs_n_msg_received.png)
![sqs_n_msg_deleted](assets/sqs_n_msg_deleted.png)

To see the sentiment analysis ongoing, go CloudWatch Logs inside our consumer function

Use CloudWatch LogInsights to find the sentiment results
```
fields @message
| sort @timestamp desc
| limit 50 | filter @message like /Sentiment from FANG companies/
```

```
Field	Value
@ingestionTime	
1680732228449
@log	
773627151292:/aws/lambda/consumer-app-HelloWorldFunction-nYGmZQbBUzaO
@logStream	
2023/04/05/[$LATEST]3802ebd018fd42f089615ec52c5523bd
@message	
{"message": "Sentiment from FANG companies: {'names': {0: 'Microsoft'}, 'wikipedia_snippit': {0: 'Microsoft Corporation is an American multinational technology corporation headquartered in Redmond, Washington.'}, 'Sentiment': {0: 'NEUTRAL'}}", "aws_request_id": "cb71ec28-7146-5474-b655-b08f833877d2"}
@timestamp	
1680732228172
aws_request_id	
cb71ec28-7146-5474-b655-b08f833877d2
message	
Sentiment from FANG companies: {'names': {0: 'Microsoft'}, 'wikipedia_snippit': {0: 'Microsoft Corporation is an American multinational technology corporation headquartered in Redmond, Washington.'}, 'Sentiment': {0: 'NEUTRAL'}}
```

### Week 11: Debug the pipeline for recording in S3 bucket
Create your own S3 bucket that you want to save the sentiment analysis results
![s3_bucket_files](assets/s3_bucket_files.png)
![s3_bucket_csv_details](assets/s3_bucket_csv_details.png)

```
# Write result to S3
write_s3(df=df, bucket="fangcomsentiment", name=names) # You need to change the bucket name here to the s3 bucket you just created
```

## Reference
* [tutorial](https://github.com/noahgift/awslambda)
* [python template](https://github.com/nogibjj/python-template)
* [Amazon Comprehend](https://docs.aws.amazon.com/comprehend/latest/dg/what-is.html)