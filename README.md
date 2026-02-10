# overview
An **end-to-end automated ETL (Extract, Transform, Load) pipeline** built entirely on AWS services. This solution enables fully automated data processing: simply upload a CSV file to an S3 bucket, and the system automatically triggers glue job, loads results to a destination bucket, and sends email notifications upon job completion—all without manual intervention.

**Fully automated**– Zero manual steps after initial file upload

**Event-driven architecture**– Components trigger each other via native AWS events

**Real-time notifications**– Email alerts on job success/failure via SNS

**Production-ready security** – Proper IAM role configuration for least-privilege access

### **Data Flow Sequence**

1. **Upload**: User uploads CSV file to `s3://<bucket-name>/raw-data/`
2. **Trigger**: S3 PUT event triggers Lambda function (filtered for `.csv` files only)
3. **Orchestration**: Lambda invokes pre-configured Glue ETL job 
4. **Processing**: Glue job performs:
    - **Extract**: Reads CSV from source S3 location
    - **Transform**: Applies your transformation
    - **Load**: Writes transformed data to `s3://<bucket-name>/transformed-data/` as single CSV file
5. **Notification**: EventBridge detects Glue job state change → triggers SNS → sends email to subscriber

## **☁️ AWS Services Used**

| Service | Purpose | Configuration Highlights |
| --- | --- | --- |
| **Amazon S3** | Source & destination storage | Two folders: `raw-data/` (input) and transformed-data`/` (output) |
| **AWS Lambda** | Event trigger & Glue orchestrator | Python runtime; S3 PUT trigger with prefix/suffix filters |
| **AWS Glue** | ETL processing  | Visual ETL job  |
| **Amazon EventBridge** | Job state monitoring | Rule tracking Glue job state changes (SUCCEEDED/FAILED) |
| **Amazon SNS** | Notification delivery | Email subscription; display name "AWS SNS" |
| **IAM** | Security & permissions | Dedicated roles per service with least-privilege access | organize this in readme file for github 


# Lambda Function Core Logic (Python)
```
`import boto3
import json

glue = boto3.client('glue')

def lambda_handler(event, context):
try:
response = glue.start_job_run(JobName='spotify-etl-job')
return {
'statusCode': 200,
'body': json.dumps(f'Started Glue job S3-glue-S3 with Job ID: {response["JobRunId"]}')
}
except Exception as e:
return {
'statusCode': 500,
'body': json.dumps(f'Error starting Glue job: {str(e)}')
}`
```
