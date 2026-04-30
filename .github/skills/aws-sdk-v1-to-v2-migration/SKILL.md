---
name: aws-sdk-v1-to-v2-migration
description: "Migrate Java code from AWS SDK v1 to v2 aligned with vtw-aws-common jar. Use when: upgrading AWS SDK, migrating com.amazonaws to software.amazon.awssdk, updating SQS/S3/DynamoDB code, fixing vtw-aws-common compatibility, migrating AWS client initialization, updating DynamoDB annotations, replacing AmazonSQSClient, replacing AmazonDynamoDBClient, replacing S3 SDK v1 classes."
---

# AWS SDK v1 to v2 Migration (vtw-aws-common aligned)

This skill migrates Java code from AWS SDK v1 (`com.amazonaws`) to AWS SDK v2 (`software.amazon.awssdk`) aligned with the upgraded `vtw-aws-common` jar utilities.

## When to Use
- Migrating any repository that uses `vtw-aws-common` jar
- Upgrading AWS SDK v1 imports to v2
- Updating DynamoDB, SQS, or S3 client code
- Fixing compilation errors after vtw-aws-common upgrade
- Updating Spring configuration for AWS beans

## Procedure

### Step 0: Pre-flight Dependency Check

Before making any changes, confirm the module actually contains AWS SDK v1 artifacts in its resolved dependency tree.

Run the following command from the module directory (where its `pom.xml` lives):

```bash
mvn dependency:tree "-Dincludes=com.amazonaws"
```

**Evaluate the output:**

- If the output contains **no `com.amazonaws` artifacts** → the module has no SDK v1 dependencies. **Stop here — no migration is needed for this module.**
- If the output lists one or more `com.amazonaws` artifacts (e.g. `aws-java-sdk-dynamodb`, `aws-java-sdk-sqs`, `aws-java-sdk-s3`, `aws-java-sdk`, `aws-java-sdk-simpleworkflow`) → SDK v1 is present. **Proceed to Step 1.**

Example output that requires migration:
```
[INFO] com.elsevier.vtw:my-module:jar:1.0.0
[INFO] \- com.amazonaws:aws-java-sdk-dynamodb:jar:1.12.792:compile
[INFO]    \- com.amazonaws:aws-java-sdk-core:jar:1.12.792:compile
```

> **Note:** Run this check for each module independently in a multi-module project. A module that inherits from a parent pom may pull in SDK v1 transitively — the `mvn dependency:tree` output will reveal this. Only compile-scoped `com.amazonaws` artifacts require migration; test-scoped transitive ones (e.g. via `aws-java-sdk-swf-libraries`) may not need code changes.

### Step 1: Analyze the Repository

Scan the repository to identify all files that need migration:

1. **Find all pom.xml files** — check for `vtw-aws-common`, `com.amazonaws` (SDK v1), and `software.amazon.awssdk` (SDK v2) dependencies
2. **Find all Java files** with AWS SDK v1 imports (`com.amazonaws.*`)
3. **Find all Java files** using vtw-aws-common helpers (`SQSHelper`, `S3Helper`, `DynamoDBDAOHelper`, `DynamoDBOMHelper`)
4. **Find Spring XML config files** referencing AWS beans (`applicationContext-aws.xml`, `AmazonSQSClientFactoryBean`, `S3Helper`)
5. **Find properties files** with AWS endpoint configurations
6. **Find log config files** with `com.amazonaws` logger references

### Step 2: Migrate pom.xml Dependencies

Apply these dependency changes. Refer to [pom.xml migration reference](./references/pom-migration.md).

**Remove** AWS SDK v1 dependencies:
```xml
<!-- REMOVE these -->
<groupId>com.amazonaws</groupId>
<artifactId>aws-java-sdk</artifactId>
<artifactId>aws-java-sdk-s3</artifactId>
<artifactId>aws-java-sdk-dynamodb</artifactId>
<artifactId>aws-java-sdk-sqs</artifactId>
<artifactId>aws-java-sdk-simpleworkflow</artifactId>
<artifactId>aws-java-sdk-swf-libraries</artifactId>
```

