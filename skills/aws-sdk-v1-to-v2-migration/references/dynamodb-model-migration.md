# DynamoDB Model Migration Reference

## Annotation Changes

SDK v2 enhanced client uses a different annotation model. Every DynamoDB-mapped POJO must be updated.

### Class-Level Annotation

**Before (SDK v1)**:
```java
import com.amazonaws.services.dynamodbv2.datamodeling.DynamoDBTable;

@DynamoDBTable(tableName = "MyTable")
public class MyEntity {
```

**After (SDK v2)**:
```java
import software.amazon.awssdk.enhanced.dynamodb.mapper.annotations.DynamoDbBean;

@DynamoDbBean
public class MyEntity {
```

> **Note**: Table name is no longer in the annotation. It's specified when creating the `DynamoDbTable` reference:
> ```java
> DynamoDbTable<MyEntity> table = enhancedClient.table("MyTable", TableSchema.fromBean(MyEntity.class));
> ```

### Partition Key (Hash Key)

**Before**:
```java
import com.amazonaws.services.dynamodbv2.datamodeling.DynamoDBHashKey;
import com.amazonaws.services.dynamodbv2.datamodeling.DynamoDBAttribute;

@DynamoDBHashKey
@DynamoDBAttribute(attributeName = "coid")
public String getCoid() { return coid; }
```

**After**:
```java
import software.amazon.awssdk.enhanced.dynamodb.mapper.annotations.DynamoDbPartitionKey;
import software.amazon.awssdk.enhanced.dynamodb.mapper.annotations.DynamoDbAttribute;

@DynamoDbPartitionKey
@DynamoDbAttribute("coid")
public String getCoid() { return coid; }
```

### Sort Key (Range Key)

**Before**:
```java
@DynamoDBRangeKey
@DynamoDBAttribute(attributeName = "updtTime")
public Long getUpdateTime() { return updateTime; }
```

**After**:
```java
@DynamoDbSortKey
@DynamoDbAttribute("updtTime")
public Long getUpdateTime() { return updateTime; }
```

### Regular Attributes

**Before**:
```java
@DynamoDBAttribute(attributeName = "actn")
public String getAction() { return action; }
```

**After**:
```java
@DynamoDbAttribute("actn")
public String getAction() { return action; }
```

---

## S3Link Field Migration

SDK v2 does **not** have an `S3Link` type. Fields of type `S3Link` must be converted to `String`.

### Before (SDK v1):
```java
import com.amazonaws.services.dynamodbv2.datamodeling.S3Link;

private S3Link intUri;
private S3Link mdLoc;

@DynamoDBAttribute(attributeName = "intURI")
public S3Link getIntUri() { return intUri; }
public void setIntUri(S3Link intUri) { this.intUri = intUri; }

@DynamoDBAttribute(attributeName = "mdLoc")
public S3Link getMdLoc() { return mdLoc; }
public void setMdLoc(S3Link mdLoc) { this.mdLoc = mdLoc; }
```

### After (SDK v2):
```java
private String intUri;
private String mdLoc;

@DynamoDbAttribute("intURI")
public String getIntUri() { return intUri; }
public void setIntUri(String intUri) { this.intUri = intUri; }

@DynamoDbAttribute("mdLoc")
public String getMdLoc() { return mdLoc; }
public void setMdLoc(String mdLoc) { this.mdLoc = mdLoc; }
```

### S3Link Usage Migration

Code that creates or reads S3Links must change:

**Before**:
```java
// Creating S3Link
auditRecord.setIntUri(mapper.createS3Link(REGION_IRELAND, NEW_BUCKET_NAME, auditRecord.getIntUri().getKey()));

// Reading S3Link
String bucket = mdRecord.getMdLoc().getBucketName();
String key = mdRecord.getMdLoc().getKey();
```

**After**:
```java
// Creating S3 URI string
auditRecord.setIntUri(String.format("s3://%s/%s", NEW_BUCKET_NAME, extractKeyFromUri(auditRecord.getIntUri())));

// Reading S3 URI
// Parse the s3:// URI to get bucket and key
URI s3Uri = URI.create(mdRecord.getMdLoc());
String bucket = s3Uri.getHost();
String key = s3Uri.getPath().substring(1); // remove leading /
```

