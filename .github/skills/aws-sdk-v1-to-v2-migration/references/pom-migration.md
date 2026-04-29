# pom.xml Migration Reference

## Complete Dependency Replacement Map

### AWS SDK v1 → v2 Group ID Change
- **v1 groupId**: `com.amazonaws`
- **v2 groupId**: `software.amazon.awssdk`

### Artifact Mappings

| SDK v1 Artifact | SDK v2 Artifact | Notes |
|---|---|---|
| `aws-java-sdk` (full SDK) | Individual modules below | Never use the full v2 BOM; pick individual modules |
| `aws-java-sdk-s3` | `s3` | S3 operations |
| `aws-java-sdk-dynamodb` | `dynamodb` + `dynamodb-enhanced` | Enhanced client for ORM-like features |
| `aws-java-sdk-sqs` | `sqs` | SQS operations |
| `aws-java-sdk-simpleworkflow` | `swf` | SWF operations |
| `aws-java-sdk-swf-libraries` | `swf` | SWF flow framework |
| `aws-java-sdk-core` | `sdk-core` + `auth` | Core + credentials |

### BOM Management (if using parent POM)

**If the parent POM manages SDK v2 versions**, you may only need:
```xml
<dependency>
    <groupId>software.amazon.awssdk</groupId>
    <artifactId>bom</artifactId>
    <version>${aws.sdk.v2.version}</version>
    <type>pom</type>
    <scope>import</scope>
</dependency>
```

Then individual dependencies without `<version>`:
```xml
<dependency>
    <groupId>software.amazon.awssdk</groupId>
    <artifactId>s3</artifactId>
</dependency>
```

### Example: Before and After

**Before (SDK v1)**:
```xml
<dependency>
    <groupId>com.amazonaws</groupId>
    <artifactId>aws-java-sdk-s3</artifactId>
    <scope>test</scope>
</dependency>
<dependency>
    <groupId>com.amazonaws</groupId>
    <artifactId>aws-java-sdk-simpleworkflow</artifactId>
</dependency>
```

**After (SDK v2)**:
```xml
<dependency>
    <groupId>software.amazon.awssdk</groupId>
    <artifactId>s3</artifactId>
    <scope>test</scope>
</dependency>
<dependency>
    <groupId>software.amazon.awssdk</groupId>
    <artifactId>swf</artifactId>
</dependency>
```

### vtw-aws-common Dependency
The `vtw-aws-common` dependency should remain unchanged:
```xml
<dependency>
    <groupId>com.elsevier.vtw.core</groupId>
    <artifactId>vtw-aws-common</artifactId>
</dependency>
```

It will transitively pull in the correct SDK v2 dependencies. Only add explicit SDK v2 dependencies if you use AWS SDK classes **directly** (not through vtw-aws-common helpers).

### Removing Exclusions
If you had exclusions for SDK v1 transitive dependencies, review whether they're still needed. SDK v2 has different transitive dependencies.