**Replace** with AWS SDK v2 equivalents:
```xml
<!-- ADD these as needed -->
<groupId>software.amazon.awssdk</groupId>
<artifactId>s3</artifactId>

<groupId>software.amazon.awssdk</groupId>
<artifactId>dynamodb</artifactId>

<groupId>software.amazon.awssdk</groupId>
<artifactId>dynamodb-enhanced</artifactId>

<groupId>software.amazon.awssdk</groupId>
<artifactId>sqs</artifactId>
```

**Keep** `vtw-aws-common` dependency — the jar itself has been upgraded.

### Step 3: Migrate Java Imports

Apply import replacements per [import migration reference](./references/import-migration.md).

#### 3a. DynamoDB Imports
| SDK v1 Import | SDK v2 Import |
|---|---|
| `com.amazonaws.services.dynamodbv2.AmazonDynamoDB` | `software.amazon.awssdk.services.dynamodb.DynamoDbClient` |
| `com.amazonaws.services.dynamodbv2.AmazonDynamoDBClient` | `software.amazon.awssdk.services.dynamodb.DynamoDbClient` |
| `com.amazonaws.services.dynamodbv2.datamodeling.DynamoDBMapper` | `software.amazon.awssdk.enhanced.dynamodb.DynamoDbEnhancedClient` |
| `com.amazonaws.services.dynamodbv2.datamodeling.DynamoDBMapperConfig` | _(removed — config via builder)_ |
| `com.amazonaws.services.dynamodbv2.datamodeling.DynamoDBTable` | `software.amazon.awssdk.enhanced.dynamodb.mapper.annotations.DynamoDbBean` |
| `com.amazonaws.services.dynamodbv2.datamodeling.DynamoDBHashKey` | `software.amazon.awssdk.enhanced.dynamodb.mapper.annotations.DynamoDbPartitionKey` |
| `com.amazonaws.services.dynamodbv2.datamodeling.DynamoDBRangeKey` | `software.amazon.awssdk.enhanced.dynamodb.mapper.annotations.DynamoDbSortKey` |
| `com.amazonaws.services.dynamodbv2.datamodeling.DynamoDBAttribute` | `software.amazon.awssdk.enhanced.dynamodb.mapper.annotations.DynamoDbAttribute` |
| `com.amazonaws.services.dynamodbv2.datamodeling.DynamoDBScanExpression` | `software.amazon.awssdk.enhanced.dynamodb.model.ScanEnhancedRequest` |
| `com.amazonaws.services.dynamodbv2.datamodeling.PaginatedScanList` | _(use PageIterable from enhanced client)_ |
| `com.amazonaws.services.dynamodbv2.datamodeling.S3Link` | _(removed — use String URIs instead)_ |
| `com.amazonaws.services.dynamodbv2.model.*` | `software.amazon.awssdk.services.dynamodb.model.*` |

#### 3b. SQS Imports
| SDK v1 Import | SDK v2 Import |
|---|---|
| `com.amazonaws.services.sqs.AmazonSQSClient` | `software.amazon.awssdk.services.sqs.SqsClient` |
| `com.amazonaws.services.sqs.AmazonSQS` | `software.amazon.awssdk.services.sqs.SqsClient` |

#### 3c. S3 Imports
| SDK v1 Import | SDK v2 Import |
|---|---|
| `com.amazonaws.services.s3.AmazonS3Client` | `software.amazon.awssdk.services.s3.S3Client` |
| `com.amazonaws.services.s3.model.ListObjectsRequest` | `software.amazon.awssdk.services.s3.model.ListObjectsRequest` |
| `com.amazonaws.services.s3.model.ObjectListing` | `software.amazon.awssdk.services.s3.model.ListObjectsResponse` |
| `com.amazonaws.services.s3.model.S3ObjectSummary` | `software.amazon.awssdk.services.s3.model.S3Object` |
| `com.amazonaws.services.s3.model.Region` | `software.amazon.awssdk.regions.Region` |
| `com.amazonaws.AmazonServiceException` | `software.amazon.awssdk.awscore.exception.AwsServiceException` |
| `com.amazonaws.AmazonClientException` | `software.amazon.awssdk.core.exception.SdkClientException` |

