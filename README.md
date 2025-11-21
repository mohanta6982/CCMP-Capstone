This project implements a fully serverless image processing pipeline. A user sends an API request with an S3 bucket and key. The request triggers a Step Function workflow, which invokes a Lambda function that:
      -Downloads the original image
      -Resizes it using Pillow
      -Saves it to an output S3 bucket

                   ┌──────────────────────┐
             │      User Client      │
             │   (Postman/ReqBin)    │
             └──────────┬───────────┘
                        │  POST /resize
                        ▼
             ┌──────────────────────┐
             │    API Gateway        │
             │   (REST API - POST)   │
             └──────────┬───────────┘
                        │ StartExecution
                        ▼
        ┌────────────────────────────────────┐
        │           Step Functions            │
        │     ResizeImageStateMachine         │
        └──────────┬─────────────────────────┘
                   │ Invoke Lambda
                   ▼
          ┌─────────────────────────┐
          │       Lambda            │
          │     resize-image        │
          │  Pillow 3.12 (Layer)    │
          └──────────┬──────────────┘
                     │
          ┌──────────┴────────────┐
          │         S3             │
          │ original-image-bucket  │
          │ resize-image-bucket    │
          │   (thumbs/)            │
          └────────────────────────┘


Used API Gateway: 
{
    "stateMachineArn": "arn:aws:states:us-east-1:836940249624:stateMachine:ResizeImageStateMachine",
    "input": "{\"bucket\": $util.escapeJavaScript($input.json('$.bucket')), \"key\": $util.escapeJavaScript($input.json('$.key'))}"
}

Lambda Execution role:

s3:GetObject
s3:PutObject
logs:CreateLogGroup
logs:CreateLogStream
logs:PutLogEvents

API Gateway Step Function Role:

states:StartExecution
logs:CreateLogGroup
logs:CreateLogStream
logs:PutLogEvents

How to Test via API Gateway (REST API):
POST
https://<api-id>.execute-api.us-east-1.amazonaws.com/prod/resize
Header
Content-Type: application/json
Body
{
  "bucket": "original-image-bucket-biddut",
  "key": "test.jpg"
}
Response Example:
{
  "executionArn": "...",
  "startDate": 1763682891421
}



          
