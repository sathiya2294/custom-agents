# Import Migration Reference

## Complete Import Replacement Map

Use find-and-replace across Java files. Each section covers one AWS service.

---

## Core / Common

```
com.amazonaws.AmazonClientException           → software.amazon.awssdk.core.exception.SdkClientException
com.amazonaws.AmazonServiceException           → software.amazon.awssdk.awscore.exception.AwsServiceException
com.amazonaws.ClientConfiguration              → software.amazon.awssdk.core.client.config.ClientOverrideConfiguration
com.amazonaws.auth.DefaultAWSCredentialsProviderChain → software.amazon.awssdk.auth.credentials.DefaultCredentialsProvider
com.amazonaws.auth.AWSCredentialsProvider      → software.amazon.awssdk.auth.credentials.AwsCredentialsProvider
com.amazonaws.regions.Regions                  → software.amazon.awssdk.regions.Region
com.amazonaws.retry.RetryPolicy                → software.amazon.awssdk.core.retry.RetryPolicy
```

---

## DynamoDB

### Client Classes
```
com.amazonaws.services.dynamodbv2.AmazonDynamoDB            → software.amazon.awssdk.services.dynamodb.DynamoDbClient
com.amazonaws.services.dynamodbv2.AmazonDynamoDBClient       → software.amazon.awssdk.services.dynamodb.DynamoDbClient
com.amazonaws.services.dynamodbv2.AmazonDynamoDBClientBuilder → software.amazon.awssdk.services.dynamodb.DynamoDbClient (use .builder())
```

### Data Modeling / Enhanced Client
```
com.amazonaws.services.dynamodbv2.datamodeling.DynamoDBMapper          → software.amazon.awssdk.enhanced.dynamodb.DynamoDbEnhancedClient
com.amazonaws.services.dynamodbv2.datamodeling.DynamoDBMapperConfig    → (removed — use builder methods on enhanced client)
com.amazonaws.services.dynamodbv2.datamodeling.DynamoDBScanExpression  → software.amazon.awssdk.enhanced.dynamodb.model.ScanEnhancedRequest
com.amazonaws.services.dynamodbv2.datamodeling.PaginatedScanList       → software.amazon.awssdk.enhanced.dynamodb.model.PageIterable
com.amazonaws.services.dynamodbv2.datamodeling.S3Link                  → (removed — use String for S3 URIs)
```

### Annotations
```
com.amazonaws.services.dynamodbv2.datamodeling.DynamoDBTable     → software.amazon.awssdk.enhanced.dynamodb.mapper.annotations.DynamoDbBean
com.amazonaws.services.dynamodbv2.datamodeling.DynamoDBHashKey   → software.amazon.awssdk.enhanced.dynamodb.mapper.annotations.DynamoDbPartitionKey
com.amazonaws.services.dynamodbv2.datamodeling.DynamoDBRangeKey  → software.amazon.awssdk.enhanced.dynamodb.mapper.annotations.DynamoDbSortKey
com.amazonaws.services.dynamodbv2.datamodeling.DynamoDBAttribute → software.amazon.awssdk.enhanced.dynamodb.mapper.annotations.DynamoDbAttribute
```

### Model Classes
```
com.amazonaws.services.dynamodbv2.model.AttributeValue      → software.amazon.awssdk.services.dynamodb.model.AttributeValue
com.amazonaws.services.dynamodbv2.model.Condition            → software.amazon.awssdk.services.dynamodb.model.Condition
com.amazonaws.services.dynamodbv2.model.ComparisonOperator   → software.amazon.awssdk.services.dynamodb.model.ComparisonOperator
com.amazonaws.services.dynamodbv2.model.ScanRequest          → software.amazon.awssdk.services.dynamodb.model.ScanRequest
com.amazonaws.services.dynamodbv2.model.ScanResult           → software.amazon.awssdk.services.dynamodb.model.ScanResponse
com.amazonaws.services.dynamodbv2.model.ReturnConsumedCapacity → software.amazon.awssdk.services.dynamodb.model.ReturnConsumedCapacity
com.amazonaws.services.dynamodbv2.model.ProvisionedThroughputExceededException → software.amazon.awssdk.services.dynamodb.model.ProvisionedThroughputExceededException
```

---

## SQS

```
com.amazonaws.services.sqs.AmazonSQS          → software.amazon.awssdk.services.sqs.SqsClient
com.amazonaws.services.sqs.AmazonSQSClient     → software.amazon.awssdk.services.sqs.SqsClient
com.amazonaws.services.sqs.model.*             → software.amazon.awssdk.services.sqs.model.*
```

---

## S3

### Client Classes
```
com.amazonaws.services.s3.AmazonS3             → software.amazon.awssdk.services.s3.S3Client
com.amazonaws.services.s3.AmazonS3Client        → software.amazon.awssdk.services.s3.S3Client
com.amazonaws.services.s3.AmazonS3ClientBuilder  → software.amazon.awssdk.services.s3.S3Client (use .builder())
```

### Model Classes
```
com.amazonaws.services.s3.model.ListObjectsRequest  → software.amazon.awssdk.services.s3.model.ListObjectsRequest
com.amazonaws.services.s3.model.ObjectListing        → software.amazon.awssdk.services.s3.model.ListObjectsResponse
com.amazonaws.services.s3.model.S3ObjectSummary      → software.amazon.awssdk.services.s3.model.S3Object
com.amazonaws.services.s3.model.PutObjectRequest     → software.amazon.awssdk.services.s3.model.PutObjectRequest
com.amazonaws.services.s3.model.GetObjectRequest     → software.amazon.awssdk.services.s3.model.GetObjectRequest
com.amazonaws.services.s3.model.DeleteObjectRequest  → software.amazon.awssdk.services.s3.model.DeleteObjectRequest
com.amazonaws.services.s3.model.Region               → software.amazon.awssdk.regions.Region
```

---

## SWF (Simple Workflow)

```
com.amazonaws.services.simpleworkflow.*  → software.amazon.awssdk.services.swf.*
```

---

## vtw-aws-common Helper Imports (UNCHANGED)

These imports from `vtw-aws-common` do NOT change — the jar handles the internal migration:

```
com.elsevier.vtw.aws.helper.SQSHelper         → NO CHANGE
com.elsevier.vtw.aws.helper.SQSMessageReturn   → NO CHANGE
com.elsevier.vtw.aws.helper.S3Helper           → NO CHANGE
com.elsevier.vtw.aws.helper.DynamoDBDAOHelper  → NO CHANGE
com.elsevier.vtw.aws.helper.DynamoDBOMHelper   → NO CHANGE
com.elsevier.vtw.aws.helper.RangeKeyOperator   → NO CHANGE
com.elsevier.vtw.aws.AmazonSQSClientFactoryBean → CHECK — may be renamed or removed in v2
```

The vtw-aws-common helper method signatures (e.g., `sqsHelper.dequeueMessages()`, `s3Helper.uploadFile()`, `dynamoDBDAOHelper.query()`) remain the same. Only the **constructor parameter types** change (e.g., `SqsClient` instead of `AmazonSQSClient`).