---

## Complete Model Class Example

### Before (SDK v1):
```java
package com.elsevier.updatename.model;

import com.amazonaws.services.dynamodbv2.datamodeling.*;

@DynamoDBTable(tableName = "")
public class VTWMetadata {
    private String coid;
    private S3Link intUri;
    private S3Link mdLoc;
    private Long sequence;

    @DynamoDBHashKey
    @DynamoDBAttribute(attributeName = "coid")
    public String getCoid() { return coid; }
    public void setCoid(String coid) { this.coid = coid; }

    @DynamoDBAttribute(attributeName = "intURI")
    public S3Link getIntUri() { return intUri; }
    public void setIntUri(S3Link intUri) { this.intUri = intUri; }

    @DynamoDBAttribute(attributeName = "mdLoc")
    public S3Link getMdLoc() { return mdLoc; }
    public void setMdLoc(S3Link mdLoc) { this.mdLoc = mdLoc; }

    @DynamoDBRangeKey
    @DynamoDBAttribute(attributeName = "sequence")
    public Long getSequence() { return sequence; }
    public void setSequence(Long sequence) { this.sequence = sequence; }
}
```

### After (SDK v2):
```java
package com.elsevier.updatename.model;

import software.amazon.awssdk.enhanced.dynamodb.mapper.annotations.DynamoDbBean;
import software.amazon.awssdk.enhanced.dynamodb.mapper.annotations.DynamoDbPartitionKey;
import software.amazon.awssdk.enhanced.dynamodb.mapper.annotations.DynamoDbSortKey;
import software.amazon.awssdk.enhanced.dynamodb.mapper.annotations.DynamoDbAttribute;

@DynamoDbBean
public class VTWMetadata {
    private String coid;
    private String intUri;
    private String mdLoc;
    private Long sequence;

    @DynamoDbPartitionKey
    @DynamoDbAttribute("coid")
    public String getCoid() { return coid; }
    public void setCoid(String coid) { this.coid = coid; }

    @DynamoDbAttribute("intURI")
    public String getIntUri() { return intUri; }
    public void setIntUri(String intUri) { this.intUri = intUri; }

    @DynamoDbAttribute("mdLoc")
    public String getMdLoc() { return mdLoc; }
    public void setMdLoc(String mdLoc) { this.mdLoc = mdLoc; }

    @DynamoDbSortKey
    @DynamoDbAttribute("sequence")
    public Long getSequence() { return sequence; }
    public void setSequence(Long sequence) { this.sequence = sequence; }
}
```

---

## DynamoDB Scan Expression Migration

### Before (SDK v1):
```java
import com.amazonaws.services.dynamodbv2.datamodeling.DynamoDBScanExpression;
import com.amazonaws.services.dynamodbv2.model.AttributeValue;
import com.amazonaws.services.dynamodbv2.model.Condition;
import static com.amazonaws.services.dynamodbv2.model.ComparisonOperator.EQ;

DynamoDBScanExpression scanExpression = new DynamoDBScanExpression();
scanExpression.addFilterCondition("category",
    new Condition().withComparisonOperator(EQ)
        .withAttributeValueList(new AttributeValue().withS(category)));
List<FixtureValue> results = helper.scanItems(FixtureValue.class, tableName, scanExpression);
```

### After (SDK v2):
```java
import software.amazon.awssdk.enhanced.dynamodb.model.ScanEnhancedRequest;
import software.amazon.awssdk.enhanced.dynamodb.Expression;
import software.amazon.awssdk.services.dynamodb.model.AttributeValue;

Expression filterExpression = Expression.builder()
    .expression("category = :cat")
    .putExpressionValue(":cat", AttributeValue.builder().s(category).build())
    .build();
ScanEnhancedRequest scanRequest = ScanEnhancedRequest.builder()
    .filterExpression(filterExpression)
    .build();
// Use with updated helper or enhanced client directly
```
