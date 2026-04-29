---
description: "AWS SDK v1 to v2 migration rules for Java files. Use when editing Java files that import com.amazonaws or use AWS SDK v1 classes like AmazonDynamoDBClient, AmazonSQSClient, DynamoDBMapper, DynamoDBTable annotation."
applyTo: "**/*.java"
---

# AWS SDK v1 to v2 Migration — Java Files

When modifying Java files that contain AWS SDK v1 code (`com.amazonaws.*`), apply these migration rules:

## Import Replacements
- `com.amazonaws.services.dynamodbv2.AmazonDynamoDB` → `software.amazon.awssdk.services.dynamodb.DynamoDbClient`
- `com.amazonaws.services.dynamodbv2.AmazonDynamoDBClient` → `software.amazon.awssdk.services.dynamodb.DynamoDbClient`
- `com.amazonaws.services.dynamodbv2.datamodeling.DynamoDBMapper` → `software.amazon.awssdk.enhanced.dynamodb.DynamoDbEnhancedClient`
- `com.amazonaws.services.dynamodbv2.datamodeling.DynamoDBTable` → `software.amazon.awssdk.enhanced.dynamodb.mapper.annotations.DynamoDbBean`
- `com.amazonaws.services.dynamodbv2.datamodeling.DynamoDBHashKey` → `software.amazon.awssdk.enhanced.dynamodb.mapper.annotations.DynamoDbPartitionKey`
- `com.amazonaws.services.dynamodbv2.datamodeling.DynamoDBRangeKey` → `software.amazon.awssdk.enhanced.dynamodb.mapper.annotations.DynamoDbSortKey`
- `com.amazonaws.services.dynamodbv2.datamodeling.DynamoDBAttribute` → `software.amazon.awssdk.enhanced.dynamodb.mapper.annotations.DynamoDbAttribute`
- `com.amazonaws.services.dynamodbv2.datamodeling.S3Link` → _(remove, use String)_
- `com.amazonaws.services.sqs.AmazonSQSClient` → `software.amazon.awssdk.services.sqs.SqsClient`
- `com.amazonaws.services.s3.model.Region` → `software.amazon.awssdk.regions.Region`
- `com.amazonaws.AmazonServiceException` → `software.amazon.awssdk.awscore.exception.AwsServiceException`
- `com.amazonaws.AmazonClientException` → `software.amazon.awssdk.core.exception.SdkClientException`
- `com.amazonaws.ClientConfiguration` → `software.amazon.awssdk.core.client.config.ClientOverrideConfiguration`
- `com.amazonaws.auth.DefaultAWSCredentialsProviderChain` → `software.amazon.awssdk.auth.credentials.DefaultCredentialsProvider`

## Annotation Replacements
- `@DynamoDBTable(tableName = "...")` → `@DynamoDbBean`
- `@DynamoDBHashKey` → `@DynamoDbPartitionKey`
- `@DynamoDBRangeKey` → `@DynamoDbSortKey`
- `@DynamoDBAttribute(attributeName = "x")` → `@DynamoDbAttribute("x")`

## S3Link → String
Replace `S3Link` field types with `String`. Store S3 references as URI strings (`s3://bucket/key`).

## vtw-aws-common helpers are unchanged
`SQSHelper`, `S3Helper`, `DynamoDBDAOHelper`, `DynamoDBOMHelper`, `RangeKeyOperator`, `SQSMessageReturn` class names and methods remain the same. Only update the constructor parameter types from SDK v1 clients to SDK v2 clients.