#### 3d. Core/Auth Imports
| SDK v1 Import | SDK v2 Import |
|---|---|
| `com.amazonaws.ClientConfiguration` | `software.amazon.awssdk.core.client.config.ClientOverrideConfiguration` |
| `com.amazonaws.auth.DefaultAWSCredentialsProviderChain` | `software.amazon.awssdk.auth.credentials.DefaultCredentialsProvider` |
| `com.amazonaws.regions.Regions` | `software.amazon.awssdk.regions.Region` |

### Step 4: Migrate DynamoDB Model Annotations

Refer to [DynamoDB model migration reference](./references/dynamodb-model-migration.md).

| SDK v1 Annotation | SDK v2 Annotation |
|---|---|
| `@DynamoDBTable(tableName = "...")` | `@DynamoDbBean` |
| `@DynamoDBHashKey` | `@DynamoDbPartitionKey` |
| `@DynamoDBRangeKey` | `@DynamoDbSortKey` |
| `@DynamoDBAttribute(attributeName = "...")` | `@DynamoDbAttribute("...")` |

**S3Link fields**: Replace `S3Link` type with `String`. Store S3 URIs as strings (`s3://bucket/key`).

### Step 5: Migrate DynamoDB Client Initialization

**SDK v1 pattern** (direct client):
```java
ClientConfiguration cfg = new ClientConfiguration();
cfg.setMaxErrorRetry(3);
client = new AmazonDynamoDBClient(cfg);
client.setEndpoint(DYNAMODB_ENDPOINT);
DynamoDBMapperConfig config = new DynamoDBMapperConfig.Builder()
    .withSaveBehavior(CLOBBER)
    .withConsistentReads(CONSISTENT)
    .build();
mapper = new DynamoDBMapper(client, config, new DefaultAWSCredentialsProviderChain());
```

**SDK v2 replacement**:
```java
DynamoDbClient client = DynamoDbClient.builder()
    .region(Region.EU_WEST_1)
    .credentialsProvider(DefaultCredentialsProvider.create())
    .overrideConfiguration(ClientOverrideConfiguration.builder()
        .retryStrategy(b -> b.maxAttempts(4))
        .build())
    .build();
DynamoDbEnhancedClient enhancedClient = DynamoDbEnhancedClient.builder()
    .dynamoDbClient(client)
    .build();
```

### Step 6: Migrate vtw-aws-common Helper Usages

The upgraded `vtw-aws-common` jar has updated its helper classes internally. Key changes:

#### SQSHelper
- Constructor now takes `SqsClient` instead of `AmazonSQSClient`
- Methods remain the same: `dequeueMessages()`, `deleteMessages()`, `enqueueSingleMessage()`
- `SQSMessageReturn` API unchanged
- `AmazonSQSClientFactoryBean` replaced → use `SqsClient.builder()` or updated factory bean

**Before (SDK v1)**:
```java
@Bean
public AmazonSQSClientFactoryBean sqsClient() {
    return new AmazonSQSClientFactoryBean();
}

@Bean
public SQSHelper sqsHelper(AmazonSQSClient amazonSQSClient) {
    return new SQSHelper(amazonSQSClient);
}
```

**After (SDK v2)**:
```java
@Bean
public SqsClient sqsClient() {
    return SqsClient.builder()
        .region(Region.EU_WEST_1)
        .credentialsProvider(DefaultCredentialsProvider.create())
        .build();
}

@Bean
public SQSHelper sqsHelper(SqsClient sqsClient) {
    return new SQSHelper(sqsClient);
}
```

#### S3Helper
- Constructor dependency changed from S3 v1 client to v2 `S3Client`
- Methods remain the same: `uploadFile()`, `listBucket()`, `deleteFile()`
- Endpoint configuration may change from string URL to `Region` based

