# Spring Configuration Migration Reference

## XML Configuration Changes

### applicationContext-aws.xml
The `vtw-aws-common` jar provides `applicationContext-aws.xml` on the classpath. After upgrading the jar, the beans it defines will use SDK v2 types internally. Projects importing this context:

```xml
<import resource="classpath:applicationContext-aws.xml" />
```

This import should continue to work. However, any **local beans** that reference SDK v1 types must be updated.

### S3Helper Bean Configuration

**Before (SDK v1)**:
```xml
<bean id="ucsS3HelperTest" class="com.elsevier.vtw.aws.helper.S3Helper" init-method="init">
    <constructor-arg ref="TimeHelper"/>
    <property name="s3Endpoint" value="${ucs.s3.endpoint}" />
</bean>
```

**After (SDK v2)** — verify against actual vtw-aws-common v2 constructor:
```xml
<bean id="ucsS3HelperTest" class="com.elsevier.vtw.aws.helper.S3Helper" init-method="init">
    <constructor-arg ref="TimeHelper"/>
    <property name="region" value="eu-west-1" />
</bean>
```

> **Important**: Check the actual `S3Helper` constructor in the upgraded `vtw-aws-common` jar. If the constructor signature changed (e.g., removed `s3Endpoint` property, added `region`), update accordingly.

---

## Java Configuration Changes

### SQS Client Bean

**Before (SDK v1)**:
```java
import com.amazonaws.services.sqs.AmazonSQSClient;
import com.elsevier.vtw.aws.AmazonSQSClientFactoryBean;

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
import software.amazon.awssdk.services.sqs.SqsClient;
import software.amazon.awssdk.regions.Region;
import software.amazon.awssdk.auth.credentials.DefaultCredentialsProvider;

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

### DynamoDB Client Bean

**Before (SDK v1)**:
```java
import com.amazonaws.services.dynamodbv2.AmazonDynamoDBClient;
import com.amazonaws.ClientConfiguration;

ClientConfiguration cfg = new ClientConfiguration();
cfg.setMaxErrorRetry(3);
AmazonDynamoDB client = new AmazonDynamoDBClient(cfg);
client.setEndpoint("dynamodb.eu-west-1.amazonaws.com");
```

**After (SDK v2)**:
```java
import software.amazon.awssdk.services.dynamodb.DynamoDbClient;
import software.amazon.awssdk.regions.Region;
import software.amazon.awssdk.core.client.config.ClientOverrideConfiguration;

DynamoDbClient client = DynamoDbClient.builder()
    .region(Region.EU_WEST_1)
    .overrideConfiguration(ClientOverrideConfiguration.builder()
        .retryStrategy(b -> b.maxAttempts(4))
        .build())
    .build();
```

### S3 Client Bean

**Before (SDK v1)**:
```java
import com.amazonaws.services.s3.AmazonS3Client;

AmazonS3Client s3Client = new AmazonS3Client();
```

**After (SDK v2)**:
```java
import software.amazon.awssdk.services.s3.S3Client;
import software.amazon.awssdk.regions.Region;

S3Client s3Client = S3Client.builder()
    .region(Region.EU_WEST_1)
    .build();
```

---

## Properties File Changes

### S3 Endpoint Properties

**Before**:
```properties
s3Path=https://s3-eu-west-1.amazonaws.com/
ucs.s3.endpoint=http://s3.amazonaws.com/
```

**After** — if the S3Helper now uses region-based configuration:
```properties
s3.region=eu-west-1
ucs.s3.region=eu-west-1
```

> **Note**: Verify against the actual `S3Helper` in vtw-aws-common v2. If it still accepts endpoint URLs, keep them. If it switched to region-based configuration, update properties and beans.

### DynamoDB Endpoint Properties

**Before**:
```properties
dynamodb.endpoint=dynamodb.eu-west-1.amazonaws.com
```

**After** — typically removed in favor of region:
```properties
dynamodb.region=eu-west-1
```

---

## Log Configuration Changes

### log4j.properties

**Before**:
```properties
log4j.logger.com.amazonaws=WARN
```

**After**:
```properties
log4j.logger.software.amazon.awssdk=WARN
```

### logback.xml

**Before**:
```xml
<logger name="com.amazonaws" level="WARN"/>
```

**After**:
```xml
<logger name="software.amazon.awssdk" level="WARN"/>
```
