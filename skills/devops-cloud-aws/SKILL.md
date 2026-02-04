---
name: devops-cloud-aws
description: AWS services including Lambda, S3, EC2, DynamoDB, CloudFormation, and serverless patterns
tags: [aws, lambda, s3, ec2, serverless, cloud]
author: Antigravity Team
version: 1.0.0
---

# AWS Cloud Development Skill

Build on Amazon Web Services.

## Lambda Functions

```javascript
// handler.js
export const handler = async (event, context) => {
  try {
    const body = JSON.parse(event.body);
    
    // Process the request
    const result = await processData(body);
    
    return {
      statusCode: 200,
      headers: {
        'Content-Type': 'application/json',
        'Access-Control-Allow-Origin': '*'
      },
      body: JSON.stringify(result)
    };
  } catch (error) {
    console.error('Error:', error);
    return {
      statusCode: 500,
      body: JSON.stringify({ error: error.message })
    };
  }
};

// With API Gateway event
export const apiHandler = async (event) => {
  const { httpMethod, pathParameters, queryStringParameters } = event;
  
  switch (httpMethod) {
    case 'GET':
      return getItem(pathParameters.id);
    case 'POST':
      return createItem(JSON.parse(event.body));
    default:
      return { statusCode: 405, body: 'Method Not Allowed' };
  }
};
```

## S3 Operations

```javascript
import { S3Client, PutObjectCommand, GetObjectCommand } from '@aws-sdk/client-s3';
import { getSignedUrl } from '@aws-sdk/s3-request-presigner';

const s3 = new S3Client({ region: 'us-east-1' });

// Upload file
async function uploadFile(bucket, key, body, contentType) {
  await s3.send(new PutObjectCommand({
    Bucket: bucket,
    Key: key,
    Body: body,
    ContentType: contentType
  }));
  return `https://${bucket}.s3.amazonaws.com/${key}`;
}

// Generate presigned URL for upload
async function getUploadUrl(bucket, key, expiresIn = 3600) {
  const command = new PutObjectCommand({ Bucket: bucket, Key: key });
  return getSignedUrl(s3, command, { expiresIn });
}

// Stream download
async function downloadFile(bucket, key) {
  const response = await s3.send(new GetObjectCommand({
    Bucket: bucket,
    Key: key
  }));
  return response.Body;
}
```

## DynamoDB

```javascript
import { DynamoDBClient } from '@aws-sdk/client-dynamodb';
import { DynamoDBDocumentClient, QueryCommand, PutCommand } from '@aws-sdk/lib-dynamodb';

const docClient = DynamoDBDocumentClient.from(new DynamoDBClient({}));

async function getItemsByUser(userId) {
  const result = await docClient.send(new QueryCommand({
    TableName: 'MyTable',
    KeyConditionExpression: 'PK = :pk',
    ExpressionAttributeValues: { ':pk': `USER#${userId}` }
  }));
  return result.Items;
}
```

## CloudFormation / SAM

```yaml
# template.yaml
AWSTemplateFormatVersion: '2010-09-09'
Transform: AWS::Serverless-2016-10-31

Globals:
  Function:
    Runtime: nodejs20.x
    Timeout: 30
    MemorySize: 256

Resources:
  ApiFunction:
    Type: AWS::Serverless::Function
    Properties:
      Handler: handler.apiHandler
      Events:
        Api:
          Type: Api
          Properties:
            Path: /items/{id}
            Method: ANY
      Policies:
        - DynamoDBCrudPolicy:
            TableName: !Ref ItemsTable

  ItemsTable:
    Type: AWS::DynamoDB::Table
    Properties:
      TableName: Items
      BillingMode: PAY_PER_REQUEST
      AttributeDefinitions:
        - AttributeName: PK
          AttributeType: S
        - AttributeName: SK
          AttributeType: S
      KeySchema:
        - AttributeName: PK
          KeyType: HASH
        - AttributeName: SK
          KeyType: RANGE
```

## Best Practices

1. **Use IAM roles** not access keys
2. **Enable CloudWatch** logging
3. **Use environment variables** for config
4. **Implement retries** with exponential backoff
5. **Use VPC endpoints** for private access