**Before (Spring XML)**:
```xml
<bean id="ucsS3HelperTest" class="com.elsevier.vtw.aws.helper.S3Helper" init-method="init">
    <constructor-arg ref="TimeHelper"/>
    <property name="s3Endpoint" value="${ucs.s3.endpoint}" />
</bean>
```

**After (Spring XML)** — update if S3Helper constructor signature changed:
```xml
<bean id="ucsS3HelperTest" class="com.elsevier.vtw.aws.helper.S3Helper" init-method="init">
    <constructor-arg ref="TimeHelper"/>
    <property name="region" value="eu-west-1" />
</bean>
```

#### DynamoDBDAOHelper
- Internal client changed to `DynamoDbClient`
- Methods remain: `query()`, `delete()`
- `RangeKeyOperator` usage unchanged

#### DynamoDBOMHelper
- Internal mapper changed to `DynamoDbEnhancedClient`
- Methods remain: `scanItems()`, `save()`, `batchSave()`, `batchDelete()`
- `DynamoDBScanExpression` replaced by `ScanEnhancedRequest`

### Step 7: Migrate Spring Configuration

#### XML Configuration
- Replace `applicationContext-aws.xml` import if the vtw-aws-common provides an updated context
- Update bean class references from v1 to v2 client types
- Replace `AmazonSQSClientFactoryBean` with v2 equivalent

#### Java Configuration
- Replace `AmazonSQSClient` parameter types with `SqsClient`
- Replace `AmazonDynamoDBClient` with `DynamoDbClient`
- Replace `AmazonS3Client` with `S3Client`

### Step 8: Migrate Exception Handling

| SDK v1 Exception | SDK v2 Exception |
|---|---|
| `AmazonServiceException` | `AwsServiceException` |
| `AmazonClientException` | `SdkClientException` |
| `ase.getErrorCode()` | `ase.awsErrorDetails().errorCode()` |
| `ase.getStatusCode()` | `ase.statusCode()` |
| `ase.getErrorType()` | `ase.awsErrorDetails().errorMessage()` |
| `ase.getRequestId()` | `ase.requestId()` |
| `ase.getMessage()` | `ase.getMessage()` |

### Step 9: Migrate Region Constants

| SDK v1 | SDK v2 |
|---|---|
| `com.amazonaws.services.s3.model.Region.EU_Ireland` | `software.amazon.awssdk.regions.Region.EU_WEST_1` |
| `Regions.EU_WEST_1` | `Region.EU_WEST_1` |

### Step 10: Update Log Configuration

In `log4j.properties` or `logback.xml`:
```properties
# BEFORE
log4j.logger.com.amazonaws=WARN

# AFTER
log4j.logger.software.amazon.awssdk=WARN
```

### Step 11: Update Properties Files

- S3 endpoint URLs (`https://s3-eu-west-1.amazonaws.com/`) may need updating to region-based configuration
- DynamoDB endpoint strings are no longer needed (use `Region` instead)

### Step 12: Validate

1. Ensure no remaining `com.amazonaws` imports (except in test mocks if needed)
2. Verify `pom.xml` has no SDK v1 dependencies
3. Run `mvn compile` to check for compilation errors
4. Run existing tests to validate behavior

## Key Gotchas

1. **S3Link removed in SDK v2** — DynamoDB `S3Link` type does not exist in v2. Replace with `String` fields storing S3 URIs.
2. **Builder pattern everywhere** — SDK v2 uses builders instead of constructors. No more `new AmazonDynamoDBClient()`.
3. **Immutable models** — SDK v2 request/response objects are immutable; use `.builder()` and `.build()`.
4. **Region is required** — SDK v2 requires explicit `Region` on clients (no more endpoint strings).
5. **DynamoDBMapperConfig removed** — Table name overrides use `DynamoDbTable` annotation or `TableSchema` in enhanced client.
6. **Scan expressions changed** — `DynamoDBScanExpression` → `ScanEnhancedRequest` with `Expression` filters.
7. **Credential provider renamed** — `DefaultAWSCredentialsProviderChain` → `DefaultCredentialsProvider.create()`.
